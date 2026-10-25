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

## Update

Docker caches the install step, so a plain rebuild keeps the old agent version. To update:

```bash
agent-update                   # rebuild agent-base and claude-box with the latest Claude
agent-update -v 2.1.0          # pin a version
agent-update -a codex          # another agent (image codex-box)
```

Each agent Dockerfile takes `ARG VERSION=latest` and passes it to its installer. With `latest`, the agent image is rebuilt without the cache.

Existing containers keep the old image. The script lists them; recreate each with `docker rm -f <name>` and `agent-run`. Login and settings persist in the config volume.

Link it like `agent-run`: `ln -s ~/development/ai-agents/bin/agent-update ~/.local/bin/agent-update`

## GitHub Token

Store a GitHub token in the macOS Keychain (prompts for the token):

```bash
security add-generic-password -a "$USER" -s gh-agent-token -w
```

Containers get it as `GH_TOKEN`. `gh` and git (HTTPS) both use it. Clone with `https://` URLs, not SSH.

## Run

Link the script into your PATH once:

```bash
ln -s ~/development/ai-agents/bin/agent-run ~/.local/bin/agent-run
```

From your project folder:

```bash
agent-run                      # claude, container claude-<folder>
agent-run -m opus              # pick a model
agent-run -a codex             # another agent (image codex-box)
agent-run -k gh-other-token    # another Keychain item
agent-run my-name              # custom container suffix
```

If the container exists, it is started and attached. `-m` and `-k` only apply when creating it; to change them, `docker rm -f <agent>-<name>` and run again.

The script runs:

```bash
GH_TOKEN=$(security find-generic-password -s gh-agent-token -w) \
docker run -dit --name claude-<project> \
  -v "$PWD":/workspace \
  -v claude-config:/home/vscode/.claude \
  -e GH_TOKEN \
  claude-box
```

- `-dit`: runs in the background with a terminal; survives SSH disconnects
- `"$PWD":/workspace`: mounts the current project
- `<agent>-config`: keeps login and settings between runs, mounted at `/home/vscode/.<agent>`

**Sign in:** on first run, open the login URL Claude shows. For API billing, add `-e ANTHROPIC_API_KEY` to the `docker run` in `bin/agent-run`.

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
2. Declare `ARG VERSION=latest` and install that version of the agent's CLI (e.g. `npm install -g @openai/codex@"$VERSION"`). Create its config folder and set `CMD`.
3. Build: `agent-update -a <agent>`
4. Run: `agent-run -a <agent>`

`agent-run` expects the image to be `<agent>-box`, the CLI command to be `<agent>` and accept `--model`, and the config folder to be `/home/vscode/.<agent>`.

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
