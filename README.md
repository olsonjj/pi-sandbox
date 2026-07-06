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
pi-sandbox --mount ~/.pi/agent         # mount additional path
pi-sandbox --image my-image            # use custom Docker image
cat file.txt | pi-sandbox -p "summarize"
```

First run builds a Docker image (one-time). Subsequent runs are instant.

## Security

- `--cap-drop=ALL` — drops all Linux capabilities
- `--security-opt=no-new-privileges` — prevents privilege escalation
- `--network none` by default — no outbound network access
- Container is removed on exit (`--rm`)
- Extra mounts default to read-only

## Requirements

- [Docker](https://docs.docker.com/get-docker/) (Podman is not currently supported)
- [Pi](https://github.com/earendil-works/pi-coding-agent) (installed in the container image automatically)
