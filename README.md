# APM - AutoPatchMaster

AutoPatchMaster (APM) automates the updating of dedicated server files. It listens for update requests via a WebSocket push stream and executes the corresponding update scripts using `screen`. A built-in HTTP server exposes health and status information.

```diff
- Source files are not published
```

## Features

- WebSocket-driven update triggers with automatic reconnect (exponential backoff)
- Command injection protection via strict game tag validation
- Health endpoint with uptime, WebSocket status and update history
- Timestamped console logging with color-coded levels
- Persistent log file (configurable path)
- Graceful shutdown on `SIGTERM`/`SIGINT`
- Async shell execution with exit code reporting

## Requirements

- Node.js >= 16.9.0

## Setup

```bash
npm install
cp .env.example .env   # adjust values as needed
npm start
```

## Configuration

All settings are configured via `.env`:

| Variable        | Default                                              | Description                        |
|-----------------|------------------------------------------------------|------------------------------------|
| `BIND_HOST`     | `127.0.0.1`                                          | IP address the HTTP server binds to |
| `BIND_PORT`     | `8080`                                               | HTTP server port                   |
| `WEBSOCKET_URL` | `wss://`                                             | WebSocket endpoint to subscribe to |
| `LOG_FILE`      | `apm.log` (next to `index.js`)                       | Path to the log file               |

## HTTP Endpoints

| Endpoint  | Method | Description                              |
|-----------|--------|------------------------------------------|
| `/`       | GET    | Health status, uptime, recent updates    |
| `/health` | GET    | Same as `/`                              |

### Example response

```json
{
  "status": "healthy",
  "response": "AutoPatchMaster (APM) running",
  "uptime_minutes": 42,
  "websocket": "connected",
  "recent_updates": [
    { "tag": "rust", "time": "2026-03-26T08:00:00.000Z" }
  ]
}
```

## WebSocket Message Format

APM expects messages in the following structure:

```json
{
  "extras": {
    "game": {
      "tag": "rust"
    }
  }
}
```

The `tag` field must match `^[a-zA-Z0-9_-]+$`. On receipt, APM runs:

```bash
screen -d -m -S <tag> bash -c "./update <tag>"
```
