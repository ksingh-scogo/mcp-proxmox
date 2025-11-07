# MCP Proxmox Server - Logging Guide

## Why `docker logs` Shows Nothing

The Proxmox MCP Server container architecture uses a **persistent container** pattern:

- **Main Process**: `sleep infinity` (keeps container running)
- **MCP Server**: Only runs when Claude Desktop does `docker exec -i proxmox-mcp-server node dist/index.js`

Since `docker logs` only shows output from the main container process (`sleep`), it will always be empty. The actual MCP server runs in separate exec sessions.

## Where Are the Logs?

All MCP server logs are written to **files inside the container**:

- `/app/logs/combined.log` - All logs (JSON format)
- `/app/logs/error.log` - Error logs only (JSON format)

## How to View Logs

### Option 1: View Recent Logs (Recommended)

```bash
./docker-run.sh viewlogs
```

This shows the last 50 log entries in pretty-printed JSON format.

### Option 2: Tail Logs in Real-Time

```bash
./docker-run.sh logs
```

This follows the log file in real-time. Press Ctrl+C to exit.

### Option 3: Manual Access

```bash
# View recent logs
docker exec proxmox-mcp-server tail -50 /app/logs/combined.log

# Follow logs in real-time
docker exec proxmox-mcp-server tail -f /app/logs/combined.log

# View error logs only
docker exec proxmox-mcp-server cat /app/logs/error.log
```

## Log Format

Logs are written in JSON format by Winston logger:

```json
{
  "component": "ProxmoxMCPServer",
  "level": "info",
  "message": "Starting Proxmox MCP Server",
  "timestamp": "2025-11-06 19:50:51",
  "host": "srv1-pve.yattle-beaver.ts.net",
  "allowElevated": true
}
```

## Log Levels

- `error` - Errors and failures
- `warn` - Warnings
- `info` - Informational messages (default)
- `debug` - Detailed debugging info

Configure via environment variable:
```bash
LOG_LEVEL=debug  # in .env file
```

## Why This Architecture?

This design is **required for Claude Desktop integration**:

1. MCP protocol uses **stdin/stdout for JSON-RPC** communication
2. Container must stay running for Claude Desktop to exec into it
3. Logs must go to **stderr or files** to avoid corrupting JSON-RPC
4. Each Claude Desktop session creates a new MCP server instance via `docker exec`

## Docker Desktop UI

The Docker Desktop logs tab will also be empty for the same reason - it shows the main process output, which is just `sleep infinity`.

Use the commands above to view actual MCP server logs.
