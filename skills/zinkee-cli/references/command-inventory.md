# Zinkee CLI command inventory

Bundled snapshot of the Zinkee CLI command surface.

## Global options

- `--version, -V`: Print version and exit (optional)
- `--json`: Output as JSON (machine-readable) (optional)
- `--profile, -p <profile>`: Config profile to use (optional)
- `--verbose`: Enable verbose output (optional)
- `--quiet`: Suppress status messages (optional)
- `--no-color`: Disable colored output (optional)

Every resource subcommand also accepts a local `--json` option after the command path.

## Command groups

| Group | Purpose | Commands |
| --- | --- | --- |
| `config` | Manage CLI configuration | `show`, `init`, `profiles` |
| `schema` | Manage Zinkee schemas | `list`, `get`, `create`, `update`, `delete` |
| `record` | Manage Zinkee records | `list`, `create`, `update`, `delete` |
| `file` | Manage Zinkee files | `upload`, `download`, `delete`, `storage-info` |
| `comment` | Manage record comments | `list`, `create` |
| `log` | View record activity logs | `get` |
| `display` | Manage Zinkee displays | `list`, `get`, `create`, `update`, `delete`, `folder/...` |
| `folder` | Manage database navigation folders | `list`, `create`, `update`, `delete`, `move` |
| `teamspace` | Manage teamspace | `list`, `create`, `update`, `delete`, `publish`, `unpublish` |

## `config`

Manage CLI configuration

### `zinkee config show`

Print the resolved configuration

### `zinkee config init`

Create a default config file

### `zinkee config profiles`

List available profiles


## `schema`

Manage Zinkee schemas

### `zinkee schema list`

List all schemas

### `zinkee schema get <schema_id>`

Get a schema by UUID or slug

Arguments:
- `<schema_id>`: Schema UUID or slug (required)

### `zinkee schema create --data <data>`

Create a new schema

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee schema update <schema_id> --data <data>`

Update an existing schema

Arguments:
- `<schema_id>`: Schema UUID or slug (required)

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee schema delete <schema_id> [--yes]`

Delete a schema

Arguments:
- `<schema_id>`: Schema UUID or slug (required)

Options:
- `--yes, -y`: Skip confirmation prompt (optional)


## `record`

Manage Zinkee records

### `zinkee record list <schema_id> [--filter-field <filter_field>] [--filter-op <filter_op>] [--filter-value <filter_value>] [--date-field <date_field>] [--date-start <date_start>] [--date-end <date_end>] [--all-fields]`

List records for a schema, with optional filters

Arguments:
- `<schema_id>`: Schema UUID or slug (required)

Options:
- `--filter-field <filter_field>`: Field UUID or slug to filter on (optional)
- `--filter-op <filter_op>`: Raw API comparison operator (e.g. `EQUAL`, `CONTAINS`, `ANY_OF`) (optional)
- `--filter-value <filter_value>`: Filter value (optional)
- `--date-field <date_field>`: Date field UUID or slug (optional)
- `--date-start <date_start>`: Date range start (optional)
- `--date-end <date_end>`: Date range end (optional)
- `--all-fields`: Show all fields including system (optional)

### `zinkee record create <schema_id> --data <data> [--id <record_id>]`

Create a new record

Arguments:
- `<schema_id>`: Schema UUID or slug (required)

Options:
- `--data <data>`: JSON data; fieldId accepts UUIDs or slugs (use '-' to read from stdin) (required)
- `--id <record_id>`: Custom record UUID (optional)

### `zinkee record update <schema_id> <record_id> --data <data>`

Update an existing record

Arguments:
- `<schema_id>`: Schema UUID or slug (required)
- `<record_id>`: Record UUID (required)

Options:
- `--data <data>`: JSON data; fieldId accepts UUIDs or slugs (use '-' to read from stdin) (required)

### `zinkee record delete <schema_id> <record_id> [--yes]`

Delete a record

Arguments:
- `<schema_id>`: Schema UUID or slug (required)
- `<record_id>`: Record UUID (required)

Options:
- `--yes, -y`: Skip confirmation prompt (optional)


## `file`

Manage Zinkee files

### `zinkee file upload <file_path> [--id <file_id>]`

Upload a file to Zinkee (S3 backed)

Note:
- In the current CLI build, passing `--id` is the reliable path. Omitting it can trigger a backend
  `400` because the multipart request still includes an empty `id` part.

Arguments:
- `<file_path>`: Path to the file to upload (required)

Options:
- `--id <file_id>`: Custom file ID (optional)

### `zinkee file download <file_id> [--output <output>]`

Download a file from Zinkee

Arguments:
- `<file_id>`: File UUID to download (required)

Options:
- `--output, -o <output>`: Output file path (optional)

### `zinkee file delete <file_id> [--yes]`

Delete a file from Zinkee

Arguments:
- `<file_id>`: File UUID to delete (required)

Options:
- `--yes, -y`: Skip confirmation prompt (optional)

### `zinkee file storage-info`

Show workspace storage usage


## `comment`

Manage record comments

### `zinkee comment list <schema_id> <record_id>`

List comments for a record

Arguments:
- `<schema_id>`: Schema UUID or slug (required)
- `<record_id>`: Record UUID (required)

### `zinkee comment create <schema_id> <record_id> --text <text>`

Add a comment to a record

Arguments:
- `<schema_id>`: Schema UUID or slug (required)
- `<record_id>`: Record UUID (required)

Options:
- `--text <text>`: Comment text (required)


## `log`

View record activity logs

### `zinkee log get <schema_id> <record_id>`

Get the activity log for a record

Arguments:
- `<schema_id>`: Schema UUID or slug (required)
- `<record_id>`: Record UUID (required)


## `display`

Manage Zinkee displays

### `zinkee display list`

List all displays

### `zinkee display get <display_id>`

Get a display by UUID

Arguments:
- `<display_id>`: Display UUID (required)

### `zinkee display create --data <data>`

Create a new display

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee display update --data <data>`

Update an existing display

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee display delete <display_id> [--yes]`

Delete a display

Arguments:
- `<display_id>`: Display UUID (required)

Options:
- `--yes, -y`: Skip confirmation prompt (optional)

### `zinkee display folder`

Manage display folders

### `zinkee display folder list`

List all display folders

### `zinkee display folder create --data <data>`

Create a new display folder

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee display folder update --data <data>`

Update a display folder

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee display folder delete <folder_id> [--yes]`

Delete a display folder

Arguments:
- `<folder_id>`: Folder UUID (required)

Options:
- `--yes, -y`: Skip confirmation prompt (optional)

### `zinkee display folder add-display <folder_id> <display_id>`

Add a display to a folder

Arguments:
- `<folder_id>`: Folder UUID (required)
- `<display_id>`: Display UUID to add (required)

### `zinkee display folder remove-display <folder_id> <display_id>`

Remove a display from a folder

Arguments:
- `<folder_id>`: Folder UUID (required)
- `<display_id>`: Display UUID to remove (required)


## `folder`

Manage database navigation folders

### `zinkee folder list`

List all database folders

### `zinkee folder create --data <data>`

Create a new database folder

Payload note:
- The JSON payload should use `folderId`, `name`, and `order`.

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee folder update --data <data>`

Update a database folder

Payload note:
- The JSON payload should use `folderId`, `name`, and `order`.

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee folder delete <folder_id> [--yes]`

Delete a database folder

Arguments:
- `<folder_id>`: Folder UUID (required)

Options:
- `--yes, -y`: Skip confirmation prompt (optional)

### `zinkee folder move <resource_id> <target_folder_id> --type <resource_type>`

Move a resource to a different folder

Arguments:
- `<resource_id>`: Resource UUID to move (required)
- `<target_folder_id>`: Target folder UUID (required)

Options:
- `--type, -t <resource_type>`: Resource type (SCHEMA, SUBSCHEMA, WORKFLOW, DISPLAY, SCHEDULE, SUBSCHEDULE) (required)


## `teamspace`

Manage teamspace

### `zinkee teamspace list`

List all teamspace folders

### `zinkee teamspace create --data <data>`

Create a new teamspace group

Payload note:
- The JSON payload should use `folderId`, `name`, and `order`.

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee teamspace update --data <data>`

Update a teamspace group

Payload note:
- The JSON payload should use `folderId`, `name`, and `order`.

Options:
- `--data <data>`: JSON data (use '-' to read from stdin) (required)

### `zinkee teamspace delete <group_id> [--yes]`

Delete a teamspace group

Arguments:
- `<group_id>`: Group UUID (required)

Options:
- `--yes, -y`: Skip confirmation prompt (optional)

### `zinkee teamspace publish <resource_id> <target_group_id> --type <resource_type> --name <resource_name>`

Publish a resource to a teamspace group

Arguments:
- `<resource_id>`: Resource UUID to publish (required)
- `<target_group_id>`: Target teamspace group UUID (required)

Options:
- `--type, -t <resource_type>`: Resource type (SCHEMA, SUBSCHEMA, WORKFLOW, DISPLAY, SCHEDULE, SUBSCHEDULE) (required)
- `--name, -n <resource_name>`: Display name for the resource (required)

### `zinkee teamspace unpublish <resource_id>`

Unpublish a resource from teamspace

Arguments:
- `<resource_id>`: Resource UUID to unpublish (required)
