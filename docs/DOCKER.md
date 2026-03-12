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

## 3. Создай директории и mount allowlist

```bash
mkdir -p store groups data logs config
docker compose run --rm nanoclaw npx tsx setup/index.ts --step mounts -- --empty
```

## 4. Запусти (из папки проекта!)

```bash
cd /path/to/nanoclaw
docker compose up -d
```

Важно: запускай из папки проекта — `PWD` нужен для путей.

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

## Ошибка EACCES на /workspace/ipc/input

Исправлено: используются bind mounts для корректных путей. Обнови, пересоздай и перезапусти:

```bash
git pull origin main
mkdir -p store groups data logs config
docker compose run --rm nanoclaw npx tsx setup/index.ts --step mounts -- --empty
docker compose down
docker compose build
docker compose up -d
```

Если были данные в старых volumes — скопируй: `docker run --rm -v nanoclaw_nanoclaw_data:/from -v $(pwd)/data:/to alpine cp -a /from/. /to/`

## Ошибка "No inputs were found in config file"

Исправлено: при запуске nanoclaw в Docker монтирование agent-runner-src пропускается (агент использует встроенный код).
