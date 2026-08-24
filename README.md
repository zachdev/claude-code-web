# claude-code-web

A Docker Compose stack that runs multiple AI coding agents with persistent sessions, all behind an nginx reverse proxy and connected to the shared `nginx-net` network.

## Services

| Service | Image | Access path | Description |
|---|---|---|---|
| **claudecodeui** | `libzonda/claudecodeui-docker:latest` | `/` | Web UI for **Claude**, **Codex**, and **Gemini** coding agents |
| **vibe-coder** | `siamakerlab/vibe-coder-server:latest` | `/vibe/` | Browser-based vibe-coding IDE |
| **redis** | `redis:7-alpine` | internal | Shared session store / cache |
| **nginx** | `nginx:alpine` | `:80` | Reverse proxy |

All agent containers bind-mount the host `/projects` directory at `/projects` inside the container and are attached to the external `nginx-net` Docker network.

## Prerequisites

- Docker Engine 24+ and Docker Compose v2
- The external `nginx-net` network must exist before starting the stack
- A `/projects` directory on the host (or adjust the bind-mount path in `docker-compose.yml`)

## Quick start

```bash
# 1. Create the shared network (once per host)
docker network create nginx-net

# 2. Create your projects directory (once per host)
sudo mkdir -p /projects

# 3. Copy and optionally edit the environment file
cp .env.example .env

# 4. Start the stack
docker compose up -d

# 5. Follow logs
docker compose logs -f
```

The Claude Code UI is now available at **http://localhost** (or the IP/hostname of your server).  
The Vibe Coder is available at **http://localhost/vibe/**.

## Session persistence

All agent state (credentials, history, configuration) is stored in named Docker volumes so it survives container restarts and upgrades:

| Volume | Contents |
|---|---|
| `claudecodeui-cloudcli` | Auth database |
| `claudecodeui-claude` | Claude Code sessions & settings |
| `claudecodeui-codex` | Codex sessions |
| `claudecodeui-gemini` | Gemini auth/config |
| `claudecodeui-plugins` | Installed plugins |
| `vibe-coder-data` | Vibe Coder project data |
| `redis-data` | Redis persistence (RDB snapshots) |

## Project files

Mount your code under `/projects` on the host. Every agent container can read and write that directory, so a project checked out at `/projects/my-app` is immediately accessible inside any agent session.

## Configuration

| Variable | Default | Description |
|---|---|---|
| `HTTP_PORT` | `80` | Host port exposed by nginx |
| `TZ` | `UTC` | Timezone for all containers |

Copy `.env.example` to `.env` to override these defaults.

## Updating

```bash
docker compose pull
docker compose up -d
```