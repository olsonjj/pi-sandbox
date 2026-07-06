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

> **Note:** `--network` is required for API access. Without it, the container
> has no outbound network and cannot reach any provider.

## Security

- `--cap-drop=ALL` — drops all Linux capabilities
- `--security-opt=no-new-privileges` — prevents privilege escalation
- `--network none` by default — no outbound network access
- Container is removed on exit (`--rm`)
- Extra mounts default to read-only

## Limitations

- **Extensions with native dependencies** — the container runs Linux. Extensions
  with binaries compiled for macOS (via `node-gyp`, etc.) will fail.
  Pure JS/TS extensions and pi packages work fine.
- **macOS UID mismatch** — the container runs as your host UID via `--user`,
  but that UID doesn't exist in the container's `/etc/passwd`. Most tools
  handle this gracefully, but some may warn or fail.
- **Windows** — not supported natively. Works under WSL.
- **No image auto-update** — the Docker image is built once. To pick up new
  pi versions or security patches, run `docker rmi pi-sandbox` and re-run.
- **No `--env` flag** — only known API keys and `PI_*` vars are forwarded.
  Arbitrary environment variables cannot be passed through.
- **Host `.pi/agent` mounted read-write** — the container can modify your
  pi config, sessions, and lock files.

## Requirements

- [Docker](https://docs.docker.com/get-docker/) (Podman is not currently supported)
- [Pi](https://github.com/earendil-works/pi-coding-agent) (installed in the container image automatically)
