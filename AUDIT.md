# Source audit of upstream e77954b (read by an AI agent; not a professional security review)

Scope: package.json, installer scripts, src/ (network, child_process, credential storage), gateway sources by grep. No code was run against any Google account.

- No npm install/postinstall scripts. `gateway:install` uses `--ignore-scripts`.
- `dist/` rebuilt from `src/` with `tsc` matches the committed `dist/` (extra files only in the committed tree: tests/types).
- Outbound hosts in non-test code: Google Cloud Code, oauth2/googleapis, generativelanguage, user-chosen provider URLs (OpenAI, Anthropic, OpenRouter, Groq, etc.), api.cloudflare.com (Workers AI), opencode.ai, models.dev (model metadata), Antigravity auto-updater endpoint. No unknown or author-owned telemetry endpoint found.
- Sensitive: it extracts Antigravity's Google OAuth client id/secret from the installed app and can run its own Google login (scope cloud-platform), storing tokens locally (AES-256-GCM, key file in ~/.gemini/antigravity/). It supports a pool of Google accounts, which can trigger Google rate limits or ToS enforcement.
- package.json claims author "Google" and homepage antigravity.google. It is metadata copied from the app, not an authenticated Google origin.
- Patching changes the app's own asar (and optionally the language server binary); keep the backup and use `npm run patch:restore`.
