# Shuffle playlist analysis (2026-04-02)

Analysis and fixes for `update_shuffle_playlist` in `pyscript/modules/spotify_services.py`.

## What the function does

A custom "smart shuffle" that reorders a Spotify playlist into a shadow/copy playlist:

1. Fetches play history from the database (played_tracks_list table)
2. Fetches playlist tracks from Spotify, matching each against play history by URI, then by name|album|artist identifier, then by name
3. Sorts by last_played (least-recently-played first)
4. Assigns synthetic dates to never-played tracks (randomized, going back from 1 year ago)
5. Runs `fix_repeat_artist_album` to spread out consecutive same-artist/album tracks
6. Splits into groups (50-150 tracks each) and shuffles within each group, preserving macro ordering
7. Runs `fix_repeat_artist_album` again within each group
8. Truncates the shadow playlist and writes the new order in batches of 50

## Bugs fixed

### 1. `updated_group` result discarded (line 499)
After calling `fix_repeat_artist_album` into `updated_group`, the code iterated the original `group` instead. The post-shuffle de-clustering had no effect.

**Fix:** Changed `for track in group` to `for track in updated_group`.

### 2. Missing `await` on async calls
`ensure_token_valid()`, `get_spotify_token()`, and recursive retry calls in `spotify_post`/`spotify_put`/`spotify_delete` were all called without `await`. Token refresh and retries were silently skipped.

**Fix:** Added `await` to all unawaited async calls.

### 4. `random.shuffle` destroyed sort order
The full track list was shuffled at line 403, immediately after being sorted by `last_played`. The intent was only to randomize never-played tracks before assigning synthetic dates, but played tracks lost their ordering too. Although a later re-sort partially restored it, `fix_repeat_artist_album` ran on the shuffled (random) order, making its nudge calculations meaningless.

**Fix:** Partition into played/unplayed lists. Only shuffle unplayed tracks. When `consider_play_date=False`, all tracks are treated as unplayed (full shuffle, as intended).

### 5. `lowest_last_played_datetime` computed from wrong set
The anchor for nudge calculations was computed from `tracks_played_last_year` (only tracks played in the last year), but `fix_repeat_artist_album` operated on all tracks including those with synthetic dates far in the past. This caused disproportionately large nudges for those tracks.

**Fix:** Compute `lowest_last_played_datetime` from the full `sorted_tracks` after synthetic dates are assigned.

## Known issues not addressed

### 3. Non-atomic playlist replacement
`truncate_playlist` deletes all tracks before adding new ones. If the process fails mid-way, the shadow playlist is left empty or partial. Accepted risk.

### 6. Group-based shuffle partially defeats sort purpose
The final sort orders everything by `last_played`, then grouping re-shuffles within each group. This is intentional (tiered randomness) but could be refined further.

### 7. Hardcoded market `NO`
The Spotify API call uses `market=NO`. Fine for private use.

### 8. Dead `use_cache` code path
`use_cache` is hardcoded to `False` at line 346. Could be cleaned up.
