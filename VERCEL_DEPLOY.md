# EcohimIF — Vercel Deploy Checklist

Покрокова інструкція для public-launch на Vercel + Neon Postgres.

---

## 1. Передумови

- Git-репозиторій (GitHub/GitLab/Bitbucket) — push current code
- Neon PostgreSQL (вже налаштовано, у Neon dashboard є `DATABASE_URL`)
- Облікові записи: Vercel, Google Cloud (OAuth), Resend, Nova Poshta
- Домен `ecohimif.ua` (DNS-доступ для додавання A/CNAME записів)

---

## 2. Vercel project setup

1. https://vercel.com → **Add New → Project** → імпорт з Git
2. **Framework Preset:** Next.js (автодетектиться)
3. **Root Directory:** `app/` (якщо моноrepo)
4. **Build Command:** `yarn build` (за замовчуванням)
5. **Install Command:** `yarn install --immutable`
6. **Node version:** 22.x у Project Settings → General → Node.js Version

---

## 3. Environment Variables (Project Settings → Environment Variables)

Скопіювати з `.env.local` у Vercel UI. **Усі ставити для Production + Preview + Development**:

| Key | Production value | Notes |
|---|---|---|
| `NEXT_PUBLIC_SITE_URL` | `https://ecohimif.ua` | Без trailing slash |
| `NEXT_PUBLIC_SITE_INDEXABLE` | `true` | **Включити лише коли всі legal-сторінки готові** |
| `DATABASE_URL` | Neon connection string з `?sslmode=require` | Скопіювати з Neon → Connection Details |
| `BETTER_AUTH_SECRET` | новий 32+ символів секрет | `openssl rand -base64 32` |
| `BETTER_AUTH_URL` | `https://ecohimif.ua` | Same as SITE_URL |
| `GOOGLE_CLIENT_ID` | з Google Cloud Console | |
| `GOOGLE_CLIENT_SECRET` | з Google Cloud Console | |
| `RESEND_API_KEY` | `re_...` | |
| `EMAIL_FROM` | `noreply@ecohimif.ua` | **Тільки після верифікації домену в Resend** |
| `EMAIL_OPERATOR` | реальний email оператора | Сюди приходять заявки |
| `NOVAPOSHTA_API_KEY` | з NP кабінету | |
| `SEED_ADMIN_EMAIL` | той email що в Resend може приймати листи | Для seed-script у production |

**УВАГА:** не зберігай дев-секрети у prod env! Згенеруй нові секрети.

---

## 4. Google OAuth — додати prod redirect URI

[Google Cloud Console](https://console.cloud.google.com/) → OAuth client → Authorized redirect URIs:
```
https://ecohimif.ua/api/auth/callback/google
```
(Залиш дев-URI для локальної розробки.)

---

## 5. Resend — верифікувати домен

1. https://resend.com → **Domains → Add Domain** → `ecohimif.ua`
2. Додай DNS-записи (`MX`, `TXT` для SPF, `TXT` для DKIM, опц. `TXT` для DMARC) у DNS-провайдера твого домену
3. Дочекайся верифікації (зазвичай 5-30 хв)
4. Після ✓ — встановіть `EMAIL_FROM=noreply@ecohimif.ua` у Vercel env vars
5. Перезапусти deployment

---

## 6. Domain → Vercel

Vercel Project → **Settings → Domains** → Add `ecohimif.ua`.

Vercel дасть DNS-інструкції:
- Apex `ecohimif.ua` → A 76.76.21.21 (або ALIAS до cname.vercel-dns.com)
- `www.ecohimif.ua` → CNAME cname.vercel-dns.com

Після пропагації DNS (5 хв-24 год) — Vercel автоматично видасть SSL Let's Encrypt cert.

---

## 7. Image uploads — переключити з local на Vercel Blob

**Поточна реалізація:** `/api/upload` записує у `public/uploads/`. На Vercel ця папка read-only, тому upload з адмінки не працюватиме у production.

**Як виправити:**
1. Vercel Dashboard → Storage → Create Database → Blob → копіюй `BLOB_READ_WRITE_TOKEN`
2. Vercel env vars → додай `BLOB_READ_WRITE_TOKEN`
3. У `src/app/api/upload/route.ts` замінити `writeFile` на:
   ```ts
   import { put } from "@vercel/blob";
   const blob = await put(`uploads/${name}`, file, { access: "public" });
   // url: blob.url
   ```
4. У `src/app/api/admin/datasheets/route.ts` — те саме для PDF

**Тимчасова альтернатива:** використовувати URL до зовнішніх зображень (Imgur/Cloudinary/S3) у полі primary image — це поточна fallback-логіка в адмін-формі.

> Я залишаю поточний код як baseline. Перемикання — окремий PR після першого deploy.

---

## 8. Database міграції на Production

Один раз перед першим deploy:

```bash
# Локально (з вашого .env.local Neon URL)
DATABASE_URL='...neon prod url...' yarn db:push
DATABASE_URL='...neon prod url...' yarn db:seed
```

АБО через Vercel CLI: `vercel env pull` → `.env.production` → `yarn db:push`.

---

## 9. Чекліст перед натисканням Deploy

- [ ] `yarn build` локально проходить без помилок
- [ ] `yarn tsc --noEmit` чистий
- [ ] `yarn check` (Biome) без помилок
- [ ] Усі env vars додано в Vercel (Production environment)
- [ ] Neon DB має схему і seed-data
- [ ] Google OAuth redirect URI додано для https://ecohimif.ua
- [ ] Resend domain верифіковано
- [ ] DNS налаштовано для домену
- [ ] `NEXT_PUBLIC_SITE_INDEXABLE=false` (поки контент-юрист не схвалить, краще noindex)
- [ ] Реальні дані у `polityka`, `umovy`, `impressum` (зараз "Чернетка")
- [ ] EU-Vertreter додано в Datenschutz (якщо UA-operator)

---

## 10. Після першого Deploy

- [ ] Зайди на /uk — homepage рендериться
- [ ] /uk/katalog — товари видно
- [ ] /uk/admin — після логіну адмін бачить адмінку
- [ ] Створи тестовий order → перевір що email прийшов на EMAIL_OPERATOR
- [ ] Lighthouse audit (Chrome DevTools) — повинно бути ≥90 всюди
- [ ] Google Search Console → Add property → submit sitemap `https://ecohimif.ua/sitemap.xml`
- [ ] Plausible/GA4 — додати трекінг (опц)
- [ ] Sentry — error tracking (опц)
- [ ] Backup стратегія БД — Neon вже робить snapshot щогодини

---

## 11. Vercel-specific recommendations

- **Function regions:** Settings → Functions → Edge/Regional → обрати близький до Neon (frankfurt-fra1 для eu-central-1 Neon)
- **Pool connections:** Neon connection string має `-pooler` у hostname (рекомендовано для serverless Next.js)
- **Edge runtime:** `proxy.ts` (next-intl middleware) — Node runtime by default (OK)
- **Revalidation:** admin pages мають `export const dynamic = "force-dynamic"` — це коректно

---

## 12. Прапор noindex → indexable

Коли все готово:
1. Vercel env vars → `NEXT_PUBLIC_SITE_INDEXABLE=true`
2. Redeploy
3. Перевір `https://ecohimif.ua/robots.txt` → `Allow: /`
4. Submit sitemap в Google Search Console
5. Очікуй індексацію 1-7 днів

---

## 13. Troubleshooting common Vercel issues

| Симптом | Причина | Як виправити |
|---|---|---|
| `ECONNREFUSED 5432` | DATABASE_URL не задано | Перевір env vars + redeploy |
| OAuth callback повертає 400 | redirect URI не додано у Google Console | Додай `https://ecohimif.ua/api/auth/callback/google` |
| Sign-in повертає `Invalid origin` | `BETTER_AUTH_URL` не match request domain | Виправ env, redeploy |
| `upload` повертає 500 на `writeFile` | Прод файлсистема read-only | Переключися на Vercel Blob (див. розділ 7) |
| Sitemap пустий | DB unreachable при build | `force-dynamic` AOK, або prefetch у ISR |
| 404 на legal pages у DE/EN | Legal pages мають тільки `[locale]` | OK, перевір що route існує |

---

## 14. Production-ready flag

Коли все з чекліста виконано — постав emojis 🚀 і запусти live!
