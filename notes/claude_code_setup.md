# Claude Code Setup on Home Assistant OS

## Problem
Claude Code auth and settings don't persist across SSH addon restarts because
`/root/.claude/` lives on the ephemeral overlay filesystem.

## Filesystem layout (HAOS SSH addon)
- `/` — Docker overlay, **ephemeral** (wiped on addon restart)
  - `/root/` lives here — including `/root/.claude/`
- `/data/` — addon persistent storage on `/dev/mmcblk1p4`
  - SSH keys, `.gitconfig`, `.vscode-server`, `.zsh_history` live here
- `/homeassistant/` and `/root/config/` — **same directory** (bind mounts to the
  same inode on `/dev/mmcblk1p4`), this is the HA config / git repo

## Solution applied (2026-04-02)
1. Moved `/root/.claude/` to `/data/.claude/` (persistent storage)
2. Created symlink: `/root/.claude -> /data/.claude`
3. Added init command to `/data/options.json` so the symlink is recreated on
   every addon restart:
   ```json
   "init_commands": [
     "ln -sf /data/.claude /root/.claude"
   ]
   ```

## If auth is lost after restart
- Check if the symlink exists: `ls -la /root/.claude`
- If not, the init_command may not have run. Recreate manually:
  ```sh
  ln -sf /data/.claude /root/.claude
  ```
- If `/data/.claude` is gone too, auth needs to be set up from scratch and the
  symlink approach re-applied (move + init_command as above)
- Re-authenticate with: `claude` (follow the OAuth flow)

## CLAUDE.md
Place project instructions at `/homeassistant/CLAUDE.md` — Claude Code reads
this automatically at conversation start. This is the git repo root.

## Memory system
Claude Code's persistent memory lives at `/data/.claude/projects/-homeassistant/memory/`
(via the symlink). The `MEMORY.md` index there is loaded every conversation.
