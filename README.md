# Memogram Docker

[![Build Memogram](https://github.com/ZeroReiNull/memogram-docker/actions/workflows/docker.yaml/badge.svg)](https://github.com/ZeroReiNull/memogram-docker/actions/workflows/docker.yaml)

Automated, multi-platform Docker builds of [Memogram](https://github.com/usememos/telegram-integration), the Telegram bot integration for [Memos](https://github.com/usememos/memos).

This is an **unofficial image-build repository**, not a fork or a modified version of Memogram. The GitHub Actions workflow downloads the upstream source and publishes images to GitHub Container Registry (GHCR).

## Container image

```text
ghcr.io/zeroreinull/memogram:latest
```

Supported platforms (when the build succeeds):

- `linux/amd64`
- `linux/arm64`

The workflow also publishes an image tag tied to the upstream Git commit:

```text
ghcr.io/zeroreinull/memogram:sha-<full-upstream-commit-sha>
```

> **Compatibility:** A successful Docker build does not guarantee that Memogram works with every Memos release. Memogram connects to a Memos **gRPC** endpoint. Check upstream compatibility before upgrading Memos, particularly across API or protocol changes.

## Quick start: Docker Compose

The example below runs Memos and Memogram in the same Compose project. Docker Compose automatically provides an internal network, so Memogram can reach Memos by the service name `memos` without exposing any extra port.

Create a `compose.yaml` file:

```yaml
services:
  memos:
    image: neosmemo/memos:stable
    container_name: memos
    restart: unless-stopped
    ports:
      - "127.0.0.1:5230:5230"
    volumes:
      - ./data:/var/opt/memos
    environment:
      MEMOS_PORT: "5230"
      MEMOS_DRIVER: "sqlite"
      MEMOS_INSTANCE_URL: "https://memos.example.com"

  memogram:
    image: ghcr.io/zeroreinull/memogram:latest
    container_name: memogram
    restart: unless-stopped
    depends_on:
      - memos
    environment:
      SERVER_ADDR: "dns:memos:5230"
      BOT_TOKEN: "REPLACE_WITH_TELEGRAM_BOT_TOKEN"
      ALLOWED_USERNAMES: "alice,bob"
```

**Replace** `memos.example.com`, `BOT_TOKEN`, and `ALLOWED_USERNAMES` with your actual settings. The Memos service configuration is an example; adapt it to your installed Memos image and release. The `SERVER_ADDR` value must point to a compatible gRPC listener, not merely an HTTP URL.

Start the services:

```bash
docker compose up -d
```

To use the bot:

1. Create a Telegram bot with [@BotFather](https://t.me/BotFather) and put its token in `BOT_TOKEN`.
2. Generate an access token in your Memos account.
3. Open your Telegram bot and send `/start <your_memos_access_token>`.
4. Send a text message to test saving a memo.

Memogram also documents photo/document uploads and `/search <keywords>`; feature availability depends on the upstream version and Memos compatibility.

## Configuration

| Environment variable | Description | Example |
| --- | --- | --- |
| `SERVER_ADDR` | Memos gRPC address reachable from the container | `dns:memos:5230` |
| `BOT_TOKEN` | Telegram bot token from BotFather | `123456:...` |
| `ALLOWED_USERNAMES` | Optional comma-separated Telegram usernames, without `@` | `alice,bob,charlie` |
| `BOT_PROXY_ADDR` | Optional Telegram Bot API proxy URL | `https://proxy.example.com` |

Usernames are matched case-insensitively by the upstream implementation. An empty or omitted `ALLOWED_USERNAMES` allows any Telegram user to interact with the bot; set an allowlist when the bot is private.

**Security:** The inline Compose example is convenient but stores your bot token in plaintext. Do **not** commit a real token to this public repository. Keep deployment files private, limit their file permissions (for example, `chmod 600 compose.yaml`), and rotate any exposed tokens. A user's Memos access token is also sensitive.

## Automatic builds

The intended GitHub Actions workflow:

1. Checks the upstream [`usememos/telegram-integration`](https://github.com/usememos/telegram-integration) `main` branch on a daily schedule.
2. Skips rebuilding when the upstream commit already has a published SHA-tagged image (for publicly accessible packages).
3. Uses Docker Buildx and QEMU to build `linux/amd64` and `linux/arm64` images.
4. Publishes `latest` and a `sha-...` tag to GHCR.
5. Supports manual runs from the **Actions** tab, including a forced rebuild if configured in the workflow.

See [GitHub Actions](https://github.com/ZeroReiNull/memogram-docker/actions) for actual run results and logs. Scheduled GitHub Actions runs can be delayed or disabled for inactive repositories. Upstream dependency or base-image changes alone may not trigger a rebuild when the upstream commit is unchanged.

### Updating a running container

Publishing a new GHCR image **does not automatically update your VPS**. Update when you're ready:

```bash
docker compose pull memogram
docker compose up -d memogram
```

For more reproducible deployments, pin an upstream `sha-...` tag instead of `latest`.

## Troubleshooting

- **Image pull denied / not found:** Check that the [Actions workflow](https://github.com/ZeroReiNull/memogram-docker/actions) completed successfully and that the GHCR package is public, or authenticate with `docker login ghcr.io`.
- **Bot does not respond:** Check `BOT_TOKEN`, `ALLOWED_USERNAMES`, network access to Telegram, and `docker compose logs memogram`.
- **gRPC connection error:** Confirm that `memos` resolves inside the Compose network, that the target port is correct, and that your Memos release actually supports the gRPC protocol required by Memogram.
- **Bot works but cannot save memos:** Verify your Memos access token and upstream Memogram compatibility with your Memos version.
- **Container starts too early:** `depends_on` controls start order, not application readiness. Inspect logs and consider a proper health check if your image supports one.

## Upstream and credits

- [Memogram source and documentation](https://github.com/usememos/telegram-integration)
- [Memos project](https://github.com/usememos/memos)
- [Docker build workflow in this repository](https://github.com/ZeroReiNull/memogram-docker/actions)

All application code belongs to the respective upstream contributors. This repository only automates building and publishing container images. For application licensing, refer to the upstream repository.
