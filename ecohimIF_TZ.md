# EcohimIF — Технічне завдання та контекст

> Цей документ — самодостатній бриф для старту проекту в новому чаті.
> Скопіюй увесь зміст і встав як перший меседж — Claude матиме повний
> контекст без перенесення історії.

---

## 1. Загальний опис проекту

**Назва робоча:** EcohimIF
**Тип:** Самостійний real-client проект, незалежний від інших.
**Розташування:** `C:\Claude\ecohimif` (Windows; TZ і test-cases лежать тут, проект створимо в підпапці або поряд)
**Замовник:** Українська компанія з Івано-Франківська, дуже різноманітний асортимент: добрива (бор тощо), грунтовки для авто, клей ПВА у різних варіаціях, сировина для пивоваріння (дріжджі, солод), та інші товари промислово-хімічного та харчового профілю. Профіль регулярно змінюється — потрібна **універсальна гнучка система карток товарів**.
**Цільовий ринок:** Україна (основний) + EU (з прицілом на DACH у майбутньому). Доставка з Івано-Франківська через **Нову Пошту** (включно з міжнародними напрямками).
**Скоп:** Універсальний e-commerce-сайт із публічним каталогом, фільтрами, кошиком, замовленням (з реєстрацією і без), datasheets/специфікаціями, формами звʼязку. Адмін-панель для 3 операторів, без зовнішнього CRM.

### Тип сайту

**Hybrid B2C/B2B e-commerce** — НЕ enterprise-portal. Покупки можливі і фізичними особами, і компаніями. Платежі поки **не через сайт** — замовлення формуються як заявки, оператор зв'язується і обробляє оплату вручну. У майбутньому — інтеграція платіжних систем.

### Цілі сайту

1. Універсальний каталог зі здатністю описати будь-який товар (рідина в літрах, сипуче в кг, штучні товари, сировина) через гнучку схему атрибутів
2. Замовлення без бар'єрів — швидке оформлення без реєстрації (cart у localStorage) + реєстрація з історією замовлень
3. Інструмент для оператора: бачить нові заявки, керує статусами, відповідає клієнту
4. Шаблон datasheets / специфікацій: PDF на завантаження або форма-конструктор → авто-PDF
5. Технологічний дизайн з інтерактивними елементами (графіки, візуальні фішки), що підкреслює якість виконання

### Основна аудиторія

- **Фермери / агрохолдинги** (добрива) — UA
- **Авто-сервіси, малярі** (грунтовки) — UA
- **Будівельні компанії, ремонтники** (клей, герметики) — UA
- **Пивоварні мікро/середні** (сировина) — UA + EU
- **Дистрибʼютори** — UA, потенційно EU
- **Кінцеві споживачі** — фіз.особи через швидке замовлення

---

## 2. Технологічний стек (фінальний)

### Frontend / Framework

| Компонент | Технологія | Версія |
|---|---|---|
| Framework | Next.js | **16.x** (App Router, React Compiler, Turbopack) |
| UI Library | React | **19.2** |
| Мова | TypeScript | **5.8 strict** |
| CSS | Tailwind CSS | **v4** (CSS-first config, Oxide engine) |
| Component primitives | shadcn/ui або **@base-ui/react** | latest |
| Icons | lucide-react | latest |
| Animations | tw-animate-css | latest |

### Інфраструктура та дані

| Компонент | Технологія | Чому |
|---|---|---|
| Database | **PostgreSQL 16** | Relational, потрібно для каталогу + замовлень |
| ORM | **Drizzle ORM** | SQL-first, типобезпечно, ~7kb |
| Auth | **Better Auth 1.2** | Email/password + **Google OAuth** + magic link |
| File storage | **@vercel/blob** (free 1GB) або S3-compatible | для datasheet PDF + product images |
| Email | **Resend** (3k free/міс) або nodemailer + SMTP | order confirmations, OAuth-emails |
| Validation | **Zod** | server-action validation |
| i18n | **next-intl 4** | UK (default) / DE / EN |
| Theme | **next-themes** | light/dark |
| PDF generation | **@react-pdf/renderer** або **puppeteer-core** | гене­рація datasheets з форми + invoice PDF |
| Charts | **recharts** | data viz на головній і в admin |
| Animations | **framer-motion** | scroll-driven, page-transitions, magnetic-effects |
| Carousel | **embla-carousel-react** | product gallery |
| Image | **sharp** (auto-installed by Next) | optimization + AVIF |

### Інтеграції

- **Нова Пошта API** — для autocomplete міст/відділень при checkout, статус ТТН
  - Документація: `https://developers.novaposhta.ua/`
  - Free tier: достатньо
- **Google OAuth** — Better Auth provider, потрібен Google Cloud Console project
- **Plausible Analytics** — privacy-friendly, DSGVO-OK без opt-in для essential metrics
- **Платіжні системи (Phase 7):** LiqPay (UA), Fondy (UA), Stripe (EU/міжнар)

### Tooling

| Компонент | Технологія |
|---|---|
| Package manager | **Yarn 4** (PnP, кешування) |
| Linter + formatter | **Biome 1.9** (заміняє ESLint + Prettier) |
| Build/dev | Turbopack (вбудований у Next 16) |
| Тести | **Vitest** (unit) + **Playwright** (e2e) — додаємо в Phase 6 |

### Розгортання

- **Hosting:** Vercel (free tier на старті) або Hetzner VPS + Coolify
- **DB:** Neon або Supabase (free tier ~3GB) або Hetzner managed Postgres
- **CDN:** Cloudflare перед Vercel/Hetzner

---

## 3. Архітектура проєкту

### Підхід

**Fullstack monolith** на Next.js App Router. Весь frontend і backend у одній кодовій базі:

- **Server Components** — читання каталогу, продуктів (fast, cacheable, ISR)
- **Server Actions** — мутації (cart, inquiries, orders)
- **Route Handlers** (`app/api/`) — webhooks (Stripe, CRM, Plausible events)
- **Middleware** (`src/proxy.ts`) — i18n routing (next-intl)

### Структура каталогу

```
src/
├── app/
│   ├── [locale]/
│   │   ├── layout.tsx                  # i18n + Header/Footer + cookies
│   │   ├── page.tsx                    # Homepage
│   │   ├── produkte/
│   │   │   ├── page.tsx                # Catalog index з фільтрами
│   │   │   └── [slug]/page.tsx         # Product detail
│   │   ├── kategorien/
│   │   │   └── [slug]/page.tsx         # Category landing
│   │   ├── ueber-uns/page.tsx          # About company
│   │   ├── qualitaet/page.tsx          # Certifications, ISO, REACH
│   │   ├── kontakt/page.tsx            # Contact + inquiry form
│   │   ├── konto/                      # Customer portal (auth required)
│   │   │   ├── layout.tsx              # Auth-protected layout
│   │   │   ├── bestellungen/page.tsx   # Order history
│   │   │   ├── angebote/page.tsx       # Active quotes
│   │   │   └── profil/page.tsx         # Company profile
│   │   ├── admin/                      # Admin panel (ADMIN role)
│   │   │   ├── produkte/page.tsx       # Product CRUD
│   │   │   ├── bestellungen/page.tsx   # Order management
│   │   │   └── anfragen/page.tsx       # Inquiry triage
│   │   ├── auth/                       # Login/register flows
│   │   │   ├── anmelden/page.tsx
│   │   │   └── registrieren/page.tsx
│   │   ├── impressum/page.tsx
│   │   ├── datenschutz/page.tsx
│   │   └── agb/page.tsx
│   ├── api/
│   │   ├── webhooks/
│   │   │   ├── stripe/route.ts
│   │   │   └── crm/route.ts
│   │   └── auth/[...all]/route.ts      # Better Auth handler
│   ├── globals.css
│   ├── layout.tsx                       # Root html/body + fonts + providers
│   ├── robots.ts                        # env-gated noindex
│   └── sitemap.ts                       # dynamic sitemap
├── components/
│   ├── brand/                           # Logo, brand mark
│   ├── controls/                        # ThemeToggle, LocaleSwitcher
│   ├── layout/                          # Header, Footer, mobile menu
│   ├── catalog/                         # ProductCard, FilterBar, ProductGrid
│   ├── product/                         # ProductGallery, SpecsTable, DatasheetList
│   ├── cart/                            # CartButton, CartDrawer, CartTotal
│   ├── forms/                           # ContactForm, InquiryForm, RegistrationForm
│   ├── admin/                           # Admin tables, edit forms
│   └── page/                            # PageHero, ContentSection, FAQ, CTA
├── content/                             # Static i18n content (about, qualität)
├── i18n/
│   ├── messages/{de,en,uk}.json
│   ├── routing.ts
│   └── request.ts
├── lib/
│   ├── db/
│   │   ├── schema.ts                    # Drizzle schemas
│   │   ├── client.ts                    # postgres pool + drizzle instance
│   │   └── seed.ts
│   ├── auth.ts                          # Better Auth config
│   ├── actions/                         # Server actions: cart, inquiry, order
│   ├── email/                           # Resend templates (DE multilingual)
│   ├── crm/                             # HubSpot/Pipedrive integration
│   └── utils.ts
├── hooks/
├── types/
└── proxy.ts                             # next-intl middleware
```

### Database schema (Drizzle) — універсальна гнучка версія

> Ключова відмінність: товари мають **довільні атрибути** (jsonb specs)
> і **гнучкі варіанти** (будь-яка одиниця виміру: L, ml, kg, g, шт, м тощо).
> Один товар може бути рідиною у літрах, інший — сипучим у кг, третій —
> штучним — все через одну таблицю.

```ts
// src/lib/db/schema.ts (sketch)

// ---------- USERS / AUTH ----------
export const users = pgTable('users', {
  id: text('id').primaryKey(),                       // Better Auth: UUID
  email: text('email').notNull().unique(),
  emailVerified: boolean('email_verified').default(false),
  name: text('name'),
  phone: text('phone'),
  // OAuth: provider info stored в окремій accounts-table (Better Auth handles)
  role: text('role', { enum: ['admin', 'operator', 'customer'] }).default('customer'),
  // optional company info (для B2B клієнтів — необовʼязково)
  companyName: text('company_name'),
  taxId: text('tax_id'),                              // ЄДРПОУ / ІПН (UA) або VAT-ID (EU)
  defaultShippingAddress: jsonb('default_shipping_address'),
  createdAt: timestamp('created_at').defaultNow(),
});

// ---------- CATALOG ----------
export const categories = pgTable('categories', {
  id: serial('id').primaryKey(),
  slug: text('slug').notNull().unique(),
  parentId: integer('parent_id'),                    // self-ref для підкатегорій
  nameUk: text('name_uk').notNull(),
  nameDe: text('name_de'),
  nameEn: text('name_en'),
  descriptionUk: text('description_uk'),
  descriptionDe: text('description_de'),
  descriptionEn: text('description_en'),
  icon: text('icon'),                                // lucide-icon name або URL
  sortOrder: integer('sort_order').default(0),
});

export const products = pgTable('products', {
  id: serial('id').primaryKey(),
  slug: text('slug').notNull().unique(),
  sku: text('sku').notNull().unique(),                // системний ID, напр. "DOB-BOR-001"
  categoryId: integer('category_id').references(() => categories.id),

  // 3-мовний контент
  nameUk: text('name_uk').notNull(),
  nameDe: text('name_de'),
  nameEn: text('name_en'),
  shortDescUk: text('short_desc_uk'),                 // 1-2 речення для карток
  shortDescDe: text('short_desc_de'),
  shortDescEn: text('short_desc_en'),
  descriptionUk: text('description_uk'),              // повний опис, markdown
  descriptionDe: text('description_de'),
  descriptionEn: text('description_en'),

  // Базові поля
  basePrice: numeric('base_price', { precision: 10, scale: 2 }),
  currency: text('currency').default('UAH'),          // UAH / EUR / USD
  inStock: boolean('in_stock').default(true),
  images: jsonb('images').$type<string[]>(),          // масив URL
  primaryImage: text('primary_image'),                // окремо щоб не парсити jsonb

  // ⭐ УНІВЕРСАЛЬНІ АТРИБУТИ — key-value pairs.
  // Кожен товар має СВІЙ набір характеристик. Приклади:
  //   клей: { "В'язкість": "8000 cP", "Час сушки": "24 год", "Колір": "Білий" }
  //   добриво: { "Активна речовина": "Бор 11%", "Форма": "Гранули", "pH": "5.5-7" }
  //   сировина: { "Вміст екстракту": "80%", "Походження": "Бельгія" }
  // Адмін додає поля довільно у формі.
  attributes: jsonb('attributes').$type<Record<string, string>>(),

  // ⭐ ФАСЕТНІ ТЕГИ — для фільтрації каталогу.
  // Адмін може позначити продукт тегами: ['добриво', 'для-злаків', 'органічне']
  // На каталог-сторінці фільтр будується автоматично з усіх тегів.
  tags: jsonb('tags').$type<string[]>(),

  // SEO + структура
  metaTitle: jsonb('meta_title').$type<{ uk?: string; de?: string; en?: string }>(),
  metaDescription: jsonb('meta_description').$type<{ uk?: string; de?: string; en?: string }>(),

  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});

// ⭐ ВАРІАНТИ — універсальні: розмір + одиниця виміру + ціна-модифікатор
export const productVariants = pgTable('product_variants', {
  id: serial('id').primaryKey(),
  productId: integer('product_id').references(() => products.id).notNull(),
  sku: text('sku').notNull().unique(),                // напр. "DOB-BOR-001-5KG"

  // Кількість + одиниця (універсально):
  amount: numeric('amount', { precision: 10, scale: 3 }).notNull(), // 0.5 / 1 / 5 / 10 / 200
  unit: text('unit', {
    enum: ['l', 'ml', 'kg', 'g', 't', 'pcs', 'm', 'm2', 'm3', 'pack', 'set'],
  }).notNull(),                                       // "L", "kg", "pcs" тощо

  packaging: text('packaging'),                       // "каністра", "мішок", "відро", "пляшка"
  price: numeric('price', { precision: 10, scale: 2 }),  // конкретна ціна варіанту (НЕ модифікатор — простіше для адміна)
  stock: integer('stock').default(0),                 // кількість на складі
  weight: numeric('weight', { precision: 8, scale: 3 }), // вага брутто кг — для НП
  sortOrder: integer('sort_order').default(0),
  isDefault: boolean('is_default').default(false),    // який варіант показувати в каталозі
});

// ---------- DATASHEETS / СПЕЦИФІКАЦІЇ ----------
// 2 способи: завантажений PDF АБО форма-конструктор → авто-PDF
export const datasheets = pgTable('datasheets', {
  id: serial('id').primaryKey(),
  productId: integer('product_id').references(() => products.id).notNull(),
  source: text('source', { enum: ['uploaded', 'form'] }).notNull(),
  // якщо source=uploaded:
  fileUrl: text('file_url'),
  fileSize: integer('file_size'),
  // якщо source=form: рендеримо PDF при запиті з цих секцій
  formContent: jsonb('form_content').$type<{
    composition?: string;
    applications?: string;
    instructions?: string;
    storage?: string;
    safety?: string;
    [key: string]: string | undefined;
  }>(),
  lang: text('lang', { enum: ['uk', 'de', 'en'] }).notNull().default('uk'),
  title: text('title').notNull(),                     // "Технічний паспорт", "Інструкція"
  version: text('version'),                           // "v1.2 / 2026-04"
  uploadedAt: timestamp('uploaded_at').defaultNow(),
});

// ---------- ORDERS / ЗАМОВЛЕННЯ ----------
// Замовлення створюється БЕЗ реєстрації (швидке) або З нею.
// userId nullable — анонімне замовлення зберігає email/phone у customer-полях.
export const orders = pgTable('orders', {
  id: serial('id').primaryKey(),
  orderNumber: text('order_number').notNull().unique(), // людинозрозумілий: "EH-2026-00042"
  userId: text('user_id').references(() => users.id),   // nullable — для guest-замовлень

  // Customer info (заповнюється або з users, або анонімно)
  customerName: text('customer_name').notNull(),
  customerEmail: text('customer_email').notNull(),
  customerPhone: text('customer_phone').notNull(),
  customerCompany: text('customer_company'),            // optional
  customerTaxId: text('customer_tax_id'),               // optional

  // Доставка через Нову Пошту
  shippingMethod: text('shipping_method', {
    enum: ['novaposhta_warehouse', 'novaposhta_courier', 'pickup'],
  }).notNull().default('novaposhta_warehouse'),
  shippingCity: text('shipping_city'),                  // напр. "Київ"
  shippingWarehouse: text('shipping_warehouse'),        // напр. "Відділення №42"
  shippingAddress: text('shipping_address'),            // повна адреса для courier
  shippingNotes: text('shipping_notes'),

  // Оплата (поки manual — оператор виставляє рахунок)
  paymentMethod: text('payment_method', {
    enum: ['invoice', 'card', 'cod'],
  }).default('invoice'),
  paymentStatus: text('payment_status', {
    enum: ['pending', 'paid', 'partial', 'refunded'],
  }).default('pending'),

  // Замовлення-статус (керує адмін)
  status: text('status', {
    enum: [
      'new',           // щойно створено
      'confirmed',     // оператор підтвердив
      'processing',    // комплектуємо
      'shipped',       // передано в НП
      'delivered',     // доставлено
      'cancelled',     // скасовано
      'returned',      // повернення
    ],
  }).default('new'),
  trackingNumber: text('tracking_number'),              // ТТН НП

  // Суми
  subtotal: numeric('subtotal', { precision: 12, scale: 2 }).notNull(),
  shippingCost: numeric('shipping_cost', { precision: 10, scale: 2 }).default('0'),
  total: numeric('total', { precision: 12, scale: 2 }).notNull(),
  currency: text('currency').default('UAH'),

  // Comm
  customerNotes: text('customer_notes'),                // клієнтське
  operatorNotes: text('operator_notes'),                // внутрішнє, не видно клієнту
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});

export const orderItems = pgTable('order_items', {
  id: serial('id').primaryKey(),
  orderId: integer('order_id').references(() => orders.id).notNull(),
  productId: integer('product_id').references(() => products.id),
  variantId: integer('variant_id').references(() => productVariants.id),

  // Snapshot ціни/назви на момент замовлення (щоб історія була стабільна)
  productNameSnapshot: text('product_name_snapshot').notNull(),
  variantLabelSnapshot: text('variant_label_snapshot'), // "5 L · каністра"
  unitPrice: numeric('unit_price', { precision: 10, scale: 2 }).notNull(),
  quantity: integer('quantity').notNull(),
  lineTotal: numeric('line_total', { precision: 12, scale: 2 }).notNull(),
});

// ---------- AUTH CART (тільки для залогінених — анонім у localStorage) ----------
export const cartItems = pgTable('cart_items', {
  id: serial('id').primaryKey(),
  userId: text('user_id').references(() => users.id).notNull(),
  productId: integer('product_id').references(() => products.id).notNull(),
  variantId: integer('variant_id').references(() => productVariants.id).notNull(),
  quantity: integer('quantity').notNull().default(1),
  addedAt: timestamp('added_at').defaultNow(),
});

// ---------- MESSAGES / СИСТЕМА ЗВ'ЯЗКУ ----------
// Клієнт пише → оператор отримує email + бачить у адмінці.
// Може бути привʼязане до замовлення або просто загальний запит.
export const messages = pgTable('messages', {
  id: serial('id').primaryKey(),
  orderId: integer('order_id').references(() => orders.id),  // optional
  userId: text('user_id').references(() => users.id),        // optional
  // якщо неавторизований:
  fromName: text('from_name').notNull(),
  fromEmail: text('from_email').notNull(),
  fromPhone: text('from_phone'),
  subject: text('subject'),
  body: text('body').notNull(),
  source: text('source', { enum: ['contact_form', 'order_chat', 'product_question'] }).notNull(),
  productId: integer('product_id').references(() => products.id), // якщо питання про продукт
  isRead: boolean('is_read').default(false),
  emailSent: boolean('email_sent').default(false),
  createdAt: timestamp('created_at').defaultNow(),
});

// Відповідь оператора → також через email + запис тут
export const messageReplies = pgTable('message_replies', {
  id: serial('id').primaryKey(),
  messageId: integer('message_id').references(() => messages.id).notNull(),
  operatorId: text('operator_id').references(() => users.id).notNull(),
  body: text('body').notNull(),
  emailSent: boolean('email_sent').default(false),
  createdAt: timestamp('created_at').defaultNow(),
});

```

### Що випало порівняно з v1 ТЗ і чому

| Видалено | Причина |
|---|---|
| `companies` (окрема таблиця) | B2B-компанії стали optional-полями `users.companyName` + `taxId` |
| `quotes` + `quoteItems` | Не використовуємо quote-workflow — клієнт сам формує замовлення, оператор підтверджує |
| `inquiries` + `inquiryItems` | Замовлення=заявка одночасно (status='new' → 'confirmed'); окремих "запитів" немає |
| `certifications` + `productCertifications` | Сертифікати — це частина `products.attributes` або вкладеного `datasheets` |
| `applications` + `productApplications` | Замінено на `products.tags` (jsonb) — гнучкіше для довільного асортименту |

---

## 4. Список реалізації — фази

### Phase 1 — Foundation (1-2 тижні)

1. ✅ Scaffold Next.js 16 + Tailwind v4 + Yarn 4 + Biome
2. ✅ i18n **UK (default)** / DE / EN з locale-routing (UA-first, бо локальний бізнес)
3. ✅ Theme system (light/dark) — next-themes
4. ✅ Brand identity v1 — палітра + типографіка + логотип (див. розділ 5)
5. ✅ Header (Logo, navigation, theme-toggle, locale-switcher, cart-icon, login-button)
6. ✅ Footer (контакти, payment-methods, legal links, social, sitemap)
7. ✅ Homepage:
   - Hero з технологічним візуалом (graphics/анімація, не аерофото заводу)
   - Categories grid із іконками
   - "Як це працює" — 4 крок-ноди до замовлення
   - Featured products (3-6 карусель)
   - About preview
   - CTA "Швидке замовлення"
8. ✅ Про нас (`/pro-nas`) — історія, команда (3 людини), location IF
9. ✅ Доставка та оплата (`/dostavka`) — Нова Пошта flow, варіанти оплати
10. ✅ Контакти (`/kontakty`) з формою → email + запис у `messages`
11. ✅ Legal pages (Impressum для DE-сторінок, Конфіденційність, Умови продажу) — placeholder
12. ✅ Cookie banner (UA-DSTU + DSGVO для DE)
13. ✅ Robots.txt + meta noindex (env-gated `NEXT_PUBLIC_SITE_INDEXABLE`)

### Phase 2 — Catalog & Database (2-3 тижні)

14. PostgreSQL + Drizzle schema (універсальна, як вище)
15. Seed-script з 30-50 fake-продуктами **різноманітного асортименту**:
    - 8 добрив (бор, азот, калій, мікроелементи)
    - 6 клеїв (ПВА різні в'язкості, поліуретан, плитковий)
    - 5 грунтовок для авто
    - 8 сировинних товарів для пивоваріння (солод, хміль, дріжджі)
    - 5 побутової хімії
16. Публічний каталог `/katalog`:
    - Sidebar з фільтрами: категорія, теги (динамічні), ціна, наявність
    - Top-bar: сортування, перегляд (грід/список), кількість на сторінці
    - Search bar з autocomplete
    - Pagination (НЕ infinite scroll — краще для SEO)
    - URL зі станом фільтрів (deep-linkable)
17. Product detail `/katalog/[slug]`:
    - Photo gallery (embla-carousel + zoom)
    - Variant selector (size + unit + price + stock)
    - Quantity stepper + "Додати в кошик" + "Купити в 1 клік"
    - Attributes table (універсальна key-value)
    - Tags як chips
    - Datasheets list (download або online-view)
    - "Поставити запитання" → форма у `messages`
    - Related products
18. Category landing `/katalog/kategoriya/[slug]`
19. Schema.org/Product structured data
20. Search API (PG fulltext: tsvector по name + description + tags)
21. ISR для product pages (revalidate 1h)

### Phase 3 — Cart + Auth + Orders (3-4 тижні)

22. **Anonymous cart** — localStorage (zod-валідовано), survive page reload
23. **Authenticated cart** — DB-backed, синхронізація з localStorage при логіні
24. CartDrawer (slide-out) з підсумком, доставкою-estimate
25. Checkout-flow `/oformlennya`:
    - Step 1: контакти (name, email, phone, optional: компанія, ЄДРПОУ)
    - Step 2: доставка (НП відділення / НП кур'єр / самовивіз)
    - Step 3: оплата (поки тільки "рахунок", "при отриманні")
    - Step 4: підтвердження
    - **Гостьове замовлення без обовʼязкової реєстрації**
26. Better Auth setup:
    - Email + password
    - **Google OAuth** провайдер
    - Magic link (опційно)
    - Email verification
    - Password reset через email
27. Реєстрація `/reestratsia` + Логін `/uvijty`
28. Особистий кабінет `/kabinet`:
    - Профіль (редагування контактів)
    - Адреси доставки
    - Історія замовлень з фільтрами по статусу
    - Деталі замовлення з ТТН-кнопкою (відкриває статус на сайті НП)
    - Список повідомлень / звернень
29. Server Action `createOrder` — створює запис, відправляє email оператору
30. Email-шаблони (Resend або nodemailer):
    - Customer: order confirmation, status updates
    - Operator: new order notification (DE/UK mix залежно від клієнта)

### Phase 4 — Admin Panel (1-2 тижні) — без зовнішнього CRM

31. Admin layout `/admin` з протекцією (role='admin' або 'operator')
32. Dashboard з KPI: нові замовлення, очікують підтвердження, суми за період
33. Products CRUD:
    - Створення/редагування з гнучкими атрибутами (динамічна форма)
    - Варіанти — таблиця додавання/видалення з drag-reorder
    - Image upload (multiple, з previews)
    - Tags input (autocomplete з існуючих)
    - Datasheet: вибір "завантажити PDF" або "форма-конструктор"
34. Categories CRUD з drag-tree
35. Orders management:
    - Список з фільтрами (статус, дата, метод доставки, оплата)
    - Деталі замовлення
    - Зміна статусу через dropdown → автоматичний email клієнту
    - Додавання внутрішніх нотаток (не видно клієнту)
    - Вписання ТТН-номера
    - Експорт у PDF (інвойс)
36. Messages inbox:
    - Список запитів від клієнтів
    - Відповідь через інтерфейс → email клієнту + збереження в `message_replies`
    - Mark as read/unread
37. Users management (тільки для role='admin')
38. Settings: налаштування email-шаблонів, контактна інформація для footer

### Phase 5 — Tech-Showcase + Design Elements (1-2 тижні) ⭐

> Розділ, який підкреслить технологічний рівень виконання.
> Деталі — у розділі 5.5 нижче.

39. Інтерактивний hero з canvas/SVG-анімацією (молекули, частинки)
40. Графіки/візуалізації (recharts): "наша географія доставки", "товарів у каталозі"
41. Калькулятор замовлення (інтерактивний — підбір продукту за параметрами)
42. Progressive image loading з blur-up
43. Smooth scroll-driven animations (Framer Motion або CSS scroll-timeline)
44. View Transitions API для page changes
45. Live performance metrics в footer ("100/100 Lighthouse")
46. Theme-toggle з view-transition smooth swap
47. Cookie/consent UI на новому рівні (не банер а modal-card)
48. Skeleton loaders для catalog/checkout
49. Magnetic hover на CTAs (опційно, не overdo)
50. Cursor-spotlight ефект (опційно)
51. Page-transition loader (top-bar progress, як у YouTube)

### Phase 6 — Polish & Launch (1 тиждень)

52. Lighthouse audit → 100/100/100/100
53. AVIF images з fallback
54. Sitemap.xml dynamic
55. OG-images per product (опційно — Vercel og)
56. Analytics (Plausible)
57. Real legal data (потребує юридичної підготовки клієнта)
58. Cookie banner з real categories
59. Deployment setup (Vercel + Neon free → Hetzner коли треба)
60. Production env-flags flip

### Phase 7 — Optional Enhancements

61. Платіжні системи (LiqPay, Fondy, Stripe)
62. Newsletter (Resend або Mailcoach)
63. Blog/Insights with MDX
64. Live chat (Crisp або custom WebSocket)
65. PWA для повторних клієнтів
66. WhatsApp/Telegram-кнопка для звʼязку
67. Loyalty/bonus система

---

## 5. Brand identity + Tech-design напрям

> Бренд **створюємо з нуля** — у клієнта поки немає логотипу, палітри, шрифтів.
> Напрям обраний клієнтом: **сучасний, технологічний, з інтерактивними фішками**.

### 5.1 Палітра — пропозиції (обрати в Phase 1 — мокапи перед імплементацією)

| Опція | Базовий | Accent | Характер |
|---|---|---|---|
| **A. Eco-Tech** ⭐ | Off-black `#0A0D0A` + warm white `#FAFAF7` | Bright lime `#A3E635` | Технологічно, енергійно, "наука + еко" |
| **B. Lab Cyan** | Deep navy `#0A2540` + soft white | Electric cyan `#06B6D4` | Hi-tech лабораторний, holographic |
| **C. Earth Premium** | Deep forest `#0F2A1F` + bone | Honey amber `#D97706` | Природно, аграрно, теплий преміум |

Рекомендую **A. Eco-Tech** — найбільше підходить під "технологічний дизайн", lime-зелений підкреслює eco + дозволяє багато viz-ефектів (graphs, particles).

### 5.2 Типографіка

- **Display/Headlines:** **Geist** (Vercel) або **Manrope** — геометричні, sharp, модерні
- **Body:** **Inter** (cross-platform fallback) або та сама Geist
- **Mono для tech-elements:** **Geist Mono** або **JetBrains Mono**
- **Display для hero:** опціонально **Instrument Serif** (контраст-засіб, премʼяльна сучасність)

### 5.3 Логотип — концепція

Wordmark "**ecohim**" в нижньому регістрі + accent точка/елемент над "i". Іконічно: молекула / абстрактний symbol що відображає трансформацію. Створюємо як SVG в `src/components/brand/Logo.tsx`. Розглядаємо 3 варіанти в Phase 1.

### 5.4 Логіка скейлу / spacing / grid

- 8px grid system
- Container max-width: 1280px (з gutter)
- Section vertical spacing: 96-128px desktop, 64px mobile
- Border-radius: 4-8px (sharp-modern, не округлий cosy)

### 5.5 Tech-Showcase елементи (Phase 5)

Конкретний список "технологічних фішок" які надаватимуть сайту wow-фактор:

**Hero-секція:**
- Інтерактивна canvas-анімація (наприклад: молекулярна сітка яка реагує на курсор)
- АБО: SVG-анімація atom-молекули що повільно обертається
- АБО: WebGL particle-field (через Three.js, light-weight)
- Текст з kinetic typography (літери що зʼявляються з різних напрямків)

**Графіки / Data viz:**
- Recharts для статистики компанії (доставлено замовлень, географія, асортимент)
- Animated counters при scroll-into-view
- Радіальна карта "Звідки наші клієнти" (через d3 або simpler SVG)
- "Калькулятор підбору" — інтерактивний UI що бере вхідні параметри → видає рекомендований продукт

**Інтеракції:**
- View Transitions API при перемиканні мови / теми (smooth color sweep)
- Magnetic-buttons на основних CTA (subtle attraction до курсора)
- Cursor-trail (опціонально) або spotlight ефект для dark-mode
- Page transitions з progress-bar (top-bar, як YouTube)
- Smooth scroll-driven animations (sections fade-in, parallax depth)
- Skeleton loaders з shimmer-ефектом

**Card-design:**
- Glass-morphism для product-cards (subtle, не overdo)
- Hover-lift з shadow expansion
- Border-glow на focus
- "Зміна стану" візуалізована — добавлення в cart → element flies до cart-icon з anim

**Preview-механіка (перед імплементацією):**
- Перед фінальним кодуванням кожної фішки — створюємо mini-mockup на окремій сторінці `/preview/[element]`
- Клієнт переглядає → схвалює → інтегруємо
- Цей підхід стримує "overcooked" дизайн

### 5.6 Inspiration / референси (приклади сайтів-зразків)

Сайти що поєднують eco-industrial + сучасний tech-look:
- **linear.app** — clean tech aesthetic
- **vercel.com** — graphical hero, smooth transitions
- **modal.com** — animated mesh background
- **anthropic.com** — Claude-style serif headings + clean layout
- **rive.app** — playful but professional motion

Не копіюємо — беремо kompozіцію, ритм, deference до контенту.

---

## 6. Legal compliance — критично

### Контекст

Сайт орієнтований на DACH-клієнтів, потенційно з UA-оператором. Тому:

| Документ | Що мати |
|---|---|
| **Impressum** | Реальне імʼя + адреса оператора (UA адреса прийнятна, **не можна fake**). § 5 TMG. |
| **Datenschutz** | DSGVO Art. 13/14 повно. Якщо UA-операт, додати **EU-Vertreter** (Art. 27) — DataReps (~€99/рік) або Prighter (~€30/міс). |
| **AGB** | B2B-умови. Треба юриста для DACH. На старті — placeholder. |
| **Cookie banner** | TTDSG: opt-in для нон-essential cookies. Plausible — opt-out не потрібен (no PII). |
| **VIES VAT validation** | Для B2B-замовлень з ЄС — перевірка VAT-ID через `https://ec.europa.eu/taxation_customs/vies/services/checkVatService` |

### Критичне правило

**До запуску в публічний індекс:**
- Усі placeholder-warning банери на legal-сторінках помітні
- `NEXT_PUBLIC_SITE_INDEXABLE=false` (default) → noindex meta + robots.txt Disallow
- Sharing з клієнтами — direct URL only

**При запуску:**
- Замінити placeholder-и на реальні дані
- Підписка на EU-Vertreter (якщо UA-operator)
- Юридичний review AGB (Avocadosich.de, IT-Recht-Kanzlei.de — €100-300)
- Flip `NEXT_PUBLIC_SITE_INDEXABLE=true`

---

## 7. Розгортання

### Staging / development

- **Vercel free tier:** unlimited deploys, custom domain `*.vercel.app`
- **Neon Postgres free:** 0.5 GB database, перфектно для початку
- **Vercel Blob:** 1 GB free для datasheets PDF
- **Resend free:** 3000 emails/місяць

### Production (коли є реальний клієнт)

- **Hetzner CX22 VPS:** €4.5/міс — Next.js + Postgres + Coolify
- **Або Vercel Pro:** €20/міс — managed
- **Hetzner Object Storage:** €1/міс — для більше datasheets
- **Resend Pro:** €20/міс — 50k emails

---

## 8. Перші кроки після створення проекту

1. **Scaffold:** ініціалізувати Next.js 16 + Yarn 4 + всі залежності
2. **Конфіги:** `tsconfig.json`, `biome.json`, `next.config.ts`, `postcss.config.mjs`, `components.json`
3. **i18n:** `src/i18n/{routing,request}.ts` + 3 message-файли (de/en/uk)
4. **Database:** `src/lib/db/{schema.ts,client.ts}` + drizzle.config.ts
5. **Auth:** `src/lib/auth.ts` (Better Auth) + `app/api/auth/[...all]/route.ts`
6. **Layout:** root layout (html/body/fonts) + locale layout (header/footer/cookies)
7. **Robots & SEO:** `app/robots.ts` + `app/sitemap.ts` (env-gated)
8. **Homepage** з усіма секціями
9. **Catalog stub:** `/produkte` index + `/produkte/[slug]` detail (mock data поки)
10. **Seed:** 20 fake-продуктів через `yarn db:seed`

### Файли що зразу треба створити

```
ecohimIF/
├── package.json
├── tsconfig.json
├── biome.json
├── components.json
├── next.config.ts
├── postcss.config.mjs
├── drizzle.config.ts
├── .env.example
├── .gitignore
├── README.md
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── globals.css
│   │   ├── robots.ts
│   │   ├── sitemap.ts
│   │   └── [locale]/
│   │       ├── layout.tsx
│   │       └── page.tsx
│   ├── i18n/
│   │   ├── routing.ts
│   │   ├── request.ts
│   │   └── messages/{de,en,uk}.json
│   ├── lib/
│   │   ├── db/{schema.ts,client.ts,seed.ts}
│   │   ├── auth.ts
│   │   └── utils.ts
│   └── proxy.ts
└── public/
```

---

## 9. Уточнення Round 1 — відповіді клієнта (квітень 2026)

> Усі 10 початкових запитань зʼясовані. Документ оновлено відповідно.
> Збережено тут дослівно для контексту майбутніх рішень.

### Q1. Категорії продуктів — які основні?

> «Дуже різноманітні, специфіка роботи в тому що вони беруть різноманітні замовлення — це можуть бути різноманітні добрива (бор тощо), може бути грунтовка для машин, може бути клей ПВА різних мастей, може бути сировина для виготовлення пива, дріжджі тощо, найрізноманітніший профіль. Тому треба якась універсальна система для створення картки товару під різноманітні потреби.»

**Рішення:** flexible product schema з `attributes: jsonb` + `tags: jsonb` + універсальні variants.

### Q2. Що таке SKU?

> «Що таке SKU?»

**Відповідь:** Stock Keeping Unit — унікальний код товару для обліку. Кожен варіант (наприклад, клей ПВА 1L vs 5L) має свій SKU. Це системне поле, не для клієнта.

### Q3. Variants — як?

> «Так, варіанти можуть бути. Якщо це умовно клей — то це різниця тари: 0.5L, 1L, 3L, 5L, 10L, 15L, 20L. Це все може бути по-різному, залежить від товару. Також може бути в кілограмах і можливо ще якісь — я не знаю ще. Бажано щоб це було якось універсально для заповнення.»

**Рішення:** `productVariants` має поля `amount` + `unit` (L/ml/kg/g/t/pcs/m/m2/m3/pack/set) + `packaging`. Адмін заповнює довільну комбінацію.

### Q4. Datasheets — хто створює?

> «Поки що PDF datasheets — їх будуть завантажувати, або заповнювати на сайті і з того формувати PDF.»

**Рішення:** `datasheets.source` має 2 варіанти — `uploaded` (готовий PDF) або `form` (заповнюється через UI → PDF генерується з `@react-pdf/renderer` або puppeteer).

### Q5. Прайси і кошик — як?

> «Прайси публічні. Має бути можливість оформити замовлення без реєстрації — швидке замовлення. Кошик працює в обох варіантах: без реєстрації (зберігається в браузері на певний період) і з реєстрацією (кошик зберігається в БД, всі замовлення з можливістю переглянути їх статус, який виставляє адмін). Має бути система звʼязку — коли людина пише повідомлення і воно відправляється на пошту адміну.»

**Рішення:**
- Анонімний кошик — `localStorage` (TTL 30 днів)
- Авторизований кошир — DB-backed `cartItems`
- Merge при логіні
- Гостьові замовлення працюють повноцінно
- `messages` + `messageReplies` таблиці для системи звʼязку

### Q6. CRM?

> «CRM ще не використовується, тому не знаю що робити в цьому пункті.»

**Рішення:** **Без зовнішнього CRM.** Вбудована admin-панель в Phase 4 виконує всі CRM-функції (управління замовленнями, повідомленнями, статусами).

### Q7. Платежі?

> «У майбутньому платежі через сайт, але наразі тільки оформлення заявки і передача до адміна/оператора на пошту і в системі, який надалі опрацьовує оплату сам.»

**Рішення:**
- Phase 1-6: payment_method = `invoice` (рахунок виставляє оператор) / `cod` (при отриманні)
- Phase 7 (optional): LiqPay / Fondy / Stripe integration

### Q8. Логістика?

> «Доставка з Івано-Франківська, Нова Пошта.»

**Рішення:**
- `shippingMethod`: `novaposhta_warehouse` / `novaposhta_courier` / `pickup`
- Інтеграція з API Нової Пошти для autocomplete міст + відділень
- Збереження ТТН в `orders.trackingNumber`
- Кнопка "Відстежити" → відкриває статус на сайті НП

### Q9. Sales-team — скільки людей?

> «Там може бути по-різному, але в основному 3 людини які працюють з цим — відповідають за ведення товарів, опрацювання заявок і т.д.»

**Рішення:** ролі `admin` (1 людина, повний доступ) + `operator` (2 людини, доступ до замовлень, повідомлень, продуктів — без users management).

### Q10. Brand?

> «Цих аспектів ще немає, треба буде придумати. Взяти щось підходяще під тематику сайту та напрямку в сторону технологічності — з натхненням нового сучасного дизайну. Використати відповідні скіли для створення дизайну, добре підібрати елементи дизайну, використати різноманітні технологічні елементи, фічі, графічні моменти, графіки тощо. Можливо дати можливість переглянути ті чи інші елементи перед імплементацією. Авторизація має бути класична пароль+логін, а також Google автентифікація, з повідомленнями на пошту — все як має бути.»

**Рішення:**
- Створюємо бренд з нуля — palette/typography/logo
- **3 варіанти палітри** на вибір (див. розділ 5.1)
- Tech-forward напрям з інтерактивними фішками (розділ 5.5)
- Preview-механіка: складні елементи спершу як mockup на `/preview/[element]`
- Auth: Better Auth з email+password **+ Google OAuth** + email-verification + password-reset

---

## 10. Інструкція для старту в новому чаті

1. Скопіюй увесь цей документ
2. Відкрий новий чат з Claude Code
3. Встав як перший меседж + додай одну з команд нижче

### Повний старт — Phase 1

> «Створи проект EcohimIF у папці `C:\Claude\ecohimif\app` (або поряд з цим ТЗ). Розпочни з Phase 1 — Foundation: scaffold + конфіги + i18n (UK default + DE + EN) + theme + Header/Footer + Homepage + Pro-nas + Dostavka + Kontakty + legal stubs + cookie banner + robots noindex. Перед фінальним вибором палітри покажи 3 варіанти (A/B/C з розділу 5.1) як мокапи. Stop after Phase 1.»

### Розширений старт — Phase 1 + 2

> «Створи проект EcohimIF + Phase 1 + Phase 2 (database + catalog). Schema візьми з ТЗ. Seed-script на 30-50 fake-продуктів різноманітного асортименту (добрива, клеї, грунтовки, пивна сировина, побутова хімія).»

### Мінімальний короткий старт

> «Створи проект EcohimIF згідно ТЗ. Universal e-commerce, UA→EU. Stack: Next 16 + Tailwind v4 + Yarn 4 + Biome + PostgreSQL + Drizzle + Better Auth (Google OAuth) + next-intl (UK/DE/EN). Phase 1: foundation + noindex.»

---

## 11. Test-cases файл

Окремо в `C:\Claude\ecohimif\ecohimIF_TestCases.md` лежить структурований набір тест-сценаріїв:
- Catalog flow (browse, filter, search)
- Product detail (variants, datasheets, attributes)
- Cart anonymous + authenticated + merge на логіні
- Checkout (guest + registered)
- Order status flow (admin → email → customer)
- Auth (login/register/Google OAuth/password-reset)
- Admin panel (CRUD products, orders, messages)
- Messages system (anonymous, authenticated, reply flow)
- Edge cases (out-of-stock, NP-API timeout, спам-захист)
- Multilingual (UK/DE/EN перехід без втрати кошика)
- Theme (light/dark persist)
- Accessibility (keyboard nav, screen reader, focus management)
- Performance (Lighthouse 100/100/100/100 cíl)

Використовувати при e2e-тестуванні (Playwright) та як manual-QA checklist перед launch.

---

**Дата документа:** 2026-04-28
**Версія:** 2.0 (після Q&A Round 1 з клієнтом)
**Розташування:** `C:\Claude\ecohimif\ecohimIF_TZ.md`
