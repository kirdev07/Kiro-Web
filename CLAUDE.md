# CLAUDE.md — Kiro Web

## Mission
Build **Kiro Web**, a modern Android application that runs and manages multiple **local** websites, PHP applications, and PHP-based social-platform bots without root, Termux, or a separately installed server app. The app includes an Android control panel and a first-party web administration panel accessible from the device and, when explicitly enabled, trusted LAN clients.

Repository: https://github.com/kirdev07/Kiro-Web
Project owner: KirDev Studio / kirdev07.
Default UI direction: clean black-and-white, Material 3, modern typography and motion.
Primary UI language: Russian; design all strings for localization.

## Non-negotiable rules
1. **This is a ground-up implementation.** Do not carry forward legacy DroidPHP Java UI, outdated dependencies, bundled executables, unsafe shell/root assumptions, old download URLs, or obsolete build system. Retain upstream history, copyright notices, and license files for attribution and auditing.
2. **Do not misrepresent progress.** Never claim Android 13–17 compatibility, no-root support, PHP execution, persistent uptime, ARM64 support, or working APK without testing and recording evidence.
3. **Preserve the main README and licensing notices.** Verify licenses of *all* bundled PHP/runtime/server/database/SFTP components and comply with attribution, redistribution and source obligations. Do not relicense third-party code by renaming it.
4. **One APK is the distribution target** (no separate app, root, Termux, or mandatory runtime download). If an embedded native runtime is not feasible for a target ABI, document the limitation rather than silently downloading executables.
5. **Local-first, secure by default.** Default listener is 127.0.0.1. LAN binding requires an explicit user opt-in, credentials and clear exposure warnings. Never enable public internet access or router port forwarding automatically.
6. **Never commit secrets**, tokens, credentials, keystores, production certificates or local databases. Use Android Keystore-backed protection for secrets and avoid logging them.
7. **No destructive repository-wide rewrite without an incremental plan.** Work in small, buildable milestones, report changed files and tests, and keep TASKS.md and ERROR.md up to date.
8. Do not introduce remote code execution via the admin API: no arbitrary shell command endpoint, no unrestricted path traversal, no unauthenticated file writes.
9. Do not remove unrelated project data or upstream notices while cleaning the old code. Keep an archived legacy tree or dedicated historical branch before removing legacy files from the new build.

## Product scope
### Sites
- Create, import, rename, enable, disable, and delete sites, with confirmation for destructive actions.
- Import site files from Android Storage Access Framework or ZIP; validate ZIP entries, canonical paths, sizes, file counts, symlinks and extraction quotas to prevent Zip Slip and decompression bombs.
- Per-site document root, optional PHP handler, logs, request counts, connection limits, status and storage use.
- Serve multiple sites using **one shared HTTP listener** and host/path routing where practical. Avoid requiring one privileged or unique port per site.
- Site limits are user-configurable admission controls (e.g., max configured sites, concurrent requests, request timeout, body size), not false promises of OS-level CPU/RAM isolation.
- Use example hosts under `.test` for documentation. Friendly LAN hostnames require a real DNS/mDNS solution; IP:port is the reliable fallback.

### PHP runtime and bots
- Package compatible PHP runtime(s) for supported ABIs; verify Android Bionic linkage, executable launch constraints, process lifetime, and distribution rights. Never assume generic Linux PHP binaries work on Android.
- Support PHP scripts for supported public APIs (e.g., Telegram/VK) with outbound HTTP, scheduled jobs, and optional long polling, respecting API terms.
- Bot manager: create/import bot, start/stop/restart, crash backoff, stdout/stderr logs, schedules, environment/secrets, resource telemetry, and explicit opt-in autostart.
- Incoming public webhooks are **out of scope for local-only v1** unless a separate, explicitly configured HTTPS-reachable ingress exists. Never imply local IPs receive internet webhooks.
- Separate web request PHP execution from long-running worker processes. Avoid spawning unlimited PHP workers.

### Web administration
- Ship the admin UI as **bundled local assets**; no external CDN required.
- Admin API: versioned, authenticated, least-privilege endpoints for status, sites, bots, logs, files, settings, network interfaces and metrics.
- Use secure session cookies (HttpOnly, SameSite, Secure when HTTPS), CSRF protections where relevant, rate limiting, password hashing with an appropriate password KDF, login lockout/backoff and session expiration.
- Password setup on first launch; no default password. LAN admin access must be opt-in and protected. Consider TLS for LAN use and clear certificate trust guidance.
- Admin panel must be responsive on phone, tablet and desktop. Expose live updates through bounded SSE/WebSocket connections with authentication.
- Display the **actual** device IP(s), port(s), reachable URLs and QR codes; do not hardcode example addresses.

### Network files and discovery
- v1: authenticated WebDAV with restricted roots and per-user permissions, if implementation is feasible.
- Later: SFTP using an audited library; do not confuse FTP with SFTP or expose plain FTP by default.
- Android/Windows/Linux network discovery is best-effort via appropriate service advertisement (mDNS/NSD). Do not promise automatic appearance in every OS file explorer.
- Never expose the whole device filesystem. Restrict access to app-owned site directories or explicitly granted SAF documents.

### Monitoring and safeguards
- Show server process health, uptime, PHP worker counts, per-site requests/errors, throughput, active connections, disk use and available app-level memory.
- Label estimated or unavailable CPU/RAM measurements accurately. No guarantee of per-site hard CPU/RAM quotas without kernel-level isolation.
- Adjustable concurrent request caps, worker caps, queue length, timeouts, upload limits, per-site disk quotas, retention policies, and automatic crash backoff.
- Logs must redact secrets, rotate, enforce storage caps and support filtering/export.

## Technical architecture (proposed; validate via prototypes)
- **Android:** Kotlin, Gradle Kotlin DSL, AndroidX, Jetpack Compose, Material 3, Coroutines/Flow, lifecycle-aware UI, repository/data layers, Room for metadata, DataStore for preferences.
- **Runtime:** Foreground service for user-visible server operation where platform policy permits; explicit notification and graceful start/stop. Android battery, background execution, boot and foreground-service rules vary by OS and target SDK. Document limitations; never promise 24/7 uptime.
- **HTTP:** Choose a maintained, license-compatible embedded HTTP stack after benchmarking and threat modeling; define an abstraction for routing, static assets, PHP dispatch, TLS and request metrics.
- **PHP:** Evaluate a legally redistributable Android-native PHP integration. Create an early real-device proof of concept before committing to the rest of the feature set. Do not fabricate binaries or fake PHP output.
- **Storage:** app-private storage for server roots, metadata and secrets; SAF for user-controlled import/export. Explicit backup/restore.
- **Admin:** local web UI + authenticated API, separate from hosted user sites by listener, origin and access policy where feasible.
- **ABI:** prioritize arm64-v8a, then assess other ABIs. Verify packaging, native execution and Play/distribution constraints on actual devices.
- **Networking:** loopback by default; explicit LAN bind to chosen interface and port; handle Wi-Fi changes, hotspot, IPv4/IPv6, conflicts and unavailable interfaces.

## Security acceptance criteria
- Threat model: untrusted PHP code and website uploads can access sensitive application capabilities; do not call this a strong sandbox. Offer clear warnings, permissions boundaries, and avoid running untrusted third-party code by default.
- Admin endpoints require auth; file APIs reject `../`, symlink escapes and alternate encodings; ZIP import has bounds; HTTP endpoints have input validation and rate limits.
- Keep credentials out of URLs, logs and source; token values masked in UI.
- No implicit public exposure, UPnP, reverse tunnels or third-party telemetry.
- Test on-device network behavior and unauthorized access from a second LAN client.

## Milestones — implement in order
**M0 — Audit and foundation**
- Inventory legacy files, licenses, native blobs, vulnerabilities and architecture. Preserve original material in an archive/branch; document what can be deleted.
- Create modern Android Gradle/Kotlin project, CI lint/tests/build, reproducible dependency versions, app ID and basic Material 3 shell.
- Add TASKS.md, ERROR.md, ARCHITECTURE.md, SECURITY.md, LICENSES.md.

**M1 — Real local HTTP proof of concept**
- Start/stop a real local HTTP server on Android without root, using foreground service.
- Serve bundled static HTML and imported static site from app-private storage.
- Show actual loopback URL, local IP and optional LAN access; test a second device.
- Implement basic status and bounded logs. Produce a working debug APK.

**M2 — Multi-site hosting**
- Site manager CRUD, routing, limits, ZIP/SAF import, per-site logs, safe deletion and persistent metadata.
- Add integration tests for routing, invalid uploads, path traversal and port collisions.

**M3 — PHP proof of concept**
- Build/package/test ARM64 PHP runtime in one APK on actual Android devices.
- Execute a simple PHP script, pass HTTP request/response correctly, constrain concurrency and handle failures.
- Document supported PHP modules, compatibility and runtime licenses.

**M4 — Secure bundled web admin**
- Implement first-run credential setup, authenticated API, dashboard, site management, logs and monitoring.
- Test session security, CSRF, brute-force controls, LAN opt-in and origin separation.

**M5 — PHP bot manager**
- Long-running PHP workers, schedules, restart policy, token storage, logs and sample Telegram/VK API integrations.
- Demonstrate bot operation without implying external webhook support.

**M6 — Network files and polish**
- Authenticated WebDAV, optional audited SFTP, network discovery, QR sharing, backup/restore, localization, accessibility, dark/light themes, performance tests and release pipeline.

**M7 — Compatibility and release**
- Device test matrix across supported Android versions (target Android 13–17 only after validation), arm64, Wi-Fi/hotspot, locked screen, battery optimization, reboot, low storage, multiple sites and concurrent bots.
- Release notes, APK checksums, signing and full third-party notices.

## Definition of done for every milestone
- Code compiles in CI and on a documented local environment.
- Relevant unit/integration tests pass; real-device tests recorded where required.
- No placeholder functionality presented as working.
- README, TASKS.md, ERROR.md, ARCHITECTURE.md and license inventory updated as applicable.
- Report: implemented behavior, exact files changed, test commands/results, remaining risks, and next action.

## Instructions to coding agents
Before editing:
1. Inspect repository structure and current branch. Identify whether legacy import is complete.
2. Read README.md, this CLAUDE.md, LICENSE.DroidPHP, LICENSE.original and any existing task/error documents.
3. Write a concise plan and execute **only the current milestone**. Do not simultaneously rewrite UI, native runtime, network layer and bot manager.
4. Prefer maintainable, tested code and well-scoped commits. Avoid unnecessary dependencies.
5. When blocked by Android platform limitations, create a minimal reproduction and document evidence rather than inventing a workaround.

Priority right now: **M0 first, then M1**. The first deliverable is a genuinely running, no-root, local HTTP server APK with a modern Android shell—not a mock dashboard.
