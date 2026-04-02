# Code Style Guidelines

How code should look: formatting, types, and conventions.

## General Principles

- **Preserve existing inline comments** (TODO, FIXME, NOTE, etc.) unless explicitly asked to remove them
- Prefer native language features and standard library over third-party dependencies
- Use centralized constants instead of magic strings

---

## Python (pyscript)

### Style

- Follow PEP 8 conventions
- Use type hints for function signatures
- Use f-strings over `.format()` or `%` formatting
- Prefer `snake_case` for functions, variables, and module names

### Pyscript-Specific

- Pyscript functions decorated with `@service`, `@event_trigger`, `@state_trigger`, etc. are Home Assistant entry points
- Use `task.unique()` for debouncing repeated triggers
- Access HA state via `state.get()` / `state.set()` and entity attribute access
- Use `log.info()` / `log.warning()` / `log.error()` for logging (pyscript provides these)
- Import shared modules from `pyscript/modules/`

---

## YAML (Home Assistant config)

### Structure

- Use 2-space indentation (Home Assistant standard)
- Keep entity IDs in `snake_case`
- Use `!include` and `!include_dir_merge_list` to split large configs
- Group related automations/scripts/sensors in dedicated files

### Secrets

- **Never hardcode secrets** in YAML — use `!secret key_name` referencing `secrets.yaml`
- See [security-review.md](security-review.md) for the full secrets policy

### Naming Conventions

- Entity IDs: `domain.descriptive_snake_case` (e.g., `sensor.living_room_temperature`)
- Automation IDs: descriptive, matching the automation's purpose
- Input helpers: prefix with purpose (e.g., `input_boolean.alarm_active`)
