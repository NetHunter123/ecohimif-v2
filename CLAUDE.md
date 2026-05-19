# EcohimIF — Project Context

> **Новий чат?** Прочитай цей файл повністю, потім `app/CLAUDE.md` (Next.js 16 застереження).
> Робоча директорія: `C:\Claude\ecohimif\app`

---

## Що це за проект

**EcohimIF** — інтернет-магазин хімічних товарів з Івано-Франківська.
Продає: добрива, клеї, ґрунтовки, пивоварна сировина, побутова хімія.
Доставка: по Україні та ЄС (Nova Poshta). Аудиторія: B2B + B2C.

Повне ТЗ: `C:\Claude\ecohimif\ecohimIF_TZ.md`
Тест-кейси: `C:\Claude\ecohimif\ecohimIF_TestCases.md`

---

## Tech Stack (точні версії — не змінювати без причини)

| Пакет | Версія | Примітка |
|---|---|---|
| Next.js | 16.2.6 | App Router, Turbopack |
| React | 19.2.4 | |
| TypeScript | ^5 | strict mode |
| Tailwind CSS | ^4 | CSS-first, `@theme inline` |
| next-intl | ^4.12.0 | i18n routing |
| next-themes | ^0.4.6 | light/dark |
| Biome | ^2.4.15 | замість ESLint+Prettier |
| Yarn | 4.14.1 | package manager |
| Node.js | 22.19.0 | |

### Критичні Breaking Changes Next.js 16
- **`middleware.ts` → `proxy.ts`** в `src/`, експортувати `proxy` (не `middleware`)
- `reactCompiler` переміщено з `experimental` на top-level (але потребує babel plugin — не використовуємо)
- Завжди читай `node_modules/next/dist/docs/` перед написанням нового коду

---

## Поточний стан: Phase 1 ЗАВЕРШЕНО ✅

### Активна палітра: **Lab Cyan (Варіант B)**
- Light: `#F0F7FF` фон · `#0A2540` текст · `#06B6D4` акцент
- Dark:  `#030D1A` фон · `#E8F4FF` текст · `#06B6D4` акцент
- Токени в: `src/app/globals.css` (`@theme inline` + `.dark`)

### Архів Eco-Tech (Варіант A — Lime Green)
- Preview: `/uk/preview/eco-tech` (inline styles, не залежить від globals)
- CSS токени: `src/styles/eco-tech.archive.css`
- Відновити: додати `data-palette="eco-tech"` на `<html>` (селектор вже в globals.css)

---

## Структура файлів

```
app/
├── src/
│   ├── app/
│   │   ├── globals.css              ← Tailwind v4 + дизайн-токени (Lab Cyan default)
│   │   ├── layout.tsx               ← root layout, Geist fonts, robots metadata
│   │   ├── page.tsx                 ← redirect → /uk
│   │   ├── robots.ts                ← env-gated indexing
│   │   ├── sitemap.ts               ← усі локалі × сторінки
│   │   └── [locale]/
│   │       ├── layout.tsx           ← NextIntlClientProvider + ThemeProvider + Header/Footer + CookieBanner
│   │       ├── page.tsx             ← Homepage (Hero, Categories, HowItWorks, FeaturedProducts, About, CTA)
│   │       ├── pro-nas/page.tsx     ← Про нас
│   │       ├── dostavka/page.tsx    ← Доставка & Оплата
│   │       ├── kontakty/page.tsx    ← Контакти (форма з Zod валідацією)
│   │       ├── polityka/page.tsx    ← Політика конфіденційності (placeholder)
│   │       ├── umovy/page.tsx       ← Умови використання (placeholder)
│   │       ├── impressum/page.tsx   ← Impressum (placeholder, для DE ринку)
│   │       └── preview/
│   │           ├── eco-tech/page.tsx     ← Пробник палітри A (inline styles)
│   │           ├── lab-cyan/page.tsx     ← Пробник палітри B (inline styles)
│   │           └── earth-premium/page.tsx ← Пробник палітри C (inline styles)
│   ├── components/
│   │   ├── brand/Logo.tsx           ← SVG логотип (hexagon + молекула + "ecohimIF")
│   │   ├── controls/
│   │   │   ├── ThemeToggle.tsx      ← "use client", useTheme, Moon/Sun icons
│   │   │   └── LocaleSwitcher.tsx   ← "use client", UA/DE/EN pill buttons
│   │   ├── layout/
│   │   │   ├── Header.tsx           ← "use client", sticky, scroll shadow, мобільний drawer
│   │   │   └── Footer.tsx           ← 4-колонний grid, адреса, платіжні бейджі
│   │   └── ui/
│   │       └── CookieBanner.tsx     ← "use client", localStorage, role="dialog"
│   ├── i18n/
│   │   ├── routing.ts               ← defineRouting, locales: ['uk','de','en'], defaultLocale: 'uk'
│   │   ├── request.ts               ← getRequestConfig, динамічний import messages
│   │   └── messages/
│   │       ├── uk.json              ← повний набір перекладів (українська)
│   │       ├── de.json              ← повний набір перекладів (німецька)
│   │       └── en.json              ← повний набір перекладів (англійська)
│   ├── lib/
│   │   └── utils.ts                 ← cn(), formatPrice(), slugify()
│   ├── proxy.ts                     ← next-intl middleware (Next.js 16: НЕ middleware.ts!)
│   └── styles/
│       └── eco-tech.archive.css     ← Архів токенів Eco-Tech
├── biome.json                       ← linter/formatter config
├── next.config.ts                   ← withNextIntl, security headers
├── tsconfig.json
├── .env.example
├── .env.local                       ← NEXT_PUBLIC_SITE_URL=http://localhost:3000
└── package.json
```

---

## i18n маршрути

| UA (default) | DE | EN | Компонент |
|---|---|---|---|
| `/uk` | `/de` | `/en` | Homepage |
| `/uk/pro-nas` | `/de/pro-nas` | `/en/pro-nas` | Про нас |
| `/uk/dostavka` | `/de/dostavka` | `/en/dostavka` | Доставка |
| `/uk/kontakty` | `/de/kontakty` | `/en/kontakty` | Контакти |
| `/uk/polityka` | — | — | Privacy |
| `/uk/umovy` | — | — | Terms |
| `/uk/impressum` | `/de/impressum` | — | Impressum |

`localePrefix: "always"` — всі URL завжди мають префікс локалі.

---

## Ключові рішення

- **Палітра**: Lab Cyan обрана після порівняння 3 варіантів (Eco-Tech / Lab Cyan / Earth Premium)
- **Tailwind v4**: CSS-first підхід з `@theme inline`, без `tailwind.config.js`
- **Biome**: замість ESLint + Prettier, конфіг у `biome.json`
- **proxy.ts**: Next.js 16 вимагає `proxy.ts` в `src/` (НЕ `middleware.ts` в root)
- **Контактна форма**: Zod схема на клієнті + honeypot field (anti-spam), без DB поки
- **Legal pages**: placeholder з robots noindex, реальний контент перед launch
- **Robots**: `NEXT_PUBLIC_SITE_INDEXABLE=false` в `.env.local` — сайт не індексується до launch
- **Каталог**: поки немає DB/CMS, фокус Phase 2 — підключити дані

---

## Що робити далі (Phase 2)

З ТЗ `ecohimIF_TZ.md` — Phase 2: Catalog & Products:
- [ ] Схема даних товарів (Zod, TypeScript types)
- [ ] Статичні дані / MDX або JSON для початкового каталогу
- [ ] Сторінка каталогу `/[locale]/kataloh/page.tsx` з фільтрацією
- [ ] Сторінка товару `/[locale]/kataloh/[slug]/page.tsx`
- [ ] Компоненти: ProductCard, ProductGrid, CategoryFilter, PriceRange
- [ ] Кошик (localStorage або server state)
- [ ] Пошук

---

## Dev сервер

```bash
# Запуск
yarn dev --port 3010

# URL
http://localhost:3010/uk

# Preview сторінки (для оцінки дизайну)
http://localhost:3010/uk/preview/eco-tech
http://localhost:3010/uk/preview/lab-cyan
http://localhost:3010/uk/preview/earth-premium
```

Launch config: `.claude/launch.json` (yarn dev --port 3010)

---

## Важливо для нового чату

1. Завжди читай `AGENTS.md` в `app/` — там застереження про Next.js 16
2. Не використовуй `middleware.ts` — тільки `src/proxy.ts`
3. Tailwind v4: немає `tailwind.config.js`, всі токени в `globals.css` через `@theme inline`
4. Форматування: `yarn check` (Biome), не ESLint
5. Активна палітра — Lab Cyan, токени: `--color-accent: #06B6D4` тощо
6. Eco-Tech збережено в `src/styles/eco-tech.archive.css` + `[data-palette="eco-tech"]` в globals
