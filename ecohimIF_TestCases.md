# EcohimIF — Тест-кейси

> Документ містить структурований набір сценаріїв для тестування продукту:
> manual-QA checklist перед launch + основа для e2e-тестів (Playwright).
> Покриває всі флоу описані в `ecohimIF_TZ.md`.
>
> **Формат:** `TC-<область>-<номер>` → передумови → кроки → очікуваний результат → пріоритет.
>
> **Пріоритети:** `P0` критично (блокує запуск) · `P1` важливо · `P2` бажано · `P3` nice-to-have

---

## Зміст

1. [Каталог та фільтрація](#1-каталог-та-фільтрація-tc-cat)
2. [Картка товару](#2-картка-товару-tc-prod)
3. [Кошик: анонімний](#3-кошик-анонімний-tc-cart-anon)
4. [Кошик: авторизований + злиття](#4-кошик-авторизований--злиття-tc-cart-auth)
5. [Checkout / Оформлення замовлення](#5-checkout--оформлення-замовлення-tc-check)
6. [Авторизація](#6-авторизація-tc-auth)
7. [Особистий кабінет](#7-особистий-кабінет-tc-kab)
8. [Адмін-панель](#8-адмін-панель-tc-adm)
9. [Система звʼязку](#9-система-звязку-tc-msg)
10. [Замовлення: статуси і email](#10-замовлення-статуси-і-email-tc-ord)
11. [Інтеграція Нової Пошти](#11-інтеграція-нової-пошти-tc-np)
12. [Мови / Багатомовність](#12-мови--багатомовність-tc-i18n)
13. [Теми / Light + Dark](#13-теми--light--dark-tc-theme)
14. [Доступність (a11y)](#14-доступність-a11y-tc-a11y)
15. [Performance + SEO](#15-performance--seo-tc-perf)
16. [Безпека](#16-безпека-tc-sec)
17. [Legal + Cookie](#17-legal--cookie-tc-legal)
18. [Edge cases / Помилки](#18-edge-cases--помилки-tc-edge)

---

## 1. Каталог та фільтрація (TC-CAT)

### TC-CAT-001 · Перегляд каталогу — базовий **P0**
**Передумови:** у БД є мінімум 20 продуктів різних категорій
**Кроки:**
1. Зайти на `/uk/katalog`
**Очікувано:** показано сітку продуктів, breadcrumbs, фільтри в sidebar, сортування зверху, pagination якщо >24 продуктів.

### TC-CAT-002 · Фільтр по категорії **P0**
**Кроки:** обрати категорію "Клеї" у sidebar
**Очікувано:** показано лише продукти цієї категорії; URL містить `?category=klei`; інші фільтри лишаються активними.

### TC-CAT-003 · Фільтр по тегах (динамічних) **P1**
**Передумови:** у БД є продукти з тегами `органічне`, `для-злаків`
**Кроки:** обрати тег `органічне`
**Очікувано:** показано продукти що мають цей тег у `products.tags`.

### TC-CAT-004 · Фільтр по ціні (range) **P1**
**Кроки:** встановити slider 100-500 грн
**Очікувано:** показано продукти у цьому діапазоні (рахуючи `basePrice` або найменший variant.price).

### TC-CAT-005 · Фільтр "в наявності" **P1**
**Кроки:** toggle "Тільки в наявності"
**Очікувано:** приховано продукти з `inStock=false` + усіма варіантами з `stock=0`.

### TC-CAT-006 · Комбінація фільтрів **P0**
**Кроки:** обрати категорію + 2 теги + ціна-range
**Очікувано:** фільтри комбінуються по AND; URL deep-linkable; reload → стан зберігається.

### TC-CAT-007 · Сортування **P1**
**Кроки:** обрати по черзі: новіші, дешевші, дорожчі, A-Я
**Очікувано:** список перебудовується; URL оновлюється; сортування комбінується з фільтрами.

### TC-CAT-008 · Пошук з autocomplete **P0**
**Кроки:** ввести у search "клей"
**Очікувано:** автодоповнення показує 5+ варіантів; натискання → перехід до результатів пошуку; PG fulltext шукає у name, description, tags.

### TC-CAT-009 · Пустий результат **P1**
**Кроки:** ввести у пошук "xyzqwerty"
**Очікувано:** показана empty-state з текстом "Нічого не знайдено" + кнопка "Скинути фільтри" + 4-6 рекомендованих продуктів.

### TC-CAT-010 · Pagination **P1**
**Передумови:** >24 продуктів
**Кроки:** перехід на сторінку 2
**Очікувано:** URL `?page=2`; scroll-to-top автоматично; кнопки prev/next активні відповідно.

### TC-CAT-011 · Перемикання grid ↔ list view **P2**
**Кроки:** клік на toggle "list/grid"
**Очікувано:** перемикання макета; preference зберігається у localStorage.

### TC-CAT-012 · Каталог mobile **P0**
**Кроки:** виставити viewport 375px
**Очікувано:** фільтри ховаються у drawer (кнопка "Фільтри"); сітка 1-2 колонки; sticky filter-button.

---

## 2. Картка товару (TC-PROD)

### TC-PROD-001 · Відкриття картки **P0**
**Кроки:** клік на product card з каталогу
**Очікувано:** перехід на `/uk/katalog/[slug]`; усі дані рендеряться; LCP < 1.5s.

### TC-PROD-002 · Photo gallery **P0**
**Кроки:** клік на thumbnail
**Очікувано:** main image змінюється з smooth transition; стрілки prev/next працюють; ESC закриває fullscreen якщо відкритий.

### TC-PROD-003 · Photo zoom **P2**
**Кроки:** клік на main image → відкриття fullscreen lightbox
**Очікувано:** lightbox з можливістю scroll по фото; ESC закриває; aria-modal.

### TC-PROD-004 · Variant selector **P0**
**Передумови:** продукт має 3 варіанти (0.5L, 1L, 5L)
**Кроки:** клік на варіант 5L
**Очікувано:** показана нова ціна, sku, наявність; "Купити" / "Додати в кошик" referencує цей variant.

### TC-PROD-005 · Якщо немає варіантів — defaults **P0**
**Передумови:** продукт без variants (тільки basePrice)
**Очікувано:** показана `basePrice`, кнопка "Додати" додає `productId` без `variantId`.

### TC-PROD-006 · Quantity stepper **P1**
**Кроки:** збільшити quantity до 10
**Очікувано:** кнопка "+" + кнопка "−" + ручний input; min=1, max=stock; якщо введено більше за stock — clamp до stock + tooltip "Залишилось N".

### TC-PROD-007 · Додавання в кошик (анонім) **P0**
**Кроки:** клік "Додати в кошик"
**Очікувано:** localStorage оновлюється; cart-icon у header показує badge "1"; toast "Додано в кошик" з кнопкою "Відкрити кошик"; flying-anim element до іконки.

### TC-PROD-008 · "Купити в 1 клік" (швидке замовлення) **P0**
**Кроки:** клік "Купити в 1 клік"
**Очікувано:** відкривається mini-modal з полями name+phone+city+warehouse → submit → створюється order у БД, email оператору, success-toast з номером замовлення.

### TC-PROD-009 · Універсальні атрибути **P1**
**Передумови:** продукт має `attributes: { "Вʼязкість": "8000 cP", "Час сушки": "24 год" }`
**Очікувано:** show as ключ-значення таблиця в розділі "Характеристики".

### TC-PROD-010 · Datasheet — uploaded PDF **P1**
**Передумови:** продукт має datasheet з `source='uploaded'`
**Очікувано:** показано кнопка "Завантажити PDF"; клік → download стартує з правильним filename.

### TC-PROD-011 · Datasheet — form-generated **P1**
**Передумови:** datasheet з `source='form'` + заповнені формою секції
**Очікувано:** клік "Відкрити PDF" → /api/datasheet/[id]/pdf генерує PDF на льоту через @react-pdf/renderer; відкривається у браузері.

### TC-PROD-012 · Питання про продукт **P1**
**Кроки:** клік "Поставити запитання" → форма з name+email+message
**Очікувано:** submit → запис у `messages` з `productId`+`source='product_question'`; email оператору; toast "Дякуємо, відповімо за 1 робочий день".

### TC-PROD-013 · Related products **P2**
**Очікувано:** в bottom картки 4 product-cards тієї ж категорії або з перетином tags.

### TC-PROD-014 · Schema.org markup **P1**
**Кроки:** view source → json-ld
**Очікувано:** містить `Product` schema з name, image, offers.price, offers.availability.

### TC-PROD-015 · Out-of-stock UX **P0**
**Передумови:** усі variants мають stock=0 або `products.inStock=false`
**Очікувано:** кнопки "Купити"/"Додати" disabled; показана плашка "Немає в наявності"; кнопка "Повідомити коли зʼявиться" → email-subscription.

---

## 3. Кошик: анонімний (TC-CART-ANON)

### TC-CART-ANON-001 · Створення кошика **P0**
**Кроки:** додати 1 продукт з каталогу
**Очікувано:** localStorage містить `ecohimif-cart` = JSON-масив { productId, variantId, quantity, addedAt }; cart-icon badge "1".

### TC-CART-ANON-002 · Перегляд кошика **P0**
**Кроки:** клік на cart-icon → відкривається CartDrawer
**Очікувано:** показано список items з фото, назвою, варіантом, ціною, quantity, кнопками +/−, "Видалити"; підсумок; кнопка "До оформлення".

### TC-CART-ANON-003 · Зміна кількості **P0**
**Кроки:** клік "+" біля товару
**Очікувано:** quantity оновлюється, total перераховується, localStorage синхронізується.

### TC-CART-ANON-004 · Видалення товару **P0**
**Кроки:** клік на "корзина" іконку поряд з товаром
**Очікувано:** товар видаляється; якщо кошик порожній — показано empty-state з кнопкою "До каталогу".

### TC-CART-ANON-005 · Persist після page reload **P0**
**Кроки:** додати в кошик → F5
**Очікувано:** кошик зберігається (читається з localStorage при mount).

### TC-CART-ANON-006 · Persist після закриття браузера **P0**
**Кроки:** додати → закрити вкладку → відкрити заново
**Очікувано:** кошик зберігається (localStorage persists поки не вичистять).

### TC-CART-ANON-007 · TTL **P2**
**Передумови:** записати у localStorage cart з timestamp 31 день тому
**Кроки:** відкрити сайт
**Очікувано:** cart очищується через TTL=30 днів, показано toast "Кошик було очищено через довгий період неактивності".

### TC-CART-ANON-008 · Не валідні дані в localStorage **P1**
**Передумови:** хтось вручну поламав JSON у localStorage
**Очікувано:** Zod-schema валідація — кошик скидається без помилки в UI; в console — warning.

### TC-CART-ANON-009 · Додавання неіснуючого продукту **P1**
**Передумови:** localStorage містить productId=999 якого не існує в БД
**Очікувано:** при rendering CartDrawer — item тихо filter-out; toast "Деякі товари більше недоступні".

### TC-CART-ANON-010 · Додавання out-of-stock товару після додавання **P1**
**Передумови:** товар у кошику, у БД stock=0
**Очікувано:** в CartDrawer показано badge "немає в наявності"; кнопка "До оформлення" disabled поки не приберуть.

---

## 4. Кошик: авторизований + злиття (TC-CART-AUTH)

### TC-CART-AUTH-001 · Перенос анонімного → DB при логіні **P0**
**Передумови:** анонім має 3 товари в localStorage cart
**Кроки:** залогінитись (existing user)
**Очікувано:** після логіну — товари переходять у `cartItems` DB-таблицю; localStorage очищується; CartDrawer показує ті самі items але теперь з DB.

### TC-CART-AUTH-002 · Злиття з існуючим DB-кошиком **P0**
**Передумови:** auth-user мав 2 товари в DB cart; анонім додав ще 1 у localStorage
**Кроки:** залогінитись
**Очікувано:** результуючий кошик = sum обох; якщо однаковий productId+variantId — quantity додаються (cap by stock).

### TC-CART-AUTH-003 · Sync між пристроями **P1**
**Передумови:** залогінений user додав 2 товари в браузері A
**Кроки:** залогінитись в браузері B
**Очікувано:** кошир показує ті самі 2 товари (читання з БД при mount).

### TC-CART-AUTH-004 · Видалення в одному пристрої = sync в другому **P2**
**Очікувано:** при next page navigation / mount у другому пристрої — кошир перечитується. Real-time sync — out of scope для MVP.

### TC-CART-AUTH-005 · Logout не очищує DB-кошик **P1**
**Кроки:** залогінений з кошиком → logout
**Очікувано:** DB cartItems зберігаються; в браузері — кошик порожній (анонім); при наступному логіні — відновиться.

---

## 5. Checkout / Оформлення замовлення (TC-CHECK)

### TC-CHECK-001 · Перехід з кошика до checkout **P0**
**Кроки:** з CartDrawer клік "До оформлення"
**Очікувано:** перехід на `/uk/oformlennya`; показано Step 1: контакти.

### TC-CHECK-002 · Step 1: контакти guest **P0**
**Кроки:** заповнити name, email, phone (required), company+taxId (optional) → Next
**Очікувано:** валідація: name ≥2 chars, email format, phone format (UA + EU); Next активний.

### TC-CHECK-003 · Step 1: контакти auth-user **P1**
**Передумови:** залогінений user з profile
**Очікувано:** поля pre-filled з даних користувача; можна редагувати; checkbox "Зберегти в профіль".

### TC-CHECK-004 · Step 2: НП відділення **P0**
**Кроки:** обрати "Нова Пошта — відділення" → ввести місто (autocomplete з NP API) → ввести/обрати відділення
**Очікувано:** список міст знаходить за 2+ символами; список відділень оновлюється після вибору міста.

### TC-CHECK-005 · Step 2: НП кур'єр **P1**
**Кроки:** обрати "НП кур'єр" → ввести місто + вулицю + будинок + квартиру
**Очікувано:** поля обовʼязкові; phone-поле зберігається з Step 1.

### TC-CHECK-006 · Step 2: Самовивіз **P2**
**Кроки:** обрати "Самовивіз з Івано-Франківська"
**Очікувано:** показано адресу складу + графік роботи; "Дата самовивозу" date-picker.

### TC-CHECK-007 · Step 3: Оплата **P1**
**Кроки:** обрати спосіб оплати (рахунок / при отриманні)
**Очікувано:** обидва доступні; "Картою на сайті" disabled з тултіпом "Скоро буде доступно" (Phase 7).

### TC-CHECK-008 · Step 4: Підтвердження + Submit **P0**
**Кроки:** перегляд усього замовлення → клік "Підтвердити замовлення"
**Очікувано:**
- Server Action `createOrder` створює запис у `orders` + `orderItems`
- Якщо auth-user — переноситься з DB-cart, після cart очищується
- Якщо guest — localStorage cart очищується
- Email клієнту "Дякуємо за замовлення EH-2026-00042"
- Email оператору з details
- Redirect на `/uk/oformlennya/dyakuyemo?order=EH-2026-00042`

### TC-CHECK-009 · Validation errors UX **P1**
**Кроки:** залишити email пустим → клік Next
**Очікувано:** inline-помилка під полем; focus на полі; toast НЕ показується (не загрожуючий UX).

### TC-CHECK-010 · NP API timeout fallback **P2**
**Передумови:** NP API недоступний
**Очікувано:** показано fallback — звичайне `<input>` для міста і відділення з warning "Перевірка доступності НП тимчасово недоступна, можемо звʼязатись пізніше".

### TC-CHECK-011 · Анонімне замовлення — пропозиція реєстрації **P2**
**Кроки:** guest submit
**Очікувано:** на success-page — banner "Створити обліковий запис щоб слідкувати за замовленням?" → одним кліком auto-register з email.

### TC-CHECK-012 · Empty cart redirect **P1**
**Кроки:** очистити кошик → перейти на `/uk/oformlennya`
**Очікувано:** redirect на /uk/katalog з toast "Кошик порожній".

---

## 6. Авторизація (TC-AUTH)

### TC-AUTH-001 · Реєстрація email+password **P0**
**Кроки:** /uk/reestratsia → name, email, password (8+) → submit
**Очікувано:** запис у `users` + verification email; перенаправлення на /uk/reestratsia/perevirka-email; показано "Перевірте пошту".

### TC-AUTH-002 · Email verification **P0**
**Кроки:** клік на лінк у email
**Очікувано:** `emailVerified=true`; auto-login; redirect на /uk/kabinet.

### TC-AUTH-003 · Login email+password **P0**
**Кроки:** /uk/uvijty → email + password
**Очікувано:** успіх → redirect на попередню сторінку або /uk/kabinet; помилка → "Невірний email або пароль".

### TC-AUTH-004 · Google OAuth **P0**
**Кроки:** клік "Увійти з Google" → consent → return
**Очікувано:** Better Auth створює user якщо новий; auto-link якщо email вже є; redirect на /uk/kabinet.

### TC-AUTH-005 · Google OAuth — link existing **P1**
**Передумови:** user зареєструвався email/password з тим самим email що Google
**Кроки:** клік Google OAuth з тим email
**Очікувано:** Better Auth links accounts; той самий user; обидва способи входу працюють.

### TC-AUTH-006 · Password reset request **P0**
**Кроки:** /uk/uvijty → "Забули пароль?" → ввести email
**Очікувано:** email з reset-link (1 година TTL); toast "Перевірте пошту".

### TC-AUTH-007 · Password reset complete **P0**
**Кроки:** клік на reset-link → ввести новий пароль → submit
**Очікувано:** password оновлено; auto-login; redirect на /uk/kabinet.

### TC-AUTH-008 · Logout **P0**
**Кроки:** клік "Вийти" в header dropdown
**Очікувано:** session знищується; redirect на homepage; cookie cleared.

### TC-AUTH-009 · Protected route без auth **P0**
**Кроки:** як guest перейти на /uk/kabinet/bestellungen
**Очікувано:** redirect на /uk/uvijty?returnUrl=/uk/kabinet/bestellungen; після login — назад.

### TC-AUTH-010 · Admin route без admin role **P0**
**Кроки:** як customer перейти на /uk/admin
**Очікувано:** 403 forbidden або redirect на homepage; не leak admin-UI у бандлі.

### TC-AUTH-011 · Rate-limit на login **P1**
**Кроки:** 10 невірних спроб за хвилину
**Очікувано:** rate-limit response 429; cooldown 5 хв; toast "Забагато спроб, спробуйте через 5 хв".

### TC-AUTH-012 · Session expiry **P2**
**Передумови:** session TTL=30 днів
**Очікувано:** після закінчення → next protected action → auto-logout → login form.

---

## 7. Особистий кабінет (TC-KAB)

### TC-KAB-001 · Перегляд профілю **P0**
**Кроки:** /uk/kabinet/profil
**Очікувано:** показано current name/email/phone/companyName/taxId; кнопка "Редагувати".

### TC-KAB-002 · Редагування профілю **P0**
**Кроки:** змінити name → Save
**Очікувано:** запис у DB; toast "Профіль оновлено"; UI оновлюється.

### TC-KAB-003 · Список адрес **P1**
**Кроки:** /uk/kabinet/adresy
**Очікувано:** показано збережені default + додаткові адреси; CRUD з ними.

### TC-KAB-004 · Історія замовлень **P0**
**Кроки:** /uk/kabinet/bestellungen
**Очікувано:** показано таблиця/cards замовлень з: order#, дата, total, status badge, кнопка "Деталі".

### TC-KAB-005 · Фільтр замовлень по статусу **P1**
**Кроки:** обрати "В обробці"
**Очікувано:** показано лише `status IN (new, confirmed, processing)`.

### TC-KAB-006 · Деталі замовлення **P0**
**Кроки:** клік "Деталі" біля замовлення
**Очікувано:** /uk/kabinet/bestellungen/[id] показує: items, доставку, оплату, статус-timeline, ТТН-кнопку якщо є.

### TC-KAB-007 · Кнопка "Відстежити ТТН" **P1**
**Передумови:** order має trackingNumber
**Очікувано:** клік → open new tab на `https://novaposhta.ua/tracking/?cargo_number=...`.

### TC-KAB-008 · Список повідомлень **P1**
**Кроки:** /uk/kabinet/povidomlennia
**Очікувано:** показано всі `messages` де `userId=current` + всі `messageReplies`; unread badge; mark-as-read при відкритті.

### TC-KAB-009 · Доступ до чужого замовлення **P0**
**Кроки:** залогінений user A намагається відкрити /uk/kabinet/bestellungen/[orderId-of-userB]
**Очікувано:** 404 або redirect на свій список з toast "Замовлення не знайдено".

---

## 8. Адмін-панель (TC-ADM)

### TC-ADM-001 · Доступ для role=admin **P0**
**Кроки:** залогінитись як admin → /uk/admin
**Очікувано:** показано dashboard з KPI: нові замовлення (count), очікують підтвердження, сума замовлень за тиждень.

### TC-ADM-002 · Доступ для role=operator **P0**
**Очікувано:** admin layout без секції "Users management"; інше доступне.

### TC-ADM-003 · Список продуктів **P0**
**Кроки:** /uk/admin/produkty
**Очікувано:** таблиця з: image-thumb, name, sku, категорія, кількість варіантів, базова ціна, stock-total, дії (edit, delete).

### TC-ADM-004 · Створення продукту **P0**
**Кроки:** клік "Новий продукт" → форма
**Очікувано:** поля: slug (auto-gen з name), sku, category-select, 3-мовний name/description, base price, currency, images (multiple drag-drop), attributes (dynamic key-value add/remove), tags (input з autocomplete існуючих).

### TC-ADM-005 · Динамічні атрибути в формі продукту **P0**
**Кроки:** "+ Додати характеристику" → вписати "Вʼязкість" / "8000 cP" → додати ще одну
**Очікувано:** список зростає, drag-reorder, видалення; зберігається у `attributes` jsonb.

### TC-ADM-006 · Варіанти продукту в формі **P0**
**Кроки:** додати variant: amount=5, unit='kg', packaging='мішок', price=350, stock=20
**Очікувано:** окрема таблиця variants всередині форми продукту; CRUD inline; isDefault toggle.

### TC-ADM-007 · Upload зображення **P0**
**Кроки:** drag 3 фото на upload zone
**Очікувано:** превʼю; reorder через drag; primary-toggle; зберігання у Vercel Blob; thumbnails в каталозі генеруються Next/Image.

### TC-ADM-008 · Datasheet — uploaded **P1**
**Кроки:** "Додати datasheet" → "Завантажити PDF" → drag PDF
**Очікувано:** збереження у Blob; запис у `datasheets` з `source='uploaded'`; preview-кнопка.

### TC-ADM-009 · Datasheet — form-конструктор **P1**
**Кроки:** "Додати datasheet" → "Заповнити форму" → заповнити секції (composition, applications, storage, safety)
**Очікувано:** збереження у `formContent` jsonb; кнопка "Превʼю PDF" → /api/datasheet/[id]/pdf генерує превʼю.

### TC-ADM-010 · Видалення продукту **P1**
**Кроки:** клік "Видалити" → confirm
**Очікувано:** soft-delete (або hard-delete але каскадно). Якщо продукт у активних orderItems — block з повідомленням "Цей продукт у активних замовленнях".

### TC-ADM-011 · Список замовлень **P0**
**Кроки:** /uk/admin/zamovlennya
**Очікувано:** таблиця з: order#, дата, customer, total, статус, метод доставки, оплата, дії; фільтри сверху.

### TC-ADM-012 · Зміна статусу замовлення **P0**
**Кроки:** dropdown біля order → змінити на "Відправлено"
**Очікувано:** UI оновлюється; email клієнту "Ваше замовлення відправлене"; запис в audit-log (опційно).

### TC-ADM-013 · Внесення ТТН **P0**
**Кроки:** в деталях замовлення — поле "ТТН Нової Пошти" → вписати 20-значний номер → save
**Очікувано:** збереження; email клієнту з ТТН-номером + лінком; статус автоматично змінюється на 'shipped'.

### TC-ADM-014 · Внутрішні нотатки **P1**
**Кроки:** в деталях замовлення додати note "Дзвонили о 12:00, обіцяли оплатити завтра"
**Очікувано:** збереження в `operatorNotes`; видно іншим operators; НЕ видно клієнту.

### TC-ADM-015 · Експорт замовлення у PDF (інвойс) **P2**
**Кроки:** клік "Сформувати рахунок"
**Очікувано:** генерується PDF з items, total, реквізитами клієнта і компанії; download.

### TC-ADM-016 · Inbox повідомлень **P0**
**Кроки:** /uk/admin/povidomlennya
**Очікувано:** список усіх `messages`; unread bold; preview body; klick → детальний view з кнопкою "Відповісти".

### TC-ADM-017 · Відповідь на повідомлення **P0**
**Кроки:** клік "Відповісти" → textarea → submit
**Очікувано:** збереження у `messageReplies`; email клієнту з відповіддю + reply-to header; mark `messages.isRead=true`.

### TC-ADM-018 · Users management **P1**
**Передумови:** role=admin
**Кроки:** /uk/admin/users
**Очікувано:** таблиця users; зміна ролі через dropdown; deactivate; reset password (відправити email).

### TC-ADM-019 · Settings **P2**
**Очікувано:** редагування контактної інформації для footer; email-шаблонів; SEO-defaults.

---

## 9. Система звʼязку (TC-MSG)

### TC-MSG-001 · Анонімне повідомлення через Kontakty **P0**
**Кроки:** /uk/kontakty → name, email, phone, subject, body → submit
**Очікувано:** запис у `messages` з `source='contact_form'`; email оператору; success-toast.

### TC-MSG-002 · Питання про продукт **P0**
**Кроки:** на картці продукту → "Поставити запитання" → submit
**Очікувано:** запис у `messages` з `productId` + `source='product_question'`; email оператору з лінком на продукт.

### TC-MSG-003 · Авторизоване повідомлення **P1**
**Передумови:** залогінений
**Кроки:** /uk/kontakty → форма pre-filled
**Очікувано:** запис у `messages` з `userId=current`; повідомлення видно у /uk/kabinet/povidomlennia.

### TC-MSG-004 · Повідомлення про конкретне замовлення **P1**
**Кроки:** /uk/kabinet/bestellungen/[id] → "Звʼязатись з оператором" → форма
**Очікувано:** запис у `messages` з `orderId` + `userId`; оператор бачить замовлення-контекст.

### TC-MSG-005 · Email-відповідь з оператора **P0**
**Кроки:** оператор відповідає в admin-панелі
**Очікувано:** клієнт отримує email; reply-to налаштовано так що відповідь клієнта приходить на email оператора; thread зберігається в `messageReplies`.

### TC-MSG-006 · Honeypot захист від спаму **P1**
**Кроки:** бот заповнює прихований honeypot-field "website"
**Очікувано:** запит mute-приймається (success-response) але НЕ зберігається; email не відправляється.

### TC-MSG-007 · Rate-limit на форму **P1**
**Кроки:** один IP надсилає 10 повідомлень за хвилину
**Очікувано:** rate-limit 429; cooldown 10 хв.

---

## 10. Замовлення: статуси і email (TC-ORD)

### TC-ORD-001 · Створення → email customer **P0**
**Кроки:** customer submit checkout
**Очікувано:** email з шаблону "Дякуємо за замовлення", містить order#, items, доставку, контакт оператора.

### TC-ORD-002 · Створення → email operator **P0**
**Очікувано:** оператор отримує email "Нове замовлення EH-2026-00042" з усіма details + лінком на admin-панель.

### TC-ORD-003 · Status flow: new → confirmed **P0**
**Кроки:** оператор підтверджує
**Очікувано:** email customer "Ваше замовлення підтверджено".

### TC-ORD-004 · Status flow: confirmed → processing → shipped + ТТН **P0**
**Очікувано:** email customer на кожному переході; ТТН-номер в email.

### TC-ORD-005 · Status flow: shipped → delivered **P0**
**Очікувано:** email customer "Замовлення доставлено. Дякуємо!"; можливо: запрошення залишити відгук (Phase 7).

### TC-ORD-006 · Status: cancelled — клієнтом **P1**
**Передумови:** order у статусі 'new' або 'confirmed'
**Кроки:** клієнт у kabinet → "Скасувати замовлення" → confirm
**Очікувано:** статус = cancelled; email оператору; email клієнту з підтвердженням.

### TC-ORD-007 · Status: cancelled — оператором **P1**
**Очікувано:** оператор може скасувати з будь-якого статусу до 'shipped'; запис причини в operatorNotes; email клієнту.

### TC-ORD-008 · Order# формат **P1**
**Очікувано:** unique sequential з префіксом року: `EH-2026-00001`, `EH-2026-00002`...

### TC-ORD-009 · Email-templates локалізація **P1**
**Передумови:** клієнт обрав DE мову при checkout
**Очікувано:** усі email клієнту приходять німецькою; оператор отримує внутрішньою мовою (UK).

### TC-ORD-010 · Order timeline у kabinet **P1**
**Очікувано:** у details показано стрічка статусів з timestamps; checked-кружальця для пройдених стейтів.

---

## 11. Інтеграція Нової Пошти (TC-NP)

### TC-NP-001 · Autocomplete міст **P0**
**Кроки:** у checkout step 2 ввести "ки"
**Очікувано:** після 2-х символів — debounced запит до NP API; показано 5-10 варіантів; ARROW-keys навігація + Enter вибір.

### TC-NP-002 · Список відділень для міста **P0**
**Передумови:** обрано місто
**Очікувано:** запит до NP API повертає відділення; список з #, адресою, графіком; пошук фільтрує.

### TC-NP-003 · ТТН status check **P2**
**Передумови:** order має trackingNumber
**Очікувано:** background-job (cron, опційно) перевіряє статус через NP API кожні 4 години; якщо changed → update orders.status; email customer.

### TC-NP-004 · NP API key environment **P0**
**Очікувано:** API key зберігається в env var `NOVAPOSHTA_API_KEY`; ніколи не у клієнтському bundle.

### TC-NP-005 · Cache cities **P1**
**Очікувано:** список міст cached в memory 24 год; не запит до NP при кожному checkout.

### TC-NP-006 · Cache warehouses per city **P1**
**Очікувано:** warehouses cached 1 година per city.

### TC-NP-007 · API timeout/error handling **P1**
**Очікувано:** якщо NP API не відповідає 5s — fallback на manual input + log error.

---

## 12. Мови / Багатомовність (TC-I18N)

### TC-I18N-001 · Default locale = UK **P0**
**Кроки:** перший візит на `/`
**Очікувано:** redirect на `/uk/...`.

### TC-I18N-002 · Перемикач мов **P0**
**Кроки:** клік на DE у LocaleSwitcher
**Очікувано:** URL змінюється на `/de/...`; контент перекладається; cookie `NEXT_LOCALE=de` зберігається.

### TC-I18N-003 · Контент продукту по мовах **P0**
**Передумови:** продукт має name_uk + name_de
**Очікувано:** при locale=de показано name_de; якщо name_de пустий → fallback на name_uk з language-marker badge.

### TC-I18N-004 · Кошик зберігається при зміні мови **P0**
**Кроки:** додати в кошик → переключити мову
**Очікувано:** items залишаються; ціни форматуються в локалізованому форматі (€/₴/$).

### TC-I18N-005 · URL-структура локалізована **P1**
**Очікувано:** UK: `/uk/katalog`, DE: `/de/katalog`, EN: `/en/catalog`. (Або всюди `/katalog` — як вирішимо в Phase 1.)

### TC-I18N-006 · Hreflang теги **P1**
**Очікувано:** на кожній сторінці `<link rel="alternate" hreflang="uk" href="..."/>` для всіх 3 мов + `x-default`.

### TC-I18N-007 · Email-templates по локалях **P1**
**Очікувано:** customer-email шаблон обирається з локалі замовлення (`orders.locale`).

### TC-I18N-008 · Date format per locale **P1**
**Очікувано:** UK: "28 квітня 2026"; DE: "28. April 2026"; EN: "April 28, 2026".

### TC-I18N-009 · Currency per locale **P1**
**Очікувано:** UK → ₴/UAH (default); DE/EN → €/EUR з конвертацією (заплановано Phase 7).

### TC-I18N-010 · Browser-detect first visit **P2**
**Кроки:** новий відвідувач з `Accept-Language: de-DE`
**Очікувано:** redirect на /de/ замість /uk/ (через next-intl middleware).

---

## 13. Теми / Light + Dark (TC-THEME)

### TC-THEME-001 · Default theme **P0**
**Очікувано:** перший візит → system-prefers-color-scheme; якщо немає preference → light.

### TC-THEME-002 · Toggle persist **P0**
**Кроки:** клікнути ThemeToggle → dark → reload
**Очікувано:** dark зберігається через next-themes (cookie/localStorage).

### TC-THEME-003 · No-flash на старті **P0**
**Очікувано:** при reload — НЕ видно flash white→black; правильна тема одразу з server-render.

### TC-THEME-004 · View Transition при перемиканні **P2**
**Очікувано:** smooth color sweep при toggle (через View Transitions API де підтримується).

### TC-THEME-005 · Контраст dark mode **P0**
**Очікувано:** WCAG AA для body text; AAA для headings.

---

## 14. Доступність (a11y) (TC-A11Y)

### TC-A11Y-001 · Skip-to-content link **P0**
**Кроки:** Tab з адресного рядка
**Очікувано:** перший фокус — на skip-link "Перейти до контенту"; ENTER → focus на main.

### TC-A11Y-002 · Keyboard navigation cart **P0**
**Очікувано:** усі кнопки cart досяжні Tab; Enter відкриває drawer; ESC закриває.

### TC-A11Y-003 · Focus trap у modal **P0**
**Кроки:** відкрити CartDrawer
**Очікувано:** Tab/Shift+Tab cycles тільки в drawer; ESC закриває; focus повертається на trigger.

### TC-A11Y-004 · Mobile menu inert когда закритий **P0**
**Очікувано:** елементи мобільного menu НЕ tabbable коли menu закритий (`inert` attribute).

### TC-A11Y-005 · ARIA labels на іконках-кнопках **P0**
**Очікувано:** усі icon-only buttons мають `aria-label`; cart-icon → "Кошик, 3 товари".

### TC-A11Y-006 · Heading hierarchy **P0**
**Очікувано:** 1 H1 на сторінці; H2 для розділів; немає stripped levels.

### TC-A11Y-007 · Form errors мають aria-describedby **P1**
**Очікувано:** input з помилкою має `aria-invalid=true` + `aria-describedby` що вказує на елемент з помилкою.

### TC-A11Y-008 · Color is not the only way **P1**
**Очікувано:** статуси замовлень мають іконку + текст, не тільки колір.

### TC-A11Y-009 · Screen reader announces add-to-cart **P1**
**Кроки:** з NVDA/VoiceOver додати в кошик
**Очікувано:** announce "Товар додано в кошик, всього 3 товари" через aria-live region.

### TC-A11Y-010 · Visible focus rings **P0**
**Очікувано:** всі interactive elements мають видимий focus-ring (НЕ outline:none без alternative).

---

## 15. Performance + SEO (TC-PERF)

### TC-PERF-001 · Lighthouse mobile **P0**
**Очікувано:** Performance ≥ 95, Accessibility = 100, Best Practices = 100, SEO = 100.

### TC-PERF-002 · Lighthouse desktop **P0**
**Очікувано:** усі 100/100/100/100.

### TC-PERF-003 · LCP < 2.5s mobile, < 1.5s desktop **P0**
**Очікувано:** на 3G slow — LCP < 2.5; на cable — < 1.5.

### TC-PERF-004 · TBT (Total Blocking Time) **P1**
**Очікувано:** < 200ms mobile.

### TC-PERF-005 · CLS (Cumulative Layout Shift) **P0**
**Очікувано:** < 0.1; усі images мають explicit width/height.

### TC-PERF-006 · Bundle size **P1**
**Очікувано:** initial JS < 200kb gzipped для головної; lazy-load для admin.

### TC-PERF-007 · Image format AVIF з fallback **P1**
**Очікувано:** next/image видає AVIF для підтримуваних браузерів, WebP fallback, JPEG legacy.

### TC-PERF-008 · ISR для product pages **P1**
**Очікувано:** product pages revalidate щогодини; admin-зміни видно після revalidate.

### TC-PERF-009 · Sitemap.xml **P0**
**Очікувано:** `/sitemap.xml` містить усі публічні URLs з 3 мов; lastmod коректні.

### TC-PERF-010 · Schema.org **P1**
**Очікувано:** Product, Organization, WebSite, BreadcrumbList JSON-LD; перевірка google-rich-results-test.

---

## 16. Безпека (TC-SEC)

### TC-SEC-001 · CSRF protection **P0**
**Очікувано:** Server Actions мають вбудовану CSRF protection Next.js.

### TC-SEC-002 · SQL Injection **P0**
**Очікувано:** усі queries через Drizzle (prepared statements); raw SQL не використовується для user input.

### TC-SEC-003 · XSS in product names/descriptions **P0**
**Кроки:** admin вводить `<script>alert(1)</script>` в описі
**Очікувано:** на frontend — escaped як текст; не виконується.

### TC-SEC-004 · Password hashing **P0**
**Очікувано:** Better Auth використовує bcrypt/argon2; ніколи не зберігає plaintext.

### TC-SEC-005 · Sessions httpOnly + Secure **P0**
**Очікувано:** session-cookies мають `HttpOnly`, `Secure` (в production), `SameSite=Lax`.

### TC-SEC-006 · Rate-limiting **P1**
**Очікувано:** на /api/auth, /api/contact, /api/order — rate-limit (Upstash Redis або in-memory для MVP).

### TC-SEC-007 · CSP headers **P1**
**Очікувано:** Content-Security-Policy налаштований у `next.config.ts` headers().

### TC-SEC-008 · File upload validation **P0**
**Очікувано:** перевірка MIME type + size limit; PDF maximum 10MB; images max 5MB; magic-bytes check на server.

### TC-SEC-009 · Admin actions auth-checked **P0**
**Очікувано:** усі admin Server Actions перевіряють role на server; не довіряти client-state.

### TC-SEC-010 · Secrets in env **P0**
**Очікувано:** жодних секретів у git; `.env.example` без реальних значень; `.env.local` у gitignore.

---

## 17. Legal + Cookie (TC-LEGAL)

### TC-LEGAL-001 · Cookie banner показується **P0**
**Кроки:** перший візит
**Очікувано:** банер у нижній частині з "Прийняти всі / Налаштувати / Відхилити".

### TC-LEGAL-002 · Cookie preferences зберігаються **P0**
**Очікувано:** вибір зберігається у `aps-cookie-consent` localStorage; банер не показується знову.

### TC-LEGAL-003 · Essential cookies без opt-in **P0**
**Очікувано:** theme, language, session — set без consent (за TTDSG це OK як technically required).

### TC-LEGAL-004 · Analytics cookies — opt-in only **P1**
**Очікувано:** Plausible завантажується лише після consent (якщо взагалі використовуємо).

### TC-LEGAL-005 · Footer legal links **P0**
**Очікувано:** Imprint/Impressum, Privacy/Datenschutz, Terms/AGB лінки в footer на всіх сторінках.

### TC-LEGAL-006 · Placeholder warnings на legal pages **P0**
**Очікувано:** до launch — банер на legal-сторінках "Ця сторінка містить placeholder-дані. Реальні дані додаються перед публічним launch."

### TC-LEGAL-007 · noindex до flip env-flag **P0**
**Очікувано:** `<meta name="robots" content="noindex,nofollow">` поки `NEXT_PUBLIC_SITE_INDEXABLE != 'true'`.

### TC-LEGAL-008 · robots.txt **P0**
**Очікувано:** /robots.txt → `Disallow: /` поки env-flag=false; `Allow: /` коли true.

---

## 18. Edge cases / Помилки (TC-EDGE)

### TC-EDGE-001 · Out-of-stock під час checkout **P0**
**Передумови:** товар у кошику; інший купив останній stock
**Кроки:** клікнути submit
**Очікувано:** check stock на server перед створенням order; помилка "Товар X закінчився, оновіть кошик"; повернути на /uk/koshyk.

### TC-EDGE-002 · Концурентне додавання в кошик **P1**
**Очікувано:** оптимістичний UI з server-reconciliation; конфлікти resolved last-wins.

### TC-EDGE-003 · Дуже довгий запит до БД (>5s) **P1**
**Очікувано:** loading-skeleton; через 10s — error UI "Завантаження триває довше ніж зазвичай" + retry.

### TC-EDGE-004 · 500 server error **P0**
**Очікувано:** custom error.tsx з повідомленням; кнопка "На головну"; sentry-log.

### TC-EDGE-005 · 404 неіснуючий продукт **P0**
**Очікувано:** custom not-found.tsx з повідомленням і пошуком/каталогом.

### TC-EDGE-006 · Email не доходить **P1**
**Очікувано:** Resend webhook оновлює `messages.emailSent`; якщо bounce — alert admin у dashboard.

### TC-EDGE-007 · JS вимкнено **P2**
**Очікувано:** базова навігація працює (Next.js SSR); кошик + checkout вимагають JS — показано noscript-message.

### TC-EDGE-008 · IE11 / старий browser **P3**
**Очікувано:** мінімальний fallback з повідомленням "Будь ласка, оновіть браузер" (Next.js не підтримує IE11 з 14).

### TC-EDGE-009 · Mobile orientation rotation **P1**
**Очікувано:** layout reflow без помилок; modal/drawer закриваються або re-position.

### TC-EDGE-010 · Slow 3G **P1**
**Очікувано:** prefetch керується розумно; image-lazy; LCP < 4s на 3G.

### TC-EDGE-011 · Adblock-розширення **P1**
**Очікувано:** сайт працює (Plausible може блокуватись — не критично).

### TC-EDGE-012 · Дублікат order при подвійному кліку submit **P0**
**Очікувано:** після першого кліку — кнопка disabled; idempotency-key на server щоб не створити 2 orders.

---

## Чек-лист перед публічним launch

- [ ] Усі P0 tests passed
- [ ] Lighthouse 100/100/100/100 на mobile + desktop
- [ ] Real-data у Impressum, Datenschutz, AGB
- [ ] Юридичний review AGB
- [ ] EU-Vertreter додано в Datenschutz (якщо UA-operator)
- [ ] Sentry або alternative error-tracking налаштовано
- [ ] Backup-стратегія БД (Neon має автобекапи; для self-hosted — daily snapshot)
- [ ] Email-deliverability перевірено (SPF + DKIM + DMARC)
- [ ] `NEXT_PUBLIC_SITE_INDEXABLE=true` flipped в production env
- [ ] Cookie banner з real categories
- [ ] Тест замовлення end-to-end з реальною оплатою (тестовий продукт за 1₴)
- [ ] Admin акаунти створені з secure passwords
- [ ] Sales-team протестувала admin-панель
- [ ] Sitemap.xml в Google Search Console
- [ ] GA4/Plausible трекінг активний
- [ ] DNS налаштовано (A/AAAA records, CAA)
- [ ] SSL certificate (Vercel/Hetzner authoauto)
- [ ] CDN перед сайтом (Cloudflare якщо self-hosted)

---

**Дата:** 2026-04-28 · оновлено 2026-05-19
**Версія:** 1.1
**Файл-партнер:** `ecohimIF_TZ.md` у тій самій папці

---

## Додаток A — Стан реалізації (станом на 2026-05-19)

Цей розділ синхронізує тест-кейси з реальним станом коду після Phase 2-4 + полірування.

### ✅ Повністю реалізовано і протестовано

| Область | TC ID | Статус |
|---|---|---|
| Каталог: список, фільтри, теги, сортування, пагінація, пошук, empty-state | TC-CAT-001 ... 010 | ✅ Live з 31 продуктом |
| Сторінка товару: галерея, variants, attributes, datasheets, related, JSON-LD, форма питання | TC-PROD-001 ... 014 | ✅ Live |
| Анонімний кошик (localStorage + Zod, TTL 30 днів) | TC-CART-ANON-001 ... 008 | ✅ Live |
| Авторизований кошик + merge при логіні | TC-CART-AUTH-001, 002 | ✅ Live (merge wired у LoginForm) |
| Checkout 4 кроки з server action `createOrder` | TC-CHECK-001 ... 009, 012 | ✅ Live |
| Auth: email+password, Google OAuth (account linking), password reset, email verification | TC-AUTH-001 ... 010 | ✅ Live |
| Kabinet: profile, orders list+detail+ТТН, повідомлення | TC-KAB-001 ... 009 | ✅ Live |
| Admin: dashboard, products CRUD, categories CRUD, orders + ТТН + auto-email, messages inbox + reply, users role mgmt | TC-ADM-001 ... 018 | ✅ Live |
| Система повідомлень (contact, product question, оператор reply) | TC-MSG-001 ... 007 | ✅ Live з honeypot |
| Order statuses + email при зміні | TC-ORD-001 ... 010 | ✅ Live (UK/DE/EN templates) |
| Nova Poshta API (autocomplete міст + відділень з кешем) | TC-NP-001 ... 007 | ✅ Live |
| i18n UK/DE/EN, hreflang | TC-I18N-001 ... 010 | ✅ Live, dropdown LocaleSwitcher |
| Theme light/dark з View Transitions API | TC-THEME-001 ... 005 | ✅ Live |
| Legal: Privacy/Terms/Impressum (draft контент, потрібен юрист) | TC-LEGAL-001 ... 008 | ✅ Draft live |
| Security headers + CSRF (Better Auth SameSite cookies) | TC-SEC-001 ... 010 | ✅ Live |

### 🔧 Часткова реалізація / TODO для production

| TC | Поточний стан | TODO для production |
|---|---|---|
| TC-PROD-002 photo gallery | Один primaryImage + 5 thumbnails (без zoom-lightbox) | Додати fullscreen lightbox для повних зображень |
| TC-PROD-003 photo zoom | Не реалізовано | Phase 5 — embla-carousel + zoom |
| TC-PROD-011 form-generated PDF datasheet | ✅ /api/datasheet/[id]/pdf через @react-pdf/renderer | — |
| TC-CHECK-010 NP API timeout fallback | ✅ 5-сек timeout + console error + plain inputs | — |
| TC-CHECK-011 guest → register pitch на success-page | Не реалізовано | Додати banner з 1-click register |
| TC-AUTH-011 rate-limit на login | Не реалізовано | Додати Upstash Redis або in-memory limiter |
| TC-ADM-007 image upload | ✅ `/api/upload` (5MB max, MIME validation, role check) — зберігає у `/public/uploads/`. На production swap на Vercel Blob | Підключити Vercel Blob для production |
| TC-ADM-008 datasheet upload | Можна задати `fileUrl` ручну, UI upload — TODO | Reuse `ImageUploader` для PDF |
| TC-ADM-015 PDF invoice | Не реалізовано | Реюз `renderDatasheetPdf` патерн |
| TC-NP-003 ТТН status cron | Не реалізовано | Phase 7 — background worker |
| TC-PERF-001 Lighthouse 100/100/100/100 | Не повністю optimized (LCP покладається на seed data URIs) | Замінити SVG data-URI на реальні JPG/AVIF + sharp оптимізація |
| TC-A11Y-001 skip-to-content link | ✅ Live | — |
| TC-A11Y-003 cart drawer focus trap | ✅ ESC close + onclick-overlay close. Tab cycle — частково (без focus-trap libraries) | Додати focus-trap для повної відповідності WAI-ARIA |
| TC-A11Y-007 aria-describedby | ✅ Через primitives Input/Select/Textarea (auto useId) | — |
| TC-SEC-006 rate-limit на /api/auth | Better Auth дефолти (anti-brute по email) | Додати IP-rate-limit на reverse proxy |
| TC-EDGE-006 Resend bounce webhook | Не імплементовано | Підключити Resend webhook → mark message.emailSent |

### 🆕 Реалізовано понад ТЗ

| Feature | Файл |
|---|---|
| LocaleSwitcher як dropdown з прапорами і native names | `src/components/controls/LocaleSwitcher.tsx` |
| LayoutChrome — мінімальний chrome на auth-сторінках без header/footer | `src/components/layout/LayoutChrome.tsx` |
| View Transitions API при перемиканні теми (radial reveal) | `src/components/controls/ThemeToggle.tsx` |
| Magnetic hover на hero CTAs | `src/components/fx/Magnetic.tsx` |
| Dev-fallback для emails: якщо `RESEND_API_KEY` пустий — лог у консоль | `src/lib/email/send.ts` |
| Drizzle Studio як `yarn db:studio` | `package.json` |
| Error boundaries для /admin, /kabinet — graceful fallback при недоступній БД | `src/app/[locale]/admin/error.tsx`, `kabinet/error.tsx` |
| Account linking (Google + email → один акаунт) | `src/lib/auth.ts` |

### Облікові дані для тестування (dev)

| Роль | Email | Пароль |
|---|---|---|
| Admin | `admingalerovich@gmail.com` | `AdminGalerovich03322475` |
| Customer | `customergalerovich@gmail.com` | `CustomerGalerovich03322475` |

### Як запустити повну прогонку

```powershell
cd C:\Claude\ecohimif\app
yarn db:push          # створити таблиці
yarn db:seed          # 5 категорій + 31 продукт + promote admin
yarn dev              # на :3000 (Google OAuth redirect_uri в Google Console)
```

Після того — `/uk/admin` як admin → перевірити dashboard з KPI; `/uk` як customer → каталог → cart → checkout (NP autocomplete) → confirm → email у /uk/kabinet/zamovlennya.

---
