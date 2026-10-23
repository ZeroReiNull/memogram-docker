# Memogram Docker

[![Build Memogram](https://github.com/ZeroReiNull/memogram-docker/actions/workflows/docker.yaml/badge.svg)](https://github.com/ZeroReiNull/memogram-docker/actions/workflows/docker.yaml)

Unofficial multi-platform Docker builds of [Memogram](https://github.com/usememos/telegram-integration), a Telegram bot for [Memos](https://github.com/usememos/memos).

**Image:** `ghcr.io/zeroreinull/memogram:latest`  
**Platforms:** `linux/amd64`, `linux/arm64`

## Quick start

Add Memogram to the same Docker Compose project as your Memos instance:

```yaml
services:
  memogram:
    image: ghcr.io/zeroreinull/memogram:latest
    restart: unless-stopped
    environment:
      SERVER_ADDR: "dns:memos:5230"
      BOT_TOKEN: "YOUR_TELEGRAM_BOT_TOKEN"
      ALLOWED_USERNAMES: "alice,bob"
```

Replace the token, usernames, and `SERVER_ADDR` with your values. The address must point to a **compatible Memos gRPC endpoint**; `memos` is the Compose service name.

```bash
docker compose up -d
```

Create a bot with [@BotFather](https://t.me/BotFather), then send `/start <memos-access-token>` to it.

## Configuration

| Variable | Purpose |
| --- | --- |
| `SERVER_ADDR` | Memos gRPC endpoint |
| `BOT_TOKEN` | Telegram bot token |
| `ALLOWED_USERNAMES` | Allowed Telegram usernames (comma-separated) |
| `BOT_PROXY_ADDR` | Optional Telegram API proxy |

> [!IMPORTANT]
> Keep tokens private. Without `ALLOWED_USERNAMES`, anyone can interact with the bot. Check compatibility with your Memos version.

## Updates

[GitHub Actions](https://github.com/ZeroReiNull/memogram-docker/actions) checks upstream daily and publishes `latest` and `sha-<commit>` images when a new revision is detected. You can also run it manually.

Update your container with:

```bash
docker compose pull memogram
docker compose up -d memogram
```

Application code belongs to the [upstream Memogram project](https://github.com/usememos/telegram-integration); this repository only builds and publishes Docker images.
