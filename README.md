# Content Factory Web — веб-версия сервиса

Сайт с Google-входом, балансом статей и генерацией через официальные API.
Хостинг: Vercel (бесплатный тариф).

## Локальный запуск

```
pip install -r requirements.txt
set FLASK_APP=api.index
flask run
```

Открой http://localhost:5000 (Google-вход заработает только после настройки ключей, см. ниже).

## Деплой на Vercel

1. Создай аккаунт на vercel.com (можно через Google).
2. Установи Vercel CLI: `npm i -g vercel`, затем в папке проекта: `vercel`.
3. В dashboard проекта добавь переменные окружения (Settings → Environment Variables):

| Переменная | Где взять | Зачем |
|---|---|---|
| `SECRET_KEY` | любой длинный случайный текст | сессии входа |
| `GOOGLE_CLIENT_ID` | Google Cloud Console (шаг ниже) | вход через Google |
| `GOOGLE_CLIENT_SECRET` | там же | вход через Google |
| `GEMINI_API_KEY` | aistudio.google.com/apikey (бесплатно) | генерация статей и фото |
| `DATABASE_URL` | neon.com (бесплатный Postgres) | база на Vercel (локально не нужен) |
| `ADMIN_KEY` | придумай | выпуск кодов пополнения |
| `ROLLPAY_API_KEY` | rollpay.app (если подключишь оплату) | вебхук платежей |

4. Настрой Google OAuth:
   - console.cloud.google.com → создай проект → «APIs & Services» → «Credentials» → «Create OAuth client ID» → тип «Web application».
   - В «Authorized redirect URIs» добавь: `https://твой-проект.vercel.app/auth/google`
   - Скопируй Client ID и Client Secret в переменные Vercel.
5. redeploy → сайт живой по адресу `https://твой-проект.vercel.app`.

## Как принимаются деньги (2 способа)

### Способ 1 — коды пополнения через Plati (работает сразу)
1. На plati.ru создай два товара: «Код пополнения 1 статья — 200 ₽» и «Код 30 статей — 5000 ₽».
2. Выпускай коды: `POST /api/admin/make-codes` с телом `{"admin_key": "ТВОЙ_ADMIN_KEY", "count": 10, "credits": 1}`.
3. Кладёшь код в товар на Plati → покупатель платит → вводит код на сайте → баланс пополнен.
Плюс: деньги приходят через Digiseller, который у тебя уже настроен.

### Способ 2 — Rollypay (автоматически, позже)
1. Зарегистрируйся на https://rollypay.io, создай магазин, получи API-ключ.
2. В личном кабинете Rollypay укажи webhook: `https://твой-проект.vercel.app/api/rollpay/webhook`.
3. Добавь переменную `ROLLPAY_API_KEY` в Vercel.
В коде уже есть заготовка вебхука: при успешной оплате начисляет кредиты по email покупателя. Перед боем надо добавить проверку подписи (в README помечено TODO).

## Что генерируется

- Текст: Gemini 2.0 Flash (бесплатная квота) — JSON со статьёй, лидом и тремя промптами фото.
- Фото: Google Imagen 3 (16:9), резерв — Pollinations (Flux).
- Telegram-пост не генерируется (убран из продукта).
- Публикация в Дзен из веба не делается (нужен браузер) — пользователь копирует текст и фото сам. Автопостинг остаётся фичей десктоп-версии.

## Файлы

- `api/index.py` — все роуты: OAuth, генерация, история, коды, вебхук.
- `api/db.py` — SQLite локально / Postgres (Neon) на Vercel.
- `api/gen.py` — генерация текста и фото.
- `public/index.html` — лендинг с ценами.
- `public/app.html` — панель пользователя.
