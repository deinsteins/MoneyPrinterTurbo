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

3. **Sandbox proxy fix.** The runtime exports a `no_proxy` list
   containing bracketed IPv6 entries (`[::1]`) that crash the Python
   HTTP stack's proxy parsing (`Invalid port: ':1]'`). The service
   unit sets a sanitised `NO_PROXY=localhost,127.0.0.1`, after which
   LLM calls succeed through the egress proxy.

## Verified behaviour

Script generation verified live through the WebUI's own service layer:
given the Indonesian topic "manfaat RAG untuk AI" it produced a
complete, coherent Indonesian video script via the gateway.

## Remaining step

Full video rendering needs a free [Pexels](https://www.pexels.com/api/)
API key for stock footage (`pexels_api_keys` in the local config).
Everything else in the pipeline — script, terms, voice, subtitles,
composition — is configured and ready.
