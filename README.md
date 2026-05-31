# 🤖 AI Lead Generation Bot — Real Estate

> Telegram бот на n8n, який скрапить сайти компаній та генерує персоналізовані cold emails для B2B-продажів у сфері нерухомості.

![Status](https://img.shields.io/badge/status-MVP-brightgreen)
![Platform](https://img.shields.io/badge/platform-n8n-orange)
![AI](https://img.shields.io/badge/AI-GPT--4o-blue)

---

## 🎯 Що робить бот?

1. **Отримує домен** компанії через Telegram (напр. `example-realty.com`)
2. **Скрапить сайт** через Firecrawl API (головна сторінка, очищений контент)
3. **Аналізує через GPT-4o**: визначає сферу, болі клієнта, знаходить контактний email
4. **Генерує cold email** — персоналізований, за структурою Hook → Pain → Value → CTA
5. **Зберігає в Google Sheets** — повний рядок з аналізом + email
6. **Сповіщує в Telegram** — готовий результат з форматуванням

---

## 📊 Архітектура

```
[Telegram Bot] ──► Отримує домен від користувача
       │
       ▼
[Extract Domain] ──► Валідація та очищення домену
       │
       ├── ❌ Невалідний ──► [Error Message] ──► Telegram
       │
       ▼ ✅ Валідний
[Processing Message] ──► "⏳ Обробляю..." в Telegram
       │
       ▼
[Firecrawl API] ──► Скрапінг сайту (onlyMainContent: true)
       │
       ▼
[Prepare LLM Input] ──► Очищення контенту + побудова промпту
       │
       ▼
[OpenAI GPT-4o] ──► Аналіз + генерація cold email (JSON)
       │
       ▼
[Parse & Format] ──► Парсинг JSON відповіді
       │
       ▼
[Google Sheets] ──► Запис рядка з усіма даними
       │
       ▼
[Telegram Notify] ──► Форматований результат користувачу
```

---

## 🚀 Швидкий старт (10 хвилин)

### Крок 1: Підготовка API ключів

| Сервіс | Де отримати | Вартість |
|--------|-------------|----------|
| **Firecrawl** | [firecrawl.dev](https://firecrawl.dev) | 500 безкоштовних скрапів |
| **OpenAI** | [platform.openai.com](https://platform.openai.com/api-keys) | ~$0.005 за запит (GPT-4o) |
| **Telegram** | [@BotFather](https://t.me/BotFather) | Безкоштовно |
| **Google Sheets** | [Google Cloud Console](https://console.cloud.google.com) | Безкоштовно |

### Крок 2: Створити Google Sheet

Створи нову Google Sheets таблицю з такими заголовками в першому рядку:

| Дата | Компанія | Домен | Сфера | Сервіс | Біль | Email | Тема | Лист |
|------|----------|-------|-------|--------|------|-------|------|------|

Скопіюй **Spreadsheet ID** з URL:
```
https://docs.google.com/spreadsheets/d/[ЦЕЙ_ID_СКОПІЮЙ]/edit
```

### Крок 3: Налаштувати Credentials в n8n

#### 3.1 — Firecrawl (Header Auth)
1. В n8n → Settings → Credentials → Add Credential
2. Тип: **Header Auth**
3. Name: `Firecrawl API Key`
4. Header Name: `Authorization`
5. Header Value: `Bearer fc-YOUR_API_KEY`

#### 3.2 — OpenAI (Header Auth)
1. Тип: **Header Auth**
2. Name: `OpenAI API Key`
3. Header Name: `Authorization`
4. Header Value: `Bearer sk-YOUR_API_KEY`

#### 3.3 — Telegram Bot
1. Тип: **Telegram API**
2. Name: `Telegram Bot`
3. Bot Token: `YOUR_BOT_TOKEN` (від @BotFather)

#### 3.4 — Google Sheets
1. Тип: **Google Sheets OAuth2 API**
2. Слідуй [офіційній інструкції n8n](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/)

### Крок 4: Імпорт воркфлоу

1. Відкрий n8n
2. Натисни **"..."** → **"Import from File"**
3. Вибери файл `workflow.json`
4. Або: **Ctrl+V** і вставь вміст файлу

### Крок 5: Підключити Credentials

Після імпорту:
1. Клікни на кожну ноду з ⚠️ (червоний трикутник)
2. Обери відповідний credential зі списку
3. В ноді **"Save to Google Sheets"** — введи свій Spreadsheet ID в поле Document
4. В ноді **"Save to Google Sheets"** — перевір назву листа (має бути `Ліди`)

> ⚠️ **Важливо:** Після імпорту перевір, що в ноді **"Firecrawl Scrape"** поле **JSON Body** має увімкнений режим Expression (іконка `fx` зліва від поля). Якщо поле починається зі знаку `=`, видали його — має залишитись тільки `{{ JSON.stringify(...) }}`.

### Крок 6: Активація

1. Натисни **"Test Workflow"** і відправ тестовий домен боту
2. Якщо все працює — натисни **"Active"** тогл вгорі

---

## 💬 Як користуватись

Відправ боту в Telegram будь-яке повідомлення з доменом:

```
example-realty.com
```

або з командою:

```
/lead example-realty.com
```

Бот прийме будь-який формат:
- `example.com` ✅
- `https://www.example.com` ✅
- `www.example.com/about` ✅
- `/lead example.com` ✅

---

## 💰 Вартість використання

| Компонент | Ціна за 1 лід | 100 лідів |
|-----------|--------------|-----------|
| Firecrawl (scrape) | ~$0.001 | $0.10 |
| OpenAI GPT-4o | ~$0.005 | $0.50 |
| Google Sheets | $0 | $0 |
| Telegram | $0 | $0 |
| **Всього** | **~$0.006** | **~$0.60** |

> 💡 100 лідів коштують менше $1. Один клієнт платить $500–5,000.

---

## 🛡️ Обробка помилок

Workflow має вбудовану обробку помилок:

- **Невалідний домен** → бот відповідає "❌ Невалідний домен" з інструкцією
- **Firecrawl API помилка** (таймаут, 500, невалідний ключ) → бот відправляє повідомлення про помилку в Telegram
- **OpenAI API помилка** (rate limit, невалідний ключ, нема балансу) → бот відправляє повідомлення про помилку в Telegram

Помилки автоматично форматуються і надсилаються користувачу, workflow не "зависає" мовчки.

---

## 🔧 Кастомізація

### Змінити нішу

1. Відкрий ноду **"Prepare LLM Input"**
2. Знайди `systemPrompt` в коді
3. Заміни "Real Estate" на свою нішу
4. Адаптуй приклади больових точок під нову індустрію

### Змінити модель

В ноді **"Prepare LLM Input"** знайди рядок:
```javascript
model: 'gpt-4o',
```

Варіанти:
- `gpt-4o` — найкраща якість ($5/1M tokens)
- `gpt-4o-mini` — швидше та дешевше ($0.15/1M tokens)
- `gpt-4.1` — новіша модель, якщо доступна

### Додати Webhook тригер

Дивись Sticky Note "💡 Webhook Alternative" у воркфлоу для інструкцій.

---

## 📁 Структура проєкту

```
ai-lead-gen-bot/
├── workflow.json           # 🎯 Головний файл — імпортуй в n8n
├── README.md               # 📖 Ця документація
├── prompts/
│   └── sdr-real-estate.txt # 📝 SDR промпт (довідкова копія)
├── .env.example            # 🔑 Шаблон API ключів (для довідки)
└── .gitignore              # 🚫 Git ignore
```

> **Примітка:** Файл `.env.example` — лише для документації. API ключі вводяться через n8n Credentials UI, а не через змінні оточення.

---

## 📈 Roadmap (v2)

- [ ] Скрапінг додаткових сторінок (/about, /contact, /services)
- [ ] Пошук email через Hunter.io API
- [ ] Автовідправка cold email через SMTP
- [ ] Batch-режим: завантаження CSV з доменами
- [ ] Дубль-чек: перевірка чи домен вже оброблений
- [ ] A/B тестування різних промптів
- [ ] LinkedIn профіль скрапінг через Firecrawl
- [ ] Інтеграція з CRM (HubSpot / Pipedrive)

---

## 📝 Ліцензія

MIT — використовуй як хочеш, зароби грошей 💪
