# Linux notes (fork-specific)

Status: **untested on a real Antigravity install.** The fork author had no Linux Antigravity available.
What was verified: the TypeScript build matches the committed `dist/`, the full upstream test suite passes on Linux (Node 22),
and a synthetic ELF fixture round-trips through patch and restore.

## Paths

The installer looks for `resources/app.asar` in, in order: `--resources PATH`, `$ANTIGRAVITY_RESOURCES`, then
`~/.local/share/Programs/antigravity`, `$XDG_DATA_HOME/{antigravity,Antigravity}` (default `~/.local/share`),
`/opt/antigravity`, `/opt/Antigravity`, `/usr/share/antigravity`, `/usr/lib/antigravity`, `/usr/local/lib/antigravity` (each with `/resources`).
Override with `--resources` or `ANTIGRAVITY_RESOURCES` for any other layout (AppImage: extract it first with `--appimage-extract`, then point at `squashfs-root/resources`).

IDE (VS Code based) lookup also honors `ANTIGRAVITY_IDE_RESOURCES`, and `/opt/Antigravity`.

## Language server binary (optional `--patch-language-server`)

On Windows this patches `bin/language_server.exe`. On Linux the binary name is detected from
`language_server`, `language_server_linux_x64`, `language_server_linux_arm`, `language_server_linux_arm64`
(first one present). Set `ANTIGRAVITY_LANGUAGE_SERVER_NAME` for another file name in `resources/bin`.
ELF files are now accepted besides Windows PE. Whether the real Linux binary contains the expected endpoint string is unverified;
if not, the installer refuses and changes nothing.

## Run the proxy as a service

`packaging/linux/antigravity-add-model.service` is a systemd user unit (untested).

## Safety

Close Antigravity first. Use `npm run patch:check` before applying and `npm run patch:restore` to undo.
Do not run this against an account you cannot afford to have rate-limited or flagged.

## Local-only mode (`AG_LOCAL_ONLY=1`)

Set `AG_LOCAL_ONLY=1` in the proxy's environment (for the systemd unit, add `Environment=AG_LOCAL_ONLY=1`). Then the proxy:

- never forwards anything to `*.googleapis.com` (native Gemini requests, sign-in/quota passthrough, model list get HTTP 403, or an empty native catalog plus your saved models);
- refuses to read Antigravity's bundled OAuth client ID/secret, so "Sign in with Antigravity" is disabled.

Saved custom models with an OpenAI-compatible or Anthropic endpoint keep working. Do not save models with provider `google` or `google-cloudcode` in this mode; those provider types still call Google on their own request path (not guarded in this change).
Limits: the Antigravity app itself may still contact Google for its own login and telemetry. This switch only controls this proxy. Not tested against a real Antigravity install; only compiled.
