# pi-sandbox

> ⚠️ **Testing only** — not yet published to npm. For development and local use.

Run [Pi](https://github.com/earendil-works/pi-coding-agent) (or any coding agent) in an isolated throwaway Docker container.

Only the current project directory is mounted. No home directory, no `~/.ssh`, no secrets.

## Install (local only)

```bash
git clone https://github.com/olsonjj/pi-sandbox.git
cd pi-sandbox
npm install -g .
```

## Usage

```bash
pi-sandbox                              # interactive
pi-sandbox -p "refactor this"          # print mode
pi-sandbox --model sonnet              # pass through any pi flags
pi-sandbox --network                   # allow outbound network (needed for API)
pi-sandbox --network=host              # use host networking (bypasses isolation)
pi-sandbox --mount ~/.pi/agent         # mount additional path (read-only)
pi-sandbox --image my-image            # use custom Docker image
pi-sandbox --cmd bash                  # run a different command inside container
pi-sandbox --rebuild                   # force rebuild of the Docker image
pi-sandbox --version                   # show wrapper and image pi versions
pi-sandbox --help                      # show all flags
cat file.txt | pi-sandbox -p "summarize"
```

First run builds a Docker image (one-time). Subsequent runs are instant.
Use `--rebuild` to pick up new pi versions or security patches. The script
also warns if the image is older than 30 days.

### Wrapper flags

| Flag | Description |
|---|---|
| `--network` | Enable outbound network (default: none) |
| `--network=<mode>` | Set Docker network mode: `bridge`, `host`, `none` |
| `--mount <path>` | Mount additional host path (read-only) |
| `--image <name>` | Use custom Docker image (default: `pi-sandbox`) |
| `--cmd <command>` | Command to run inside container (default: `pi`) |
| `--rebuild` | Force rebuild of the Docker image |
| `--version` | Show wrapper and image pi versions |
| `--help` | Show help |

All other arguments are passed through to `pi`.

> **Warning:** `--network=host` bypasses all network isolation. The container
> shares the host's network stack.

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `PI_SANDBOX_IMAGE` | `pi-sandbox` | Docker image name |
| `PI_SANDBOX_NODE` | `22` | Node.js version for the image |

## Providers

API keys are forwarded from your host environment. Set them before running:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
# ... then run with --network
pi-sandbox --network -p "hello"
```

Supported provider keys forwarded automatically:

Anthropic, OpenAI, Google, Groq, DeepSeek, Mistral, xAI, Cerebras,
Gemini, Vertex AI, Azure OpenAI, AWS Bedrock, OpenRouter, Cloudflare,
Together, Fireworks, HuggingFace, Ollama

Any `PI_*` environment variables are also forwarded.

> **Note:** `--network` is required for API access. Without it, the container
> has no outbound network and cannot reach any provider.

## How it works

- **Project directory** — `$PWD` is mounted at `/workspace` (read-write)
- **node_modules** — an anonymous Docker volume hides host `node_modules`
  from the container, preventing leaks
- **Temp directories** — `/tmp` and the sandbox home are `tmpfs` (ephemeral,
  in-memory, lost on exit)
- **Pi config** — `~/.pi/agent` is mounted read-write so pi can write lock
  files and sessions
- **Extra mounts** — paths added with `--mount` are mounted read-only

## Security

- `--cap-drop=ALL` — drops all Linux capabilities
- `--security-opt=no-new-privileges` — prevents privilege escalation
- `--network none` by default — no outbound network access
- Container is removed on exit (`--rm`)
- Extra mounts default to read-only
- Container runs as your host UID/GID (not root)

## Limitations

- **Extensions with native dependencies** — the container runs Linux. Extensions
  with binaries compiled for macOS (via `node-gyp`, etc.) will fail.
  Pure JS/TS extensions and pi packages work fine.
- **macOS UID mismatch** — the container runs as your host UID via `--user`,
  but that UID doesn't exist in the container's `/etc/passwd`. Most tools
  handle this gracefully, but some may warn or fail.
- **Windows** — not supported natively. Works under WSL.
- **No image auto-update** — the Docker image is built once. Use `--rebuild`
  to pick up new pi versions or security patches. The script warns if the
  image is older than 30 days.
- **No `--env` flag** — only known API keys and `PI_*` vars are forwarded.
  Arbitrary environment variables cannot be passed through.
- **Host `.pi/agent` mounted read-write** — the container can modify your
  pi config, sessions, and lock files.

## Uninstall

```bash
npm uninstall -g pi-sandbox
docker rmi pi-sandbox
```

## Troubleshooting

| Problem | Solution |
|---|---|
| "Docker is not installed" | Install [Docker Desktop](https://docs.docker.com/get-docker/) |
| "Docker daemon is not running" | Start Docker Desktop |
| Image build fails | Check your internet connection. Docker Hub or npm may be down. |
| "permission denied" on files | Files created in the container are owned by your host UID. If you previously ran without `--user`, old files may be owned by root. Run `sudo chown -R $USER .` |
| Container can't reach API | Make sure you used `--network` and your API key is exported |
| pi version is outdated | Run `pi-sandbox --rebuild` |

## Requirements

- [Docker](https://docs.docker.com/get-docker/) (Podman is not currently supported)
- [Pi](https://github.com/earendil-works/pi-coding-agent) (installed in the container image automatically)
