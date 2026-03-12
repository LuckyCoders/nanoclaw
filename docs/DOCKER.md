# Запуск NanoClaw в Docker (Telegram)

## 1. Отредактируй `.env`

```bash
nano .env
```

Заполни:
- `OPENROUTER_API_KEY` — ключ с [openrouter.ai/keys](https://openrouter.ai/keys)
- `TELEGRAM_BOT_TOKEN` — токен от @BotFather

## 2. Собери образ агента

```bash
./container/build.sh
```

## 3. Создай mount allowlist (один раз)

```bash
docker compose run --rm nanoclaw npx tsx setup/index.ts --step mounts -- --empty
```

## 4. Запусти

```bash
docker compose up -d
```

## 5. Зарегистрируй Telegram-чат

1. Напиши боту в Telegram `/chatid` — он вернёт ID чата
2. Зарегистрируй чат:

```bash
docker compose exec nanoclaw npx tsx setup/index.ts --step register -- \
  --jid "tg:ТВОЙ_CHAT_ID" \
  --name "Main" \
  --folder "telegram_main" \
  --channel telegram \
  --no-trigger-required \
  --is-main
```

Для групп: добавь бота в группу, отключи Group Privacy в @BotFather, затем `/chatid` в группе.

## Логи

```bash
docker compose logs -f nanoclaw
# или
tail -f logs/nanoclaw.log  # внутри volume
```

## Остановка

```bash
docker compose down
```

## Ошибка "No inputs were found in config file"

Если агент падает с `TS18003: No inputs were found`, пересобери образ и очисти кэш:

```bash
docker compose down
docker compose build --no-cache
docker compose up -d
# Удали пустую папку agent-runner-src (если была создана до фикса)
docker compose exec nanoclaw rm -rf /app/data/sessions/telegram_main/agent-runner-src 2>/dev/null || true
docker compose restart nanoclaw
```
