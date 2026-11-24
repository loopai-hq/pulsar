---
name: pulsar
description: Install, start, connect and stop Pulsar, the local Qwen3.8-27B, Qwen3.6-35B-A3B and Inkling 2 Mini server for Apple Silicon. Use when the user wants to run Pulsar or point a coding agent (OpenCode, Claude Code, Codex, Hermes) at it.
---

# Pulsar

Pulsar serves Qwen3.8-27B, Qwen3.6-35B-A3B or Inkling 2 Mini on one Apple silicon Mac, with the OpenAI and Anthropic
APIs on `http://127.0.0.1:8000`. Commands below run from the `pulsar` folder of the download. If you don't know where
it is, ask the user.

## 1. Check the Mac

- Apple silicon (M3 or newer), macOS 26.4 or later, Python 3.12 to 3.14.
- 24 GB Mac: before starting, ask the user to run `sudo sysctl iogpu.wired_limit_mb=20480` themselves (it resets on
  reboot). Never run sudo for them. Then add `SPLASH_TEXT_ONLY=1` in front of the serve command in step 3.

## 2. Install (once)

Download `pulsar-1.2.1-macos-arm64.tar.gz` from https://github.com/loopai-hq/pulsar/releases/latest, then:

```bash
tar -xzf pulsar-1.2.1-macos-arm64.tar.gz && cd pulsar
```

If `./pulsar` says macOS quarantined the download, run the `xattr` command it prints. Each model downloads on its
first `serve`. Building from source needs Xcode; see the README.

## 3. Start the server

Run one in its own terminal or as a background process, and leave it running:

```bash
./pulsar serve --model mlx-community/Qwen3.8-27B-4bit                            # 20 GB
./pulsar serve --model mlx-community/Qwen3.6-35B-A3B-4bit                        # 21 GB
./pulsar serve --model mlx-community/Inkling-2-Mini-4bit                         # 17 GB
```

- Inkling 2 Mini: Pulsar also fetches its draft, the 0.6 GB `mtp.safetensors` of `thinkingmachines/Inkling-2-Mini`,
  by itself, and serves the model's text. It takes guesses straight from the prompt by default
  (`SPLASH_PROMPT_LOOKUP=0` turns that off).
- The first start prepares Pulsar's GPU kernels for this Mac; later starts are ready in seconds.
- It's ready when it prints `Ready` and `curl -s http://127.0.0.1:8000/status` shows `"ready": true`.
- The defaults are the tuned settings. Don't add other `SPLASH_` variables.
- `--port 8080` picks another port. `--max-context 128K` sets the context limit (the default is automatic).
- If startup prints a memory breakdown and stops, the model doesn't fit: close apps, or on a 24 GB Mac do the steps in
  section 1.

## 4. Connect a coding agent

With the server running, from the `pulsar` folder:

```bash
./pulsar opencode    # or: ./pulsar claude / ./pulsar codex / ./pulsar hermes
```

- The agent starts pointed at Pulsar, with the model and context limit set.
- Arguments after `--` go to the agent, e.g. `./pulsar claude -- -p "hello"`.
- Set `SPLASH_PORT` if the server isn't on port 8000.

To add Pulsar to OpenCode's own config (`opencode.json`) instead, use an OpenAI-compatible provider. For Inkling 2 Mini:

```json
{
  "provider": {
    "pulsar": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Pulsar",
      "options": { "baseURL": "http://127.0.0.1:8000/v1", "apiKey": "local" },
      "models": { "mlx-community/Inkling-2-Mini-4bit": { "name": "Inkling 2 Mini", "reasoning": true } }
    }
  },
  "model": "pulsar/mlx-community/Inkling-2-Mini-4bit"
}
```

For another model, use the `--model` name the server was started with, e.g. `mlx-community/Qwen3.8-27B-4bit`.

Any other app: OpenAI API at `http://127.0.0.1:8000/v1`, Anthropic API at `http://127.0.0.1:8000`, and the model the
server was started with. Any API key works unless the server was started with `SPLASH_API_KEY`.

## 5. Stop

Press Ctrl+C in the server terminal, or stop the background process you started.
