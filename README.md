# AgentDesk Messenger Bot

FastAPI webhook service that auto-replies to a Facebook Page's Messenger inbox
using the Gemini API. One service can serve multiple client pages via
`clients.yaml` (page_id -> business name, products/prices, policies, language).

## Files

- `app.py` — FastAPI app: `GET /webhook` (Meta verification), `POST /webhook`
  (message events, signature-checked, replies via Gemini + Graph API)
- `clients.yaml` — per-client page config; unlisted pages use a built-in default
- `requirements.txt`, `Procfile`, `render.yaml` — Render free-tier deployment
- `.env.example` — environment variable template (no secrets in code)

## Run locally

```bash
pip install -r requirements.txt
cp .env.example .env   # fill in real values, never commit
uvicorn app:app --reload
```

Expose it for Meta with ngrok: `ngrok http 8000`, then use the https URL as the
callback URL below.

## Meta-side setup (3 clicks for the user)

1. **Create the app**: go to developers.facebook.com → My Apps → Create App →
   choose "Other" → app type "Business" → name it (e.g. `AgentDesk Bot`) → Create.
2. **Add Messenger**: in the app dashboard → Add Product → Messenger → Set up.
3. **Webhook**: Messenger → Settings → Webhooks → Add Callback URL:
   - Callback URL: `https://<your-render-service>.onrender.com/webhook`
   - Verify Token: the exact value of your `VERIFY_TOKEN` env var
   - Click Verify and Save.
4. **Subscribe to events**: in the same Webhooks section → Add Subscriptions →
   select your Page → tick `messages` and `messaging_postbacks` → Save.
5. **Page token**: Messenger → Settings → Access Tokens → connect your Facebook
   Page → Generate Token (this grants `pages_messaging`) → save as
   `PAGE_ACCESS_TOKEN`.
6. **App secret**: App Settings → Basic → copy the App Secret → save as
   `APP_SECRET`.
7. **Render env vars**: on your Render web service → Environment → add
   `VERIFY_TOKEN`, `APP_SECRET`, `PAGE_ACCESS_TOKEN`, `GEMINI_API_KEY`
   (free key from aistudio.google.com), and `GEMINI_MODEL` (`gemini-2.0-flash`).
8. **Test**: as a page admin/tester, send the Page a message on Messenger and
   watch the Render logs.

## App review note

`pages_messaging` requires App Review (and usually business verification) before
the bot can reply to the general public. While the app is in Development mode it
only works for users with a role on the app (admins, developers, testers).
Request the permission via App Review → Permissions and Features when ready.

## Notes

- Replies are generated in the background so Meta always gets its 200 fast.
- Echoes, delivery confirmations, and read receipts are ignored (no reply loops).
- Text-only for now: messages with attachments but no text are skipped.
- Render free web services sleep after ~15 min idle; the first message after a
  sleep takes ~30-60s while the service wakes. Use a paid plan or a keep-alive
  for production.
