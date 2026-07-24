# Dokumentasi Software Specification — Jati Prime Furniture App

**Versi v3 (Full SRS)** — dari UX/UI spec menjadi Software Requirement Specification lengkap: arsitektur sistem, entity-relationship & data contract, component library, design tokens, serta per-halaman: spesifikasi ukuran, API mapping, animasi, acceptance criteria, edge case, developer notes, priority & requirement ID, dependency, dan component relationship.

---

## Daftar Isi

**Fondasi Arsitektur & Data**
- [00. Project Architecture](#00-project-architecture) — tujuan sistem, modul, tech stack, struktur folder, version history, NFR global, system flow diagram
- [00. Entity Relationship & Data Dictionary](#00-entity-relationship--data-dictionary) — ERD, relasi, data dictionary, data contract JSON
- [00. Component Library](#00-component-library-reusable-ui-components) — Button, Card, Input, Modal, Drawer, Bottom Sheet, Toast, Snackbar, Accordion, Tabs, Stepper, Gallery, Badge, Avatar

**Fondasi Desain**
- [00. Design System & Design Tokens](#00-design-system--design-tokens)
- [00. API Mapping & Database Schema](#00-api-mapping--database-schema)

**A. Halaman Customer (sesuai mockup)** 🟢
1. Home (Beranda) — Critical
2. Kategori — High
3. Produk (Listing/PLP) — Critical
4. Detail Produk (PDP) — Critical
5. Keranjang (Cart) — Critical
6. Checkout — Critical
7. Tentang Kami — Low
8. Galeri — Medium
9. Testimoni — Medium
10. FAQ — Low
11. Kontak Kami — Medium
12. Akun Saya — High

**B. Halaman Customer Tambahan** 🟡
13. Wishlist (Favorit) — Medium
14. Login — Critical
15. Register — Critical
16. Pesanan Saya — High
17. Tracking Pesanan — High

**C. Admin Panel** 🟡
18. Admin — Dashboard — High
19. Admin — Manajemen Produk — Critical
20. Admin — Manajemen Kategori — High
21. Admin — Manajemen Pesanan — Critical
22. Admin — Manajemen Pelanggan — Medium

---

> **Legenda**: 🟢 = sesuai mockup gambar. 🟡 = rancangan usulan tambahan.
>
> **Struktur tiap halaman**: (1) Tujuan → (2) Layout → (3–7) Komponen/State/Interaksi/UX/Responsive → (8) Spesifikasi Ukuran → (9) Design Token → (10) Animasi → (11) Acceptance Criteria → (12) Edge Case → (13) Developer Notes → **(14) Priority & Requirement ID → (15) Dependency → (16) Component Relationship**.



---

# 00 — Project Architecture

## 1. Tujuan Sistem
Jati Prime App adalah platform e-commerce vertikal untuk brand furniture kayu jati, terdiri dari 2 sisi:
- **Customer App** — katalog produk, transaksi (cart → checkout → tracking), konten brand (galeri, testimoni, FAQ, kontak).
- **Admin Panel** — pengelolaan katalog, pesanan, dan pelanggan oleh internal tim Jati Prime.

Tujuan bisnis utama: mengkonversi pengunjung menjadi pembeli furniture custom/ready-stock, dengan kanal komunikasi utama via WhatsApp, serta membangun kepercayaan lewat konten (galeri workshop, testimoni, garansi).

## 2. Modul Sistem

| Modul | Cakupan Halaman |
|---|---|
| **Catalog** | Home, Kategori, Produk, Detail Produk, Wishlist |
| **Transaction** | Keranjang, Checkout, Pesanan Saya, Tracking |
| **Identity** | Login, Register, Akun Saya |
| **Content/Marketing** | Tentang Kami, Galeri, Testimoni, FAQ, Kontak |
| **Admin — Catalog Management** | Admin Produk, Admin Kategori |
| **Admin — Operations** | Admin Dashboard, Admin Pesanan, Admin Pelanggan |

## 3. Arsitektur Aplikasi (High-Level)

```
┌─────────────────────┐        ┌─────────────────────┐
│   Customer App       │        │    Admin Panel        │
│ (Mobile-first Web/    │        │  (Web, desktop-first) │
│  Native App)          │        │                       │
└──────────┬───────────┘        └──────────┬───────────┘
           │                                │
           │            REST API            │
           └───────────────┬────────────────┘
                            │
                  ┌─────────▼─────────┐
                  │    API Gateway /    │
                  │   Backend Service   │
                  │  (Auth, Catalog,    │
                  │   Order, Content)   │
                  └─────────┬─────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                    │
┌───────▼──────┐   ┌────────▼───────┐   ┌────────▼───────┐
│   Database     │   │  Object Storage │   │  3rd Party      │
│ (PostgreSQL/    │   │ (Gambar produk, │   │ (WhatsApp API,  │
│  MySQL)         │   │  galeri, avatar)│   │  Maps, Payment) │
└────────────────┘   └────────────────┘   └────────────────┘
```

## 4. Struktur Folder (Usulan — Frontend, contoh berbasis React/Next.js)

```
src/
├── app/                     # Routing per halaman (App Router)
│   ├── (customer)/
│   │   ├── home/
│   │   ├── kategori/
│   │   ├── produk/[slug]/
│   │   ├── keranjang/
│   │   ├── checkout/
│   │   ├── akun/
│   │   ├── wishlist/
│   │   ├── pesanan/
│   │   └── tracking/[id]/
│   └── admin/
│       ├── dashboard/
│       ├── produk/
│       ├── kategori/
│       ├── pesanan/
│       └── pelanggan/
├── components/
│   ├── ui/                  # Reusable: Button, Card, Input, Modal, dst (lihat 00-component-library.md)
│   └── domain/               # ProductCard, OrderCard, dsb (spesifik bisnis)
├── lib/
│   ├── api/                  # API client per modul (products.ts, orders.ts, dst)
│   └── hooks/                 # useCart, useWishlist, useAuth, dst
├── styles/
│   └── tokens.css            # Design tokens (lihat 00-design-system.md)
└── types/                    # TypeScript interfaces (lihat 00-entity-relationship.md)
```

## 5. Tech Stack (Usulan)

| Layer | Teknologi |
|---|---|
| Frontend Customer | React Native / Next.js (PWA) — mobile-first |
| Frontend Admin | React (Next.js/Vite), desktop-first |
| State Management | React Query (server state) + Zustand/Context (UI state) |
| Backend | Node.js (NestJS/Express) atau Laravel |
| Database | PostgreSQL |
| Object Storage | S3-compatible (gambar produk, galeri, avatar) |
| Auth | JWT + Refresh Token |
| 3rd Party | WhatsApp Business API (chat), Google Maps API (kontak), Payment Gateway (Midtrans/Xendit — untuk transfer bank/VA) |
| Notifikasi | Push Notification (FCM) + WhatsApp notification (status pesanan) |

## 6. Diagram Hubungan Modul

```
Identity (Login/Register/Akun)
      │
      ▼
Catalog (Home → Kategori → Produk → Detail Produk → Wishlist)
      │
      ▼
Transaction (Keranjang → Checkout → Pesanan Saya → Tracking)
      │
      ▼
Content/Marketing (Testimoni ← dipicu setelah Pesanan "Selesai")

Admin — Catalog Management ──► mengelola data yang tampil di Catalog (customer)
Admin — Operations ──► memproses data dari Transaction (customer) & memberi update ke Tracking
```

## 7. Non-Functional Requirements (Global — berlaku semua halaman)

| Kategori | Target |
|---|---|
| Load Time (initial) | < 2 detik (3G fast / 4G) |
| LCP (Largest Contentful Paint) | < 2.5 detik |
| CLS (Cumulative Layout Shift) | < 0.1 |
| FID/INP | < 200ms |
| Accessibility | WCAG 2.1 Level AA |
| SEO (Lighthouse score) | ≥ 95 (halaman customer publik) |
| Uptime API | ≥ 99.5% |
| Skalabilitas | Mendukung minimal 1.000 concurrent user tanpa degradasi > 20% response time |
| Keamanan | HTTPS wajib, password hashing (bcrypt/argon2), rate-limiting endpoint auth |
| Kompatibilitas Browser | 2 versi terbaru Chrome, Safari, Firefox, Edge; iOS Safari & Chrome Android |
| Bahasa | Bahasa Indonesia (default), struktur siap untuk multi-bahasa (i18n-ready) di masa depan |

> Target NFR spesifik per halaman (jika ada pengecualian/penekanan khusus) dicantumkan di section "Non-Functional Requirement" masing-masing halaman.

## 8. Version History / Roadmap

| Versi | Scope | Status |
|---|---|---|
| **v1.0 (MVP)** | Home, Kategori, Produk, Detail Produk, Keranjang, Checkout, Login, Register, Akun Saya | Rencana rilis awal |
| **v1.1** | Wishlist, Pesanan Saya, Tracking, Testimoni, FAQ, Kontak, Tentang Kami, Galeri | Rilis susulan (peningkatan retensi & trust) |
| **v1.2** | Admin Panel penuh (Dashboard, Produk, Kategori, Pesanan, Pelanggan) | Rilis untuk kebutuhan operasional internal |
| **v1.3 (usulan)** | 3D Viewer produk (rotate 360°), Custom Furniture Configurator, integrasi Payment Gateway VA otomatis | Belum direncanakan, potensi pengembangan lanjutan |
| **v1.4 (usulan)** | Multi-bahasa (EN), Program Loyalitas/Referral, Live Chat in-app | Belum direncanakan |

## 9. System Flow Diagram (End-to-End)

```
[Login/Register]
        │
        ▼
[Home] ──► [Kategori] ──► [Produk] ──► [Detail Produk]
                                              │
                              ┌───────────────┼───────────────┐
                              ▼                                ▼
                        [Tambah ke Keranjang]           [Tambah ke Wishlist]
                              │
                              ▼
                        [Keranjang]
                              │
                              ▼
                        [Checkout] ──(butuh)──► [Alamat] + [Metode Pengiriman] + [Metode Pembayaran]
                              │
                              ▼
                        [Buat Pesanan] ──► [Payment/Konfirmasi]
                              │
                              ▼
                        [Pesanan Saya] ──► [Tracking Pesanan]
                              │
                              ▼
                    (Status: Selesai) ──► [Beri Ulasan] ──► muncul di [Testimoni]

--- Sisi Admin (paralel) ---
[Admin Pesanan] ──(update status/resi)──► sinkron ke [Tracking] customer
[Admin Produk]/[Admin Kategori] ──(kelola data)──► tampil di [Kategori]/[Produk]/[Detail Produk] customer
[Admin Pelanggan] ──(monitor/blokir)──► mempengaruhi akses [Login] customer
```


---

# 00 — Entity Relationship & Data Dictionary

> Melengkapi `00-api-database-mapping.md` (yang fokus ke skema tabel per field) dengan: diagram ERD visual, penjelasan relasi antar entitas, data dictionary lengkap (tipe, constraint, nullable), dan **Data Contract** — bentuk objek JSON persis yang dikirim API ke frontend per entitas.

## 1. Diagram ERD (Ascii)

```
┌───────────────┐        ┌───────────────┐        ┌───────────────┐
│    users        │1      N│   addresses     │        │   categories    │
│ id (PK)         │◄──────►│ id (PK)          │        │ id (PK)         │
│ name            │        │ user_id (FK)     │        │ name            │
│ email           │        │ recipient_name   │        │ slug            │
│ phone           │        │ phone            │        │ icon_url        │
│ password_hash   │        │ full_address     │        │ sort_order      │
│ avatar_url      │        │ is_default       │        └────────┬────────┘
│ is_active       │        └───────────────┘                 │1
└───┬───────┬─────┘                                            │
   1│       │1                                                 │N
    │       │                                          ┌────────▼────────┐
    │N      │N                                          │    products      │
┌───▼────┐ ┌▼──────────┐  ┌───────────────┐            │ id (PK)          │
│ carts   │ │ wishlists  │  │    orders       │           │ slug             │
│ id      │ │ id         │  │ id (PK)         │           │ name             │
│ user_id │ │ user_id    │  │ user_id (FK)    │           │ description      │
└───┬────┘ │ product_id │  │ address_id (FK) │           │ price            │
   1│      └────────────┘  │ shipping_method │           │ stock            │
    │N                     │ payment_method  │           │ category_id (FK) │
┌───▼─────────┐            │ status          │           │ material         │
│ cart_items   │           │ courier_name    │           │ finishing        │
│ id           │           │ resi_no         │           │ dimension        │
│ cart_id (FK) │           │ subtotal        │           │ capacity         │
│ product_id   │───────────│ total           │           │ warranty         │
│ qty          │      N   1│ created_at      │           │ rating_avg       │
└─────────────┘            └────────┬────────┘           │ review_count     │
                                    1│                     │ is_active        │
                                     │N                    └────────┬─────────┘
                            ┌────────▼─────────┐                  1│
                            │   order_items      │                  │N
                            │ id                 │          ┌────────▼─────────┐
                            │ order_id (FK)      │          │  product_images    │
                            │ product_id (FK)    │          │ id                 │
                            │ qty                │          │ product_id (FK)    │
                            │ price_at_purchase  │          │ image_url          │
                            └───────────────────┘           │ sort_order         │
                                                             └───────────────────┘
                            ┌───────────────────┐
                            │     reviews         │
                            │ id                  │
                            │ product_id (FK,null)│───────► products (N:1, nullable = review toko)
                            │ user_id (FK)        │───────► users (N:1)
                            │ rating              │
                            │ comment             │
                            │ photos              │
                            └───────────────────┘

┌───────────────┐         ┌───────────────┐
│ gallery_photos  │        │     faqs        │
│ id              │        │ id              │
│ image_url       │        │ question        │
│ tag             │        │ answer          │
│ related_product │──────► │ sort_order      │
│   _id (FK,null) │products│                 │
└───────────────┘         └───────────────┘
```

## 2. Penjelasan Relasi

| Relasi | Kardinalitas | Keterangan |
|---|---|---|
| `users` → `addresses` | 1 : N | Satu user bisa punya banyak alamat, salah satu ditandai `is_default` |
| `users` → `carts` | 1 : 1 | Satu user hanya punya satu keranjang aktif |
| `carts` → `cart_items` | 1 : N | Satu keranjang berisi banyak baris item |
| `cart_items` → `products` | N : 1 | Banyak baris cart_item bisa merujuk produk yang sama (user berbeda) |
| `users` → `orders` | 1 : N | Riwayat transaksi user |
| `orders` → `order_items` | 1 : N | Snapshot item yang dibeli dalam satu transaksi |
| `orders` → `addresses` | N : 1 | Order merujuk 1 alamat pengiriman (snapshot alamat saat order dibuat, idealnya di-copy bukan hanya FK, lihat Developer Notes) |
| `categories` → `products` | 1 : N | Satu kategori punya banyak produk |
| `products` → `product_images` | 1 : N | Galeri gambar per produk |
| `products` → `reviews` | 1 : N | Ulasan spesifik produk (`product_id` terisi) |
| `users` → `reviews` | 1 : N | Riwayat ulasan yang pernah dibuat user |
| `users` → `wishlists` → `products` | N : N (via junction table) | Relasi many-to-many favorit user ke produk |
| `products` → `gallery_photos` | 1 : N (opsional) | Cross-link foto galeri ke produk terkait |

## 3. Data Dictionary — Detail Constraint

### Entity: `products`
| Field | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | UUID | PK, not null | |
| slug | varchar(160) | unique, not null | URL-friendly, auto-generate dari `name` |
| name | varchar(160) | not null | |
| description | text | nullable | |
| price | decimal(14,2) | not null, >= 0 | dalam Rupiah, tanpa desimal sen |
| stock | integer | not null, default 0, >= 0 | |
| category_id | UUID | FK → categories.id, not null | |
| material | varchar(100) | nullable | |
| finishing | varchar(100) | nullable | |
| dimension | varchar(100) | nullable | format bebas, mis. "200 x 100 x 75 cm" |
| capacity | varchar(50) | nullable | mis. "6 Kursi", kosong untuk produk non-set |
| warranty | varchar(50) | nullable | mis. "5 Tahun" |
| rating_avg | decimal(2,1) | default 0, computed | di-update via trigger/job saat ada review baru |
| review_count | integer | default 0, computed | |
| is_active | boolean | default true | soft-delete flag |
| created_at / updated_at | timestamp | not null | |

### Entity: `orders`
| Field | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | UUID | PK | |
| invoice_no | varchar(30) | unique, not null | format `JP-YYYYMMDD-XXX` |
| user_id | UUID | FK → users.id | |
| address_snapshot | jsonb | not null | **copy** data alamat saat order dibuat (bukan hanya FK — lihat Developer Notes) |
| shipping_method | enum(`reguler`,`express`) | not null | |
| shipping_cost | decimal(14,2) | not null, default 0 | |
| payment_method | enum(`transfer_bank`,`cod`) | not null | |
| status | enum(`menunggu_pembayaran`,`diproses`,`dikirim`,`selesai`,`dibatalkan`) | not null, default `menunggu_pembayaran` | |
| courier_name | varchar(60) | nullable | terisi saat status `dikirim` |
| resi_no | varchar(60) | nullable | terisi saat status `dikirim` |
| subtotal | decimal(14,2) | not null | |
| total | decimal(14,2) | not null | subtotal + shipping_cost |
| created_at | timestamp | not null | |

## 4. Data Contract (Bentuk Objek JSON per Entitas)

### `Product` (response `/api/products/{slug}`)
```json
{
  "id": "prod_01HZY...",
  "slug": "meja-makan-jati-solid",
  "name": "Meja Makan Jati Solid",
  "description": "Meja premium berbahan kayu jati solid Grade A...",
  "price": 14800000,
  "stock": 8,
  "category": { "id": "cat_sofa", "name": "Meja Makan", "slug": "meja-makan" },
  "images": [
    { "url": "https://cdn.jatiprime.id/p/1.jpg", "sort_order": 0 },
    { "url": "https://cdn.jatiprime.id/p/2.jpg", "sort_order": 1 }
  ],
  "specs": {
    "material": "Kayu Jati Solid Grade A",
    "finishing": "Natural Oil",
    "dimension": "200 x 100 x 75 cm",
    "capacity": "6 Kursi",
    "warranty": "5 Tahun"
  },
  "rating_avg": 4.9,
  "review_count": 73,
  "is_wishlisted": false,
  "is_active": true
}
```

### `CartItem` (response `/api/cart`)
```json
{
  "id": "cart_item_01",
  "product": { "id": "prod_01", "name": "Meja Makan Jati Solid", "price": 14800000, "image": "https://cdn.../1.jpg", "stock": 8 },
  "qty": 1,
  "line_total": 14800000
}
```

### `Order` (response `/api/orders/{id}`)
```json
{
  "id": "order_01",
  "invoice_no": "JP-20260724-001",
  "status": "dikirim",
  "address": { "recipient_name": "Bagus Sajiwo", "phone": "0812-3456-7890", "full_address": "Jl. Raya Jepara Bangsri KM. 7, Jepara, Jawa Tengah 59453" },
  "shipping_method": "reguler",
  "shipping_cost": 0,
  "payment_method": "transfer_bank",
  "courier_name": "JNE",
  "resi_no": "JNE1234567890",
  "items": [
    { "product_name": "Meja Makan Jati Solid", "qty": 1, "price_at_purchase": 14800000 }
  ],
  "subtotal": 27550000,
  "total": 27550000,
  "created_at": "2026-07-20T10:00:00Z"
}
```

### `Review` (response `/api/reviews`)
```json
{
  "id": "rev_01",
  "user": { "name": "Andi Pratama", "avatar_url": null },
  "rating": 5,
  "comment": "Kualitas produk sangat bagus, finishing rapi...",
  "photos": ["https://cdn.../r1.jpg", "https://cdn.../r2.jpg"],
  "created_at": "2026-07-22T08:00:00Z"
}
```

### `Category` (response `/api/categories`)
```json
{
  "id": "cat_sofa",
  "name": "Sofa",
  "slug": "sofa",
  "icon_url": "https://cdn.../icon-sofa.svg",
  "product_count": 24
}
```

## 5. Developer Notes
- **Snapshot alamat & harga di order**: jangan hanya simpan `address_id`/`product_id` sebagai referensi murni — simpan salinan datanya (`address_snapshot` jsonb, `order_items.price_at_purchase` + nama produk) agar riwayat pesanan tidak berubah/rusak jika data master (alamat dihapus, harga produk naik) berubah di kemudian hari.
- `rating_avg` & `review_count` pada `products` sebaiknya berupa computed/cached field yang di-update via background job atau database trigger setiap ada review baru — jangan hitung agregat real-time di setiap request Detail Produk (performa).
- Semua `id` menggunakan UUID (bukan auto-increment integer) untuk menghindari enumeration attack pada endpoint publik seperti `/api/products/{id}`.


---

# 00 — Component Library (Reusable UI Components)

> Daftar seluruh komponen UI reusable yang dipakai berulang di 22 halaman. Setiap komponen didefinisikan sekali di sini; halaman individual cukup mereferensikan nama komponennya (lihat section "Component Relationship" di tiap halaman) — bukan mendefinisikan ulang.

---

## 1. Button

### Varian
| Varian | Background | Text Color | Border | Penggunaan |
|---|---|---|---|---|
| Primary | `color/primary` | `#FFFFFF` | none | CTA utama (Checkout, Tambah ke Keranjang) |
| Secondary/Outline | transparent | `color/primary` | 1.5px `color/primary` | Chat WhatsApp, Konsultasi Gratis |
| Destructive | transparent | `color/danger` | none / 1px `color/danger` | Hapus, Batalkan Pesanan, Keluar |
| Disabled | `color/text-disabled` | `#FFFFFF` | none | Field belum lengkap |
| Text/Link | transparent | `color/primary` | none | "Lihat Semua", "Ubah" |

### Ukuran
| Size | Height | Padding Horizontal | Font | Radius |
|---|---|---|---|---|
| Large | 52px | 24px | `type/button` 16px/600 | `radius/md` (12px) atau `radius/pill` untuk CTA hero |
| Medium | 44px | 20px | 14px/600 | `radius/md` |
| Small | 36px | 16px | 13px/600 | `radius/sm` |

### State
- Default, Hover (desktop, darken 8%), Pressed (`scale(0.98)`, darken 12%), Disabled (opacity 50% + `color/text-disabled`), Loading (spinner menggantikan label, tombol tetap ukuran sama agar tidak layout-shift).

### Animasi
- Press: `scale(0.98)`, 100ms, ease-out.
- Loading spinner: rotate 360° infinite, 800ms/putaran.

---

## 2. Card

### Varian
| Varian | Radius | Shadow | Padding | Penggunaan |
|---|---|---|---|---|
| Product Card | `radius/lg` (16px) | `shadow/md` | 12px | Grid produk (Home, Produk, Wishlist) |
| Info Card | `radius/lg` | `shadow/sm` | 16px | Address Card, Ringkasan Belanja, Profile Card |
| Order Card | `radius/lg` | `shadow/sm` | 16px | Pesanan Saya |
| Promo/Banner Card | `radius/lg` | none | 20px | Hero Banner, Custom Furniture Banner |

### Anatomy (Product Card)
Image (aspect 4:3, radius 18px) → Wishlist icon (overlay top-right) → Title → Price → Rating.

---

## 3. Input (Text Field)

| Properti | Nilai |
|---|---|
| Height | 44–48px |
| Radius | `radius/md` |
| Border default | 1px `color/border` |
| Border focus | 1.5px `color/primary` |
| Border error | 1.5px `color/danger` |
| Padding horizontal | 12px |
| Placeholder color | `color/text-disabled` |
| Helper/error text | `type/body-sm`, margin-top 4px |

Varian: Text, Password (dengan toggle show/hide), Search (dengan leading icon + optional trailing filter icon), Textarea (min-height 100px).

---

## 4. Modal (Dialog)

| Properti | Nilai |
|---|---|
| Backdrop | `color/overlay` (`rgba(0,0,0,0.35–0.5)`) |
| Container width | 90% (mobile), max 400px |
| Radius | `radius/lg` |
| Padding | 24px |
| Posisi | Center screen |
| Animasi masuk | Fade backdrop (200ms) + scale-in container (0.95→1, 250ms ease-out) |
| Animasi keluar | Fade + scale-out (200ms) |

Penggunaan: Konfirmasi hapus item, konfirmasi logout, konfirmasi batalkan pesanan.

---

## 5. Drawer (Side Menu)

| Properti | Nilai |
|---|---|
| Width | 80% layar (max 320px) |
| Posisi | Slide dari kiri |
| Backdrop | `color/overlay` |
| Animasi | Slide-in 300ms ease-out, slide-out 250ms ease-in |

Penggunaan: Side menu dari ikon hamburger di Top Navigation Bar (opsional, untuk navigasi tambahan).

---

## 6. Bottom Sheet

| Properti | Nilai |
|---|---|
| Radius top | `radius/xl` (20px) di kedua sudut atas |
| Drag handle | 32×4px, `color/border`, center top, margin 8px |
| Max height | 85% viewport |
| Backdrop | `color/overlay` |
| Animasi | Slide-up 300ms ease-out (masuk), slide-down 250ms ease-in (keluar), dismissible via drag-down atau tap backdrop |

Penggunaan: Filter lanjutan (halaman Produk), prompt login saat guest tap wishlist, opsi share.

---

## 7. Toast

| Properti | Nilai |
|---|---|
| Posisi | Top of screen, di bawah top nav, margin 16px |
| Width | Auto, max 90% |
| Radius | `radius/md` |
| Background | `color/text-primary` (dark) untuk info netral; `color/success`/`color/danger` untuk status |
| Durasi tampil | 3000ms auto-dismiss |
| Animasi | Slide-down + fade (250ms in), fade-out (200ms out) |

Penggunaan: "Ditambahkan ke keranjang", "Nomor rekening disalin", error umum non-blocking.

---

## 8. Snackbar

| Properti | Nilai |
|---|---|
| Posisi | Bottom of screen, di atas Bottom Navigation, margin 12px |
| Width | Full-width minus margin 16px kiri-kanan |
| Radius | `radius/md` |
| Background | `color/text-primary` |
| Action button | Teks warna `color/primary` terang/aksen, di kanan (mis. "Undo") |
| Durasi tampil | 5000ms (lebih lama dari Toast karena ada actionable) |
| Animasi | Slide-up + fade (300ms in), slide-down (200ms out) |

Penggunaan: "Item dihapus, Undo" (Keranjang), "Item dihapus dari wishlist, Undo".

**Perbedaan Toast vs Snackbar**: Toast = notifikasi pasif tanpa aksi (posisi atas), Snackbar = notifikasi dengan aksi lanjutan seperti Undo (posisi bawah).

---

## 9. Accordion

| Properti | Nilai |
|---|---|
| Row height (collapsed) | 52px |
| Chevron icon | 20×20px, rotate 180° saat expand |
| Animasi expand | Height auto-animate 250ms ease-in-out |
| Divider antar item | 1px `color/border` |

Penggunaan: FAQ.

---

## 10. Tabs / Filter Chip

| Properti | Nilai |
|---|---|
| Height | 36px |
| Radius | `radius/pill` |
| Active | background `color/primary`, text putih |
| Inactive | border 1px `color/border`, text `color/text-secondary` |
| Gap antar tab | 8px |
| Animasi switch | Background crossfade 150ms |

Penggunaan: Filter Produk (Semua/Terbaru/Terlaris), Galeri (Semua/Workshop/Produk/Customer), Pesanan Saya (status).

---

## 11. Stepper

### 11a. Quantity Stepper
| Properti | Nilai |
|---|---|
| Button size | 28×28px |
| Border | 1px `color/border` |
| Angka | `type/h3`, min-width 24px |

Penggunaan: Keranjang.

### 11b. Numbered Step (Progress Stepper)
| Properti | Nilai |
|---|---|
| Circle size | 24px |
| Background aktif/selesai | `color/primary` |
| Background pending | `color/border` |
| Connector line | 2px, vertikal (Tracking) atau tersirat via urutan section (Checkout) |

Penggunaan: Checkout (4 step horizontal-implicit), Tracking Pesanan (step vertikal eksplisit).

---

## 12. Gallery / Image Carousel

| Properti | Nilai |
|---|---|
| Main image height | 320–420px tergantung konteks |
| Thumbnail size | 56×56px |
| Thumbnail active border | 2px `color/primary` |
| Swipe animasi | Slide horizontal 300ms ease-in-out |

Penggunaan: Detail Produk (galeri produk), Galeri (grid + lightbox fullscreen).

---

## 13. Badge

| Varian | Background | Text | Penggunaan |
|---|---|---|---|
| Success | `rgba(62,142,90,0.12)` | `color/success` | "Stok Tersedia", status "Selesai" |
| Warning | `rgba(224,169,62,0.12)` | `color/warning` | status "Diproses" |
| Info | `rgba(59,110,165,0.12)` | `color/info` | status "Dikirim" |
| Danger | `rgba(229,57,53,0.12)` | `color/danger` | "Stok Habis", status "Dibatalkan" |
| Neutral/Count | `color/primary` | putih | Badge jumlah item keranjang (angka kecil di ikon) |

Ukuran: height 20–24px, radius `radius/sm`, padding horizontal 8px, font `type/body-sm` 700.

---

## 14. Avatar

| Properti | Nilai |
|---|---|
| Size Small | 32×32px (list pelanggan admin, review item) |
| Size Medium | 40×40px (review item customer-facing) |
| Size Large | 64×64px (Profile Card, Akun Saya) |
| Radius | 50% (circle) |
| Fallback | Inisial nama, background warna solid dari palet turunan `color/primary` |

---

## 15. Ringkasan Pemetaan Komponen ke Halaman

| Komponen | Digunakan di Halaman |
|---|---|
| Button | Semua halaman |
| Card (Product) | Home, Produk, Wishlist, Pesanan Saya (thumbnail), Detail Produk (terkait) |
| Input | Produk (search), Login, Register, Checkout (alamat form), Admin Produk |
| Modal | Keranjang (konfirmasi hapus), Akun Saya (konfirmasi logout), Pesanan Saya (batalkan) |
| Drawer | Home (side menu, opsional semua halaman) |
| Bottom Sheet | Produk (filter lanjutan), Login prompt dari Wishlist |
| Toast | Detail Produk, Kontak (salin), Checkout |
| Snackbar | Keranjang, Wishlist |
| Accordion | FAQ |
| Tabs/Filter Chip | Produk, Galeri, Pesanan Saya, Admin Pesanan |
| Stepper (Qty) | Keranjang |
| Stepper (Progress) | Checkout, Tracking |
| Gallery | Detail Produk, Galeri |
| Badge | Detail Produk, Pesanan Saya, Admin Produk, Admin Pesanan |
| Avatar | Akun Saya, Testimoni, Admin Pelanggan |


---

# 00 — Design System & Design Tokens

> File ini adalah **sumber kebenaran tunggal (single source of truth)** untuk seluruh nilai visual (warna, tipografi, spacing, radius, shadow, animasi) yang dipakai di 22 halaman. Semua halaman lain merujuk ke token di sini — jangan hardcode ulang nilai di tempat lain.

## 1. Color Tokens

| Token | Hex | Penggunaan |
|---|---|---|
| `color/primary` | `#6F7863` | Tombol utama, ikon aktif, bottom nav aktif |
| `color/primary-hover` | `#5C6552` | State hover/pressed tombol primary |
| `color/primary-dark` | `#4A5142` | Header gelap, footer, text on-light emphasis |
| `color/body-bg` | `#F8F5EF` | Background utama seluruh app (cream) |
| `color/surface` | `#FFFFFF` | Background card, modal, input field |
| `color/border` | `#E5E1D8` | Border card, divider, input outline |
| `color/text-primary` | `#2B2B26` | Judul, nama produk, teks utama |
| `color/text-secondary` | `#6B6B63` | Deskripsi, label, teks sekunder |
| `color/text-disabled` | `#B5B0A6` | Placeholder, teks nonaktif |
| `color/success` | `#3E8E5A` | Badge "Stok Tersedia", status "Selesai" |
| `color/warning` | `#E0A93E` | Badge "Diproses", peringatan stok menipis |
| `color/danger` | `#E53935` | Error, "Stok Habis", tombol hapus/destruktif |
| `color/info` | `#3B6EA5` | Badge "Dikirim", link informasi |
| `color/rating-star` | `#F5B301` | Ikon bintang rating |
| `color/overlay` | `rgba(0,0,0,0.35)` | Overlay gambar hero, modal backdrop |

## 2. Typography Scale

| Token | Font Family | Size | Weight | Line Height | Penggunaan |
|---|---|---|---|---|---|
| `type/display` | Playfair Display | 32px | 700 | 40px | Hero headline (Home) |
| `type/h1` | Playfair Display | 24px | 700 | 32px | Judul halaman (mis. "Tentang Kami") |
| `type/h2` | Playfair Display | 20px | 600 | 28px | Judul section (mis. "Koleksi Terbaik") |
| `type/h3` | Inter | 16px | 600 | 24px | Nama produk, judul card |
| `type/body` | Inter | 14px | 400 | 20px | Paragraf deskripsi |
| `type/body-sm` | Inter | 12px | 400 | 18px | Caption, label kecil, timestamp |
| `type/price` | Inter | 18px | 700 | 24px | Harga produk |
| `type/button` | Inter | 16px | 600 | 20px | Teks tombol |
| `type/nav-label` | Inter | 11px | 500 | 14px | Label bottom navigation |

**Font fallback stack**: `Playfair Display, Georgia, serif` untuk heading; `Inter, -apple-system, Segoe UI, sans-serif` untuk body/UI.

## 3. Spacing System (base grid 8px)

| Token | Nilai | Penggunaan |
|---|---|---|
| `space/xs` | 4px | Gap ikon-label kecil |
| `space/sm` | 8px | Gap antar elemen dalam 1 komponen (mis. rating - jumlah ulasan) |
| `space/md` | 16px | Padding card, gap grid antar kartu |
| `space/lg` | 24px | Padding section, margin antar blok konten |
| `space/xl` | 32px | Margin antar section besar |
| `space/2xl` | 48px | Padding top/bottom section hero |
| `space/container` | 16px | Padding kiri-kanan layar (screen margin) mobile |
| `space/section-mobile` | 40px | Jarak antar section di mobile |
| `space/section-desktop` | 80px | Jarak antar section di breakpoint desktop |

## 4. Border Radius Tokens

| Token | Nilai | Penggunaan |
|---|---|---|
| `radius/sm` | 8px | Chip, badge, filter tab |
| `radius/md` | 12px | Input field, tombol standar |
| `radius/lg` | 16px | Card produk, card ringkasan |
| `radius/xl` | 18–20px | Gambar produk, hero image |
| `radius/pill` | 999px | Tombol CTA utama (mis. "Jelajahi Koleksi"), avatar |

## 5. Elevation / Shadow Tokens

| Token | Nilai CSS | Penggunaan |
|---|---|---|
| `shadow/sm` | `0 1px 2px rgba(0,0,0,0.05)` | List item, input focus ring |
| `shadow/md` | `0 4px 12px rgba(0,0,0,0.08)` | Product card default |
| `shadow/lg` | `0 8px 24px rgba(0,0,0,0.12)` | Modal, bottom sheet, card hover/pressed |
| `shadow/nav` | `0 -2px 8px rgba(0,0,0,0.06)` | Bottom navigation bar (shadow ke atas) |

## 6. Animation & Motion Tokens

| Token | Durasi | Easing | Penggunaan |
|---|---|---|---|
| `motion/fast` | 150ms | `cubic-bezier(0.4,0,0.2,1)` | Toggle wishlist, tap feedback tombol |
| `motion/base` | 250ms | `cubic-bezier(0.4,0,0.2,1)` | Expand accordion, transisi tab |
| `motion/slow` | 400ms | `cubic-bezier(0.4,0,0.2,1)` | Page transition, modal open/close |
| `motion/hero` | 800ms | `ease-out` | Fade-in hero banner saat halaman dimuat |
| `motion/skeleton` | 1200ms loop | `ease-in-out` | Shimmer efek pada skeleton loader |

## 7. Iconography
- Style: **outline/line-icon**, stroke width 1.5px, ukuran default 24×24px (bottom nav: 22×22px).
- Sumber ikon disarankan: Lucide Icons / Feather Icons agar konsisten di seluruh app.

## 8. Breakpoints

| Token | Lebar | Kolom Grid Produk |
|---|---|---|
| `bp/mobile` | 0–599px | 2 kolom |
| `bp/tablet` | 600–1023px | 3 kolom |
| `bp/desktop` | ≥1024px | 4 kolom |

## 9. Grid & Container
- Max-width container desktop: 1200px, center-aligned.
- Grid gap: `space/md` (16px) sebagai default di semua breakpoint.
- Screen margin (padding kiri-kanan) mobile: `space/container` (16px).


---

# 00 — API Mapping & Database Schema

> File ini adalah rujukan bersama untuk backend & frontend developer: daftar endpoint per halaman, serta skema database inti. Halaman individual akan mereferensikan endpoint dari sini di section "API Mapping" masing-masing.

## 1. Ringkasan Endpoint per Halaman

| Halaman | Method | Endpoint | Response Utama |
|---|---|---|---|
| Home | GET | `/api/home` | `hero[]`, `categories[]`, `best_sellers[]` |
| Kategori | GET | `/api/categories` | `id, name, icon_url, product_count` |
| Produk | GET | `/api/products?search=&category=&sort=&page=` | `products[]` (paginated), `total` |
| Detail Produk | GET | `/api/products/{slug}` | `product` (full detail + gallery + specs) |
| Detail Produk | POST | `/api/wishlist/{product_id}` | toggle wishlist |
| Keranjang | GET | `/api/cart` | `items[]`, `subtotal`, `total` |
| Keranjang | PATCH | `/api/cart/{item_id}` | update qty |
| Keranjang | DELETE | `/api/cart/{item_id}` | hapus item |
| Checkout | GET | `/api/checkout/summary` | `address`, `shipping_methods[]`, `payment_methods[]`, `summary` |
| Checkout | POST | `/api/orders` | membuat order baru → `order_id` |
| Tentang Kami | GET | `/api/pages/about` | konten statis (CMS) |
| Galeri | GET | `/api/gallery?tag=` | `photos[]` (paginated) |
| Testimoni | GET | `/api/reviews?sort=&page=` | `summary {avg, total, breakdown[]}`, `reviews[]` |
| FAQ | GET | `/api/faq` | `faq[]` (question, answer) |
| Kontak | GET | `/api/contact` | info kontak & lokasi (CMS) |
| Akun Saya | GET | `/api/me` | profil user |
| Wishlist | GET | `/api/wishlist` | `products[]` |
| Login | POST | `/api/auth/login` | `token`, `user` |
| Register | POST | `/api/auth/register` | `token`, `user` |
| Pesanan Saya | GET | `/api/orders?status=` | `orders[]` (paginated) |
| Tracking | GET | `/api/orders/{id}/tracking` | `timeline[]`, `courier {name, resi}` |
| Admin Dashboard | GET | `/api/admin/stats` | `sales_summary`, `recent_orders[]`, `low_stock[]` |
| Admin Produk | GET/POST/PUT/DELETE | `/api/admin/products` | CRUD produk |
| Admin Kategori | GET/POST/PUT/DELETE | `/api/admin/categories` | CRUD kategori |
| Admin Pesanan | GET/PATCH | `/api/admin/orders` | list & update status |
| Admin Pelanggan | GET/PATCH | `/api/admin/customers` | list & blokir/aktifkan |

## 2. Autentikasi
- Semua endpoint `/api/me`, `/api/cart`, `/api/orders`, `/api/wishlist`, `/api/admin/*` membutuhkan header `Authorization: Bearer {token}`.
- Endpoint publik (tanpa auth): `/api/home`, `/api/categories`, `/api/products`, `/api/products/{slug}`, `/api/faq`, `/api/contact`, `/api/gallery`, `/api/reviews`.

## 3. Database Schema (Entitas Inti)

### 3.1 Tabel `users`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| name | varchar | |
| email | varchar (unique) | |
| phone | varchar | |
| password_hash | varchar | |
| avatar_url | varchar (nullable) | |
| is_active | boolean | default true, dipakai fitur blokir akun (admin) |
| created_at | timestamp | |
| updated_at | timestamp | |

### 3.2 Tabel `categories`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| name | varchar | mis. "Sofa", "Meja Makan" |
| slug | varchar (unique) | |
| icon_url | varchar | |
| sort_order | integer | untuk reorder tampilan grid Kategori |
| is_active | boolean | |

### 3.3 Tabel `products`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| slug | varchar (unique) | |
| name | varchar | |
| description | text | |
| price | decimal | |
| stock | integer | |
| category_id | UUID (FK → categories.id) | |
| material | varchar | mis. "Kayu Jati Solid Grade A" |
| finishing | varchar | mis. "Natural Oil" |
| dimension | varchar | mis. "200 x 100 x 75 cm" |
| capacity | varchar (nullable) | mis. "6 Kursi" |
| warranty | varchar | mis. "5 Tahun" |
| rating_avg | decimal | agregat dari `reviews` |
| review_count | integer | agregat dari `reviews` |
| is_active | boolean | |
| created_at | timestamp | |
| updated_at | timestamp | |

### 3.4 Tabel `product_images`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| product_id | UUID (FK) | |
| image_url | varchar | |
| sort_order | integer | urutan thumbnail di galeri produk |

### 3.5 Tabel `carts` & `cart_items`
| Field | Tipe | Keterangan |
|---|---|---|
| cart.id | UUID (PK) | |
| cart.user_id | UUID (FK) | |
| cart_item.id | UUID (PK) | |
| cart_item.cart_id | UUID (FK) | |
| cart_item.product_id | UUID (FK) | |
| cart_item.qty | integer | |

### 3.6 Tabel `orders` & `order_items`
| Field | Tipe | Keterangan |
|---|---|---|
| order.id | UUID (PK) | |
| order.invoice_no | varchar (unique) | mis. "JP-20260724-001" |
| order.user_id | UUID (FK) | |
| order.address_id | UUID (FK) | |
| order.shipping_method | enum(`reguler`,`express`) | |
| order.shipping_cost | decimal | |
| order.payment_method | enum(`transfer_bank`,`cod`) | |
| order.status | enum(`menunggu_pembayaran`,`diproses`,`dikirim`,`selesai`,`dibatalkan`) | |
| order.courier_name | varchar (nullable) | |
| order.resi_no | varchar (nullable) | |
| order.subtotal | decimal | |
| order.total | decimal | |
| order.created_at | timestamp | |
| order_item.id | UUID (PK) | |
| order_item.order_id | UUID (FK) | |
| order_item.product_id | UUID (FK) | |
| order_item.qty | integer | |
| order_item.price_at_purchase | decimal | snapshot harga saat transaksi |

### 3.7 Tabel `addresses`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| user_id | UUID (FK) | |
| recipient_name | varchar | |
| phone | varchar | |
| full_address | text | |
| is_default | boolean | |

### 3.8 Tabel `reviews`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| product_id | UUID (FK, nullable) | null jika review keseluruhan toko |
| user_id | UUID (FK) | |
| rating | integer (1-5) | |
| comment | text | |
| photos | json/array varchar | |
| created_at | timestamp | |

### 3.9 Tabel `wishlists`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| user_id | UUID (FK) | |
| product_id | UUID (FK) | |
| created_at | timestamp | |

### 3.10 Tabel `gallery_photos`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| image_url | varchar | |
| tag | enum(`workshop`,`produk`,`customer`) | |
| related_product_id | UUID (FK, nullable) | cross-link ke produk |

### 3.11 Tabel `faqs`
| Field | Tipe | Keterangan |
|---|---|---|
| id | UUID (PK) | |
| question | varchar | |
| answer | text | |
| sort_order | integer | |

## 4. Relasi Utama (ERD Ringkas)
```
users 1───N addresses
users 1───N orders
users 1───N wishlists
users 1───1 carts
carts 1───N cart_items ───N products
orders 1───N order_items ───N products
products N───1 categories
products 1───N product_images
products 1───N reviews
```


---

## 1. Home (Beranda) 🟢


### 1. Tujuan Halaman
Halaman utama aplikasi Jati Prime Furniture. Berfungsi sebagai pintu masuk pengguna, menampilkan value proposition brand, kategori unggulan, dan produk terbaik untuk mendorong eksplorasi lebih lanjut.

### 2. Struktur Layout (top → bottom)

### 2.1 Top Navigation Bar
- **Ikon Hamburger (☰)** — kiri, membuka side drawer menu (opsional, untuk navigasi tambahan: About, Kebijakan, Bahasa, dll).
- **Logo "JATI PRIME Furniture"** — center, tap → reload/scroll to top Home.
- **Ikon Keranjang (🛒)** — kanan, menampilkan badge jumlah item jika keranjang tidak kosong. Tap → navigasi ke halaman **5. Keranjang**.

### 2.2 Hero Banner
- Gambar full-width ruang tamu dengan sofa jati.
- **Headline**: "Furniture berkualitas dari kayu jati pilihan untuk melengkapi rumah impian Anda."
- **Sub-headline**: "Desain elegan, kokoh, dan dibuat dengan penuh ketelitian."
- **CTA Button**: "Jelajahi Koleksi →" (primary button, warna hijau tua/olive #5C6B4A atau senada tema) → navigasi ke halaman **3. Produk** (menampilkan semua produk).

### 2.3 Trust Badges (Icon Row)
4 kolom ikon + label kecil di bawah hero:
- Kayu Pilihan
- Desain Eksklusif
- Garansi Produk
- Pengiriman Aman

Setiap badge bersifat informational (non-tappable) atau tap → scroll ke section relevan di halaman Tentang Kami.

### 2.4 Section "Koleksi Terbaik"
- **Header row**: judul "Koleksi Terbaik" (kiri) + link teks "Lihat Semua" (kanan) → navigasi ke halaman **3. Produk**.
- **Product Card (horizontal scroll atau grid 2 kolom)**:
  - Gambar produk (rasio 1:1 rounded corner)
  - Nama produk (contoh: "Meja Makan Jati Solid")
  - Harga format Rupiah (contoh: "Rp 14.800.000")
  - Tap card → navigasi ke **4. Detail Produk** dengan product ID terkait.

### 2.5 Bottom Navigation Bar (Sticky)
5 item ikon + label, aktif state highlight (warna gelap/olive):
1. **Beranda** (aktif di halaman ini)
2. **Kategori** → halaman 2
3. **Favorit** → halaman Wishlist
4. **Chat** → deep link ke WhatsApp / live chat
5. **Akun** → halaman 12 (jika sudah login) / halaman Login (jika belum)

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Hero Banner | Image + Text overlay | Bisa carousel jika ada >1 promo |
| CTA Button | Primary Button | Warna solid, radius besar (pill/rounded) |
| Product Card | Card | Shadow tipis, radius 12px |
| Bottom Nav | Fixed Navigation | Selalu terlihat, 5 slot maksimal |

### 4. State & Kondisi
- **Loading**: skeleton loader pada hero & product card saat data belum siap.
- **Empty koleksi terbaik**: tampilkan pesan "Belum ada produk unggulan" + ilustrasi.
- **Keranjang badge**: muncul hanya jika count > 0.
- **Logged out**: ikon Akun mengarahkan ke Login/Register, bukan langsung ke profil.

### 5. Interaksi & Navigasi
```
Home
 ├─ Tap Logo/Hamburger → Side Menu
 ├─ Tap CTA "Jelajahi Koleksi" → Produk (semua)
 ├─ Tap Icon Keranjang → Keranjang
 ├─ Tap "Lihat Semua" → Produk
 ├─ Tap Product Card → Detail Produk
 └─ Bottom Nav → Kategori / Favorit / Chat / Akun
```

### 6. Pertimbangan UX
- Hero headline harus singkat (maks 2 baris) agar tidak memakan viewport di layar kecil.
- Product card sebaiknya menampilkan rating (⭐) jika tersedia untuk meningkatkan kepercayaan, konsisten dengan halaman Produk.
- Gunakan lazy-loading gambar untuk performa scroll yang mulus.
- Warna brand: earth-tone (cream background #F7F3EC, olive/dark green accent, coklat kayu untuk foto produk) — konsisten di seluruh halaman.

### 7. Responsive Notes
- Mobile-first (viewport ~375–430px).
- Grid produk: 2 kolom di mobile, bisa 3–4 kolom di tablet/desktop breakpoint.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Top Navigation Bar
| Properti | Nilai |
|---|---|
| Height | 56px |
| Padding horizontal | 16px (`space/container`) |
| Icon size | 24×24px |
| Logo height | 28px |
| Background | `color/body-bg`, sticky, shadow saat scroll >20px |

### Hero Banner
| Properti | Nilai |
|---|---|
| Height | 420px (mobile) |
| Padding top | 24px |
| Padding bottom | 24px |
| Border radius | 16px (`radius/lg`), inset margin 16px kiri-kanan |
| Background | Image, `object-fit: cover` |
| Overlay | Linear-gradient `rgba(0,0,0,0) 40% → rgba(0,0,0,0.45) 100%` (bawah lebih gelap agar teks terbaca) |
| Headline font | `type/display` — Playfair Display 28px/700 (disesuaikan turun dari 32px agar muat 2 baris di mobile) |
| Sub-headline | `type/body` 14px/400, warna `#FFFFFF`, max-width 90% |
| Gap headline → subtitle | 8px |
| Gap subtitle → button | 16px |
| Button height | 48px |
| Button radius | `radius/pill` (999px) |
| Button padding horizontal | 24px |

### Trust Badge Row
| Properti | Nilai |
|---|---|
| Kolom | 4, equal width |
| Icon size | 28×28px |
| Gap icon → label | 4px (`space/xs`) |
| Label font | `type/body-sm`, center align |
| Padding vertical section | 24px |

### Section "Koleksi Terbaik"
| Properti | Nilai |
|---|---|
| Section margin top | 32px (`space/xl`) |
| Header row padding | 16px horizontal |
| "Lihat Semua" font | `type/body-sm`, warna `color/primary`, underline on press |
| Grid | 2 kolom, gap 16px (`space/md`) |

### Product Card (lihat spesifikasi detail di `produk.md` — komponen ini reuse)

### Bottom Navigation Bar
| Properti | Nilai |
|---|---|
| Height | 64px + safe-area-inset-bottom |
| Icon size | 22×22px |
| Label font | `type/nav-label` |
| Active color | `color/primary` |
| Inactive color | `color/text-secondary` |
| Shadow | `shadow/nav` |

### 9. Design Token yang Digunakan
`color/primary`, `color/body-bg`, `color/text-primary`, `color/text-secondary`, `type/display`, `type/body`, `type/body-sm`, `radius/lg`, `radius/pill`, `space/container`, `space/md`, `space/xl`, `shadow/nav`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Hero Banner | Fade + slight scale-in (0.98→1) | 800ms (`motion/hero`) | ease-out | Saat halaman dimuat |
| Trust Badge Row | Stagger fade-up (delay 80ms per item) | 300ms per item | ease-out | Setelah hero selesai fade |
| Product Card | Translate-Y(-4px) + shadow naik ke `shadow/lg` | 150ms (`motion/fast`) | ease-out | On press/hover |
| Navbar | Background blur + shadow muncul | 200ms | linear | Scroll > 80px |
| Skeleton Loader | Shimmer looping | 1200ms (`motion/skeleton`) | ease-in-out | Saat data loading |

### 11. Acceptance Criteria
- [ ] Hero banner tampil dengan headline, sub-headline, dan tombol CTA yang tap-able.
- [ ] Tombol "Jelajahi Koleksi" mengarahkan ke halaman Produk (semua produk, tanpa filter).
- [ ] 4 trust badge tampil rata dan tidak terpotong pada layar 360px–430px.
- [ ] Section "Koleksi Terbaik" menampilkan minimal 2 produk, maksimal sesuai API (`limit` default 6).
- [ ] Cumulative Layout Shift (CLS) < 0.1 — gunakan `width`/`height` attribute pada semua gambar.
- [ ] Semua gambar (hero, produk) menggunakan lazy loading kecuali hero (priority load / eager, karena above-the-fold).
- [ ] Bottom navigation dapat diakses via keyboard/tab order dan screen reader (aria-label per ikon).
- [ ] Badge jumlah item keranjang update real-time tanpa reload halaman.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Koneksi internet lambat/putus | Fetch `/api/home` gagal/timeout | Tampilkan state error dengan tombol "Coba Lagi" (retry), skeleton dihilangkan |
| Data hero kosong (CMS belum diisi) | `hero[]` kosong dari API | Sembunyikan section hero, tampilkan langsung ke Trust Badge |
| Koleksi Terbaik kosong | `best_sellers[]` kosong | Tampilkan pesan "Belum ada produk unggulan" + ilustrasi, sembunyikan link "Lihat Semua" |
| Gambar produk gagal dimuat | `onerror` event pada `<img>` | Tampilkan placeholder abu-abu dengan ikon gambar (broken image icon) |
| User belum login, tap ikon Akun | Klik bottom nav "Akun" | Redirect ke halaman Login, bukan error |

### 13. Developer Notes
- Gunakan `<picture>` + `srcset` untuk hero banner agar resolusi gambar menyesuaikan device (mobile/tablet/desktop), format prioritas: AVIF → WebP → JPEG fallback.
- Hero banner sebaiknya di-*priority load* (bukan lazy) karena berada di viewport pertama (above-the-fold) — pengaruh langsung ke LCP (Largest Contentful Paint).
- Product card di section "Koleksi Terbaik" me-reuse komponen yang sama persis dengan grid di halaman Produk — jangan duplikasi komponen, cukup import/reuse dengan props berbeda (`variant="compact"` jika perlu).
- Cache response `/api/home` di client (stale-while-revalidate, TTL ± 5 menit) agar transisi antar tab tidak terasa lambat saat kembali ke Home.
- Pastikan tombol CTA "Jelajahi Koleksi" menggunakan elemen `<a>`/`<Link>` asli (bukan `onClick` div) untuk SEO & aksesibilitas.

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-HOME-001 | Hero banner harus tampil dengan headline, sub-headline, dan CTA yang tap-able |
| REQ-HOME-002 | CTA "Jelajahi Koleksi" harus mengarahkan ke halaman Produk (semua produk) |
| REQ-HOME-003 | Section "Koleksi Terbaik" harus menampilkan minimal 2 produk dari API |
| REQ-HOME-004 | Ikon Keranjang harus menampilkan badge jumlah item real-time |
| REQ-HOME-005 | Bottom Navigation harus dapat diakses via keyboard & screen reader |

### 15. Dependency

```
Home
 └─ bergantung pada
     ├─ API /api/home (Hero, Category preview, Best Sellers)
     └─ tidak bergantung pada modul lain (entry point aplikasi)

Home ──diperlukan oleh──► Kategori, Produk, Keranjang, Akun (via navigasi Bottom Nav)
```

### 16. Component Relationship

```
Hero Banner (Card variant: Promo/Banner)
   ↓
Button (CTA "Jelajahi Koleksi")
   ↓
Product Card (reuse dari 00-component-library.md)
   ↓
Bottom Navigation (shared, semua halaman)
```


---

## 2. Kategori 🟢


### 1. Tujuan Halaman
Menampilkan seluruh kategori produk furniture agar pengguna dapat menjelajah berdasarkan tipe barang yang dicari, serta menawarkan layanan custom furniture.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon panah kembali / hamburger (kiri)
- Logo "JATI PRIME" (center)
- Ikon Keranjang (kanan) dengan badge

### 2.2 Judul Halaman
- Teks besar "Kategori" (heading, left-aligned)

### 2.3 Grid Kategori (3 kolom)
Kartu kategori berupa: ikon/gambar representatif + label nama kategori di bawahnya. Kartu berbentuk rounded square dengan background lembut.

Daftar kategori (3 baris x 3 kolom = 9 item):
1. Sofa
2. Meja Makan
3. Kursi
4. Tempat Tidur
5. Lemari
6. Rak TV
7. Meja Kerja
8. Buffet / Credenza
9. Aksesori

Tap kartu kategori → navigasi ke **3. Produk** dengan filter kategori otomatis ter-apply (query param `?category=sofa` dsb).

### 2.4 Banner "Custom Furniture"
- Card besar full-width di bagian bawah grid.
- Gambar kursi custom + teks "Custom Furniture — Sesuai Keinginan Anda"
- **Tombol "Konsultasi Gratis"** (button outline/pill) → membuka chat WhatsApp/form konsultasi custom order.

### 2.5 Bottom Navigation Bar
Sama seperti halaman Home, dengan **Kategori** dalam state aktif.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Category Card | Icon Card | Ikon line-art/flat, ukuran seragam, 3 per baris |
| Custom Banner | Promo Card | Aspect ratio landscape, CTA menonjol |

### 4. State & Kondisi
- **Kategori kosong produk**: tetap tampil di grid (kategori adalah struktur tetap), namun saat diklik dan hasil kosong, tampilkan empty state di halaman Produk.
- **Loading**: skeleton grid 9 kotak abu-abu saat data kategori dimuat dari API.

### 5. Interaksi & Navigasi
```
Kategori
 ├─ Tap salah satu kartu kategori → Produk (filtered by kategori)
 ├─ Tap "Konsultasi Gratis" → Chat WhatsApp / Form Custom Order
 └─ Bottom Nav → Beranda / Favorit / Chat / Akun
```

### 6. Pertimbangan UX
- Ikon kategori harus mudah dikenali walau kecil (gunakan gaya konsisten: outline, ketebalan garis sama).
- Grid 3 kolom dipilih agar nama kategori panjang (misalnya "Buffet / Credenza") tetap terbaca tanpa terpotong — gunakan font size lebih kecil (10–11px) atau line-wrap 2 baris.
- Banner custom furniture ditempatkan di bawah agar tidak mengganggu scan cepat kategori utama, namun tetap terlihat tanpa scroll berlebihan.

### 7. Responsive Notes
- Grid dapat berubah menjadi 4–6 kolom pada layar tablet/desktop.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Category Card
| Properti | Nilai |
|---|---|
| Grid | 3 kolom, gap 12px |
| Card size | 108×108px (square, responsive terhadap lebar layar) |
| Border radius | 16px (`radius/lg`) |
| Background | `color/surface` |
| Icon/Image size | 40×40px, center |
| Label font | `type/body-sm`, 2 baris max, text-align center |
| Padding internal | 12px |
| Shadow | `shadow/sm` |

### Custom Furniture Banner
| Properti | Nilai |
|---|---|
| Height | 140px |
| Border radius | 16px |
| Padding internal | 20px |
| Judul font | `type/h3`, warna putih |
| Tombol "Konsultasi Gratis" | height 40px, radius `radius/pill`, border 1.5px putih, background transparan |

### 9. Design Token yang Digunakan
`color/surface`, `color/text-primary`, `type/h1`, `type/body-sm`, `radius/lg`, `radius/pill`, `space/md`, `shadow/sm`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Category Card | Scale-down (0.96) | 100ms (`motion/fast`) | ease-out | On press (tap feedback) |
| Grid | Stagger fade-up per baris (delay 60ms) | 250ms | ease-out | Saat halaman dimuat |
| Custom Banner | Slight zoom background image (1.0→1.05) | 4000ms loop | ease-in-out | Idle/ambient (opsional) |

### 11. Acceptance Criteria
- [ ] 9 kartu kategori tampil rapi 3 kolom tanpa nama kategori terpotong.
- [ ] Tap kartu kategori mengarahkan ke Produk dengan filter kategori ter-apply (`?category={slug}`).
- [ ] Tombol "Konsultasi Gratis" membuka WhatsApp dengan pesan pre-filled ("Halo, saya ingin konsultasi custom furniture").
- [ ] Semua ikon kategori memiliki `alt` text sesuai nama kategori.
- [ ] CLS < 0.1, gambar ikon kategori menggunakan dimensi tetap.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Kategori tanpa produk | Tap kategori dengan `product_count = 0` | Tetap navigasi ke Produk, tampilkan empty state "Belum ada produk di kategori ini" |
| Gagal load data kategori | API error | Skeleton grid diganti pesan error + tombol retry |
| WhatsApp tidak terpasang | Tap "Konsultasi Gratis" | Fallback membuka WhatsApp Web via browser |

### 13. Developer Notes
- Ikon kategori disimpan sebagai SVG (bukan raster) agar tajam di semua kepadatan layar (1x/2x/3x) dan ukuran file kecil.
- Slug kategori harus URL-safe (lowercase, dash-separated) agar konsisten dipakai sebagai query param di halaman Produk.
- Urutan tampil kategori mengikuti field `sort_order` dari `/api/categories` — jangan hardcode urutan di frontend.

### 14. Priority & Requirement ID

| Priority | **High** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-KATEGORI-001 | Grid 9 kategori harus tampil rapi 3 kolom tanpa nama terpotong |
| REQ-KATEGORI-002 | Tap kartu kategori harus mengarahkan ke Produk dengan filter ter-apply |
| REQ-KATEGORI-003 | Tombol "Konsultasi Gratis" harus membuka WhatsApp dengan pesan pre-filled |

### 15. Dependency

```
Kategori
 └─ bergantung pada
     ├─ API /api/categories
     └─ Home (sebagai entry navigasi utama)

Kategori ──diperlukan oleh──► Produk (query param filter kategori)
```

### 16. Component Relationship

```
Category Card (grid icon)
   ↓
Button (Konsultasi Gratis → WhatsApp deep link)
   ↓
Produk (halaman tujuan filter)
```


---

## 3. Produk (Listing / PLP) 🟢


### 1. Tujuan Halaman
Menampilkan daftar produk (semua atau hasil filter kategori/pencarian) dalam bentuk grid, dilengkapi search bar dan filter cepat.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon hamburger/back (kiri)
- Logo "JATI PRIME" (center)
- Ikon Keranjang + badge (kanan)

### 2.2 Search Bar
- Input field dengan placeholder "Cari produk.." + ikon kaca pembesar (kiri input).
- **Ikon Filter** (kanan search bar, bentuk slider/funnel) → membuka bottom sheet filter lanjutan (rentang harga, material, ukuran, rating).

### 2.3 Filter Tab (Chip/Pill horizontal scroll)
Opsi tab filter cepat:
- Semua (default aktif)
- Terbaru
- Terlaris
- Harga tertinggi

Tab aktif memiliki background solid (dark olive), tab non-aktif outline/transparent.

### 2.4 Grid Produk (2 kolom)
Setiap **Product Card** berisi:
- Gambar produk (rasio persegi, rounded corner)
- **Ikon Love/Wishlist** (pojok kanan atas gambar, toggle favorit)
- Nama produk (contoh: "Meja Makan Jati Solid")
- Harga (contoh: "Rp 14.800.000")
- Rating bintang + jumlah ulasan (contoh: "⭐4.9 (73)")

Tap card (di luar ikon love) → navigasi ke **4. Detail Produk**.
Tap ikon love → toggle add/remove ke Wishlist (dengan micro-animation, tidak pindah halaman).

### 2.5 Bottom Navigation Bar
State aktif: **Produk** (ikon "Kategori" pada bottom nav global, namun konteksnya listing produk — perlu konsistensi label; lihat catatan UX di bawah).

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Search Input | Text Field | Autocomplete opsional, debounce 300ms |
| Filter Chip | Toggle Pill | Single-select untuk sort, multi bisa untuk filter lanjutan |
| Product Card | Card | Sama seperti Home tapi + wishlist icon & rating |

### 4. State & Kondisi
- **Hasil kosong** (misal pencarian tidak ditemukan): tampilkan ilustrasi + teks "Produk tidak ditemukan" + tombol "Reset Filter".
- **Loading**: skeleton card grid saat fetch data / ganti filter.
- **Infinite scroll / pagination**: load more produk saat mencapai akhir list, tampilkan spinner kecil.
- **Wishlist toggle**: perlu login; jika belum login, tampilkan prompt login saat tap ikon love.

### 5. Interaksi & Navigasi
```
Produk
 ├─ Input Search → filter hasil real-time
 ├─ Tap ikon Filter → Bottom Sheet Filter Lanjutan
 ├─ Tap Tab (Semua/Terbaru/Terlaris/Harga tertinggi) → reorder grid
 ├─ Tap ikon Love pada card → toggle Wishlist
 ├─ Tap Card → Detail Produk
 └─ Bottom Nav → Beranda / Kategori / Favorit / Chat / Akun
```

### 6. Pertimbangan UX
- Konsistensi bottom navigation: pada mockup, label "Produk" di bottom nav menggantikan "Kategori" saat berada di halaman ini — sebaiknya bottom nav global tetap 5 item tetap (Beranda, Kategori, Favorit, Chat, Akun) dan halaman Produk diakses sebagai sub-halaman dari Kategori/Beranda tanpa mengubah label nav, untuk menghindari kebingungan struktur informasi.
- Rating & jumlah ulasan penting untuk kepercayaan (social proof) — pastikan selalu tampil walau produk baru (tampilkan "Produk Baru" jika belum ada ulasan).
- Search bar sebaiknya sticky saat scroll agar user bisa mengubah pencarian kapan saja.

### 7. Responsive Notes
- Grid 2 kolom (mobile) → 3–4 kolom (tablet/desktop).
- Filter chip row scrollable horizontal jika opsi bertambah.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Search Bar
| Properti | Nilai |
|---|---|
| Height | 44px |
| Radius | 12px (`radius/md`) |
| Border | 1px solid `color/border` |
| Icon search size | 20×20px, warna `color/text-secondary` |
| Icon filter (funnel) | 20×20px, area tap 40×40px |
| Padding horizontal | 12px |
| Font | `type/body`, 14px |

### Filter Tab (Chip)
| Properti | Nilai |
|---|---|
| Height | 36px |
| Padding horizontal | 16px |
| Radius | `radius/pill` |
| Gap antar chip | 8px |
| Active background | `color/primary` |
| Active text | `#FFFFFF` |
| Inactive border | 1px `color/border` |
| Inactive text | `color/text-secondary` |

### Product Card
| Properti | Nilai |
|---|---|
| Grid | 2 kolom, gap 16px |
| Image aspect ratio | 4:3 |
| Image radius | 18px (`radius/xl`) |
| Card radius | 16px (`radius/lg`) |
| Card shadow | `shadow/md` |
| Wishlist icon posisi | top-right, offset 8px dari tepi gambar |
| Wishlist icon size | 20×20px, background `rgba(255,255,255,0.9)`, circle 32×32px |
| Padding internal (di bawah gambar) | 12px |
| Nama produk font | `type/h3`, max 2 baris (ellipsis) |
| Harga font | `type/price` — 18px/700 |
| Rating font | `type/body-sm`, warna `color/rating-star` untuk ikon bintang |
| Gap nama → harga | 4px |
| Gap harga → rating | 4px |
| Hover/press state | `translateY(-4px)`, shadow naik ke `shadow/lg` |

### 9. Design Token yang Digunakan
`color/primary`, `color/border`, `color/rating-star`, `type/h3`, `type/price`, `type/body-sm`, `radius/md`, `radius/lg`, `radius/xl`, `radius/pill`, `space/md`, `shadow/md`, `shadow/lg`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Product Card | TranslateY(-4px) + shadow → `shadow/lg` | 200ms (`motion/base`) | ease-out | Hover (desktop) / press (mobile) |
| Wishlist Icon | Scale bounce (1→1.3→1) + fill color transition | 300ms | spring/ease-out | Tap toggle wishlist |
| Filter Tab switch | Background color crossfade | 150ms (`motion/fast`) | ease-in-out | Tap tab |
| Grid reorder (ganti sort) | Fade-out lama → fade-in baru | 250ms | ease-in-out | Ganti filter Terbaru/Terlaris/dst |
| Search result update | Debounce 300ms lalu fade content | 200ms | ease-out | Input search berubah |

### 11. Acceptance Criteria
- [ ] Input search memfilter hasil dengan debounce 300ms (tidak fetch di setiap keystroke).
- [ ] Tab filter (Semua/Terbaru/Terlaris/Harga tertinggi) bersifat single-select dan reorder grid tanpa reload halaman.
- [ ] Toggle wishlist berhasil optimistic-update di UI sebelum konfirmasi API selesai (dengan rollback jika gagal).
- [ ] Grid tidak menyebabkan layout shift saat gambar dimuat (gunakan aspect-ratio CSS).
- [ ] Infinite scroll/pagination memuat data tambahan sebelum user mencapai betul-betul akhir list (prefetch di 80% scroll).
- [ ] Semua interaktif elemen (chip, card, ikon wishlist) memiliki target tap minimal 40×40px (aksesibilitas).

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Pencarian tanpa hasil | Query search tidak match produk | Ilustrasi "Produk tidak ditemukan" + tombol "Reset Filter" |
| Tap wishlist saat belum login | Guest user tap ikon love | Tampilkan bottom sheet/modal prompt login, jangan langsung redirect paksa |
| Stok produk = 0 tapi tetap tampil di grid | `stock = 0` | Tambahkan badge kecil "Stok Habis" di pojok gambar card |
| Filter kategori dari URL tidak valid | Slug kategori di query param tidak ada di DB | Tampilkan semua produk + toast "Kategori tidak ditemukan" |
| Gagal load halaman berikutnya (pagination) | API error saat scroll | Tampilkan tombol "Muat Ulang" di bagian bawah grid, bukan infinite spinner |

### 13. Developer Notes
- Reuse komponen `ProductCard` yang sama persis di halaman Home (Koleksi Terbaik), Wishlist, dan Pesanan Saya — buat sebagai shared component dengan props (`showWishlistIcon`, `showRating`, dll).
- State filter & search sebaiknya disimpan di URL query params (bukan hanya local state) agar shareable/bisa di-refresh tanpa kehilangan filter.
- Gunakan `IntersectionObserver` untuk infinite scroll, bukan scroll-event listener manual (performa lebih baik).

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-PRODUK-001 | Search bar harus memfilter hasil dengan debounce 300ms |
| REQ-PRODUK-002 | Filter tab (Semua/Terbaru/Terlaris/Harga tertinggi) harus single-select dan reorder grid |
| REQ-PRODUK-003 | Toggle wishlist harus optimistic-update dengan rollback jika API gagal |
| REQ-PRODUK-004 | Grid harus mendukung infinite scroll/pagination |
| REQ-PRODUK-005 | Filter kategori dari halaman Kategori harus ter-apply otomatis via query param |

### 15. Dependency

```
Produk
 └─ bergantung pada
     ├─ API /api/products
     ├─ Kategori (query param filter, opsional)
     └─ Login (hanya untuk aksi wishlist; browsing tetap bisa tanpa login)

Produk ──diperlukan oleh──► Detail Produk, Wishlist, Keranjang (alur pembelian)
```

### 16. Component Relationship

```
Input (Search) + Tabs (Filter Chip)
   ↓
Product Card
   ↓        ↘
Wishlist    Detail Produk
```


---

## 4. Detail Produk (PDP) 🟢


### 1. Tujuan Halaman
Menyajikan informasi lengkap satu produk untuk meyakinkan pengguna melakukan pembelian: galeri foto, harga, spesifikasi, dan opsi kontak/beli.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri, kembali ke halaman sebelumnya (Produk/Home).
- Ikon **Share** (kanan) — membagikan link produk.
- Ikon **Wishlist/Love** (kanan, sebelah share) — toggle simpan produk.
- Tidak ada logo brand di header ini (fokus pada konten produk).

### 2.2 Galeri Gambar Produk
- Gambar utama full-width (hero image ruang makan dengan meja jati).
- **Thumbnail strip** (4 gambar kecil di bawah gambar utama) — tap thumbnail mengganti gambar utama (swipeable gallery, indikator dot opsional).

### 2.3 Info Utama Produk
- **Nama produk**: "Meja Makan Jati Solid" (heading besar, bold).
- **Harga**: "Rp 14.800.000" (warna aksen/hijau tua, font besar).
- **Rating & Stok**: baris berisi "⭐4.9 (73 ulasan)" (kiri) dan "Stok Tersedia" (kanan, badge hijau).
- Deskripsi singkat produk: "Meja premium berbahan kayu jati solid Grade A, finishing natural oil yang menonjolkan serat kayu asli. Kuat, kokoh, dan tahan hingga puluhan tahun."

### 2.4 Spesifikasi (Table/List)
| Field | Contoh Value |
|---|---|
| Material | Kayu Jati Solid Grade A |
| Finishing | Natural Oil |
| Ukuran | 200 x 100 x 75 cm |
| Kapasitas | 6 Kursi |
| Garansi | 5 Tahun |

Ditampilkan sebagai list 2 kolom (label kiri, value kanan, rata kanan).

### 2.5 Action Bar (Sticky Bottom)
Dua tombol berdampingan:
- **"Chat WhatsApp"** (outline button, ikon WhatsApp) → deep link ke WhatsApp dengan pesan pre-filled berisi nama produk.
- **"Tambah ke Keranjang"** (solid/primary button) → menambahkan item ke keranjang (quantity default 1) + toast konfirmasi "Ditambahkan ke keranjang" + update badge cart di header lain.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Image Gallery | Carousel + Thumbnail | Swipe & tap-to-select |
| Spec Table | Key-Value List | Alternating row jika perlu keterbacaan |
| Sticky CTA Bar | Fixed Bottom Bar | Selalu terlihat walau scroll deskripsi panjang |

### 4. State & Kondisi
- **Stok habis**: badge berubah menjadi "Stok Habis" (merah/abu), tombol "Tambah ke Keranjang" disabled, diganti opsi "Beri Tahu Saya" atau tetap bisa "Chat WhatsApp" untuk pre-order.
- **Sudah di wishlist**: ikon love terisi penuh (filled) berbeda dari outline.
- **Varian produk** (jika ada, misal pilihan warna/ukuran): tampilkan selector chip di atas action bar (tidak terlihat di mockup, opsional untuk pengembangan lanjut).
- **Loading galeri**: skeleton/placeholder blur saat gambar dimuat.

### 5. Interaksi & Navigasi
```
Detail Produk
 ├─ Tap Back → Halaman sebelumnya
 ├─ Tap Share → Native share sheet
 ├─ Tap Love → Toggle wishlist
 ├─ Tap Thumbnail → Ganti gambar utama
 ├─ Tap "Chat WhatsApp" → Buka WhatsApp
 └─ Tap "Tambah ke Keranjang" → Update cart + toast + tetap di halaman ini
```

### 6. Pertimbangan UX
- Sticky action bar penting agar CTA utama selalu dapat diakses tanpa scroll ke bawah, khususnya untuk produk dengan deskripsi/spesifikasi panjang.
- Rating & jumlah ulasan sebaiknya tappable → scroll/jump ke section ulasan produk (jika ditambahkan di bawah spesifikasi pada pengembangan lanjut, terhubung dengan data di halaman Testimoni).
- Harga harus format Rupiah konsisten (titik sebagai pemisah ribuan) di seluruh aplikasi.
- Tombol "Tambah ke Keranjang" sebaiknya berubah menjadi "Lihat Keranjang" sesaat setelah ditekan (micro-interaction) sebagai feedback visual instan.

### 7. Responsive Notes
- Galeri gambar tetap 1 kolom penuh di semua breakpoint mobile; pada tablet/desktop bisa side-by-side dengan info produk (gallery kiri, info kanan).

### 8. Spesifikasi Komponen (Detail Ukuran)

### Image Gallery
| Properti | Nilai |
|---|---|
| Main image height | 320px |
| Main image radius | 0 (full-bleed di dalam container, atau 16px jika inset) |
| Thumbnail size | 56×56px |
| Thumbnail radius | 10px |
| Thumbnail gap | 8px |
| Active thumbnail border | 2px solid `color/primary` |

### Info Utama
| Properti | Nilai |
|---|---|
| Nama produk font | `type/h1` — 22px/700 |
| Harga font | `type/price` — 20px/700, warna `color/primary-dark` |
| Rating + Stok row | `type/body-sm`, gap 8px antara rating dan badge stok |
| Badge "Stok Tersedia" | height 24px, radius `radius/sm`, background `rgba(62,142,90,0.12)`, text `color/success` |
| Deskripsi font | `type/body`, line-height 20px, margin top 12px |

### Spesifikasi Table
| Properti | Nilai |
|---|---|
| Row height | 40px |
| Label font | `type/body-sm`, warna `color/text-secondary` |
| Value font | `type/body`, warna `color/text-primary`, align right |
| Divider | 1px `color/border` antar baris |

### Sticky Action Bar
| Properti | Nilai |
|---|---|
| Height | 64px + safe-area-bottom |
| Padding horizontal | 16px |
| Gap antar tombol | 12px |
| "Chat WhatsApp" button | height 48px, radius `radius/md`, border 1.5px `color/success`, background transparan |
| "Tambah ke Keranjang" button | height 48px, radius `radius/md`, background `color/primary`, flex-grow 1.5 (lebih lebar dari tombol WhatsApp) |

### 9. Design Token yang Digunakan
`color/primary`, `color/primary-dark`, `color/success`, `color/border`, `type/h1`, `type/price`, `type/body`, `type/body-sm`, `radius/md`, `radius/sm`, `space/md`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Main Image | Crossfade antar gambar | 250ms (`motion/base`) | ease-in-out | Tap thumbnail / swipe |
| Wishlist Icon (header) | Scale bounce + fill | 300ms | spring | Tap toggle |
| "Tambah ke Keranjang" | Label berubah sementara jadi "Ditambahkan ✓" | 1500ms hold lalu revert | ease-out | Tap tombol |
| Toast konfirmasi | Slide-up dari bawah + fade | 300ms in / 200ms out | ease-out | Setelah add to cart sukses |
| Spesifikasi table | Tidak ada animasi (static render) | – | – | – |

### 11. Acceptance Criteria
- [ ] Galeri gambar dapat di-swipe dan thumbnail dapat di-tap untuk mengganti gambar utama.
- [ ] Harga, rating, dan status stok tampil akurat sesuai data API (tidak ada hardcode).
- [ ] Tombol "Tambah ke Keranjang" disabled otomatis jika `stock = 0`.
- [ ] Tombol "Chat WhatsApp" membuka deep link `wa.me` dengan pesan pre-filled berisi nama & link produk.
- [ ] Sticky action bar tetap terlihat penuh saat user scroll deskripsi/spesifikasi yang panjang.
- [ ] Toggle wishlist tersimpan dan konsisten saat user kembali ke halaman ini dari halaman lain.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Stok habis | `stock = 0` | Badge berubah jadi "Stok Habis" (merah), tombol "Tambah ke Keranjang" → nonaktif & label berubah jadi "Beri Tahu Saya" |
| Gambar produk gagal load | Network error pada image | Placeholder abu-abu dengan ikon gambar |
| Produk dihapus/nonaktif saat user sedang membuka halaman | API return 404 saat refresh | Tampilkan halaman "Produk tidak ditemukan" + tombol kembali ke Produk |
| Tap "Tambah ke Keranjang" berkali-kali cepat (double tap) | Multiple rapid taps | Debounce/disable tombol sesaat (300ms) untuk mencegah duplikasi item di keranjang |
| Internet terputus saat tap "Tambah ke Keranjang" | Request gagal | Toast error "Gagal menambahkan ke keranjang, coba lagi" + tombol tetap dalam state semula (bukan optimistic-locked) |

### 13. Developer Notes
- Gunakan `<picture>`+`srcset` untuk gambar produk, resolusi minimal 2x untuk layar retina.
- Preload gambar thumbnail berikutnya (`rel="preload"`) untuk transisi galeri yang mulus.
- Data spesifikasi (`material`, `finishing`, `dimension`, dst.) sebaiknya di-render dari array key-value dinamis dari API, bukan field terpisah hardcoded, agar admin bisa menambah field spesifikasi baru tanpa perlu update aplikasi (lihat `admin/produk.md`).
- Deep link WhatsApp format: `https://wa.me/62812xxxxxxx?text={encoded_message}` — pastikan nomor & template pesan disimpan sebagai config, bukan hardcode di komponen.

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-DETAIL-001 | Galeri gambar harus dapat di-swipe & thumbnail dapat mengganti gambar utama |
| REQ-DETAIL-002 | Tombol "Tambah ke Keranjang" harus disabled otomatis jika stok = 0 |
| REQ-DETAIL-003 | Tombol "Chat WhatsApp" harus membuka deep link dengan pesan pre-filled nama produk |
| REQ-DETAIL-004 | Sticky action bar harus tetap terlihat penuh saat scroll deskripsi panjang |
| REQ-DETAIL-005 | Toggle wishlist harus konsisten across halaman (Produk, Wishlist, Detail Produk) |

### 15. Dependency

```
Detail Produk
 └─ bergantung pada
     ├─ API /api/products/{slug}
     ├─ Produk / Home / Galeri (sebagai sumber navigasi masuk)
     └─ Login (untuk aksi wishlist, opsional untuk lihat detail)

Detail Produk ──diperlukan oleh──► Keranjang (add to cart), Wishlist, WhatsApp (chat)
```

### 16. Component Relationship

```
Gallery (image + thumbnail)
   ↓
Badge (Stok Tersedia/Habis)
   ↓
Button Primary (Tambah ke Keranjang) + Button Outline (Chat WhatsApp)
   ↓
Toast ("Ditambahkan ke keranjang")
   ↓
Keranjang
```


---

## 5. Keranjang (Cart) 🟢


### 1. Tujuan Halaman
Menampilkan seluruh item yang telah ditambahkan pengguna, memungkinkan pengubahan jumlah/penghapusan item, dan menampilkan ringkasan belanja sebelum checkout.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Keranjang" — center.
- Link teks **"Ubah"** — kanan, mengaktifkan mode edit (misalnya menampilkan checkbox untuk hapus banyak item sekaligus / mode select-multiple).

### 2.2 List Item Keranjang
Setiap **Cart Item Row** terdiri dari:
- Thumbnail gambar produk (kiri, kotak kecil rounded).
- Nama produk (contoh: "Meja Makan Jati Solid").
- Harga satuan (contoh: "Rp 14.800.000").
- **Stepper Quantity**: tombol `−` | angka jumlah | tombol `+`.
- **Ikon Hapus (tempat sampah)** — di ujung kanan row, tap → hapus item dari keranjang (idealnya dengan konfirmasi atau undo snackbar).

Contoh 3 item pada mockup:
1. Meja Makan Jati Solid — Rp 14.800.000 — qty 1
2. Kursi Makan Jati Anyaman — Rp 2.250.000 — qty 6
3. Lemari 3 Pintu Jati — Rp 10.500.000 — qty 1

### 2.3 Ringkasan Belanja (Card)
- Judul "Ringkasan Belanja".
- Baris "Subtotal (3 Item)" → nilai total harga barang.
- Baris "Ongkir" → "Gratis" (atau nominal jika dikenakan biaya, tergantung metode pengiriman yang dipilih nanti).
- Garis pemisah (divider).
- Baris "**Total**" (bold, font lebih besar) → nilai akhir.

### 2.4 CTA Button
- **"Checkout (3 Item)"** — tombol solid full-width di bagian bawah, menampilkan jumlah item dinamis sesuai isi keranjang → navigasi ke **6. Checkout**.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Cart Item Row | List Item | Swipe-to-delete opsional selain ikon trash |
| Quantity Stepper | Stepper Input | Min 1, max sesuai stok tersedia |
| Summary Card | Info Card | Update real-time saat quantity berubah |
| Checkout Button | Primary Button | Disabled jika keranjang kosong |

### 4. State & Kondisi
- **Keranjang kosong**: tampilkan ilustrasi keranjang kosong + teks "Keranjang Anda masih kosong" + tombol "Mulai Belanja" → navigasi ke Produk.
- **Quantity mencapai batas stok**: tombol `+` disabled, tampilkan pesan kecil "Stok tersisa: X".
- **Quantity = 1 lalu tap `−`**: memicu konfirmasi hapus item (dialog "Hapus produk ini dari keranjang?").
- **Update harga real-time**: subtotal & total harus recalculate instan setiap stepper berubah tanpa reload halaman.

### 5. Interaksi & Navigasi
```
Keranjang
 ├─ Tap Back → Halaman sebelumnya
 ├─ Tap "Ubah" → Mode edit (multi-select hapus)
 ├─ Tap +/− pada item → Update qty & subtotal
 ├─ Tap ikon Hapus → Hapus item (dengan konfirmasi/undo)
 └─ Tap "Checkout (N Item)" → Checkout
```

### 6. Pertimbangan UX
- Nomor pada tombol Checkout ("3 Item") membantu pengguna mengonfirmasi jumlah barang sebelum lanjut — pastikan angka ini selalu sinkron dengan jumlah unique product (bukan total quantity) atau dijelaskan konsisten di seluruh app.
- Sediakan micro-feedback (misal snackbar "Item dihapus, Undo") agar penghapusan tidak terasa destruktif/tanpa jalan kembali.
- Ongkir "Gratis" perlu syarat jelas (misal minimum belanja) — bisa ditambahkan info tooltip agar user tidak kaget saat metode pengiriman lain berbayar di halaman Checkout.

### 7. Responsive Notes
- List item tetap 1 kolom pada semua breakpoint; ringkasan belanja bisa menjadi sidebar sticky pada layar lebar (tablet/desktop).

### 8. Spesifikasi Komponen (Detail Ukuran)

### Cart Item Row
| Properti | Nilai |
|---|---|
| Row height | 88px |
| Thumbnail size | 64×64px |
| Thumbnail radius | 12px |
| Gap thumbnail → info | 12px |
| Nama produk font | `type/h3` |
| Harga satuan font | `type/body-sm`, `color/text-secondary` |
| Divider antar row | 1px `color/border` |

### Quantity Stepper
| Properti | Nilai |
|---|---|
| Button size | 28×28px (bulat/rounded 8px) |
| Border | 1px `color/border` |
| Angka qty font | `type/h3`, min-width 24px center align |
| Gap antar elemen stepper | 8px |

### Ikon Hapus
| Properti | Nilai |
|---|---|
| Size | 20×20px |
| Area tap | 40×40px |
| Warna | `color/danger` |

### Ringkasan Belanja Card
| Properti | Nilai |
|---|---|
| Padding internal | 16px |
| Radius | `radius/lg` |
| Background | `color/surface` |
| Shadow | `shadow/sm` |
| Row height | 32px |
| Divider sebelum Total | 1px `color/border`, margin vertical 8px |
| Total font | `type/h2`, warna `color/primary-dark` |

### CTA Checkout Button
| Properti | Nilai |
|---|---|
| Height | 52px |
| Radius | `radius/md` |
| Background | `color/primary` |
| Font | `type/button` |

### 9. Design Token yang Digunakan
`color/danger`, `color/border`, `color/surface`, `color/primary`, `color/primary-dark`, `type/h2`, `type/h3`, `type/body-sm`, `type/button`, `radius/md`, `radius/lg`, `shadow/sm`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Quantity Stepper | Angka fade+slide saat berubah | 150ms (`motion/fast`) | ease-out | Tap +/− |
| Item dihapus | Slide-out ke kiri + collapse height | 250ms (`motion/base`) | ease-in | Tap ikon hapus / konfirmasi |
| Subtotal/Total | Number count-up/down animation | 200ms | ease-out | Qty berubah |
| Snackbar "Item dihapus, Undo" | Slide-up dari bawah | 300ms in | ease-out | Setelah hapus item |

### 11. Acceptance Criteria
- [ ] Perubahan quantity langsung memperbarui subtotal & total tanpa reload halaman.
- [ ] Tombol `+` disabled otomatis saat qty mencapai batas stok tersedia.
- [ ] Tap `−` saat qty = 1 memicu dialog konfirmasi hapus, bukan otomatis menghapus.
- [ ] Tombol "Checkout (N Item)" menampilkan jumlah item unik sesuai isi keranjang secara real-time.
- [ ] Keranjang kosong menampilkan empty state dengan CTA "Mulai Belanja".
- [ ] Data keranjang tersinkron dengan akun user (bukan hanya local storage) agar konsisten lintas device.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Produk di keranjang kehabisan stok saat checkout | Stok berubah jadi 0 di server sejak ditambahkan | Tampilkan badge "Stok Habis" pada item terkait, disable tombol Checkout sampai item dihapus/diupdate |
| Qty melebihi stok tersedia | User set qty > stock | Auto-clamp ke stok maksimum + toast info "Stok tersisa: X" |
| Gagal update qty (network error) | API request gagal | Rollback qty ke nilai sebelumnya + toast error |
| Harga produk berubah sejak ditambahkan ke keranjang | Admin update harga di tengah proses user | Tampilkan notice kecil "Harga telah diperbarui" pada item terkait sebelum checkout |

### 13. Developer Notes
- Simpan cart di backend (linked ke `user_id`) sejak user login, bukan hanya di local storage, agar keranjang tidak hilang saat ganti device/browser.
- Gunakan optimistic UI update untuk stepper qty (langsung update UI, sync ke API di background) demi responsivitas, dengan rollback jika API gagal.
- Snackbar undo hapus item sebaiknya punya window waktu ±5 detik sebelum benar-benar dihapus permanen dari backend.

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-CART-001 | Perubahan qty harus update subtotal & total real-time tanpa reload |
| REQ-CART-002 | Tombol `+` harus disabled saat qty mencapai batas stok |
| REQ-CART-003 | Qty = 1 lalu tap `−` harus memicu dialog konfirmasi hapus |
| REQ-CART-004 | Data keranjang harus tersinkron ke akun (backend), bukan hanya local storage |
| REQ-CART-005 | Tombol Checkout harus menampilkan jumlah item real-time |

### 15. Dependency

```
Keranjang
 └─ bergantung pada
     ├─ API /api/cart
     ├─ Detail Produk (sumber item ditambahkan)
     └─ Login (cart tersinkron ke akun user)

Keranjang ──diperlukan oleh──► Checkout
```

### 16. Component Relationship

```
Product Card (compact/list variant) + Stepper (Quantity)
   ↓
Icon Button (Hapus) → Snackbar ("Item dihapus, Undo")
   ↓
Card (Ringkasan Belanja)
   ↓
Button Primary (Checkout)
```


---

## 6. Checkout 🟢


### 1. Tujuan Halaman
Memandu pengguna menyelesaikan pesanan melalui alur bertahap: alamat pengiriman, metode pengiriman, metode pembayaran, hingga konfirmasi pesanan.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Checkout" — center.

### 2.2 Section 1 — Alamat Pengiriman
- Nomor urut **"1"** dalam lingkaran (step indicator).
- Judul "Alamat Pengiriman" + link **"Ubah"** (kanan) → membuka halaman/modal pilih atau edit alamat.
- **Card Alamat**:
  - Nama penerima: "Bagus Sajiwo" (bold)
  - Alamat lengkap: "Jl. Raya Jepara Bangsri KM. 7, Jepara, Jawa Tengah 59453"
  - Nomor telepon: "0812-3456-7890"

### 2.3 Section 2 — Metode Pengiriman
- Nomor urut **"2"**.
- Judul "Metode Pengiriman".
- **Radio Option List**:
  - "Pengiriman Reguler" — "Gratis • Estimasi 3-5 hari" (default terpilih, radio filled hijau).
  - "Pengiriman Express" — "Rp 150.000 • Estimasi 1-2 hari" (radio kosong).

### 2.4 Section 3 — Metode Pembayaran
- Nomor urut **"3"**.
- Judul "Metode Pembayaran".
- **Radio Option List**:
  - "Transfer Bank" — sub-info "BCA • 1234567890 a.n. Jati Prime" (terpilih default).
  - "COD (Bayar di Tempat)" — opsi kedua.

### 2.5 Section 4 — Ringkasan Pesanan
- Nomor urut **"4"**.
- Judul "Ringkasan Pesanan".
- Baris "Subtotal (3 Item)" → nominal.
- Baris "Ongkir" → "Gratis" (mengikuti metode pengiriman terpilih; update jika pilih Express).
- Baris "**Total**" (bold) → nominal akhir (menyesuaikan biaya ongkir dinamis).

### 2.6 CTA Button
- **"Buat Pesanan"** — tombol solid full-width di bagian bawah → memproses order, menampilkan konfirmasi (halaman sukses / status pesanan), lalu mengarah ke halaman **Pesanan** atau **Tracking**.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Step Number Badge | Circular Badge | Angka 1–4, warna solid dark olive |
| Address Card | Info Card | Border tipis, background putih/cream |
| Radio Option | Radio Button List | Single-select per section |
| Order Summary | Summary Block | Update dinamis sesuai pilihan step 2 |

### 4. State & Kondisi
- **Belum ada alamat tersimpan**: tampilkan CTA "Tambah Alamat" menggantikan card alamat, mengarah ke form alamat baru.
- **Pilih Pengiriman Express**: Ongkir & Total pada Ringkasan Pesanan otomatis ter-update (+Rp150.000).
- **Pilih COD**: kemungkinan menampilkan info tambahan/catatan biaya COD jika ada.
- **Validasi sebelum submit**: tombol "Buat Pesanan" harus memastikan alamat, metode pengiriman, dan metode pembayaran semuanya terisi/terpilih; jika tidak, tampilkan pesan error inline.
- **Loading saat submit**: tombol berubah menjadi spinner/loading state, disabled agar tidak double-submit.

### 5. Interaksi & Navigasi
```
Checkout
 ├─ Tap Back → Keranjang
 ├─ Tap "Ubah" alamat → Pilih/Edit Alamat
 ├─ Pilih radio Metode Pengiriman → Update Ringkasan Pesanan
 ├─ Pilih radio Metode Pembayaran → (tampilkan info tambahan jika perlu)
 └─ Tap "Buat Pesanan" → Proses order → Halaman Konfirmasi/Pesanan
```

### 6. Pertimbangan UX
- Struktur numbered-step (1–4) sangat membantu pengguna memahami progres tanpa perlu multi-page wizard — pertahankan pola single-page checkout ini untuk mengurangi drop-off.
- Info bank/no. rekening pada "Transfer Bank" sebaiknya memiliki tombol "Salin" untuk memudahkan copy nomor rekening saat transfer manual.
- Pastikan opsi COD menjelaskan area jangkauan (tidak semua kota bisa COD) agar tidak terjadi kegagalan pesanan di kemudian hari.
- Total akhir harus selalu ter-highlight jelas (font besar, warna kontras) sebagai elemen paling penting sebelum submit.

### 7. Responsive Notes
- Layout single-column tetap dipertahankan di semua breakpoint karena sifatnya sequential/step-by-step; pada desktop dapat ditambahkan sidebar ringkasan sticky di kanan.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Step Number Badge
| Properti | Nilai |
|---|---|
| Size | 24×24px, circle |
| Background | `color/primary` |
| Font | `type/body-sm`, 700, warna putih |
| Gap ke judul section | 8px |

### Address Card
| Properti | Nilai |
|---|---|
| Padding | 16px |
| Radius | `radius/lg` |
| Border | 1px `color/border` |
| Nama penerima font | `type/h3` |
| Alamat & telepon font | `type/body`, `color/text-secondary` |

### Radio Option (Metode Pengiriman/Pembayaran)
| Properti | Nilai |
|---|---|
| Row height | 56px |
| Radio button size | 20×20px |
| Radio selected color | `color/primary` |
| Label font | `type/h3` |
| Sub-info font | `type/body-sm`, `color/text-secondary` |
| Divider antar opsi | 1px `color/border` |

### Ringkasan Pesanan
Sama seperti Ringkasan Belanja pada `keranjang.md` (reuse komponen).

### CTA "Buat Pesanan"
| Properti | Nilai |
|---|---|
| Height | 52px |
| Radius | `radius/md` |
| Background | `color/primary` |
| Font | `type/button` |
| Sticky | Ya, menempel di bawah viewport |

### 9. Design Token yang Digunakan
`color/primary`, `color/border`, `type/h3`, `type/body`, `type/body-sm`, `type/button`, `radius/lg`, `radius/md`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Radio Option | Dot fill scale-in | 150ms (`motion/fast`) | ease-out | Tap pilih opsi |
| Ringkasan Pesanan (Ongkir/Total) | Number update fade+recalculate | 200ms | ease-out | Ganti metode pengiriman |
| "Buat Pesanan" saat submit | Button → loading spinner state | instan swap | – | Tap submit |
| Transisi ke halaman konfirmasi | Slide-in dari kanan / fade | 400ms (`motion/slow`) | ease-in-out | Setelah order berhasil dibuat |

### 11. Acceptance Criteria
- [ ] Ongkir & Total pada Ringkasan Pesanan otomatis update saat metode pengiriman diganti (Reguler ↔ Express).
- [ ] Tombol "Buat Pesanan" disabled sampai alamat, metode pengiriman, dan metode pembayaran semuanya terisi/terpilih.
- [ ] Tombol submit menampilkan loading state dan mencegah double-submit (idempotent request / disable saat proses).
- [ ] Setelah order berhasil, keranjang otomatis dikosongkan dan user diarahkan ke halaman konfirmasi/Pesanan Saya.
- [ ] Info rekening bank dapat di-copy dengan satu tap (copy-to-clipboard + toast "Nomor rekening disalin").

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Belum ada alamat tersimpan | User baru/alamat dihapus | Tampilkan CTA "Tambah Alamat" menggantikan Address Card |
| Submit order saat salah satu item di keranjang sudah habis stok | Race condition stok | Tampilkan error spesifik item mana yang bermasalah sebelum order dibuat, jangan proses partial order |
| Gagal koneksi saat submit | Network timeout | Tampilkan retry dialog, order belum tercatat (pastikan backend idempotent agar tidak double-order jika user retry) |
| Pilih COD di luar area jangkauan | Alamat di luar zona COD | Disable opsi COD dengan keterangan "Tidak tersedia untuk wilayah ini" |

### 13. Developer Notes
- Gunakan idempotency key pada request `POST /api/orders` untuk mencegah order duplikat jika user menekan submit berkali-kali akibat koneksi lambat.
- Validasi ulang stok & harga di sisi backend saat submit (jangan hanya percaya data dari client), untuk mencegah race condition antara halaman Keranjang dan Checkout.
- Nomor rekening & info pembayaran sebaiknya dikelola dari CMS/admin settings, bukan hardcode di frontend, agar mudah diubah tanpa deploy ulang.

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-CHECKOUT-001 | Ongkir & Total harus otomatis update saat metode pengiriman diganti |
| REQ-CHECKOUT-002 | Tombol "Buat Pesanan" harus disabled sampai alamat, pengiriman, dan pembayaran lengkap |
| REQ-CHECKOUT-003 | Submit harus mencegah double-order (idempotency key) |
| REQ-CHECKOUT-004 | Sistem harus validasi ulang stok & harga di backend saat submit (tidak percaya data client) |
| REQ-CHECKOUT-005 | Setelah order berhasil, keranjang harus otomatis dikosongkan |

### 15. Dependency

```
Checkout
 └─ bergantung pada
     ├─ Login (harus sudah masuk)
     ├─ Keranjang (harus ada minimal 1 item)
     ├─ Alamat (harus ada minimal 1 alamat tersimpan)
     └─ API /api/checkout/summary, /api/orders

Checkout ──diperlukan oleh──► Pesanan Saya, Tracking, Admin — Manajemen Pesanan
```

> Catatan: sesuai contoh dari tim — **jika Login belum selesai dibangun, Checkout belum bisa di-develop/di-test penuh** karena bergantung pada sesi user & data alamat tersimpan.

### 16. Component Relationship

```
Stepper (Progress: 1-Alamat → 2-Pengiriman → 3-Pembayaran → 4-Ringkasan)
   ↓
Card (Address Card) + Radio Group (Metode Pengiriman/Pembayaran)
   ↓
Card (Ringkasan Pesanan)
   ↓
Button Primary (Buat Pesanan) → Modal/halaman konfirmasi
   ↓
Pesanan Saya
```


---

## 7. Tentang Kami 🟢


### 1. Tujuan Halaman
Membangun kepercayaan (trust) dan brand story dengan menjelaskan latar belakang, visi, dan misi Jati Prime Furniture.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Logo "JATI PRIME" — center.

### 2.2 Judul & Deskripsi
- Heading "Tentang Kami".
- Paragraf deskripsi: "Jati Prime adalah brand furniture premium yang berbasis di Jepara, Indonesia. Kami berkomitmen menghadirkan produk furniture berkualitas tinggi dengan desain elegan dan material terbaik."

### 2.3 Gambar Workshop
- Foto pengrajin sedang bekerja di workshop kayu (membangun kredibilitas craftsmanship).

### 2.4 Icon Feature Row (3 kolom)
- Material Terbaik
- Pengerjaan Detail
- Desain Eksklusif

Setiap kolom: ikon kecil + label teks di bawahnya, non-interaktif (informational).

### 2.5 Visi Kami
- Sub-heading "Visi Kami".
- Paragraf: "Menjadi brand furniture jati premium terpercaya di Indonesia dan mancanegara."

### 2.6 Misi Kami
- Sub-heading "Misi Kami".
- **Bullet list**:
  - Menggunakan material kayu jati berkualitas terbaik
  - Menghadirkan desain yang elegan & timeless
  - Memberikan pelayanan terbaik untuk pelanggan

### 2.7 Bottom Navigation Bar
Standar 5 item (Beranda, Kategori, Favorit, Chat, Akun) — tidak ada item yang aktif secara eksplisit karena halaman ini diakses dari luar tab utama (misalnya via side menu atau footer link).

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Content Image | Static Image | Full-width, rounded corner |
| Icon Feature | Icon + Label | 3-column grid, ikon monoline |
| Bullet List | List | Misi Kami, gaya checklist/point |

### 4. State & Kondisi
Halaman ini sepenuhnya statis (content page), tidak memerlukan state loading/error kompleks selain standar loading gambar.

### 5. Interaksi & Navigasi
```
Tentang Kami
 ├─ Tap Back → Halaman sebelumnya
 └─ Bottom Nav → Beranda / Kategori / Favorit / Chat / Akun
```

### 6. Pertimbangan UX
- Tambahkan CTA di akhir halaman (misalnya "Lihat Koleksi Kami" atau "Kunjungi Galeri") agar halaman ini tidak menjadi dead-end dan tetap mendorong konversi.
- Foto workshop autentik (bukan stok generik) sangat penting untuk memperkuat cerita "dibuat dengan penuh ketelitian" yang disebut di Home.
- Konsistensi tone teks (Bahasa Indonesia formal-ramah) harus sama dengan halaman lain seperti FAQ dan Kontak.

### 7. Responsive Notes
- Single column di mobile; pada desktop, konten teks & gambar dapat disusun 2 kolom side-by-side.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Content Image
| Properti | Nilai |
|---|---|
| Height | 200px |
| Radius | `radius/lg` |
| Margin vertical | 16px |

### Icon Feature Row
| Properti | Nilai |
|---|---|
| Kolom | 3, equal width |
| Icon size | 32×32px |
| Gap icon → label | 8px |
| Label font | `type/body-sm`, center |

### Bullet List (Misi Kami)
| Properti | Nilai |
|---|---|
| Bullet icon | Check (✓) 16×16px, warna `color/primary` |
| Gap bullet → teks | 8px |
| Line height | 20px |
| Gap antar item | 12px |

### 9. Design Token yang Digunakan
`color/primary`, `color/text-primary`, `type/h1`, `type/h2`, `type/body`, `type/body-sm`, `radius/lg`, `space/lg`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Section (fade-in saat scroll) | Fade + translateY(20px→0) | 300ms (`motion/base`) | ease-out | Scroll into view |

### 11. Acceptance Criteria
- [ ] Seluruh konten (teks, gambar) ter-render dari CMS/API, bukan hardcode di kode aplikasi.
- [ ] Gambar workshop menggunakan lazy loading (bukan above-the-fold).
- [ ] Halaman tetap dapat diakses & terbaca baik tanpa JavaScript (SSR/prerendered) untuk kebutuhan SEO dasar.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Konten CMS kosong/belum diisi | Field kosong dari API | Sembunyikan section terkait, jangan tampilkan area kosong |
| Gambar gagal dimuat | Network error | Placeholder abu-abu |

### 13. Developer Notes
- Halaman ini idealnya dikelola via CMS (misal field `about_description`, `vision`, `mission[]`) agar tim non-teknis bisa update konten tanpa deploy ulang aplikasi.
- Pertimbangkan menambah CTA di akhir halaman ("Lihat Koleksi Kami") sebagai low-effort improvement untuk konversi.

### 14. Priority & Requirement ID

| Priority | **Low** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-ABOUT-001 | Seluruh konten harus ter-render dari CMS/API, bukan hardcode |
| REQ-ABOUT-002 | Gambar workshop harus lazy-loaded (bukan above-the-fold) |

### 15. Dependency

```
Tentang Kami
 └─ bergantung pada
     └─ API/CMS /api/pages/about (tidak bergantung modul transaksional)

Tidak ada halaman lain yang bergantung langsung pada Tentang Kami.
```

### 16. Component Relationship

```
Card (Content Image)
   ↓
Icon Feature Row (informational, non-interaktif)
   ↓
Bullet List (Misi Kami)
```


---

## 8. Galeri 🟢


### 1. Tujuan Halaman
Menampilkan dokumentasi visual proses produksi, hasil produk, dan foto pelanggan untuk memperkuat kredibilitas dan inspirasi desain interior.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Galeri" — center.

### 2.2 Filter Tab (Chip horizontal)
- Semua (default aktif, background solid gelap)
- Workshop
- Produk
- Customer

### 2.3 Grid Foto (Masonry / Grid 2 kolom, tinggi bervariasi)
Menampilkan campuran foto:
- Proses workshop (pengrajin mengukir/mengamplas kayu)
- Foto produk jadi (ruang makan, ruang tamu)
- Foto rumah pelanggan (customer showcase)

Tap foto → membuka **lightbox/fullscreen viewer** dengan swipe kiri/kanan antar foto dan tombol close (×).

### 2.4 Bottom Navigation Bar
Standar 5 item.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Filter Chip | Toggle Pill | Single-select, filter grid berdasarkan tag |
| Masonry Grid | Image Grid | Variable height agar natural/tidak monoton |
| Lightbox Viewer | Modal Overlay | Fullscreen, swipeable, pinch-to-zoom |

### 4. State & Kondisi
- **Tab kosong** (misal belum ada foto "Customer"): tampilkan pesan "Belum ada foto pada kategori ini".
- **Loading**: skeleton grid abu-abu saat gambar dimuat, progressive image loading (blur-up).
- **Infinite scroll**: load lebih banyak foto saat mencapai bagian bawah grid.

### 5. Interaksi & Navigasi
```
Galeri
 ├─ Tap Back → Halaman sebelumnya
 ├─ Tap Filter Tab → Update grid sesuai kategori
 ├─ Tap Foto → Buka Lightbox Fullscreen
 └─ Bottom Nav → Beranda / Kategori / Favorit / Chat / Akun
```

### 6. Pertimbangan UX
- Kategori "Produk" pada galeri sebaiknya tappable ke Detail Produk terkait jika foto tersebut terhubung dengan item yang dijual (cross-link meningkatkan konversi).
- Foto "Customer" (UGC - user generated content) memperkuat social proof; pertimbangkan menambahkan nama/lokasi singkat pelanggan di caption saat foto dibuka di lightbox.
- Pastikan rasio gambar tidak terdistorsi (gunakan object-fit: cover dengan cropping cerdas).

### 7. Responsive Notes
- Grid 2 kolom (mobile) → 3–4 kolom masonry (tablet/desktop).

### 8. Spesifikasi Komponen (Detail Ukuran)

### Filter Tab
Sama seperti Filter Tab pada `produk.md` (reuse komponen).

### Masonry Grid
| Properti | Nilai |
|---|---|
| Kolom | 2 (mobile) |
| Gap | 8px |
| Radius per foto | 12px |
| Height | variable (masonry, min 140px – max 260px) |

### Lightbox Viewer
| Properti | Nilai |
|---|---|
| Background | `rgba(0,0,0,0.9)` |
| Close icon size | 28×28px, posisi top-right, margin 16px |
| Transisi buka | Fade + scale (0.9→1) |

### 9. Design Token yang Digunakan
`color/overlay`, `radius/sm`, `radius/md`, `space/sm`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Grid foto | Fade-in progresif per gambar (blur-up) | 300ms | ease-out | Saat gambar selesai load |
| Lightbox | Fade + scale-in | 250ms (`motion/base`) | ease-out | Tap foto |
| Swipe antar foto lightbox | Slide horizontal | 300ms | ease-in-out | Swipe kiri/kanan |

### 11. Acceptance Criteria
- [ ] Tap foto membuka lightbox fullscreen dengan navigasi swipe antar foto.
- [ ] Filter tab memfilter grid tanpa reload halaman penuh.
- [ ] Infinite scroll memuat foto tambahan sebelum user mencapai akhir grid.
- [ ] Gambar menggunakan progressive/blur-up loading agar tidak terasa "pop-in" mendadak.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Tab kosong (mis. belum ada foto Customer) | `photos[] = []` untuk tag tertentu | Pesan "Belum ada foto pada kategori ini" |
| Gambar gagal dimuat di lightbox | Network error | Placeholder + tombol "Muat Ulang" pada foto tersebut |

### 13. Developer Notes
- Simpan gambar dalam beberapa resolusi (thumbnail untuk grid, full-size untuk lightbox) agar tidak memuat file besar hanya untuk tampilan kecil.
- Foto dengan tag "Produk" yang punya `related_product_id` sebaiknya menampilkan tombol kecil "Lihat Produk" di lightbox untuk cross-link ke Detail Produk.

### 14. Priority & Requirement ID

| Priority | **Medium** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-GALLERY-001 | Tap foto harus membuka lightbox fullscreen dengan navigasi swipe |
| REQ-GALLERY-002 | Filter tab harus memfilter grid tanpa reload halaman penuh |
| REQ-GALLERY-003 | Grid harus mendukung infinite scroll |

### 15. Dependency

```
Galeri
 └─ bergantung pada
     ├─ API /api/gallery
     └─ Produk (opsional, untuk cross-link foto bertag "Produk")

Tidak ada halaman lain yang bergantung langsung pada Galeri.
```

### 16. Component Relationship

```
Tabs (Filter Chip: Semua/Workshop/Produk/Customer)
   ↓
Gallery (Masonry Grid)
   ↓
Gallery (Lightbox Viewer) ──opsional──► Detail Produk (jika foto bertag Produk)
```


---

## 9. Testimoni 🟢


### 1. Tujuan Halaman
Menampilkan ringkasan rating keseluruhan brand serta daftar ulasan individual dari pelanggan untuk membangun kepercayaan calon pembeli.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Testimoni" — center.

### 2.2 Rating Summary Card
- Angka besar **"4.9"** (rating rata-rata keseluruhan).
- Bintang (⭐⭐⭐⭐⭐) di bawah angka.
- Teks "Dari 327 Ulasan".
- **Bar chart breakdown** per bintang (5 → 1), menampilkan proporsi jumlah ulasan tiap rating:
  - 5 bintang: bar penuh, jumlah 290
  - 4 bintang: bar pendek, jumlah 31
  - 3 bintang: jumlah 4
  - 2 bintang: jumlah 1
  - 1 bintang: jumlah 1

### 2.3 List Ulasan (Review List)
Setiap **Review Item** terdiri dari:
- Avatar bulat (foto profil reviewer, placeholder jika tidak ada).
- Nama reviewer (contoh: "Andi Pratama").
- Rating bintang individual (⭐⭐⭐⭐⭐).
- Timestamp relatif (contoh: "2 hari yang lalu").
- Teks ulasan (contoh: "Kualitas produk sangat bagus, finishing rapi dan sesuai ekspektasi. Pengiriman cepat dan aman.").
- **Foto lampiran ulasan** (opsional, 2-3 thumbnail kecil produk yang diterima customer).

Contoh reviewer pada mockup: Andi Pratama, Siti Rahmawati, Budi Santoso — masing-masing dengan teks dan foto berbeda.

### 2.4 Bottom Navigation Bar
Standar 5 item.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Rating Summary | Stat Card | Angka besar + breakdown bar chart |
| Rating Bar | Horizontal Progress Bar | Proporsional terhadap total ulasan |
| Review Item | List Item | Avatar + teks + foto opsional |

### 4. State & Kondisi
- **Belum ada ulasan**: tampilkan empty state "Belum ada ulasan" (jarang terjadi karena brand sudah established, tapi perlu handle untuk produk baru).
- **Load more**: pagination/infinite scroll untuk memuat ulasan tambahan (total 327 ulasan tidak mungkin tampil sekaligus).
- **Foto ulasan tap** → buka lightbox fullscreen (serupa Galeri).

### 5. Interaksi & Navigasi
```
Testimoni
 ├─ Tap Back → Halaman sebelumnya
 ├─ Tap Foto ulasan → Lightbox fullscreen
 ├─ Scroll ke bawah → Load more ulasan
 └─ Bottom Nav → Beranda / Kategori / Favorit / Chat / Akun
```

### 6. Pertimbangan UX
- Tambahkan filter/sort ulasan (misal "Terbaru", "Rating Tertinggi", "Dengan Foto") untuk membantu user menemukan ulasan relevan dengan cepat, mengingat volume ulasan (327) cukup besar.
- Bar breakdown rating sebaiknya tappable → filter list ulasan sesuai jumlah bintang yang dipilih.
- Konsistensi rating produk individual di halaman Detail Produk (misal 4.9 dari 73 ulasan) dengan rating keseluruhan brand (4.9 dari 327 ulasan) — perlu dijelaskan bedanya di UI (produk spesifik vs keseluruhan toko) agar tidak membingungkan.

### 7. Responsive Notes
- List ulasan tetap single column; rating summary card dapat menjadi sticky sidebar di layar lebar.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Rating Summary Card
| Properti | Nilai |
|---|---|
| Angka besar (4.9) font | `type/display`, 36px/700 |
| Bintang icon size | 16×16px, warna `color/rating-star` |
| Bar chart height | 6px per bar, radius 4px |
| Bar chart fill color | `color/rating-star` |
| Bar chart track color | `color/border` |
| Gap antar baris bar | 8px |

### Review Item
| Properti | Nilai |
|---|---|
| Avatar size | 40×40px, circle |
| Nama font | `type/h3` |
| Timestamp font | `type/body-sm`, `color/text-secondary` |
| Comment font | `type/body`, margin top 8px |
| Foto lampiran size | 56×56px, radius 8px, gap 8px antar foto |
| Divider antar review | 1px `color/border`, margin vertical 16px |

### 9. Design Token yang Digunakan
`color/rating-star`, `color/border`, `color/text-secondary`, `type/display`, `type/h3`, `type/body`, `type/body-sm`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Bar chart breakdown | Fill width animate dari 0 → nilai aktual | 600ms | ease-out | Saat halaman/section dimuat |
| Review list | Fade-in + stagger per item | 200ms per item | ease-out | Scroll into view / load |
| Foto lampiran | Tap → buka lightbox (reuse dari Galeri) | 250ms | ease-out | Tap foto |

### 11. Acceptance Criteria
- [ ] Rating summary menampilkan angka rata-rata dan breakdown per bintang yang akurat sesuai data agregat.
- [ ] List ulasan mendukung pagination/infinite scroll (total 327 ulasan tidak dimuat sekaligus).
- [ ] Tap foto lampiran ulasan membuka lightbox fullscreen.
- [ ] Data rating produk individual (di Detail Produk) dan rating toko keseluruhan (di halaman ini) diambil dari sumber data yang jelas berbeda dan tidak tertukar.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Belum ada ulasan sama sekali | `total = 0` | Empty state "Belum ada ulasan" |
| Reviewer tanpa foto profil | `avatar_url = null` | Tampilkan avatar inisial nama sebagai fallback |
| Ulasan tanpa foto lampiran | `photos = []` | Sembunyikan row foto, tampilkan hanya teks |

### 13. Developer Notes
- Rating & breakdown sebaiknya dihitung/cache di backend (bukan dihitung ulang di client dari seluruh raw data) untuk performa, terutama dengan volume ratusan ulasan.
- Sediakan opsi sort/filter ulasan ("Terbaru", "Rating Tertinggi", "Dengan Foto") sebagai peningkatan lanjutan mengingat volume ulasan besar.

### 14. Priority & Requirement ID

| Priority | **Medium** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-TESTI-001 | Rating summary harus akurat sesuai data agregat backend |
| REQ-TESTI-002 | List ulasan harus mendukung pagination (327 ulasan tidak dimuat sekaligus) |
| REQ-TESTI-003 | Rating produk individual & rating toko keseluruhan tidak boleh tertukar sumber datanya |

### 15. Dependency

```
Testimoni
 └─ bergantung pada
     ├─ API /api/reviews
     └─ Pesanan Saya (ulasan idealnya dipicu setelah status "Selesai")

Testimoni ditampilkan juga (versi ringkas) di Detail Produk (rating_avg, review_count).
```

### 16. Component Relationship

```
Card (Rating Summary + Bar Chart)
   ↓
Avatar + Review Item List
   ↓
Gallery (Lightbox untuk foto lampiran ulasan)
```


---

## 10. FAQ 🟢


### 1. Tujuan Halaman
Menjawab pertanyaan umum pelanggan seputar produk, pemesanan, dan pengiriman dalam format accordion agar mudah dipindai (scannable).

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "FAQ" — center.

### 2.2 Accordion List
Setiap **FAQ Item** berupa baris berisi:
- Teks pertanyaan (kiri).
- Ikon chevron (`⌄` / panah bawah, kanan) — indikator expand/collapse.

Tap baris → expand menampilkan jawaban di bawah pertanyaan (accordion, hanya 1 atau beberapa item bisa terbuka bersamaan), ikon chevron berubah arah (`⌃`) saat terbuka.

Daftar pertanyaan pada mockup:
1. Apakah produk terbuat dari kayu jati asli?
2. Apakah bisa custom ukuran / desain?
3. Berapa lama proses pengerjaan furniture?
4. Bagaimana cara perawatan furniture jati?
5. Apakah ada garansi produk?
6. Bagaimana cara pemesanan?
7. Apakah bisa COD (Bayar di Tempat)?
8. Kirim ke luar kota / luar negeri bisa?
9. Bagaimana jika barang rusak saat pengiriman?

### 2.3 Bottom Navigation Bar
Standar 5 item.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Accordion Item | Expandable List Item | Divider antar item, animasi expand smooth (~200ms) |

### 4. State & Kondisi
- **Default**: semua item collapsed (jawaban tersembunyi).
- **Expanded**: satu atau lebih item terbuka menampilkan teks jawaban lengkap.
- **Loading**: jika FAQ diambil dari CMS/API, tampilkan skeleton list sebelum konten muncul.

### 5. Interaksi & Navigasi
```
FAQ
 ├─ Tap Back → Halaman sebelumnya
 ├─ Tap pertanyaan → Expand/Collapse jawaban
 └─ Bottom Nav → Beranda / Kategori / Favorit / Chat / Akun
```

### 6. Pertimbangan UX
- Tambahkan search bar kecil di atas list jika jumlah FAQ terus bertambah, agar user bisa mencari kata kunci tertentu.
- Setiap jawaban FAQ terkait pemesanan/pengiriman/COD sebaiknya berisi link cross-reference ke halaman terkait (misal jawaban "Bagaimana cara pemesanan?" bisa berisi link ke halaman Produk atau Checkout).
- Sertakan CTA di akhir list: "Masih ada pertanyaan? Hubungi Kami" → navigasi ke halaman **Kontak** — agar pengguna yang tidak menemukan jawaban tetap memiliki jalur bantuan.

### 7. Responsive Notes
- Accordion list tetap single column di semua breakpoint; lebar maksimum dibatasi (misal 600–700px) pada desktop agar teks jawaban tetap nyaman dibaca.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Accordion Item
| Properti | Nilai |
|---|---|
| Row height (collapsed) | 52px |
| Padding horizontal | 16px |
| Pertanyaan font | `type/h3` |
| Jawaban font | `type/body`, `color/text-secondary`, padding top 8px, padding bottom 16px |
| Chevron icon size | 20×20px |
| Divider antar item | 1px `color/border` |

### 9. Design Token yang Digunakan
`color/border`, `color/text-secondary`, `type/h3`, `type/body`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Accordion expand/collapse | Height auto-animate + chevron rotate 180° | 250ms (`motion/base`) | ease-in-out | Tap pertanyaan |

### 11. Acceptance Criteria
- [ ] Tap pertanyaan meng-expand jawaban dengan animasi smooth, tanpa layout jump mendadak.
- [ ] Lebih dari satu item dapat terbuka bersamaan (bukan strict single-open), kecuali ditentukan lain oleh tim produk.
- [ ] Konten FAQ dapat diakses via keyboard (Enter/Space untuk expand) dan screen reader (aria-expanded).

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Data FAQ kosong dari API | `faq[] = []` | Tampilkan pesan "Belum ada FAQ tersedia" + CTA ke Kontak Kami |
| Jawaban sangat panjang | Teks > beberapa paragraf | Accordion tetap scroll natural mengikuti tinggi konten, tidak dipotong |

### 13. Developer Notes
- Render FAQ dari data terstruktur (`/api/faq`) agar admin dapat menambah/mengubah pertanyaan tanpa deploy ulang.
- Tambahkan CTA di akhir list ("Masih ada pertanyaan? Hubungi Kami") yang link ke halaman Kontak.

### 14. Priority & Requirement ID

| Priority | **Low** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-FAQ-001 | Tap pertanyaan harus expand jawaban dengan animasi smooth |
| REQ-FAQ-002 | FAQ harus dapat diakses via keyboard & screen reader (aria-expanded) |

### 15. Dependency

```
FAQ
 └─ bergantung pada
     └─ API/CMS /api/faq (tidak bergantung modul lain)

FAQ ──terhubung ke──► Kontak Kami (CTA "Masih ada pertanyaan?")
```

### 16. Component Relationship

```
Accordion (list pertanyaan)
   ↓
Button Text (CTA ke Kontak Kami)
```


---

## 11. Kontak Kami 🟢


### 1. Tujuan Halaman
Memberikan berbagai kanal komunikasi resmi (WhatsApp, telepon, email, alamat, sosial media) dan lokasi toko fisik melalui peta.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Kontak Kami" — center.

### 2.2 Header Card
- Judul "Hubungi Kami".
- Sub-teks "Kami siap membantu Anda".

### 2.3 List Kontak (Icon + Info)
- **WhatsApp** (ikon hijau) — "0812-3456-7890" → tap membuka deep link WhatsApp.
- **Telepon** (ikon) — "(0291) 123456" → tap memicu dial-out (`tel:` link).
- **Email** (ikon) — "info@jatiprime.id" → tap membuka mail client (`mailto:` link).
- **Alamat** (ikon pin) — "Jl. Raya Jepara Bangsri KM. 7, Jepara, Jawa Tengah 59453" → tap bisa membuka aplikasi maps eksternal.

### 2.4 Sosial Media
- Judul kecil "Ikuti Kami".
- Row ikon: Instagram, Facebook, YouTube, dan satu ikon lagi (misal TikTok/Twitter) — masing-masing tap membuka profil sosial media terkait di browser/app.

### 2.5 Peta Lokasi
- Embed **Google Maps** menampilkan pin lokasi "Jati Prime Furniture Jepara".
- Tap peta → membuka aplikasi Maps eksternal dengan rute ke lokasi.

### 2.6 Bottom Navigation Bar
Standar 5 item.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Contact List Item | Icon + Text Row | Tappable, deep-link sesuai jenis kontak |
| Social Icon Row | Icon Button Group | Circular icon, warna brand konsisten |
| Map Embed | Static/Interactive Map | Bisa non-interaktif thumbnail yang membuka app maps saat tap |

### 4. State & Kondisi
- **Tidak ada aplikasi WhatsApp/Maps terpasang**: fallback ke browser (WhatsApp Web / Google Maps web).
- **Loading peta**: tampilkan placeholder abu-abu dengan ikon pin statis sebelum peta ter-load penuh.

### 5. Interaksi & Navigasi
```
Kontak Kami
 ├─ Tap Back → Halaman sebelumnya
 ├─ Tap WhatsApp → Buka WhatsApp chat
 ├─ Tap Telepon → Dial number
 ├─ Tap Email → Buka mail client
 ├─ Tap Alamat/Peta → Buka Maps eksternal
 ├─ Tap ikon Sosial Media → Buka profil terkait
 └─ Bottom Nav → Beranda / Kategori / Favorit / Chat / Akun
```

### 6. Pertimbangan UX
- Tambahkan jam operasional (misal "Senin–Sabtu, 08.00–17.00 WIB") di dekat header agar pelanggan tahu kapan waktu terbaik menghubungi.
- Ikon WhatsApp sebaiknya paling menonjol (primary color) karena kemungkinan menjadi kanal komunikasi utama, konsisten dengan tombol "Chat WhatsApp" di halaman Detail Produk.
- Pastikan alamat lengkap dapat di-copy dengan mudah (tap-to-copy) untuk kebutuhan pengiriman/kurir pihak ketiga.

### 7. Responsive Notes
- Single column di mobile; pada desktop, list kontak dan peta dapat disusun 2 kolom side-by-side.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Contact List Item
| Properti | Nilai |
|---|---|
| Row height | 56px |
| Icon size | 24×24px, warna sesuai channel (WhatsApp hijau, dsb) |
| Label font | `type/body-sm`, `color/text-secondary` |
| Value font | `type/h3` |
| Gap icon → teks | 12px |

### Social Icon Row
| Properti | Nilai |
|---|---|
| Icon size | 20×20px |
| Circle background | 36×36px, `color/surface` |
| Gap antar icon | 12px |

### Map Embed
| Properti | Nilai |
|---|---|
| Height | 180px |
| Radius | `radius/lg` |
| Pin icon size | 32×32px |

### 9. Design Token yang Digunakan
`color/success` (WhatsApp), `color/surface`, `color/text-secondary`, `type/h3`, `type/body-sm`, `radius/lg`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Contact List Item | Background highlight sekilas | 100ms | ease-out | Tap (tap feedback) |
| Map | Fade-in saat tile selesai load | 300ms | ease-out | Load selesai |

### 11. Acceptance Criteria
- [ ] Tap WhatsApp membuka deep link `wa.me` dengan fallback ke WhatsApp Web bila app tidak terpasang.
- [ ] Tap Telepon memicu native dialer (`tel:`).
- [ ] Tap Email membuka mail client (`mailto:`).
- [ ] Tap Alamat/Peta membuka aplikasi Maps eksternal dengan koordinat yang benar.
- [ ] Semua ikon sosial media mengarah ke URL profil yang valid dan terbuka di tab/app baru.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Peta gagal dimuat (no internet/API key issue) | Map tile error | Tampilkan placeholder statis alamat + pesan "Peta tidak dapat dimuat" |
| Device tanpa aplikasi Maps/WhatsApp | Tap link terkait | Fallback ke versi web (Google Maps web, WhatsApp Web) |

### 13. Developer Notes
- Data kontak (no. WA, telepon, email, alamat, koordinat) dikelola dari CMS/admin settings agar mudah diperbarui tanpa deploy ulang.
- Gunakan static map image sebagai fallback ringan (bukan full interactive map SDK) untuk performa loading yang lebih cepat di koneksi lambat.

### 14. Priority & Requirement ID

| Priority | **Medium** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-KONTAK-001 | Tap WhatsApp/Telepon/Email harus memicu deep link native yang sesuai |
| REQ-KONTAK-002 | Peta harus menampilkan pin lokasi toko yang akurat |

### 15. Dependency

```
Kontak Kami
 └─ bergantung pada
     └─ API/CMS /api/contact (tidak bergantung modul transaksional)

FAQ ──terhubung ke──► Kontak Kami
```

### 16. Component Relationship

```
List Item (Contact: WhatsApp/Telepon/Email/Alamat)
   ↓
Icon Button Row (Sosial Media)
   ↓
Map Embed
```


---

## 12. Akun Saya 🟢


### 1. Tujuan Halaman
Pusat pengelolaan profil pengguna dan akses cepat ke fitur-fitur terkait akun: pesanan, wishlist, alamat, pengaturan, dan bantuan.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Akun Saya" — center.

### 2.2 Profile Card
- Avatar foto profil (bulat, placeholder default jika belum upload).
- Nama pengguna: "Bagus Sajiwo" (bold, besar).
- Email: "bagussajiwo@example.com".
- Nomor telepon: "0812-3456-7890".
- Link teks **"Ubah Profil >"** — mengarah ke form edit profil.

### 2.3 Menu List
Daftar menu berbentuk list item dengan ikon kiri + label + chevron kanan (`>`):
1. **Pesanan Saya** → riwayat & status pesanan (halaman `pesanan.md`).
2. **Wishlist Saya** → daftar produk favorit (halaman `wishlist.md`).
3. **Alamat Saya** → kelola daftar alamat pengiriman.
4. **Pengaturan Akun** → ubah password, notifikasi, bahasa, dll.
5. **Bantuan** → mengarah ke FAQ / Kontak / pusat bantuan.
6. **Keluar** (Logout) — item terakhir, biasanya warna teks berbeda (merah/abu) untuk menandakan aksi destruktif/berbeda dari navigasi biasa.

### 2.4 Bottom Navigation Bar
State aktif: **Akun**.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Profile Card | Info Card | Avatar + info ringkas + edit link |
| Menu List Item | Navigation List | Icon + Label + Chevron, divider antar item |
| Logout Item | Destructive List Item | Konfirmasi dialog sebelum eksekusi |

### 4. State & Kondisi
- **Belum login**: halaman ini tidak dapat diakses; ikon "Akun" pada bottom nav mengarahkan ke halaman **Login** terlebih dahulu.
- **Tap "Keluar"**: munculkan dialog konfirmasi "Yakin ingin keluar?" dengan tombol "Batal" / "Keluar" sebelum benar-benar logout dan redirect ke Home/Login.
- **Avatar kosong**: tampilkan inisial nama sebagai placeholder (misal "BS") dengan background warna solid.

### 5. Interaksi & Navigasi
```
Akun Saya
 ├─ Tap Back → Halaman sebelumnya
 ├─ Tap "Ubah Profil" → Form Edit Profil
 ├─ Tap "Pesanan Saya" → Pesanan
 ├─ Tap "Wishlist Saya" → Wishlist
 ├─ Tap "Alamat Saya" → Kelola Alamat
 ├─ Tap "Pengaturan Akun" → Pengaturan
 ├─ Tap "Bantuan" → FAQ/Kontak
 ├─ Tap "Keluar" → Dialog konfirmasi → Logout
 └─ Bottom Nav → Beranda / Kategori / Favorit / Chat
```

### 6. Pertimbangan UX
- Tampilkan badge angka kecil pada "Pesanan Saya" jika ada pesanan dengan status baru/perlu tindakan (misal "1 pesanan baru dikirim").
- Konsistensi ikon antar menu list harus seragam (line-icon style, ukuran & ketebalan sama) sesuai gaya ikon di seluruh aplikasi.
- Pastikan info kontak (email/telepon) pada profile card tidak dapat diedit langsung dari sana — arahkan selalu melalui "Ubah Profil" untuk menjaga satu sumber kebenaran (single source of truth) form edit.

### 7. Responsive Notes
- Single column list di semua breakpoint; pada desktop bisa ditampilkan sebagai 2 kolom (sidebar menu kiri, konten kanan) mengikuti pola dashboard akun pada umumnya.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Profile Card
| Properti | Nilai |
|---|---|
| Avatar size | 64×64px, circle |
| Nama font | `type/h2` |
| Email/telepon font | `type/body-sm`, `color/text-secondary` |
| "Ubah Profil" link font | `type/body-sm`, `color/primary`, underline |
| Padding card | 16px |
| Radius | `radius/lg` |
| Shadow | `shadow/sm` |

### Menu List Item
| Properti | Nilai |
|---|---|
| Row height | 52px |
| Icon size | 22×22px |
| Label font | `type/h3` |
| Chevron size | 16×16px, `color/text-secondary` |
| Divider | 1px `color/border` |
| "Keluar" text color | `color/danger` |

### 9. Design Token yang Digunakan
`color/primary`, `color/danger`, `color/border`, `color/text-secondary`, `type/h2`, `type/h3`, `type/body-sm`, `radius/lg`, `shadow/sm`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Menu List Item | Background highlight sekilas | 100ms | ease-out | Tap (tap feedback) |
| Dialog konfirmasi logout | Fade + scale-in (0.95→1) | 200ms | ease-out | Tap "Keluar" |

### 11. Acceptance Criteria
- [ ] Tap "Keluar" memunculkan dialog konfirmasi sebelum benar-benar logout.
- [ ] Semua item menu menavigasi ke halaman yang benar sesuai daftar interaksi.
- [ ] Avatar menampilkan inisial nama sebagai fallback jika `avatar_url` kosong.
- [ ] Halaman ini hanya bisa diakses oleh user yang sudah login (redirect ke Login jika sesi tidak valid/expired).

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Sesi login expired saat membuka halaman | Token invalid/expired | Redirect otomatis ke halaman Login dengan pesan "Sesi berakhir, silakan masuk kembali" |
| Data profil gagal dimuat | API error | Skeleton diganti pesan error + tombol retry |
| Avatar upload gagal (di halaman Ubah Profil) | Upload error | Toast error, foto lama tetap dipertahankan |

### 13. Developer Notes
- Cache data profil di client (short TTL) agar transisi antar tab terasa instan, namun tetap re-validate token di background.
- Konfirmasi logout sebaiknya juga menghapus token/cache lokal (cart guest-merge, dsb.) secara bersih agar tidak ada data user lama tertinggal di device.

### 14. Priority & Requirement ID

| Priority | **High** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-AKUN-001 | Halaman hanya bisa diakses oleh user yang sudah login |
| REQ-AKUN-002 | Tap "Keluar" harus memunculkan dialog konfirmasi sebelum logout |
| REQ-AKUN-003 | Sesi expired harus redirect otomatis ke Login dengan pesan yang jelas |

### 15. Dependency

```
Akun Saya
 └─ bergantung pada
     ├─ Login (wajib sudah autentikasi)
     └─ API /api/me

Akun Saya ──menu ke──► Pesanan Saya, Wishlist, Alamat Saya, Pengaturan Akun, Bantuan (FAQ/Kontak)
```

### 16. Component Relationship

```
Card (Profile Card) + Avatar
   ↓
List Item (Menu: Pesanan/Wishlist/Alamat/Pengaturan/Bantuan/Keluar)
   ↓
Modal (Konfirmasi Logout)
```


---

## 13. Wishlist (Favorit) 🟡


> **Catatan**: Halaman ini tidak muncul secara eksplisit pada mockup yang diberikan, namun direferensikan oleh ikon **"Favorit"** pada Bottom Navigation Bar di semua halaman. Dokumentasi berikut disusun konsisten dengan pola desain (design system) yang sudah ada pada halaman Produk & Detail Produk.

### 1. Tujuan Halaman
Menampilkan daftar produk yang telah ditandai sebagai favorit (disimpan via ikon ♡ di halaman Produk/Detail Produk) agar pengguna mudah kembali dan membelinya nanti.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Wishlist Saya" / "Favorit" — center.

### 2.2 Grid Produk Favorit
Menggunakan **Product Card** yang identik dengan halaman Produk (gambar, ikon love terisi penuh, nama, harga, rating). Tap ikon love → hapus dari wishlist (dengan konfirmasi/undo snackbar).

### 2.3 Bottom Navigation Bar
State aktif: **Favorit**.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Product Card | Card | Reuse komponen dari halaman Produk |

### 4. State & Kondisi
- **Kosong**: ilustrasi hati kosong + teks "Belum ada produk favorit" + tombol "Jelajahi Produk" → Produk.
- **Belum login**: prompt login sebelum menampilkan wishlist tersimpan (jika wishlist disinkron ke akun).

### 5. Interaksi & Navigasi
```
Wishlist
 ├─ Tap Back → Halaman sebelumnya
 ├─ Tap Card → Detail Produk
 ├─ Tap ikon Love (filled) → Hapus dari wishlist
 └─ Bottom Nav → Beranda / Kategori / Chat / Akun
```

### 6. Pertimbangan UX
- Tambahkan tombol "Tambah Semua ke Keranjang" agar user dapat langsung checkout barang favorit sekaligus.
- Sinkronkan wishlist antar device jika user login (simpan di backend, bukan hanya local storage).

### 7. Responsive Notes
- Grid 2 kolom (mobile) → 3–4 kolom (tablet/desktop), identik dengan halaman Produk.

### 8. Spesifikasi Komponen (Detail Ukuran)
Reuse penuh komponen `Product Card` dari `produk.md` (lihat spesifikasi ukuran di sana) — perbedaan hanya ikon wishlist selalu dalam state "filled".

### 9. Design Token yang Digunakan
Sama seperti `produk.md`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Item dihapus dari wishlist | Fade-out + collapse grid item | 250ms (`motion/base`) | ease-in | Tap ikon love (filled → unfilled) |

### 11. Acceptance Criteria
- [ ] Tap ikon love (filled) langsung menghapus item dari wishlist dengan feedback visual instan.
- [ ] Grid menyesuaikan (reflow) tanpa gap kosong setelah item dihapus.
- [ ] Data wishlist tersinkron ke akun (bukan hanya local), sehingga konsisten di semua device.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Wishlist kosong | `products[] = []` | Ilustrasi hati kosong + CTA "Jelajahi Produk" |
| Produk di wishlist sudah dihapus/nonaktif oleh admin | Produk `is_active = false` | Tampilkan item dengan badge "Tidak Tersedia", disable tap-to-detail |

### 13. Developer Notes
- Gunakan tabel `wishlists` (lihat `00-api-database-mapping.md`) dengan constraint unique (`user_id`, `product_id`) untuk mencegah duplikasi entry.

### 14. Priority & Requirement ID

| Priority | **Medium** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-WISHLIST-001 | Tap ikon love (filled) harus langsung menghapus item dari wishlist |
| REQ-WISHLIST-002 | Data wishlist harus tersinkron ke akun, konsisten di semua device |

### 15. Dependency

```
Wishlist
 └─ bergantung pada
     ├─ Login (wajib sudah autentikasi)
     ├─ Produk / Detail Produk (sumber toggle wishlist)
     └─ API /api/wishlist

Wishlist ──diakses dari──► Bottom Navigation ("Favorit"), Akun Saya (menu "Wishlist Saya")
```

### 16. Component Relationship

```
Product Card (reuse dari 00-component-library.md)
   ↓
Snackbar ("Item dihapus dari wishlist, Undo")
```


---

## 14. Login 🟡


> **Catatan**: Tidak tampil pada mockup yang diberikan. Halaman ini diperlukan sebagai prasyarat akses ke halaman **Akun Saya**, **Wishlist**, dan **Checkout** bagi pengguna yang belum memiliki sesi aktif. Disusun konsisten dengan gaya visual brand (cream background, tombol olive solid, tipografi serif untuk heading).

### 1. Tujuan Halaman
Memverifikasi identitas pengguna agar dapat mengakses fitur personal (akun, riwayat pesanan, wishlist, checkout).

### 2. Struktur Layout

### 2.1 Header
- Logo "JATI PRIME Furniture" (center, atas).
- Judul "Masuk ke Akun Anda".

### 2.2 Form Login
- Input **Email / No. HP** (text field).
- Input **Password** (secure field + ikon show/hide password).
- Link teks "Lupa Password?" (kanan, kecil) → navigasi ke halaman reset password.
- **Tombol "Masuk"** (primary, full-width).

### 2.3 Alternatif Login
- Divider "atau" dengan garis di kedua sisi.
- Tombol login sosial (opsional): Google, Facebook.

### 2.4 Footer
- Teks "Belum punya akun? **Daftar**" → navigasi ke halaman **Register**.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Text Field | Input | Validasi format email/no. HP real-time |
| Password Field | Secure Input | Toggle visibility |
| Primary Button | Button | Disabled sampai kedua field terisi valid |

### 4. State & Kondisi
- **Error kredensial salah**: tampilkan pesan inline merah di bawah field terkait ("Email atau password salah").
- **Loading**: tombol "Masuk" menampilkan spinner saat proses autentikasi.
- **Field kosong**: tombol disabled / validasi saat submit.

### 5. Interaksi & Navigasi
```
Login
 ├─ Tap "Lupa Password?" → Reset Password
 ├─ Tap "Masuk" → Validasi → Akun Saya (sukses) / Error inline (gagal)
 └─ Tap "Daftar" → Register
```

### 6. Pertimbangan UX
- Untuk checkout tamu (guest checkout), pertimbangkan opsi "Lanjutkan sebagai Tamu" agar tidak memaksa registrasi hanya untuk membeli satu kali.
- Gunakan auto-fill/keyboard type sesuai field (numeric untuk no. HP, email keyboard untuk email).

### 7. Responsive Notes
- Form terpusat (max-width ~400px) pada semua breakpoint agar tetap nyaman dibaca di layar besar.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Text Field / Password Field
| Properti | Nilai |
|---|---|
| Height | 48px |
| Radius | `radius/md` |
| Border | 1px `color/border`, fokus → 1.5px `color/primary` |
| Padding horizontal | 12px |
| Font | `type/body` |
| Gap antar field | 16px |

### Primary Button "Masuk"
| Properti | Nilai |
|---|---|
| Height | 52px |
| Radius | `radius/md` |
| Background | `color/primary` (disabled: `color/text-disabled`) |

### 9. Design Token yang Digunakan
`color/primary`, `color/border`, `color/danger` (error state), `type/body`, `radius/md`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Input error | Shake horizontal kecil (±4px) + border merah | 300ms | ease-in-out | Validasi gagal |
| Tombol "Masuk" | Swap ke spinner loading | instan | – | Submit form |

### 11. Acceptance Criteria
- [ ] Validasi format email/no. HP real-time sebelum submit.
- [ ] Pesan error kredensial salah tampil jelas tanpa menyebutkan field mana yang salah secara spesifik (email atau password) demi keamanan (mencegah user enumeration).
- [ ] Tombol "Masuk" disabled sampai kedua field terisi.
- [ ] Password field punya toggle show/hide.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Salah password 5x berturut-turut | Rate limiting | Tampilkan captcha atau cooldown sementara untuk mencegah brute-force |
| Akun diblokir admin | Login dengan akun `is_active = false` | Pesan "Akun Anda dinonaktifkan, hubungi Kontak Kami" |
| Internet terputus saat submit | Network timeout | Tombol kembali ke state normal + toast error "Gagal terhubung, coba lagi" |

### 13. Developer Notes
- Simpan token di secure storage (bukan localStorage biasa untuk web — gunakan httpOnly cookie bila memungkinkan; untuk app mobile gunakan secure keychain/keystore).
- Terapkan rate-limiting di backend endpoint `/api/auth/login` untuk mencegah brute force.

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-LOGIN-001 | Validasi format email/no. HP harus real-time sebelum submit |
| REQ-LOGIN-002 | Endpoint login harus memiliki rate-limiting untuk mencegah brute force |
| REQ-LOGIN-003 | Token harus disimpan di secure storage (bukan localStorage biasa) |

### 15. Dependency

```
Login
 └─ tidak bergantung pada modul lain (fondasi paling awal — foundational)

Login ──diperlukan oleh──► Akun Saya, Wishlist, Checkout, Pesanan Saya, Tracking
```

> Ini adalah salah satu modul paling **Critical** dan menjadi dependency untuk hampir seluruh fitur transaksional — harus selesai lebih dulu.

### 16. Component Relationship

```
Input (Email/HP) + Input (Password, secure)
   ↓
Button Primary (Masuk)
   ↓
Akun Saya / Checkout (redirect setelah sukses)
```


---

## 15. Register (Daftar Akun) 🟡


> **Catatan**: Tidak tampil pada mockup yang diberikan. Disusun sebagai pasangan dari halaman Login, mengikuti pola visual & komponen yang sama.

### 1. Tujuan Halaman
Memungkinkan pengguna baru membuat akun untuk mengakses fitur personal aplikasi (wishlist, riwayat pesanan, checkout lebih cepat).

### 2. Struktur Layout

### 2.1 Header
- Logo "JATI PRIME Furniture" (center, atas).
- Judul "Buat Akun Baru".

### 2.2 Form Register
- Input **Nama Lengkap**.
- Input **Email**.
- Input **No. HP**.
- Input **Password** (dengan indikator kekuatan password).
- Input **Konfirmasi Password**.
- Checkbox "Saya menyetujui Syarat & Ketentuan serta Kebijakan Privasi" (dengan link tappable ke masing-masing dokumen).
- **Tombol "Daftar"** (primary, full-width, disabled sampai semua field valid & checkbox dicentang).

### 2.3 Footer
- Teks "Sudah punya akun? **Masuk**" → navigasi ke halaman **Login**.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Text Field | Input | Validasi real-time (format email, no. HP numerik) |
| Password Strength Indicator | Progress Bar | Lemah/Sedang/Kuat |
| Checkbox + Link | Toggle | Wajib dicentang sebelum submit |

### 4. State & Kondisi
- **Email sudah terdaftar**: pesan error inline "Email sudah digunakan, silakan login".
- **Password tidak cocok**: pesan error di field Konfirmasi Password.
- **Loading**: tombol "Daftar" menampilkan spinner saat proses pembuatan akun.
- **Sukses**: redirect ke halaman verifikasi (OTP email/SMS) atau langsung ke Home dengan sesi aktif.

### 5. Interaksi & Navigasi
```
Register
 ├─ Isi form → Validasi real-time per field
 ├─ Tap "Daftar" → Proses registrasi → Verifikasi/Home
 └─ Tap "Masuk" → Login
```

### 6. Pertimbangan UX
- Pertimbangkan opsi daftar via nomor WhatsApp mengingat brand ini banyak berinteraksi via WhatsApp (Chat WhatsApp muncul di beberapa halaman lain).
- Sertakan verifikasi OTP untuk keamanan akun sebelum dapat login penuh.

### 7. Responsive Notes
- Form terpusat (max-width ~400px), konsisten dengan halaman Login.

### 8. Spesifikasi Komponen (Detail Ukuran)
Reuse `Text Field`/`Password Field` dari `login.md`. Tambahan:

### Password Strength Indicator
| Properti | Nilai |
|---|---|
| Height bar | 4px |
| Radius | 2px |
| Warna Lemah | `color/danger` |
| Warna Sedang | `color/warning` |
| Warna Kuat | `color/success` |

### Checkbox Syarat & Ketentuan
| Properti | Nilai |
|---|---|
| Size | 20×20px |
| Radius | 4px |
| Link warna | `color/primary`, underline |

### 9. Design Token yang Digunakan
`color/danger`, `color/warning`, `color/success`, `color/primary`, sisanya sama seperti `login.md`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Password Strength Bar | Width + color transition | 200ms | ease-out | Input password berubah |
| Checkbox | Check mark draw-in | 150ms | ease-out | Tap checkbox |

### 11. Acceptance Criteria
- [ ] Password strength indicator update real-time saat mengetik.
- [ ] Tombol "Daftar" disabled sampai semua field valid & checkbox syarat dicentang.
- [ ] Pesan error "Email sudah digunakan" tampil jelas dengan CTA ke halaman Login.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Email sudah terdaftar | Submit dengan email existing | Error inline + link "Masuk di sini" |
| Password & Konfirmasi tidak cocok | Validasi mismatch | Error inline di field Konfirmasi Password |
| No. HP format tidak valid | Input bukan format Indonesia (+62/08xx) | Error inline real-time |

### 13. Developer Notes
- Kirim OTP verifikasi (email/SMS) sebelum akun aktif penuh, untuk mengurangi akun fiktif/spam.
- Validasi kekuatan password minimal di backend juga (jangan hanya client-side) demi keamanan.

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-REGISTER-001 | Password strength indicator harus update real-time |
| REQ-REGISTER-002 | Tombol "Daftar" harus disabled sampai semua field valid & checkbox dicentang |
| REQ-REGISTER-003 | Validasi kekuatan password harus dilakukan juga di backend, bukan hanya client |

### 15. Dependency

```
Register
 └─ tidak bergantung pada modul lain (fondasi paling awal, sejajar dengan Login)

Register ──diperlukan oleh──► Login, Akun Saya, dan seluruh fitur yang butuh autentikasi
```

### 16. Component Relationship

```
Input (Nama/Email/HP/Password/Konfirmasi Password)
   ↓
Checkbox (Syarat & Ketentuan)
   ↓
Button Primary (Daftar)
   ↓
Login (redirect setelah sukses / verifikasi OTP)
```


---

## 16. Pesanan Saya 🟡


> **Catatan**: Tidak tampil eksplisit pada mockup, namun direferensikan sebagai menu "Pesanan Saya" di halaman **Akun Saya**. Disusun konsisten dengan pola card & tab yang digunakan pada halaman Produk (filter tab) dan Keranjang (item card).

### 1. Tujuan Halaman
Menampilkan riwayat dan status seluruh pesanan yang pernah dibuat pengguna, dari diproses hingga selesai.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Pesanan Saya" — center.

### 2.2 Filter Tab Status (horizontal scroll chip)
- Semua
- Diproses
- Dikirim
- Selesai
- Dibatalkan

### 2.3 List Pesanan (Order Card)
Setiap **Order Card** berisi:
- No. Invoice / ID Pesanan (contoh: "#JP-20260724-001").
- Tanggal pesanan.
- Badge status (warna berbeda per status: kuning=diproses, biru=dikirim, hijau=selesai, abu/merah=dibatalkan).
- Thumbnail 1–3 produk dalam pesanan tersebut + teks "+N produk lainnya" jika lebih dari 3.
- Total pembayaran.
- **Tombol aksi kontekstual**:
  - Jika "Dikirim" → tombol "Lacak Pesanan" → navigasi ke halaman **Tracking**.
  - Jika "Selesai" → tombol "Beli Lagi" dan "Beri Ulasan".
  - Jika "Diproses" → tombol "Batalkan Pesanan" (dengan konfirmasi).

Tap card (area non-tombol) → detail pesanan lengkap (rincian item, alamat, metode pembayaran — mirip ringkasan pada halaman Checkout tapi read-only).

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Status Filter Chip | Toggle Pill | Single-select |
| Order Card | Card | Status badge + thumbnail produk + CTA kontekstual |
| Status Badge | Label Badge | Warna sesuai status |

### 4. State & Kondisi
- **Belum ada pesanan**: ilustrasi + teks "Belum ada pesanan" + tombol "Mulai Belanja" → Produk.
- **Loading**: skeleton card list.
- **Batalkan pesanan**: dialog konfirmasi sebelum eksekusi, status berubah menjadi "Dibatalkan" dan card pindah ke tab terkait.

### 5. Interaksi & Navigasi
```
Pesanan Saya
 ├─ Tap Back → Akun Saya
 ├─ Tap Filter Tab → Filter list sesuai status
 ├─ Tap Card → Detail Pesanan (read-only)
 ├─ Tap "Lacak Pesanan" → Tracking
 ├─ Tap "Beli Lagi" → Tambahkan item ke Keranjang
 ├─ Tap "Beri Ulasan" → Form review produk
 └─ Tap "Batalkan Pesanan" → Dialog konfirmasi
```

### 6. Pertimbangan UX
- Badge status harus konsisten warna & label di seluruh aplikasi (termasuk notifikasi push jika ada).
- "Beli Lagi" mempercepat repeat purchase — fitur penting untuk retensi pelanggan furniture (biasanya cross-sell aksesori).

### 7. Responsive Notes
- List card single column di mobile; grid 2 kolom pada tablet/desktop.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Filter Tab Status
Reuse Filter Tab dari `produk.md`.

### Order Card
| Properti | Nilai |
|---|---|
| Padding | 16px |
| Radius | `radius/lg` |
| Shadow | `shadow/sm` |
| Invoice ID font | `type/body-sm`, `color/text-secondary` |
| Status Badge height | 24px, radius `radius/sm` |
| Thumbnail size | 48×48px, radius 8px, gap 6px antar thumbnail |
| Total font | `type/h3` |
| Tombol aksi height | 40px, radius `radius/md` |

### Status Badge Colors
| Status | Background | Text |
|---|---|---|
| Diproses | `rgba(224,169,62,0.12)` | `color/warning` |
| Dikirim | `rgba(59,110,165,0.12)` | `color/info` |
| Selesai | `rgba(62,142,90,0.12)` | `color/success` |
| Dibatalkan | `rgba(229,57,53,0.12)` | `color/danger` |

### 9. Design Token yang Digunakan
`color/warning`, `color/info`, `color/success`, `color/danger`, `type/h3`, `type/body-sm`, `radius/lg`, `radius/sm`, `radius/md`, `shadow/sm`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Filter Tab switch | Fade-out list lama → fade-in list baru | 200ms (`motion/base`) | ease-in-out | Ganti tab status |
| Order Card | Slide-out saat dibatalkan lalu pindah tab | 250ms | ease-in | Konfirmasi batalkan pesanan |

### 11. Acceptance Criteria
- [ ] Filter tab status memfilter list pesanan sesuai status yang benar.
- [ ] Tombol aksi kontekstual (Lacak/Beli Lagi/Batalkan) sesuai dengan status pesanan masing-masing.
- [ ] "Batalkan Pesanan" memunculkan dialog konfirmasi sebelum eksekusi.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Belum ada pesanan sama sekali | `orders[] = []` | Empty state + CTA "Mulai Belanja" |
| Coba batalkan pesanan yang sudah "Dikirim" | Status tidak eligible dibatalkan | Tombol "Batalkan" tidak ditampilkan sama sekali untuk status ini |
| Produk dalam pesanan lama sudah dihapus dari katalog | `product_id` tidak ditemukan | Tampilkan nama produk snapshot dari `order_items` (bukan join real-time ke `products`), disable "Beli Lagi" untuk item tsb |

### 13. Developer Notes
- Data nama & harga produk di riwayat pesanan harus disimpan sebagai **snapshot** pada saat transaksi (`order_items.price_at_purchase`, nama produk ikut disimpan), bukan join langsung ke tabel `products` yang bisa berubah/terhapus.

### 14. Priority & Requirement ID

| Priority | **High** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-PESANAN-001 | Filter tab status harus memfilter list pesanan sesuai status benar |
| REQ-PESANAN-002 | Data nama/harga produk di riwayat harus berupa snapshot, bukan join real-time |
| REQ-PESANAN-003 | "Batalkan Pesanan" harus memunculkan dialog konfirmasi |

### 15. Dependency

```
Pesanan Saya
 └─ bergantung pada
     ├─ Login
     ├─ Checkout (sumber data order dibuat)
     └─ API /api/orders

Pesanan Saya ──diperlukan oleh──► Tracking, Testimoni (trigger ulasan setelah "Selesai")
```

### 16. Component Relationship

```
Tabs (Filter Status)
   ↓
Card (Order Card) + Badge (Status)
   ↓
Button (Lacak Pesanan / Beli Lagi / Beri Ulasan / Batalkan)
   ↓
Tracking / Keranjang / Testimoni
```


---

## 17. Tracking Pesanan 🟡


> **Catatan**: Tidak tampil pada mockup, direferensikan dari tombol "Lacak Pesanan" pada halaman **Pesanan Saya**. Disusun mengikuti pola numbered-step yang sudah dipakai pada halaman **Checkout**.

### 1. Tujuan Halaman
Menampilkan status pengiriman pesanan secara real-time/bertahap agar pengguna tahu posisi barang mereka.

### 2. Struktur Layout

### 2.1 Top Navigation Bar
- Ikon back (←) — kiri.
- Judul "Lacak Pesanan" — center.

### 2.2 Ringkasan Pesanan Singkat
- No. Invoice, tanggal pesan, estimasi tiba.

### 2.3 Timeline Status (Vertical Stepper)
Menggunakan pola circular step badge (seperti nomor 1-4 di Checkout) namun berorientasi vertikal dengan garis penghubung:
1. **Pesanan Dibuat** — timestamp.
2. **Pembayaran Dikonfirmasi** — timestamp.
3. **Sedang Diproses / Produksi** — timestamp (khusus custom furniture bisa lebih lama).
4. **Dikirim** — nomor resi + nama kurir/ekspedisi.
5. **Tiba di Tujuan** — timestamp (jika sudah selesai).

Step yang sudah terlewati ditandai check (✓) hijau, step aktif ditandai highlight, step mendatang abu-abu/kosong.

### 2.4 Info Kurir (jika status "Dikirim")
- Nama ekspedisi, nomor resi (dengan tombol "Salin"), tombol "Lacak di Situs Ekspedisi" (link eksternal).

### 2.5 CTA
- Tombol "Hubungi Kami" jika ada kendala pengiriman → navigasi ke Kontak/Chat WhatsApp.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Vertical Stepper | Timeline Component | Reuse gaya step badge dari Checkout |
| Courier Info Card | Info Card | No. resi + copy button |

### 4. State & Kondisi
- **Belum ada update kurir**: step "Dikirim" tampil abu-abu/pending.
- **Gagal kirim/retur**: tambahkan step khusus dengan warna merah/peringatan.
- **Loading**: skeleton stepper saat data status dimuat.

### 5. Interaksi & Navigasi
```
Tracking
 ├─ Tap Back → Pesanan Saya
 ├─ Tap "Salin" no. resi → Copy to clipboard
 ├─ Tap "Lacak di Situs Ekspedisi" → Buka browser eksternal
 └─ Tap "Hubungi Kami" → Kontak/WhatsApp
```

### 6. Pertimbangan UX
- Untuk furniture custom, estimasi waktu produksi bisa cukup lama (mingguan) — tampilkan estimasi tanggal per step agar ekspektasi pelanggan terkelola baik, merujuk jawaban FAQ "Berapa lama proses pengerjaan furniture?".
- Notifikasi push/email otomatis saat status berubah akan meningkatkan kepercayaan tanpa pelanggan perlu cek manual.

### 7. Responsive Notes
- Vertical timeline tetap 1 kolom di semua breakpoint karena sifatnya kronologis/sequential.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Vertical Stepper
| Properti | Nilai |
|---|---|
| Step circle size | 24×24px |
| Connector line width | 2px |
| Connector line (completed) color | `color/success` |
| Connector line (pending) color | `color/border` |
| Gap antar step | 24px |
| Label font | `type/h3` |
| Timestamp font | `type/body-sm`, `color/text-secondary` |

### Courier Info Card
| Properti | Nilai |
|---|---|
| Padding | 16px |
| Radius | `radius/lg` |
| "Salin" button height | 32px, radius `radius/sm` |

### 9. Design Token yang Digunakan
`color/success`, `color/border`, `type/h3`, `type/body-sm`, `radius/lg`, `radius/sm`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Step aktif baru (status update) | Circle scale-in + connector line fill progresif | 400ms (`motion/slow`) | ease-out | Status pesanan berubah |
| Tombol "Salin" | Icon berubah jadi checkmark sesaat | 1000ms hold | ease-out | Tap salin resi |

### 11. Acceptance Criteria
- [ ] Timeline menampilkan step yang sudah terlewati dengan jelas berbeda dari step mendatang.
- [ ] Nomor resi dapat disalin dengan satu tap.
- [ ] Link "Lacak di Situs Ekspedisi" membuka halaman resmi ekspedisi terkait dengan nomor resi terisi otomatis (jika didukung).

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Status "Dikirim" tapi resi belum diinput admin | Data resi kosong | Step "Dikirim" tetap aktif tapi info kurir tampil "Menunggu info pengiriman" |
| Pesanan diretur/gagal kirim | Status khusus | Tambahkan step warna merah "Pengiriman Bermasalah" + CTA "Hubungi Kami" |

### 13. Developer Notes
- Struktur timeline sebaiknya generic/data-driven (`timeline[]` dari API) agar mudah menambah step baru (misal "Quality Check") tanpa perlu ubah kode frontend.

### 14. Priority & Requirement ID

| Priority | **High** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-TRACKING-001 | Timeline harus jelas membedakan step terlewati vs mendatang |
| REQ-TRACKING-002 | Nomor resi harus dapat disalin dengan satu tap |

### 15. Dependency

```
Tracking
 └─ bergantung pada
     ├─ Pesanan Saya (entry point)
     └─ Admin — Manajemen Pesanan (sumber update status & no. resi)

Tidak ada halaman lain yang bergantung langsung pada Tracking.
```

### 16. Component Relationship

```
Stepper (Progress, vertikal)
   ↓
Card (Courier Info) + Button (Salin Resi)
   ↓
Button (Hubungi Kami) → Kontak Kami / WhatsApp
```


---

## 18. Admin — Dashboard 🟡


> **Catatan**: Panel admin tidak tampil pada mockup yang diberikan (mockup hanya mencakup customer-facing app). Dokumentasi berikut adalah rancangan usulan struktur admin panel yang selaras dengan data & entitas yang muncul di sisi customer (produk, kategori, pesanan, pelanggan).

### 1. Tujuan Halaman
Memberikan ringkasan performa toko (sales overview) bagi admin/pemilik brand Jati Prime dalam satu tampilan (bird's-eye view).

### 2. Struktur Layout

### 2.1 Sidebar Navigasi (Desktop) / Bottom Nav (Mobile Admin)
- Dashboard (aktif)
- Produk
- Kategori
- Pesanan
- Pelanggan
- Pengaturan Toko
- Logout

### 2.2 Top Bar
- Judul "Dashboard".
- Info admin login (avatar + nama) di kanan.
- Notifikasi bell icon (pesanan baru, stok menipis, dll).

### 2.3 Stat Cards (Grid 4 kolom)
- Total Penjualan (Bulan Ini) — dengan indikator naik/turun % vs bulan lalu.
- Total Pesanan Baru.
- Produk Terlaris.
- Pelanggan Baru.

### 2.4 Grafik Penjualan
- Line/bar chart tren penjualan 7/30 hari terakhir, dengan filter rentang tanggal.

### 2.5 Tabel Pesanan Terbaru
- Kolom: No. Invoice, Pelanggan, Total, Status, Tanggal — 5-10 baris terbaru, link "Lihat Semua" → Admin Pesanan.

### 2.6 Widget Stok Menipis
- List produk dengan stok di bawah ambang batas (misal < 5 unit), link cepat ke Admin Produk untuk restock.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Stat Card | KPI Card | Angka besar + trend indicator |
| Chart | Line/Bar Chart | Interactive tooltip per titik data |
| Data Table | Table | Sortable columns, pagination |

### 4. State & Kondisi
- **Loading**: skeleton pada stat cards & chart saat data dimuat.
- **Tidak ada data** (toko baru): tampilkan state kosong dengan panduan "Mulai tambahkan produk pertama Anda".

### 5. Interaksi & Navigasi
```
Dashboard
 ├─ Klik Stat Card → Detail terkait (misal Total Pesanan → Admin Pesanan)
 ├─ Klik "Lihat Semua" tabel → Admin Pesanan
 ├─ Klik produk stok menipis → Admin Produk (edit stok)
 └─ Sidebar → Produk / Kategori / Pesanan / Pelanggan
```

### 6. Pertimbangan UX
- Prioritaskan informasi actionable (pesanan baru perlu diproses, stok menipis) di atas metrik vanity agar admin langsung tahu apa yang harus dikerjakan hari itu.
- Gunakan warna status konsisten dengan yang ada di sisi customer (halaman Pesanan Saya) agar tim internal & data customer selaras.

### 7. Responsive Notes
- Desktop-first (panel admin umumnya diakses via laptop/PC); tetap sediakan versi mobile-responsive untuk cek cepat dari HP.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Stat Card
| Properti | Nilai |
|---|---|
| Grid | 4 kolom (desktop), 2 kolom (mobile) |
| Padding | 20px |
| Radius | `radius/lg` |
| Angka utama font | `type/display`, 28px/700 |
| Label font | `type/body-sm` |
| Trend indicator | ikon panah + persentase, hijau (`color/success`) naik / merah (`color/danger`) turun |

### Chart Penjualan
| Properti | Nilai |
|---|---|
| Height | 280px |
| Line color | `color/primary` |
| Grid line color | `color/border` (opacity 0.5) |

### Data Table Pesanan Terbaru
| Properti | Nilai |
|---|---|
| Row height | 48px |
| Header background | `color/body-bg` |
| Header font | `type/body-sm`, 600, uppercase |

### 9. Design Token yang Digunakan
`color/primary`, `color/success`, `color/danger`, `color/border`, `type/display`, `type/body-sm`, `radius/lg`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Stat Card angka | Count-up dari 0 ke nilai aktual | 800ms | ease-out | Saat halaman dimuat |
| Chart | Line draw-in dari kiri ke kanan | 600ms | ease-out | Saat data dimuat |

### 11. Acceptance Criteria
- [ ] Stat card menampilkan data real-time (bukan cache statis lama) dengan indikator trend yang akurat.
- [ ] Tabel pesanan terbaru menampilkan maksimal 10 baris dengan link "Lihat Semua".
- [ ] Widget stok menipis hanya menampilkan produk dengan stok di bawah ambang batas yang dikonfigurasi.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Toko baru tanpa data penjualan | Data kosong | State kosong dengan panduan "Mulai tambahkan produk pertama Anda" |
| Gagal load salah satu widget | API partial failure | Widget lain tetap tampil normal, hanya widget yang gagal menampilkan error kecil + retry |

### 13. Developer Notes
- Cache data dashboard di server (mis. 1-5 menit) karena query agregat (total penjualan, dsb.) relatif berat jika dihitung real-time setiap request.
- Gunakan role-based access agar hanya admin dengan permission tertentu yang bisa melihat data finansial sensitif.

### 14. Priority & Requirement ID

| Priority | **High** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-ADMIN-DASH-001 | Stat card harus menampilkan data real-time dengan indikator trend akurat |
| REQ-ADMIN-DASH-002 | Widget stok menipis harus sesuai ambang batas yang dikonfigurasi |

### 15. Dependency

```
Admin Dashboard
 └─ bergantung pada
     ├─ Admin Manajemen Produk (data stok)
     ├─ Admin Manajemen Pesanan (data penjualan)
     └─ Admin Manajemen Pelanggan (data pelanggan baru)

Ini adalah halaman agregat — harus dibangun setelah 3 modul admin lain punya data.
```

### 16. Component Relationship

```
Card (Stat Card ×4)
   ↓
Chart (Line/Bar Penjualan)
   ↓
Data Table (Pesanan Terbaru) → Admin Manajemen Pesanan
```


---

## 19. Admin — Manajemen Produk 🟡


> **Catatan**: Tidak tampil pada mockup customer-facing. Rancangan usulan agar admin dapat mengelola data yang ditampilkan pada halaman **Produk** & **Detail Produk** di sisi customer.

### 1. Tujuan Halaman
Mengelola (CRUD) seluruh data produk furniture: informasi, harga, stok, gambar, dan spesifikasi.

### 2. Struktur Layout

### 2.1 Top Bar
- Judul "Manajemen Produk".
- Tombol "**+ Tambah Produk**" (kanan atas, primary button).

### 2.2 Search & Filter Bar
- Input pencarian nama produk.
- Filter dropdown: Kategori, Status Stok (Tersedia/Habis), Urutkan (Terbaru/Terlaris/Harga).

### 2.3 Tabel Produk
Kolom:
- Thumbnail gambar
- Nama Produk
- Kategori
- Harga
- Stok
- Rating (read-only, agregat dari ulasan customer)
- Status (Aktif/Nonaktif — toggle switch untuk tampil/sembunyi di aplikasi customer)
- Aksi (Edit ✏️ / Hapus 🗑️)

### 2.4 Form Tambah/Edit Produk (Modal atau Halaman Terpisah)
Field-field sesuai spesifikasi yang tampil di halaman **Detail Produk** customer:
- Nama Produk
- Kategori (dropdown, terhubung ke Admin Kategori)
- Harga
- Deskripsi
- Material
- Finishing
- Ukuran
- Kapasitas
- Garansi
- Stok
- Upload Gambar (multiple, drag-and-drop, dengan urutan gambar utama & thumbnail)
- Toggle Status Aktif/Nonaktif

Tombol "Simpan" dan "Batal".

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Data Table | Table | Sortable, pagination, bulk-select checkbox |
| Product Form | Form Modal/Page | Validasi field wajib sebelum simpan |
| Image Uploader | Drag & Drop Zone | Preview thumbnail, reorder via drag |

### 4. State & Kondisi
- **Hapus produk**: dialog konfirmasi (produk yang sudah pernah dipesan sebaiknya di-nonaktifkan, bukan dihapus permanen, agar riwayat pesanan lama tetap valid).
- **Stok = 0**: otomatis update status tampilan di customer app menjadi "Stok Habis" (sinkron dengan halaman Detail Produk).
- **Validasi form**: field wajib (nama, harga, kategori, minimal 1 gambar) harus terisi sebelum submit.

### 5. Interaksi & Navigasi
```
Manajemen Produk
 ├─ Klik "+ Tambah Produk" → Form Tambah Produk
 ├─ Klik Edit (✏️) → Form Edit Produk (prefilled)
 ├─ Klik Hapus (🗑️) → Dialog konfirmasi
 ├─ Toggle Status → Update visibilitas di app customer
 └─ Filter/Search → Update tabel
```

### 6. Pertimbangan UX
- Reuse struktur field spesifikasi persis sama dengan yang tampil di halaman Detail Produk customer, agar tidak ada field yang terlewat/tidak sinkron.
- Sediakan bulk-action (aktifkan/nonaktifkan/hapus banyak produk sekaligus) untuk efisiensi admin dengan katalog besar.

### 7. Responsive Notes
- Desktop-first dengan tabel data lengkap; pada mobile, tabel dapat berubah menjadi list card ringkas per produk.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Data Table
| Properti | Nilai |
|---|---|
| Row height | 56px |
| Thumbnail size | 40×40px, radius 6px |
| Toggle Status switch | 40×22px |
| Aksi icon size | 18×18px |

### Form Tambah/Edit Produk
| Properti | Nilai |
|---|---|
| Input height | 44px |
| Textarea (deskripsi) height | 120px min |
| Image Uploader dropzone height | 160px, border dashed 2px `color/border` |
| Tombol Simpan height | 44px, background `color/primary` |

### 9. Design Token yang Digunakan
`color/primary`, `color/border`, `radius/md`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Toggle Status switch | Slide knob | 150ms | ease-out | Tap switch |
| Image upload preview | Fade-in thumbnail baru | 200ms | ease-out | Upload selesai |

### 11. Acceptance Criteria
- [ ] Form validasi field wajib (nama, harga, kategori, minimal 1 gambar) sebelum submit berhasil.
- [ ] Toggle status langsung mempengaruhi visibilitas produk di aplikasi customer tanpa delay signifikan.
- [ ] Upload gambar mendukung drag-and-drop dan reorder urutan gambar.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Hapus produk yang sudah pernah dipesan | Produk terkait ke `order_items` | Larang hard-delete, arahkan admin untuk nonaktifkan (`is_active=false`) saja |
| Upload gambar gagal (ukuran terlalu besar/format salah) | Validasi file | Toast error dengan keterangan format & ukuran maksimum yang didukung |

### 13. Developer Notes
- Field spesifikasi (Material, Finishing, Ukuran, dst.) sebaiknya disimpan fleksibel (bisa key-value dinamis) agar mendukung kategori produk baru dengan atribut berbeda di masa depan.
- Gunakan soft-delete (`is_active=false`) bukan hard-delete untuk menjaga integritas data histori pesanan.

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-ADMIN-PRODUK-001 | Hapus produk yang sudah pernah dipesan harus diblokir (soft-delete saja) |
| REQ-ADMIN-PRODUK-002 | Form harus validasi field wajib sebelum submit berhasil |
| REQ-ADMIN-PRODUK-003 | Toggle status harus langsung mempengaruhi visibilitas di aplikasi customer |

### 15. Dependency

```
Admin Manajemen Produk
 └─ bergantung pada
     └─ Admin Manajemen Kategori (dropdown kategori pada form produk)

Admin Manajemen Produk ──menyuplai data ke──► Kategori, Produk, Detail Produk (customer)
```

### 16. Component Relationship

```
Data Table (list produk)
   ↓
Modal/Form (Tambah/Edit Produk) + Input + Image Uploader
   ↓
Toggle Switch (Status Aktif/Nonaktif) → mempengaruhi Produk (customer)
```


---

## 20. Admin — Manajemen Kategori 🟡


> **Catatan**: Tidak tampil pada mockup customer-facing. Rancangan usulan agar admin dapat mengelola kategori yang tampil pada halaman **Kategori** di sisi customer.

### 1. Tujuan Halaman
Mengelola daftar kategori produk (Sofa, Meja Makan, Kursi, dst.) yang digunakan untuk navigasi & filter di aplikasi customer.

### 2. Struktur Layout

### 2.1 Top Bar
- Judul "Manajemen Kategori".
- Tombol "**+ Tambah Kategori**" (kanan atas).

### 2.2 Tabel/Grid Kategori
Kolom/Item:
- Ikon/Gambar kategori (thumbnail kecil sesuai yang tampil di grid Kategori customer).
- Nama Kategori.
- Jumlah Produk terkait (badge angka).
- Urutan tampil (drag handle untuk reorder — menentukan urutan grid di sisi customer).
- Aksi (Edit ✏️ / Hapus 🗑️).

### 2.3 Form Tambah/Edit Kategori
- Nama Kategori.
- Upload Ikon/Gambar kategori.
- Slug/URL (auto-generate dari nama, dapat diedit).
- Toggle Status Aktif/Nonaktif.

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Sortable List/Table | Drag & Drop List | Reorder mempengaruhi urutan tampil di customer app |
| Category Form | Form Modal | Field minimal: nama & ikon |

### 4. State & Kondisi
- **Hapus kategori yang masih punya produk terkait**: tampilkan peringatan "Kategori ini memiliki N produk, pindahkan produk ke kategori lain sebelum menghapus" (mencegah broken reference).
- **Kategori kosong (0 produk)**: tetap dapat tampil di customer app namun sebaiknya diberi badge "Segera Hadir" jika memang disengaja, atau disembunyikan otomatis.

### 5. Interaksi & Navigasi
```
Manajemen Kategori
 ├─ Klik "+ Tambah Kategori" → Form Tambah
 ├─ Drag handle → Reorder urutan tampil
 ├─ Klik Edit → Form Edit (prefilled)
 └─ Klik Hapus → Validasi produk terkait → Konfirmasi/Peringatan
```

### 6. Pertimbangan UX
- Batasi jumlah kategori utama (idealnya tetap di kisaran 8-10 seperti pada mockup customer) agar grid Kategori di app tetap rapi (3 kolom x beberapa baris) tanpa perlu scroll berlebihan.
- Ikon kategori baru harus mengikuti gaya visual (line-art/flat, warna) yang konsisten dengan ikon kategori existing.

### 7. Responsive Notes
- Desktop-first; drag-and-drop reorder idealnya tetap didukung pada tablet (touch-friendly).

### 8. Spesifikasi Komponen (Detail Ukuran)

### Sortable List
| Properti | Nilai |
|---|---|
| Row height | 56px |
| Drag handle icon size | 20×20px |
| Thumbnail size | 32×32px, radius 6px |
| Badge jumlah produk | height 20px, radius `radius/sm` |

### 9. Design Token yang Digunakan
`radius/sm`, `radius/md`.

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Reorder drag | Row mengikuti posisi drag + row lain shift | real-time (spring) | – | Drag handle |

### 11. Acceptance Criteria
- [ ] Drag-and-drop reorder langsung tersimpan (auto-save `sort_order`) tanpa perlu tombol simpan terpisah.
- [ ] Hapus kategori yang masih memiliki produk terkait diblokir dengan pesan peringatan yang jelas.

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Hapus kategori dengan produk terkait | `product_count > 0` | Blokir hapus + pesan "Pindahkan N produk ke kategori lain terlebih dahulu" |

### 13. Developer Notes
- Batasi jumlah kategori utama yang tampil di grid customer (idealnya ≤10) agar UX halaman Kategori tetap rapi.

### 14. Priority & Requirement ID

| Priority | **High** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-ADMIN-KATEGORI-001 | Drag-and-drop reorder harus auto-save `sort_order` |
| REQ-ADMIN-KATEGORI-002 | Hapus kategori dengan produk terkait harus diblokir dengan pesan jelas |

### 15. Dependency

```
Admin Manajemen Kategori
 └─ tidak bergantung pada modul admin lain (dasar/fondasi katalog)

Admin Manajemen Kategori ──diperlukan oleh──► Admin Manajemen Produk (dropdown kategori), Kategori (customer)
```

### 16. Component Relationship

```
Sortable List (drag handle)
   ↓
Modal/Form (Tambah/Edit Kategori)
   ↓
Kategori (customer, urutan tampil grid)
```


---

## 21. Admin — Manajemen Pesanan 🟡


> **Catatan**: Tidak tampil pada mockup customer-facing. Rancangan usulan agar admin dapat memproses pesanan yang dibuat customer melalui halaman **Checkout**, dan statusnya tersinkron ke halaman customer **Pesanan Saya** & **Tracking**.

### 1. Tujuan Halaman
Mengelola seluruh siklus hidup pesanan: verifikasi pembayaran, update status pengiriman, hingga penyelesaian pesanan.

### 2. Struktur Layout

### 2.1 Top Bar
- Judul "Manajemen Pesanan".
- Tombol export data (misal "Export ke Excel") — opsional.

### 2.2 Filter Tab Status
- Semua / Menunggu Pembayaran / Diproses / Dikirim / Selesai / Dibatalkan
(konsisten dengan status yang tampil di halaman customer **Pesanan Saya**).

### 2.3 Tabel Pesanan
Kolom:
- No. Invoice
- Nama Pelanggan
- Tanggal Pesan
- Jumlah Item
- Total Pembayaran
- Metode Pembayaran (Transfer Bank / COD — sesuai opsi di halaman Checkout)
- Status
- Aksi (Lihat Detail / Update Status)

### 2.4 Detail Pesanan (Modal/Halaman Terpisah)
- Info pelanggan & alamat pengiriman (sesuai data dari Checkout).
- Rincian item yang dipesan (nama, qty, harga).
- Metode pengiriman terpilih (Reguler/Express).
- Bukti transfer (jika metode Transfer Bank — upload/lihat bukti dari customer).
- **Dropdown Update Status**: Menunggu Pembayaran → Diproses → Dikirim (+ input No. Resi & Ekspedisi) → Selesai.
- Tombol "Simpan Perubahan".

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Status Filter Tab | Toggle Pill | Sama seperti pola tab pada Produk/Pesanan customer |
| Data Table | Table | Sortable, searchable, pagination |
| Status Dropdown | Select Input | Perubahan status memicu notifikasi ke customer |
| Resi Input | Text Field | Wajib diisi saat status diubah ke "Dikirim" |

### 4. State & Kondisi
- **Update status ke "Dikirim"**: field No. Resi & Ekspedisi wajib diisi sebelum simpan (data ini muncul di halaman **Tracking** customer).
- **Pesanan COD**: tidak perlu verifikasi bukti transfer, langsung dapat diproses.
- **Pesanan Transfer Bank belum ada bukti**: status default "Menunggu Pembayaran", admin tidak bisa lanjut ke "Diproses" sampai bukti diverifikasi (atau ada tombol override manual dengan catatan).

### 5. Interaksi & Navigasi
```
Manajemen Pesanan
 ├─ Filter Tab Status → Update tabel
 ├─ Klik baris → Detail Pesanan
 ├─ Verifikasi bukti transfer → Update status "Diproses"
 ├─ Isi No. Resi → Update status "Dikirim" → Sinkron ke Tracking customer
 └─ Update status "Selesai" → Memicu notifikasi minta ulasan ke customer
```

### 6. Pertimbangan UX
- Setiap perubahan status idealnya otomatis mengirim notifikasi (push/WhatsApp/email) ke customer, mengingat brand ini sangat mengandalkan kanal WhatsApp.
- Sediakan catatan internal (admin notes) per pesanan untuk komunikasi antar staf (misal catatan custom request dari customer).

### 7. Responsive Notes
- Desktop-first dengan tabel lengkap; tetap sediakan tampilan mobile-responsive ringkas untuk cek status cepat.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Data Table & Detail Panel
| Properti | Nilai |
|---|---|
| Row height | 56px |
| Status Dropdown height | 36px, radius `radius/sm` |
| Resi Input height | 40px |
| Upload bukti transfer preview | 120×120px thumbnail |

### 9. Design Token yang Digunakan
`radius/sm`, `radius/md`, status badge colors (lihat `pesanan.md` customer untuk mapping warna status).

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Status Dropdown update | Badge warna crossfade | 150ms | ease-in-out | Simpan perubahan status |

### 11. Acceptance Criteria
- [ ] Update status ke "Dikirim" mewajibkan input No. Resi & nama Ekspedisi sebelum bisa disimpan.
- [ ] Perubahan status memicu notifikasi otomatis ke customer (push/WhatsApp/email).
- [ ] Detail pesanan menampilkan seluruh info yang sama seperti yang diinput customer di Checkout (alamat, metode pengiriman, metode pembayaran).

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Bukti transfer tidak jelas/blur | Admin review manual | Sediakan opsi "Minta Bukti Ulang" yang mengirim notifikasi ke customer |
| Update status gagal tersimpan (network) | API error | Tampilkan error, status tidak berubah di UI sampai berhasil |

### 13. Developer Notes
- Setiap perubahan status sebaiknya tercatat sebagai audit log (siapa admin yang mengubah, kapan) untuk akuntabilitas.
- Integrasi API ekspedisi (jika tersedia) untuk auto-update status "Selesai" berdasarkan status pengiriman real dari kurir.

### 14. Priority & Requirement ID

| Priority | **Critical** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-ADMIN-PESANAN-001 | Update status ke "Dikirim" harus mewajibkan No. Resi & Ekspedisi terisi |
| REQ-ADMIN-PESANAN-002 | Perubahan status harus memicu notifikasi otomatis ke customer |
| REQ-ADMIN-PESANAN-003 | Setiap perubahan status harus tercatat sebagai audit log |

### 15. Dependency

```
Admin Manajemen Pesanan
 └─ bergantung pada
     └─ Checkout (customer) — sumber data pesanan masuk

Admin Manajemen Pesanan ──menyuplai data ke──► Tracking (customer), Admin Dashboard
```

### 16. Component Relationship

```
Tabs (Filter Status) + Data Table
   ↓
Detail Panel (info pesanan + bukti transfer)
   ↓
Select (Update Status) + Input (No. Resi)
   ↓
Tracking (customer, sinkron real-time)
```


---

## 22. Admin — Manajemen Pelanggan 🟡


> **Catatan**: Tidak tampil pada mockup customer-facing. Rancangan usulan agar admin dapat melihat & mengelola data pelanggan yang mendaftar melalui halaman **Register**, serta memantau aktivitas terkait profil di halaman **Akun Saya** customer.

### 1. Tujuan Halaman
Memberikan visibilitas terhadap seluruh pelanggan terdaftar: data kontak, riwayat pesanan, dan status akun.

### 2. Struktur Layout

### 2.1 Top Bar
- Judul "Manajemen Pelanggan".
- Input pencarian (nama/email/no. HP).

### 2.2 Tabel Pelanggan
Kolom:
- Avatar + Nama
- Email
- No. HP
- Tanggal Bergabung
- Total Pesanan (jumlah transaksi)
- Total Belanja (lifetime value)
- Status Akun (Aktif/Diblokir)
- Aksi (Lihat Detail / Blokir-Aktifkan)

### 2.3 Detail Pelanggan (Modal/Halaman Terpisah)
- Info profil lengkap (sama seperti data di halaman **Akun Saya** customer: nama, email, no. HP, avatar).
- Daftar alamat tersimpan.
- Riwayat pesanan (list ringkas, link ke Admin Pesanan per invoice).
- Riwayat ulasan yang pernah diberikan (terhubung ke data di halaman **Testimoni** customer).

### 3. Komponen UI
| Komponen | Tipe | Keterangan |
|---|---|---|
| Data Table | Table | Sortable (Total Belanja, Tanggal Bergabung), searchable |
| Customer Detail Panel | Side Panel/Modal | Ringkasan 360° satu pelanggan |
| Status Toggle | Switch | Blokir/aktifkan akun pelanggan |

### 4. State & Kondisi
- **Blokir akun**: pelanggan tidak dapat login (halaman **Login** customer), tampilkan alasan singkat wajib diisi admin saat blokir.
- **Pelanggan tanpa pesanan** (baru daftar): Total Pesanan = 0, tetap tampil di list.
- **Pencarian tanpa hasil**: tampilkan pesan "Pelanggan tidak ditemukan".

### 5. Interaksi & Navigasi
```
Manajemen Pelanggan
 ├─ Search → Filter tabel
 ├─ Klik baris → Detail Pelanggan
 ├─ Klik riwayat pesanan → Admin Pesanan (invoice terkait)
 └─ Toggle Status → Blokir/Aktifkan akun
```

### 6. Pertimbangan UX
- Highlight pelanggan dengan Total Belanja tinggi (misal label "Pelanggan VIP") untuk membantu tim admin memprioritaskan layanan/promo khusus.
- Pastikan data sensitif (no. HP, alamat) hanya dapat diakses oleh role admin yang berwenang (role-based access control).

### 7. Responsive Notes
- Desktop-first dengan tabel data lengkap; versi mobile dapat menampilkan list card ringkas per pelanggan.

### 8. Spesifikasi Komponen (Detail Ukuran)

### Data Table & Detail Panel
| Properti | Nilai |
|---|---|
| Row height | 56px |
| Avatar size | 32×32px |
| Status Toggle | 40×22px |
| Detail Panel width | 400px (side panel, desktop) |

### 9. Design Token yang Digunakan
`radius/md`, `color/danger` (status diblokir), `color/success` (status aktif).

### 10. Animasi
| Elemen | Efek | Durasi | Easing | Trigger |
|---|---|---|---|---|
| Detail Panel | Slide-in dari kanan | 300ms (`motion/base`) | ease-out | Klik baris pelanggan |
| Status Toggle | Slide knob + badge color update | 150ms | ease-out | Blokir/Aktifkan |

### 11. Acceptance Criteria
- [ ] Blokir akun mewajibkan admin mengisi alasan singkat sebelum konfirmasi.
- [ ] Pelanggan yang diblokir tidak dapat login di sisi customer (`Login` menampilkan pesan sesuai).
- [ ] Data sensitif (no. HP, alamat) hanya terlihat oleh admin dengan permission yang sesuai (role-based access).

### 12. Edge Case

| Kondisi | Trigger | Hasil yang Diharapkan |
|---|---|---|
| Pelanggan baru daftar, belum ada transaksi | `total_orders = 0` | Tetap tampil normal di list dengan nilai 0, bukan error |
| Pencarian tanpa hasil | Query tidak match | Pesan "Pelanggan tidak ditemukan" |

### 13. Developer Notes
- Terapkan audit log untuk aksi blokir/aktifkan akun (siapa admin, kapan, alasan) demi akuntabilitas dan compliance data privasi.

### 14. Priority & Requirement ID

| Priority | **Medium** |
|---|---|

| Req ID | Requirement |
|---|---|
| REQ-ADMIN-PELANGGAN-001 | Blokir akun harus mewajibkan alasan singkat sebelum konfirmasi |
| REQ-ADMIN-PELANGGAN-002 | Data sensitif hanya terlihat oleh admin dengan permission sesuai |

### 15. Dependency

```
Admin Manajemen Pelanggan
 └─ bergantung pada
     └─ Register/Login (customer) — sumber data pelanggan terdaftar

Admin Manajemen Pelanggan ──mempengaruhi──► Login (customer, jika akun diblokir)
```

### 16. Component Relationship

```
Data Table (list pelanggan)
   ↓
Detail Panel (Side Panel: profil + riwayat pesanan + ulasan)
   ↓
Toggle Switch (Blokir/Aktifkan) → mempengaruhi Login (customer)
```
