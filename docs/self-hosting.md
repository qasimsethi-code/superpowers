# Self-Hosting the Brainstorm Server

The Superpowers brainstorm skill includes a local HTTP + WebSocket server that serves visual mockups and wireframes to your browser during brainstorming sessions. By default it binds to `127.0.0.1` and is only accessible from the same machine. For remote development environments, shared team servers, or containerized setups, you can configure it to accept connections from other hosts.

## When Self-Hosting Is Useful

- **Remote development environments** — GitHub Codespaces, Gitpod, or any cloud-based dev box where Claude runs on one machine and your browser is on another
- **Containerized agents** — running Claude inside Docker where the container's localhost is not reachable from your host machine
- **Shared team servers** — a single machine running Claude on behalf of multiple developers

## Quick Start

By default, the server binds to `127.0.0.1`:

```bash
scripts/start-server.sh --project-dir /path/to/project
# Returns: {"type":"server-started","url":"http://localhost:<port>", ...}
```

To accept connections from any interface:

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host your-server-hostname
```

`--url-host` controls the hostname shown in the returned JSON URL — set it to whatever hostname your browser will use to reach the server.

## Configuration

The server reads configuration from environment variables (set automatically by `start-server.sh`):

| Variable | Default | Description |
|---|---|---|
| `BRAINSTORM_HOST` | `127.0.0.1` | Interface to bind (use `0.0.0.0` for all interfaces) |
| `BRAINSTORM_PORT` | random (49152–65535) | Port to listen on |
| `BRAINSTORM_URL_HOST` | derived from `BRAINSTORM_HOST` | Hostname shown in the returned URL |
| `BRAINSTORM_DIR` | `/tmp/brainstorm-<pid>-<ts>` | Session directory for content and state files |
| `BRAINSTORM_OWNER_PID` | parent process PID | Server auto-exits when this PID dies |

### `start-server.sh` flags

| Flag | Description |
|---|---|
| `--host <addr>` | Bind address (default: `127.0.0.1`) |
| `--url-host <hostname>` | Hostname in the returned URL |
| `--project-dir <path>` | Store session files under `<path>/.superpowers/brainstorm/` instead of `/tmp` |
| `--foreground` | Run in current process (required for some environments) |
| `--background` | Force background mode |

## Docker

Run the server in a container and expose it to the host:

```dockerfile
FROM node:20-alpine
WORKDIR /superpowers
COPY skills/brainstorming/scripts/ ./scripts/
EXPOSE 8080
CMD ["node", "scripts/server.cjs"]
```

```bash
docker run -e BRAINSTORM_HOST=0.0.0.0 \
           -e BRAINSTORM_PORT=8080 \
           -e BRAINSTORM_URL_HOST=localhost \
           -e BRAINSTORM_DIR=/tmp/brainstorm \
           -p 8080:8080 \
           my-brainstorm-server
```

The server requires no external npm packages — only Node.js built-ins.

## Remote / Containerized Environments

In environments where the agent and the browser are on different hosts (Codespaces, Gitpod, remote SSH), you typically need:

1. **Bind to `0.0.0.0`** so the server accepts non-loopback connections
2. **Set `--url-host`** to the hostname your browser can resolve
3. **Forward or expose the port** through your platform's port-forwarding mechanism

```bash
# Example: agent running on a remote machine reachable as "dev.example.com"
scripts/start-server.sh \
  --project-dir /workspace/project \
  --host 0.0.0.0 \
  --url-host dev.example.com
```

The returned JSON will then contain `"url":"http://dev.example.com:<port>"`, which you can open directly.

### GitHub Codespaces

Codespaces automatically forwards ports. Bind to `0.0.0.0` and use the Codespaces-generated domain for `--url-host`, or leave `--url-host` unset and use the Codespaces port-forwarding UI:

```bash
scripts/start-server.sh \
  --project-dir /workspaces/project \
  --host 0.0.0.0
```

Then open the forwarded port URL from the Ports panel.

### Gitpod

Same approach:

```bash
scripts/start-server.sh \
  --project-dir /workspace/project \
  --host 0.0.0.0 \
  --url-host $(gp url <port> | sed 's|https://||')
```

## Security Considerations

- The server serves HTML files from the session's `content/` directory. It does not execute code from those files server-side.
- No authentication is built in. If binding to `0.0.0.0`, ensure the port is only accessible to trusted users (firewall, VPN, or platform access controls).
- The server auto-exits when the agent process that started it terminates (`BRAINSTORM_OWNER_PID` is monitored). It also exits after 30 minutes of inactivity.
- Use `--project-dir` rather than relying on `/tmp` when you want mockup files to persist after the session ends.

## Troubleshooting

**URL is unreachable from the browser**

The server is bound to `127.0.0.1`. Restart with `--host 0.0.0.0 --url-host <your-hostname>`.

**Server starts but page shows only "Waiting for agent..."**

The agent hasn't pushed a screen yet, or it wrote to a different session directory. Check that the agent used the same `--project-dir` path when it started the server.

**Server exits immediately**

The owner PID (`BRAINSTORM_OWNER_PID`) may have been resolved incorrectly. In containerized environments the process tree may look different. You can pass `--foreground` to keep the server in the foreground and diagnose from stdout.

**Port conflict**

Set `BRAINSTORM_PORT` explicitly, or let the server pick a random port (default). The returned JSON always contains the actual port in use.
