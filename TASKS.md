# Whisper Integration Tasks

**Plan**: [/home/moot/.claude/plans/zippy-dazzling-hinton.md](/home/moot/.claude/plans/zippy-dazzling-hinton.md)

## Goal

Extend llama-swappo to serve whisper-stt as a backend, providing unified OpenAI + Ollama API for Strix Halo (gfx1151).

---

## Tasks (TDD Approach)

### Phase 1: Test Infrastructure

- [ ] **1.1** Write integration test for `/v1/audio/transcriptions` endpoint with mock whisper backend
- [ ] **1.2** Write test for whisper model config parsing (cmd, env, inference-path)
- [ ] **1.3** Write test for persistent group behavior (whisper stays loaded when LLM swaps)
- [ ] **1.4** Write test for multipart form file upload routing to whisper backend

### Phase 2: Container Build

- [ ] **2.1** Update `docker/llama-swappo-halo/Dockerfile.llama-swappo-halo` to copy whisper-server binary from whisper-stt-rocm image
- [ ] **2.2** Copy whisper shared libraries to `/usr/local/lib/`
- [ ] **2.3** Verify whisper-server runs with `--inference-path /v1/audio/transcriptions`
- [ ] **2.4** Run container smoke test: whisper-server starts and responds to `/health`

### Phase 3: Configuration

- [ ] **3.1** Add `whisper` group to `applications/ai/llama-swappo/llama-swappo/config.yaml` with `persistent: true`
- [ ] **3.2** Add `whisper-large-v3-turbo` model config with correct cmd, env, aliases
- [ ] **3.3** Verify config loads without errors
- [ ] **3.4** Run test: model appears in `/v1/models` and `/api/tags` endpoints

### Phase 4: Deployment

- [ ] **4.1** Add `stt-models` volume mount to `applications/ai/llama-swappo/deployment.yaml`
- [ ] **4.2** Update justfile build recipe if needed
- [ ] **4.3** Deploy to k3s cluster
- [ ] **4.4** Run test: whisper model loads on startup (if preload configured)

### Phase 5: End-to-End Verification

- [ ] **5.1** Test `/v1/audio/transcriptions` with real audio file through llama-swappo
- [ ] **5.2** Test concurrent LLM + STT requests (verify group isolation)
- [ ] **5.3** Test whisper stays loaded when LLM model swaps
- [ ] **5.4** Verify OpenAI client compatibility (e.g., `openai.audio.transcriptions.create`)

---

## Acceptance Criteria

```bash
# STT works through unified API
curl http://llama.krohnos.io/v1/audio/transcriptions \
  -F model="whisper-large-v3-turbo" \
  -F file=@test.mp3

# Returns transcription in OpenAI format
{"text": "Hello world", ...}
```

---

## Notes

- No Go code changes required - llama-swappo already has `/v1/audio/transcriptions` endpoint
- whisper-server's `--inference-path` flag provides OpenAI compatibility
- Whisper group uses `persistent: true` to stay loaded alongside LLMs
