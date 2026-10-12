# Content Factory Web — сайт с продажей статей

Сайт на Vercel (бесплатно) + твой домашний компьютер как движок генерации.

## Как это работает

```
Покупатель -> Сайт (Vercel) -> Очередь заказов (Postgres/Neon)
                                     |
Твой ПК (web_agent.py) <-------------+
   Claude Web через Playwright (твой логин)
   Gemini Web / Pollinations для фото
                                     |
Покупатель видит статью и фото <-----+
```

- Сайт принимает деньги (коды пополнения, позже Rollypay) и учёт пользователей (вход через Google).
- Статьи пишет **твой компьютер** — твой сохранённый вход в Claude (Playwright), как в десктопной программе.
- Пока открыто окно агента на ПК — заказы выдаются. Закрыл окно — заказы ждут (кредиты не пропадают).
- Если заказ ждал дольше 20 минут (ПК выключен) — кредит автоматически возвращается покупателю.

## Часть 1. Движок на твоём ПК

Папка `C:\Igor\Generation_Statey_Dzen\AI_Content_Factory_Pro`:

1. Открой `agent_config.json` и впиши:
   - `site_url` — адрес сайта после деплоя, например `https://content-factory-web.vercel.app`
   - `agent_token` — любой придуманный пароль (например, 20 случайных букв и цифр). Тот же самый токен добавь на Vercel в переменную `AGENT_TOKEN`.
2. Запуск агента: двойной клик по `START_AGENT.bat`. Окно с надписью «Жду заказы» должно висеть открытым.
3. Впервые: если Claude Web попросит вход — открой в браузере агента вход (как в программе) и залогинься один раз.

Тест связки без браузера: поставь в `agent_config.json` `"fake": true` — агент будет возвращать тестовую статью. Потом верни `false`.

## Часть 2. Сайт на Vercel

1. Загрузи папку `content-factory-web` на GitHub (New repository -> Public -> upload files).
2. vercel.com -> Add New -> Project -> импорт репозитория -> Deploy.
3. Settings -> Environment Variables, добавь:
   - `SECRET_KEY` — любая длинная абракадабра (сессии)
   - `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` — из Google Cloud Console (OAuth)
   - `AGENT_TOKEN` — тот же токен, что в agent_config.json
   - `DATABASE_URL` — строка из neon.com (бесплатный Postgres)
   - `ADMIN_KEY` — пароль для выпуска кодов пополнения
4. Deployments -> Redeploy.
5. В Google Cloud Console добавь redirect URI: `https://ТВОЙ-АДРЕС.vercel.app/auth/google`.

## Деньги

**Коды пополнения (работает уже сейчас):**
1. Выпусти коды: `POST /api/admin/make-codes` с телом `{"admin_key": "ТВОЙ_ADMIN_KEY", "count": 10, "credits": 1}` (для пакета — `"credits": 30`).
2. Продавай коды через Plati/Digiseller: товар «1 статья — 200 ₽» и «30 статей — 5000 ₽», в цифровой доставке — код `CF-XXXX-XXXX`.
3. Покупатель вводит код в панели сайта -> баланс пополнен.

**Rollypay (позже):** вебхук `/api/rollpay/webhook` уже есть (заготовка). Перед боем добавить проверку подписи.

## Важно знать

- Тариф Vercel Hobby бесплатный, сайт живёт по адресу `https://....vercel.app`.
- Один Claude-аккаунт = один движок. Не продавай больше статей в день, чем выдаёт твой лимит Claude (примерно 30–50). Иначе заказы будут стоять в очереди.
- Сайт без агента не генерирует ничего: хранит заказы в очереди до 20 минут, потом возвращает кредит.

## Файлы

- `api/index.py` — все роуты: OAuth, очередь заказов, коды, вебхук, API агента.
- `api/db.py` — SQLite локально / Postgres (Neon) на Vercel.
- `public/index.html` — лендинг с ценами.
- `public/app.html` — панель покупателя.
- `web_agent.py` + `agent_config.json` + `START_AGENT.bat` — движок на твоём ПК (в папке продукта).
