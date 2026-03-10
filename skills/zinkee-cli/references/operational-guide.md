# Zinkee CLI operational guide

## Default mode

- Prefer `zinkee --json --quiet ...` so stdout stays machine-readable and stderr stays quiet.
- Use `jq` for post-processing. Do not pipe CLI JSON into Python one-liners.
- If the execution environment adds extra rules around `zinkee`, apply those rules first and use
  this guide for the CLI-specific details.

## Auth and target selection

- `ZINKEE_API_KEY` overrides the API key stored in the selected profile.
- `-p/--profile` selects the profile from `~/.config/zinkee/config.toml`.
- `ZINKEE_CONFIG_DIR` overrides the config directory if the environment needs a non-default
  location.
- `base_url` comes from the selected profile. If profile loading fails but `ZINKEE_API_KEY` is
  present, the CLI falls back to the production API base URL.
- Treat `zinkee config show` as sensitive output because it can expose API keys in plain text.

## Discovery workflow

- Start with `schema list`, then `schema get <schema>` before record writes or field-level filters.
- Schema and field parameters accept UUIDs or slugs. Slugs are usually the fastest stable handle.
- Record payloads and record responses still use field UUIDs, so build a map when you need to read
  or transform `data` objects.

Example field map:

```bash
zinkee --json --quiet schema get employees \
  | jq '[.fields[] | {(.slug): .id}] | add'
```

## Shell safety

- Prefer direct pipes into `jq` instead of storing large JSON blobs in shell variables.
- In `zsh`, use `printf '%s\n' "$VAR"` instead of `echo "$VAR"` when piping JSON.
- Avoid `cat <<'EOF' | zinkee ...` for payloads. Prefer `echo ... | zinkee ... --data -` or pass
  the JSON directly to `--data`.

## Output and errors

- In JSON mode, successful commands write the parsed API payload to stdout and exit with code `0`.
- In JSON mode, command-level failures raised by the CLI itself are emitted as JSON errors with a
  non-zero exit code.
- Usage and argument parsing failures from Typer or Click are not guaranteed to be JSON, even when
  `--json` is present. Automation should treat malformed commands and missing arguments as a
  separate class of errors.

## Record payload rules

- `record create` and `record update` expect a JSON array of objects shaped like
  `{"fieldId": "<slug-or-uuid>", "value": ...}`.
- A single object is auto-wrapped into a one-item array.
- Use `--data -` when the payload is built by `jq` or contains nested JSON.
- `record list` fetches all matching records in one call. Narrow large queries with
  `--filter-field`, `--filter-op`, `--filter-value`, `--date-field`, `--date-start`, and
  `--date-end`.
- `OptionField` filtering is a known limitation. When it fails, fetch the records and filter
  client-side with `jq`.

### Field value cheat sheet

- `TextField`: plain string.
- `NumberField`: string form of the number, not a raw JSON number.
- `DateField`: full ISO-8601 timestamp string, usually with `T00:00:00.000Z`.
- `OptionField`: option ID from the schema definition.
- `FileField`: JSON-encoded string containing an array with file metadata; upload first.
- `ReferenceField`: target record display value string, not the target UUID.
- Skip `FormulaField`, `AutoIncrementalField`, `HeritageField`, and system fields in write
  payloads.

## Schema workflow

- `schema create` auto-generates the schema UUID if the payload omits `id`.
- `schema update` should send the full schema object, not a hand-written partial patch. The safest
  pattern is `schema get | jq ... | schema update --data -`.

## Display workflow

- `display create` should create the display shell first with `"widgets": []`.
- Add or modify widgets with `display update`.
- Strip read-only widget fields such as `viewConfigId` before sending update payloads back.
- Display names live at `.additionalProperties.name`, not at a top-level `.name`.

## Folder and teamspace workflow

- `folder move` and `teamspace publish` require explicit resource types.
- Supported resource types are `SCHEMA`, `SUBSCHEMA`, `WORKFLOW`, `DISPLAY`, `SCHEDULE`, and
  `SUBSCHEDULE`.

## Deep references

- If the installed CLI exposes a different surface than this bundled guide, inspect
  `zinkee --help`, `zinkee <group> --help`, and `zinkee <group> <command> --help`.
