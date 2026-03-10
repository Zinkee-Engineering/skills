# Zinkee CLI operational guide

## Default mode

- Prefer `zinkee --json --quiet ...` so stdout stays machine-readable and stderr stays quiet.
- Use `jq` for post-processing. Do not pipe CLI JSON into Python one-liners.
- `schema list` and `display list` return very large full objects, not small summaries. Project
  them immediately with `jq` before bringing the output back into the conversation.
- If the execution environment adds extra rules around `zinkee`, apply those rules first and use
  this guide for the CLI-specific details.

## Auth and target selection

- `ZINKEE_API_KEY` overrides the API key stored in the selected profile.
- `-p/--profile` selects the profile from `~/.config/zinkee/config.toml`.
- `ZINKEE_CONFIG_DIR` overrides the config directory if the environment needs a non-default
  location.
- `base_url` comes from the selected profile. If profile loading fails but `ZINKEE_API_KEY` is
  present, the CLI falls back to the production API base URL.
- `zinkee config profiles` is the safe way to verify available profiles without exposing secrets.
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

Example compact summaries:

```bash
zinkee --json --quiet schema list \
  | jq '[.[] | {id, name, slug, fieldCount: (.fields | length)}]'

zinkee --json --quiet display list \
  | jq '[.[] | {id, name: .additionalProperties.name, layoutTemplate: .additionalProperties.layoutTemplate, widgetCount: (.widgets | length)}]'
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
- Some successful write commands still return `{}` or even a refreshed list instead of the created
  object. Do not assume a uniform success payload across resource types.
- Usage and argument parsing failures from Typer or Click are not guaranteed to be JSON, even when
  `--json` is present. Automation should treat malformed commands and missing arguments as a
  separate class of errors.

## Record payload rules

- `record create` and `record update` expect a JSON array of objects shaped like
  `{"fieldId": "<slug-or-uuid>", "value": ...}`.
- A single object is auto-wrapped into a one-item array.
- Use `--data -` when the payload is built by `jq` or contains nested JSON.
- `record create` and `record update` return a validation summary with `validUUIDs`,
  `invalidUUIDs`, and `validationErrors`, not the full record body.
- `record list` fetches all matching records in one call. Narrow large queries with
  `--filter-field`, `--filter-op`, `--filter-value`, `--date-field`, `--date-start`, and
  `--date-end`.
- `record list` returns rows under `.[] .data`, and the field values inside `.data` are keyed by
  field UUID.
- For text and option filters, use backend comparison operators such as `EQUAL`, `CONTAINS`, or
  `ANY_OF`. The literal operator example `=` is not reliable.
- `OptionField` filtering is a known limitation. When it fails, fetch the records and filter
  client-side with `jq`.

Example record round-trip:

```bash
zinkee --json --quiet record create employees --data \
  '[{"fieldId":"employee-name","value":"Ada"}]'

zinkee --json --quiet record list employees --filter-field employee-name --filter-op EQUAL --filter-value Ada \
  | jq '.[].data'
```

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
- On `schema create`, send only user-defined fields. The backend injects system fields
  automatically, so including `_id`, `_createdAt`, `_updatedAt`, and similar fields can break the
  create with duplicated-column errors.
- A minimal schema create can work with `fields`, `highlightFieldId`, and `viewsConfig: []`.
- The backend may rewrite `id`, `name`, or `slug` on create to resolve collisions. Always trust
  the returned schema object rather than the requested values.
- `schema update` should send the full schema object, not a hand-written partial patch. The safest
  pattern is `schema get | jq ... | schema update --data -`.

Minimal schema create example:

```bash
FIELD_ID="$(uuidgen | tr '[:upper:]' '[:lower:]')"
printf '%s\n' "{
  \"name\": \"Scratch Employees\",
  \"slug\": \"scratch-employees\",
  \"highlightFieldId\": \"$FIELD_ID\",
  \"auditable\": false,
  \"commentable\": false,
  \"fields\": [
    {
      \"id\": \"$FIELD_ID\",
      \"fieldType\": \"TextField\",
      \"headerName\": \"Name\",
      \"slug\": \"employee-name\",
      \"required\": false,
      \"unique\": false,
      \"system\": false,
      \"defaultValue\": \"\",
      \"textType\": \"NORMAL\",
      \"additionalProperties\": {
        \"headerName\": \"Name\",
        \"conditionalFormatting\": [],
        \"options\": [],
        \"required\": false,
        \"descriptionFields\": [],
        \"useTimeFormat\": false,
        \"filterFields\": [],
        \"textType\": \"NORMAL\"
      }
    }
  ],
  \"viewsConfig\": [],
  \"additionalProperties\": {}
}" | zinkee --json --quiet schema create --data -
```

## Display workflow

- `display create` works with a minimal shell payload: `id`, `variables: []`, `widgets: []`,
  `additionalProperties.name`, `additionalProperties.layoutTemplate`, and `additionalProperties.slots`.
- `display create` should create the display shell first with `"widgets": []`.
- Add or modify widgets with `display update`.
- Strip read-only widget fields such as `viewConfigId` before sending update payloads back.
- Display names live at `.additionalProperties.name`, not at a top-level `.name`.
- `display folder list` returns folder `id`, `properties`, and `displays`; do not expect a stable
  human-readable `name` field there.

Minimal display create example:

```bash
DISPLAY_ID="$(uuidgen | tr '[:upper:]' '[:lower:]')"
printf '%s\n' "{
  \"id\": \"$DISPLAY_ID\",
  \"variables\": [],
  \"widgets\": [],
  \"additionalProperties\": {
    \"name\": \"Scratch Display\",
    \"layoutTemplate\": \"single_view\",
    \"slots\": \"{\\\"main\\\":[]}\"
  },
  \"crossWidgetFilter\": null
}" | zinkee --json --quiet display create --data -
```

## Folder and teamspace workflow

- `folder create` and `teamspace create` expect `folderId`, not `id`, in the payload.
- `folder create`, `folder delete`, `teamspace create`, and `teamspace delete` can return the
  refreshed full list instead of a single object.
- `folder move` and `teamspace publish` require explicit resource types.
- Supported resource types are `SCHEMA`, `SUBSCHEMA`, `WORKFLOW`, `DISPLAY`, `SCHEDULE`, and
  `SUBSCHEDULE`.

Minimal folder create example:

```bash
FOLDER_ID="$(uuidgen | tr '[:upper:]' '[:lower:]')"
printf '%s\n' "{\"folderId\":\"$FOLDER_ID\",\"name\":\"Scratch Folder\",\"order\":99}" \
  | zinkee --json --quiet folder create --data -
```

## File workflow

- `file upload` advertises `--id` as optional, but the current CLI implementation still sends an
  `id` form part. In practice, pass `--id <uuid>` to avoid backend `400` errors.
- Successful `file upload` and `file delete` calls may return `{}`.
- In JSON mode, `file download` returns `{"file": "...", "size": ...}` after writing to disk.

Example upload and cleanup:

```bash
FILE_ID="$(uuidgen | tr '[:upper:]' '[:lower:]')"
zinkee --json --quiet file upload ./notes.txt --id "$FILE_ID"
zinkee --json --quiet file download "$FILE_ID" --output ./notes-copy.txt
zinkee --json --quiet file delete "$FILE_ID" --yes
```

## Deep references

- If the installed CLI exposes a different surface than this bundled guide, inspect
  `zinkee --help`, `zinkee <group> --help`, and `zinkee <group> <command> --help`.
