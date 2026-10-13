# Agent Containers

Run AI coding agents in isolated Docker containers. Each agent has its own image, config, and credentials, built on a shared base.

## Layout

```
~/docker/
├── base/Dockerfile      # shared OS, user, tools
├── claude/Dockerfile    # Claude Code
└── <agent>/Dockerfile   # one folder per additional agent
```

## Build

Build the base first; agent images depend on it.

```bash
docker build -t agent-base ~/docker/base
docker build -t claude-box ~/docker/claude
```

Rebuild the base after changing it, then rebuild each agent.

## Run

From your project folder:

```bash
docker run -dit --name claude-<project> \
  -v "$PWD":/workspace \
  -v claude-config:/home/vscode/.claude \
  claude-box
```

- `-dit`: runs in the background with a terminal; survives SSH disconnects
- `"$PWD":/workspace`: mounts the current project
- `claude-config`: keeps login and settings between runs

**Sign in:** on first run, open the login URL Claude shows. For API billing, add `-e ANTHROPIC_API_KEY` to pass the key from your host environment.

## Daily Use

| Task | Command |
|---|---|
| Open Claude | `docker attach claude-<project>` |
| Detach (keeps running) | `Ctrl+p` then `Ctrl+q` |
| Open a shell | `docker exec -it claude-<project> bash` |
| List containers | `docker ps -a` |
| Stop | `docker stop claude-<project>` |
| Restart | `docker start claude-<project>` |
| Remove | `docker rm -f claude-<project>` |

## Add an Agent

1. Create `~/docker/<agent>/Dockerfile` starting with `FROM agent-base`.
2. Install the agent's CLI, create its config folder, set `CMD`.
3. Build: `docker build -t <agent>-box ~/docker/<agent>`
4. Run with its own config volume (`<agent>-config`) and API key.

Never put API keys in a Dockerfile.

## Add Project Tools

If a project needs a language runtime so the agent can run tests, extend the agent image in the project folder:

```dockerfile
FROM claude-box
USER root
RUN apt-get update && apt-get install -y --no-install-recommends python3 \
 && rm -rf /var/lib/apt/lists/*
USER vscode
```

## Notes

- Containers run as non-root user `vscode` (UID 1000), so files created in `/workspace` match your host user.
- Claude Code auto-updates. For fixed versions, add `DISABLE_AUTOUPDATER=1` to the Dockerfile `ENV` and update by rebuilding.
- For panes and windows, run tmux on the host, not in containers. See [Multiple Panes](#multiple-panes).

## Multiple Panes

Run tmux on the host and attach to containers from its panes:

```bash
tmux new -s agents
docker attach claude-<projectA>      # pane 1
# Ctrl+b %  to split
docker attach claude-<projectB>      # pane 2
```

- One host session can show several containers. tmux inside a container only sees that container.
- Containers already survive disconnects (`-dit`), so tmux inside them adds nothing.
- Nested tmux means pressing the prefix key twice to reach the inner session.
- Detach with `Ctrl+p` then `Ctrl+q` before closing a pane. Closing an attached pane or pressing `Ctrl+c` can stop the container.
