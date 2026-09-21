# Render + GitHub deployment

This project runs a Telegram bot and exposes a small Flask health endpoint at
`/health` for Render and UptimeRobot.

## Before pushing

1. Confirm that `main.py` contains no token or private key.
2. Keep `.env`, `inf/`, and `upload_bots/` out of Git. They are ignored by
   `.gitignore`.
3. Rotate the Telegram bot token immediately if it was ever committed or
   shared publicly.

## Render

The included `render.yaml` creates a Python web service with:

- `python main.py` as the start command
- `/health` as the health-check path
- a 1 GB persistent disk mounted at `/var/data`
- `DATA_DIR=/var/data`, so the SQLite database and uploaded files survive
  normal restarts and deploys

The Render service must have these environment variables:

- `BOT_TOKEN` (required)
- `ADMIN_ID` (required numeric Telegram user ID)
- `BINANCE_API_KEY`, `BINANCE_SECRET_KEY`, and `BINANCE_PAY_ID` (only if
  Binance Pay features are used)

Do not put any of these values in GitHub or in this document.

## UptimeRobot

After the Render service becomes live, create an HTTP(s) monitor:

- URL: `https://YOUR-RENDER-DOMAIN.onrender.com/health`
- Monitor type: HTTP(s)
- Expected status: `200`
- Interval: 5 minutes or slower

The monitor should check `/health`, not the Telegram webhook or a private
admin route.

## Important operating note

This bot uses Telegram long polling. Run only one live instance with the same
`BOT_TOKEN`; a second instance can cause polling conflicts and duplicate or
missing updates.