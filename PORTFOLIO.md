# Portfolio Notes — Deni Hidayat (deinsteins)

This repository is my working fork of
[harry0703/MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo),
an AI short-video generator (topic → script → stock footage → TTS
voiceover → subtitles → finished MP4). The upstream code is the starting
point; **the deployment and integration work described below is mine**,
done on my own server in October 2026.

## My work on this deployment

1. **Server deployment.** Installed and operated as a native systemd
   service (Streamlit WebUI on port 8501) in a restricted sandbox
   without Docker — dependencies resolved with `uv`, and a restore
   script recreates the service after every VM replacement.

2. **LLM rewired to a self-hosted gateway.** The default provider
   configuration was replaced with my own 9Router instance
   (OpenAI-compatible endpoint): `llm_provider = "openai"`,
   `openai_base_url` pointing at the gateway, model `deepseek-chat`
   (chosen after the gateway's Claude route hit its monthly quota).
   The gateway key lives only in the local `config.toml`, which is
   git-ignored and **not** committed here.

3. **Sandbox proxy fixes.**
   - The runtime exports a `no_proxy` list containing bracketed IPv6
     entries (`[::1]`) that crash the Python HTTP stack's proxy
     parsing (`Invalid port: ':1]'`). The service unit sets a
     sanitised `NO_PROXY=localhost,127.0.0.1`.
   - `app/services/voice.py` (committed in this fork):
     `create_edge_tts_communicate` now forwards the configured egress
     proxy to `edge_tts.Communicate` and rebuilds edge_tts'
     certifi-pinned SSL context from `SSL_CERT_FILE`, because the
     sandbox proxy demands preemptive auth and MITM-intercepts TLS.

4. **Footage source switched to Pixabay.** Pexels had paused new API
   key issuance, so `video_source = "pixabay"` with a Pixabay key
   (local config only).

5. **Narration via ElevenLabs.** The built-in Edge TTS websocket
   cannot traverse this sandbox's TLS-intercepting proxy (the proxy
   mangles the websocket upgrade), so narration uses ElevenLabs
   (`eleven_multilingual_v2`) with a personal Voice Design voice —
   free-plan API access blocks library voices but allows self-created
   ones.

## Verified behaviour

Full pipeline verified live, end to end: topic "manfaat RAG untuk AI"
→ Indonesian script via the gateway → search terms → 5 Pixabay clips
downloaded → ElevenLabs narration → subtitles → final render
**1080×1920 (9:16), 23.7 seconds, H.264 + AAC**.
