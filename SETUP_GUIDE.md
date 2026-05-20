# EcohimIF — Setup Guide (повна інструкція)

Цей документ — покрокова інструкція щоб повністю запустити проект з нуля
до робочого e-commerce з адмінкою.

**Робоча директорія:** `C:\Claude\ecohimif\app`
**Файл секретів:** `C:\Claude\ecohimif\app\.env.local`

---

## 1. Що потрібно встановити локально

| Інструмент | Версія | Перевірка |
|---|---|---|
| Node.js | 22.19.0 LTS | `node -v` |
| Yarn | 4.14.1 | `yarn -v` (corepack увімкнено) |
| Docker Desktop | будь-яка | `docker -v` |
| (опц.) `psql` | 16 | `psql --version` |

Якщо чогось не вистачає — встанови з офіційних сайтів.

---

## 2. PostgreSQL — найшвидший шлях через Docker

### Варіант A: локальний Docker (рекомендую для розробки)

```powershell
# Запустити контейнер з Postgres 16
docker run -d --name ecohimif-pg `
  -p 5432:5432 `
  -e POSTGRES_PASSWORD=postgres `
  -e POSTGRES_USER=postgres `
  -e POSTGRES_DB=ecohimif `
  postgres:16

# Перевірити що запущено
docker ps --filter name=ecohimif-pg
```

Зупиняти: `docker stop ecohimif-pg`, знову: `docker start ecohimif-pg`.

`DATABASE_URL` тоді буде:
```
postgresql://postgres:postgres@localhost:5432/ecohimif
```

### Варіант B: Neon (free hosted)

1. Зайди на https://neon.tech → Sign in with GitHub
2. Створи проект "ecohimif", регіон Europe
3. Скопіюй **Connection string** (рядок виду `postgresql://user:pass@ep-xxx.eu-central-1.aws.neon.tech/neondb?sslmode=require`)
4. Встав у `.env.local` як `DATABASE_URL`

### Варіант C: Supabase (free hosted)

Аналогічно Neon — Settings → Database → Connection string.

---

## 3. Заповнення .env.local

Відкрий `C:\Claude\ecohimif\app\.env.local` та встанови такі змінні:

```ini
# ── Базові ────────────────────────────────────────────
NEXT_PUBLIC_SITE_URL=http://localhost:3010
NEXT_PUBLIC_SITE_INDEXABLE=false

# ── PostgreSQL ────────────────────────────────────────
# Локальний Docker:
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/ecohimif
# Або Neon/Supabase — встав свій рядок

# ── Better Auth ───────────────────────────────────────
# Згенеруй ОДИН РАЗ: openssl rand -base64 32  (або з PS: [Convert]::ToBase64String((1..32 | %{Get-Random -Max 256})))
BETTER_AUTH_SECRET=ЗАМІНИ_НА_СВОЇ_32_СИМВОЛИ
BETTER_AUTH_URL=http://localhost:3010

# ── Google OAuth (опційно — без них працює email/password) ─────
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# ── Resend (опційно — без нього листи логуються в консоль) ─────
RESEND_API_KEY=
EMAIL_FROM=noreply@ecohimif.ua
EMAIL_OPERATOR=твоя_пошта@gmail.com

# ── Nova Poshta (для checkout autocomplete міст, опційно) ───────
NOVAPOSHTA_API_KEY=

# ── Seed admin ────────────────────────────────────────
SEED_ADMIN_EMAIL=admin@ecohimif.ua
SEED_ADMIN_PASSWORD=admin12345
```

---

## 4. API-ключі — звідки взяти

### 🔑 BETTER_AUTH_SECRET (обов'язково)

Будь-який випадковий рядок 32+ символів. Швидко:

```powershell
# PowerShell:
[Convert]::ToBase64String([Security.Cryptography.RandomNumberGenerator]::GetBytes(32))
```

```bash``
# Bash:
openssl rand -base64 32
```

### 🔑 Google OAuth (опціонально, для "Увійти через Google")

1. **Google Cloud Console:** https://console.cloud.google.com/
2. Створити проект "EcohimIF"
3. **APIs & Services → OAuth consent screen** — заповнити (User Type: External, App name "EcohimIF", User support email — твоя пошта)
4. **APIs & Services → Credentials → Create Credentials → OAuth client ID**
   - Application type: **Web application**
   - Authorized redirect URIs:
     ```
     http://localhost:3010/api/auth/callback/google
     https://ecohimif.ua/api/auth/callback/google
     ```
5. Скопіювати **Client ID** → `GOOGLE_CLIENT_ID`, **Client Secret** → `GOOGLE_CLIENT_SECRET`

Якщо не задано — кнопка "Увійти через Google" не буде працювати, але інше працює.

### 🔑 Resend (опціонально, для реальних emails)

1. https://resend.com → Sign up (3000 free emails/міс)
2. **API Keys → Create API Key** → скопіюй у `RESEND_API_KEY`
3. **Domains → Add Domain** — додати свій домен `ecohimif.ua` (потрібен DNS-доступ для DKIM/SPF записів) ABO використати їхній тестовий sandbox-домен на старті
4. `EMAIL_FROM=noreply@твій-домен.com` (або `onboarding@resend.dev` для тестів)

Якщо не задано — всі листи логуються в консоль розробки (можна тестувати флоу без реальних листів).

### 🔑 Nova Poshta API (опціонально, для checkout autocomplete)

1. https://novaposhta.ua/ → Особистий кабінет → Налаштування → API
2. "Створити ключ" → `NOVAPOSHTA_API_KEY`
3. Поточна реалізація checkout використовує plain inputs для міста/відділення. Інтеграція з NP API (autocomplete) — наступний крок (TODO у Phase 3 enhancements).

### 🔑 Vercel Blob (для майбутніх uploaded images/datasheets)

Поки не використовується — продукти мають `primaryImage` як URL або data-URI (через адмін-форму). Якщо захочеш upload з диска:
1. https://vercel.com → проект → Storage → Create Database → Blob
2. Скопіюй `BLOB_READ_WRITE_TOKEN` у `.env.local`
3. Доробити в `ProductForm` upload-zone (поки немає)

---

## 5. Запуск БД — створення таблиць і даних

```powershell
cd C:\Claude\ecohimif\app

# 1. Створити таблиці згідно зі схемою
yarn db:push

# 2. Залити 31 тестовий продукт + 5 категорій
yarn db:seed
```

Якщо `db:push` запитає підтвердження — натисни Yes.
Перевірити що все добре: `yarn db:studio` → відкриє Drizzle Studio у браузері з усіма таблицями.

---

## 6. Створення адмін-аккаунту

1. Запустити dev: `yarn dev --port 3010`
2. Зайти на http://localhost:3010/uk/reestratsia
3. Зареєструватися з email `admin@ecohimif.ua` (або тим, що в `SEED_ADMIN_EMAIL`) і паролем мін. 8 символів
4. Знову виконати `yarn db:seed` — скрипт побачить існуючого користувача і **промоутне його до role=admin**
5. Зайти на http://localhost:3010/uk/admin — повинна відкритися адмінка

> Альтернатива (швидше): один раз через Drizzle Studio або psql виконати:
> ```sql
> UPDATE users SET role='admin', email_verified=true WHERE email='твоя@пошта.ua';
> ```

---

## 7. Структура для testing — карта URL

| URL | Що там | Хто має доступ |
|---|---|---|
| `/uk` | Homepage | усі |
| `/uk/katalog` | Каталог з фільтрами | усі |
| `/uk/katalog/[slug]` | Сторінка товару | усі |
| `/uk/oformlennya` | Checkout (4 кроки) | усі (guest або auth) |
| `/uk/oformlennya/dyakuyemo` | Success-сторінка | усі |
| `/uk/uvijty` | Логін | гості |
| `/uk/reestratsia` | Реєстрація | гості |
| `/uk/vidnovyty-parol` | Запит скидання | гості |
| `/uk/skydanya-parolyu?token=...` | Введення нового паролю | гості з токеном |
| `/uk/kabinet` | Профіль | auth |
| `/uk/kabinet/zamovlennya` | Мої замовлення | auth |
| `/uk/kabinet/povidomlennya` | Мої повідомлення | auth |
| `/uk/admin` | Дашборд (KPI) | admin/operator |
| `/uk/admin/produkty` | Список товарів | admin/operator |
| `/uk/admin/produkty/[id]` | Edit товару | admin/operator |
| `/uk/admin/produkty/new` | Створення товару | admin/operator |
| `/uk/admin/kategorii` | Категорії CRUD | admin/operator |
| `/uk/admin/zamovlennya` | Замовлення | admin/operator |
| `/uk/admin/zamovlennya/[id]` | Деталі замовлення | admin/operator |
| `/uk/admin/povidomlennya` | Inbox | admin/operator |
| `/uk/admin/koristuvachi` | Користувачі | тільки admin |

---

## 8. Команди розробки

```powershell
yarn dev --port 3010    # Dev сервер
yarn build              # Production build
yarn start              # Запуск production
yarn check              # Biome lint + format check
yarn format             # Auto-fix формат
yarn tsc --noEmit       # Type-check
yarn db:push            # Sync schema → DB
yarn db:generate        # Згенерувати SQL міграції
yarn db:studio          # Drizzle Studio
yarn db:seed            # Seed-data + promote admin
```

---

## 9. Що далі (мій план перевірки після setup)

Після того як ти налаштуєш PostgreSQL і виконаєш `db:push` + `db:seed`:

### Етап A: Smoke test (15 хв)
Я пройду по основних флоу:
1. Homepage → каталог → товар → додати в кошик → checkout → success
2. Реєстрація → email verification → логін → kabinet
3. Admin → створити товар → подивитися в каталозі
4. Admin → змінити статус замовлення → перевірити email (в консолі або реальний)
5. Контактна форма → перевірити що повідомлення в admin/povidomlennya

### Етап B: Перевірка дизайну (30 хв)
- Responsive: 375/768/1280/1920px
- Light/dark mode на всіх ключових сторінках
- Перевірка palette accessibility (WCAG AA контраст)
- Hover/focus states
- Loading states (skeleton/spinner)
- Empty states (порожній кошик, ніч не знайдено)

### Етап C: Логічні та структурні помилки (30 хв)
- Edge cases: stock=0, неіснуючий товар, чужий order у kabinet
- Server actions: помилки, race conditions
- Auth-protection всіх admin-routes
- CSRF/XSS перевірка
- SEO: meta-tags, JSON-LD, sitemap

### Етап D: Testcases з ecohimIF_TestCases.md (1-2 год)
Пройдемось по P0-tests:
- TC-CAT (каталог)
- TC-PROD (товар)
- TC-CART (кошик anonim + auth)
- TC-CHECK (checkout)
- TC-AUTH (login/register/reset)
- TC-ADM (admin основні)
- TC-MSG (повідомлення)
- TC-ORD (статуси + email)

### Етап E: Розширення тестових даних (опц.)
Якщо потрібно більше — створю seed з:
- 100+ продуктами
- 5-10 fake-замовленнями різних статусів
- Декількома test-користувачами (customer + operator)

---

## 10. Чек-лист "готово до перевірки"

Перед тим як писати мені "перевір":

- [ ] PostgreSQL запущений (`docker ps` показує контейнер) АБО Neon URL валідний
- [ ] `.env.local` має реальний `DATABASE_URL`
- [ ] `BETTER_AUTH_SECRET` — згенерований 32-char секрет
- [ ] `yarn db:push` пройшов без помилок
- [ ] `yarn db:seed` показав "Seed complete"
- [ ] `yarn dev --port 3010` стартує без червоних помилок
- [ ] Відкривається `/uk/katalog` і показує **31 продукт** (не 0)
- [ ] Зареєстрований admin-користувач
- [ ] (опц.) Google OAuth налаштований
- [ ] (опц.) Resend API key валідний

Коли цей чек-лист виконано — напиши мені "запусти перевірку", і я пройдуся по всіх етапах вище.

---

## 11. Відомі моменти / TODO

Ці речі не критичні для запуску, але варто знати:

1. **Image upload у адмінці** — поки тільки URL/data-URI. Інтеграція з Vercel Blob — окрема задача.
2. **Nova Poshta autocomplete** — у checkout поки plain inputs. Інтеграція з NP API — окрема задача.
3. **PDF datasheets конструктор** — `datasheets.source='form'` зберігається в БД, але endpoint `/api/datasheet/[id]/pdf` для генерації PDF з @react-pdf/renderer ще не імплементовано.
4. **Real Lighthouse 100/100/100/100** — не оптимізовано (потребує перевірки картинок, font-display, etc.).
5. **Tech-showcase (Phase 5)** — Magnetic buttons, View Transitions, animated charts, etc. — окремий блок робіт.
6. **Реальні legal-сторінки** — Impressum/AGB/Datenschutz — placeholder. Юридичну редакцію робить клієнт перед launch.

---

## 12. Якщо щось не працює

| Симптом | Можлива причина | Як перевірити |
|---|---|---|
| `ECONNREFUSED 5432` | Postgres не запущений | `docker ps` |
| `password authentication failed` | Невірний `DATABASE_URL` | Скопіювати зі step 2 |
| Каталог показує "База даних не підключена" | `db:push` не виконувався | Запустити `yarn db:push && yarn db:seed` |
| Google login не працює | OAuth credentials порожні | Заповнити `GOOGLE_CLIENT_ID/SECRET` або інакше не використовувати |
| Email не приходить | `RESEND_API_KEY` порожній — це нормально | Дивися консоль dev-сервера, там `[email:DEV-MODE]` блоки |
| Admin не пускає (redirect на /kabinet) | role != admin | Drizzle Studio → users → set role='admin' |
| 500 на `/uk/admin/*` | DB недоступна під час SSR | Error boundary покаже інструкцію |
| TypeScript errors | Кеш | `Remove-Item -Recurse -Force .next` |

---

Готово. Як буде налаштовано — пиши, продовжимо з перевіркою.
