# Product Requirements Document (PRD): Bagisto 2.5.x

## 1. Executive Summary & Product Vision

Bagisto is an open-source, enterprise-grade e-commerce platform built on the Laravel 13 framework, PHP 8.4+, Vue.js 3, Tailwind CSS 4, and Vite 6. It is designed to deliver exceptional performance, modular extensibility, and global scalability across multiple channels, currencies, and languages. The platform operates on MySQL 8.0+, MariaDB 10.11+, and PostgreSQL 16+ as first-class supported database engines.

The architecture is built around 42 independent, autoloaded packages residing in `packages/Webkul/`. Every data domain leverages the Contract, Model, Proxy, and Repository architectural pattern to guarantee extensibility without modifying core code.

---

## 2. Product Architecture & Technical Foundations

### 2.1 Core Technology Stack
- **Backend Framework**: Laravel 13 on PHP 8.4+.
- **Frontend Architecture**: Vue.js 3, Tailwind CSS 4, Vite 6.
- **Supported Databases**: MySQL (8.0, 8.4, 9.7), MariaDB (10.11, 11.8, 12.3), and PostgreSQL (10, 14, 18).
- **Extension Pattern**: Prettus L5 Repository Pattern, Konekt Concord model/proxy binding, and event-driven hooks.
- **Caching & Acceleration**: Full Page Cache (FPC), Redis/database session and cache drivers, dynamic ImageCache.

### 2.2 Architectural Tenets
- **Contract / Model / Proxy Pattern**: All core entities are defined by interfaces (Contracts) and wrapped with Proxies, allowing third-party packages to substitute core models at runtime without modifying upstream source code.
- **Repository Abstraction**: Direct Eloquent model queries in controllers, listeners, or services are strictly abstracted behind repositories extending `Webkul\Core\Eloquent\Repository`.
- **Event-Driven Lifecycle**: Dot-delimited event pairs (`before` and `after`) fire across all major system lifecycle actions to enable modular extension.

---

## 3. Core Feature Specifications

### 3.1 Catalog & Product Management (`Webkul/Product`, `Webkul/Category`, `Webkul/Attribute`, `Webkul/BookingProduct`)

#### Product Types
The catalog engine provides native support for seven distinct product types:
- **Simple Product**: Physical inventory items with a single SKU, dedicated stock, and direct shipping requirements.
- **Configurable Product**: Single parent products with selectable attribute options (e.g., color, size) mapped to child simple variants with distinct inventory and pricing.
- **Virtual Product**: Non-physical items that require no shipping or physical fulfillment (e.g., memberships, digital warranties).
- **Downloadable Product**: Digital items featuring secure file uploads, download link limits, expiry durations, and sample preview links.
- **Grouped Product**: Collections of distinct simple products presented on a single product page for bundled purchasing.
- **Bundle Product**: Customizable product kits where customers configure multiple options and add-ons with dynamic price calculations.
- **Booking Product**: Specialized reservation products supporting five booking modes:
  - *Appointment Booking*: Slot-based hourly appointments with daily break times.
  - *Event Ticket Booking*: Multi-tier ticket inventory, venue scheduling, and attendee limits.
  - *Rental Booking*: Daily and hourly equipment or property reservations.
  - *Table Booking*: Restaurant and venue table reservation with capacity controls.
  - *Default Booking*: Standard time-slot reservations.

#### Entity-Attribute-Value (EAV) System (`Webkul/Attribute`)
- **Custom Attributes**: Administrators can create unlimited custom attributes across types: text, textarea, boolean, select, multiselect, datetime, date, price, image, file, and checkbox.
- **Attribute Families**: Group attributes into reusable families (e.g., Clothing, Electronics) that dictate product creation forms and validation rules.
- **Validation & Scoping**: Attributes can be scoped globally, per-channel, or per-locale, with configurable frontend visibility, search indexing, and layered navigation filtering.

#### Category Tree (`Webkul/Category`)
- **Nested Set Hierarchy**: High-performance hierarchical tree structure using nested-set database architecture.
- **Channel Scoping**: Independent root category trees configurable per sales channel.
- **Display Configurations**: Category pages support products-only, description-only, or combined display modes with custom banners and rich text descriptions.

#### Dynamic Pricing Engines
- **Tier Pricing**: Quantity-based volume discounts per product.
- **Customer Group Pricing**: Segmented pricing rules tailored to specific customer groups (e.g., Wholesale, VIP).
- **Special Pricing**: Scheduled promotional prices with strict start and end dates.

---

### 3.2 Inventory & Multi-Warehouse Fulfillment (`Webkul/Inventory`)

- **Multi-Inventory Sources**: Define multiple physical warehouses, retail locations, or distribution centers.
- **Channel Source Allocation**: Link multiple inventory sources to specific channels with configurable priority weighting.
- **Stock Tracking & Backorders**: Real-time stock decrementing, backorder allowance toggles, and minimum inventory thresholds with low-stock email triggers.

---

### 3.3 Sales & Order Management (`Webkul/Sales`, `Webkul/Checkout`)

#### Order Lifecycle Pipeline
- **Order Processing**: Complete transactional state machine managing status transitions: Pending, Processing, Completed, Cancelled, Closed, and Fraud.
- **Invoicing**: Partial and full invoice generation with automated PDF invoice rendering.
- **Shipments**: Partial and full shipment creation with carrier tracking code assignment.
- **Refunds & Credit Memos**: Full or partial refunds against invoices with automatic inventory restitution.
- **Transactions**: Audit-compliant tracking of gateway transaction identifiers and authorization codes.

#### Checkout Workflow (`Webkul/Checkout`)
- **Flexible Checkout Flow**: Support for guest checkout, registered customer checkout, and saved multi-address checkout.
- **Address Validation**: Billing and shipping address capture with dynamic country/state selector integrations.
- **Order Review**: Transparent line-item breakdowns including discounts, taxes, and shipping rates prior to final placement.

---

### 3.4 Promotions, Coupons & Rules (`Webkul/CartRule`, `Webkul/CatalogRule`, `Webkul/Rule`)

- **Cart Promotion Rules (`CartRule`)**: Automated or coupon-code-triggered discounts based on complex cart conditions (cart subtotal, quantity, total weight, payment method, shipping method, or customer group).
- **Cart Actions**: Percentage discounts, fixed amount discounts, free shipping overrides, and Buy X Get Y promotions.
- **Catalog Price Rules (`CatalogRule`)**: Pre-calculated catalog-wide discounts scheduled across categories and attribute conditions.
- **Coupon Code Management**: Single coupon codes or automated bulk generation of unique, single-use coupon pools with expiration dates and usage limits.

---

### 3.5 Returns, Exchanges & RMA (`Webkul/RMA`)

- **Return Merchandise Authorization**: Customer-facing return request pipeline for delivered orders.
- **Resolution Types**: Configurable support for refund, replacement, or repair.
- **Return Conditions**: Customer uploads supporting images and provides reason codes.
- **Order Cancellation**: Configurable window allowing customers to cancel pending orders directly from their account panel.
- **Custom RMA Fields**: Admin-configurable dynamic fields for gathering return details.

---

### 3.6 Payment & Shipping Ecosystem

#### Integrated Payment Gateways (`Webkul/Payment`)
- **Offline Drivers**: Cash on Delivery (COD) and Bank Wire / Money Transfer with custom instructions.
- **Stripe (`Webkul/Stripe`)**: Stripe Elements integration, 3D Secure 2 compliance, and webhook-driven payment confirmation.
- **PayPal (`Webkul/Paypal`)**: PayPal Smart Buttons, PayPal Express Checkout, and standard gateway processing.
- **Razorpay (`Webkul/Razorpay`)**: Native integration supporting UPI, Netbanking, cards, and digital wallets.
- **PayU (`Webkul/PayU`)**: Regional payment processing for credit/debit cards and Netbanking.
- **PhonePe (`Webkul/PhonePe`)**: Direct payment processing supporting UPI, QR code generation, and card transactions.
- **PayGlocal (`Webkul/PayGlocal`)**: Cross-border international transactions with currency conversion.

#### Shipping Method Framework (`Webkul/Shipping`)
- **Flat Rate Shipping**: Channel-specific flat fee per order or per item.
- **Free Shipping**: Minimum order threshold-driven free shipping option.
- **Table Rate Shipping**: Dynamic shipping rate calculation based on country, state, postal code, weight range, and order subtotal.

---

### 3.7 Multi-Store, Multi-Currency & Internationalization (`Webkul/Core`, `Webkul/Tax`)

- **Multi-Channel Architecture**: Host multiple distinct storefronts from a single Bagisto instance, each with its own domain, theme, default currency, default locale, inventory sources, and root category.
- **23 Supported Locales**: Pre-packaged translations for 23 languages (Arabic, Bengali, Catalan, German, English, Spanish, Persian, French, Hebrew, Hindi, Indonesian, Italian, Japanese, Dutch, Polish, Brazilian Portuguese, Romanian, Russian, Sinhala, Swahili, Turkish, Ukrainian, Chinese).
- **RTL Support**: Native Right-to-Left script layout alignment for Arabic and Persian.
- **Multi-Currency & Exchange Rates**: Display prices in multiple currencies with automated exchange rate updates via configured external currency APIs.
- **Tax Engine (`Webkul/Tax`)**: Complex tax category mapping, tax rate tables (country, state, postal code range), combined tax rules, and support for inclusive/exclusive pricing display.

---

### 3.8 Customer Experience & Storefront (`Webkul/Shop`, `Webkul/Customer`, `Webkul/SocialLogin`, `Webkul/SocialShare`)

- **Modern Responsive Storefront**: Built on Vue.js 3 and Tailwind CSS 4 with high Core Web Vitals scores and mobile-first responsiveness.
- **Customer Self-Service Portal**: Order history tracking, downloadable link access, address book management, review history, and wishlist management.
- **Interactive Shopping Tools**: Layered navigation with attribute filters, keyword search, product comparisons, and product reviews with star ratings.
- **Social Login (`Webkul/SocialLogin`)**: OAuth single-sign-on integration for Google, Facebook, Twitter/X, GitHub, and LinkedIn.
- **Social Sharing (`Webkul/SocialShare`)**: Integrated social sharing buttons for Facebook, Twitter/X, Pinterest, WhatsApp, and LinkedIn.

---

### 3.9 Administration & Operations (`Webkul/Admin`, `Webkul/User`, `Webkul/DataGrid`, `Webkul/Theme`, `Webkul/CMS`)

- **Analytics Dashboard**: Real-time sales summaries, total revenue, order counts, average order values, top-selling products, customer counts, and stock warning widgets.
- **Interactive DataGrid Framework (`Webkul/DataGrid`)**: Uniform listing engine across all admin entities featuring instant search, multi-column sorting, compound filtering, column show/hide controls, mass actions (status update, deletion), and batch exports to CSV/XLS.
- **Visual Theme Customizer (`Webkul/Theme`)**: Drag-and-and-drop section customizer to configure storefront hero carousels, category carousels, featured product sliders, and custom HTML/image banners.
- **Role-Based Access Control (RBAC) (`Webkul/User`)**: Fine-grained Access Control List (ACL) permissions defining role-level access per navigation menu and controller action, backed by Two-Factor Authentication (2FA).
- **Content Management System (`Webkul/CMS`)**: Rich-text page authoring for static company pages (Terms, About, Privacy Policy) with custom URL keys and channel visibility controls.

---

### 3.10 High-Volume Data Transfer (`Webkul/DataTransfer`)

- **Batch Import Pipeline**: High-throughput queued data import system supporting CSV, XLS, XLSX, and XML formats.
- **Supported Import Domains**: Products (simple, configurable, downloadable, etc.), Categories, Customers, and Tax Rates.
- **Batch Processing & Error Resilience**: Streamed chunking to avoid memory overflow, comprehensive validation reporting, and row-level error logging.

---

### 3.11 Artificial Intelligence Engine (`Webkul/MagicAI`)

- **Generative AI Integration**: Powered by Laravel AI SDK and modern LLM providers.
- **Catalog Automation**: Automated generation of rich product descriptions, bullet points, feature lists, and translated copy.
- **SEO Generation**: One-click generation of SEO meta titles, meta keywords, and meta descriptions.
- **Image Generation**: Automated text-to-image creation for product and category banners.

---

### 3.12 Regulatory Compliance & Consumer Protection (`Webkul/GDPR`, `Webkul/EUWithdrawal`, `Webkul/Omnibus`)

- **GDPR Compliance (`Webkul/GDPR`)**: Customer self-service data portability (export all account data in JSON/PDF), data rectification requests, account deletion requests (right to be forgotten), and cookie consent banners.
- **EU Statutory Withdrawal (`Webkul/EUWithdrawal`)**: Compliant 14-day statutory withdrawal request submission form, automated email notification, and admin withdrawal request review pipeline.
- **EU Price Transparency (`Webkul/Omnibus`)**: Automatic tracking and display of the lowest price charged for a product during the preceding 30 days to prevent deceptive promotional pricing.

---

### 3.13 Marketing & SEO (`Webkul/Marketing`, `Webkul/Sitemap`)

- **SEO & URL Rewrites**: Custom URL keys for products, categories, and CMS pages with automatic 301/302 redirects on slug modifications.
- **Search Terms Analytics**: Track storefront search queries, query frequency, and map search terms to specific landing URLs.
- **XML Sitemaps (`Webkul/Sitemap`)**: Automated generation of search-engine-ready XML sitemaps per channel.
- **Email Newsletters**: Subscription management and promotional email campaign dispatch.

---

## 4. Non-Functional Requirements (NFRs)

### 4.1 Performance & Scalability
- **Page Response Times**: Cached pages served via Full Page Cache (FPC) must return within 200ms.
- **Database Scalability**: All database queries must be optimized for execution under 50ms, with zero N+1 query regressions across product listings and checkout.
- **Asset Optimization**: Vite 6 build pipelines produce minified, tree-shaken CSS and JS chunks with asset hashing for optimal CDN cacheability.

### 4.2 Security Standards
- **Authentication & Sessions**: Passwords hashed with Bcrypt (cost factor 12). Sessions encrypted with secure, HTTP-only, SameSite cookies.
- **CSRF & Injection Defenses**: Strict Cross-Site Request Forgery token validation on all state-altering requests. Parameterized SQL queries via Eloquent PDO bindings to prevent SQL injection.
- **Input Sanitization**: SVG and HTML content sanitized via SVG-Sanitize and Purifier before storage and rendering.

### 4.3 Database Compatibility & Reliability
- **Multi-Engine Support**: Every schema migration, index, and raw SQL query must function identically across MySQL 8.0+, MariaDB 10.11+, and PostgreSQL 16+.
- **Transaction Safety**: All multi-step order placement, invoice capture, and refund operations must run inside atomic database transactions with proper rollback handling.

