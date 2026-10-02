# TimeTrack

TimeTrack is a small time-tracking app for recording employee hours against projects. It includes a browser interface, a REST API, and an MCP server backed by the same SQLite database.

## Features

- View all logged time entries.
- Log hours with an employee, project, date, and optional description.
- See project totals broken down by employee.
- Query the REST API or connect an MCP client.
- Start with sample entries in a new, empty database.

## Requirements

- Python 3.14 or newer
- [uv](https://docs.astral.sh/uv/)

## Run the application

From the project directory, install the locked dependencies:

```bash
uv sync
```

Start the web app and REST API:

```bash
uv run uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

Then open:

- Web app: <http://127.0.0.1:8000/>
- Interactive API documentation: <http://127.0.0.1:8000/docs>

### Run MCP by itself

In a second terminal, start the MCP server on port 8001:

```bash
uv run fastmcp run main.py:mcp --transport http --host 127.0.0.1 --port 8001
```

Connect an MCP client to `http://127.0.0.1:8001/mcp`. This serves the MCP interface; the browser UI and REST API are served separately by the Uvicorn command above. Both processes use the same SQLite database file.

## REST API

| Method | Path | Description |
| --- | --- | --- |
| `GET` | `/api/entries` | List all time entries. |
| `GET` | `/api/projects` | List projects with logged time. |
| `GET` | `/api/projects/{project}/summary` | Get project total hours and employee breakdown. |
| `GET` | `/api/timesheet/{employee_name}` | Get an employee's entries. Optional `start_date` and `end_date` query parameters filter by date. |
| `POST` | `/api/entries` | Add a time entry. |

Example request to add an entry:

```bash
curl -X POST http://127.0.0.1:8000/api/entries \
	-H "Content-Type: application/json" \
	-d '{"employee_name":"Asha Patel","project":"Website Redesign","entry_date":"2026-10-02","hours":2.5,"description":"Accessibility fixes"}'
```

Dates use `YYYY-MM-DD`. Hours must be greater than zero.

## Database

The app creates `timetrack.db` in the project directory when it starts. If the database has no time entries, it inserts a small set of sample entries. Existing data is retained on later starts.

Set `TIMETRACK_DB_PATH` to store the database at another location. For example, in PowerShell:

```powershell
$env:TIMETRACK_DB_PATH = "C:\data\timetrack.db"
uv run uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

The local database may contain personal or business time records. Keep it out of source control unless you intentionally want to publish that data.

## Project files

- `main.py` - FastAPI application, REST routes, and MCP setup.
- `database.py` - SQLite schema, initialization, and query functions.
- `static/` - Browser UI assets.
- `pyproject.toml` and `uv.lock` - Project metadata and pinned dependency resolution.

This project is configured for local development and does not add authentication. Do not expose it to an untrusted network without adding appropriate access controls.
