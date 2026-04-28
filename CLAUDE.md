# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

File này cung cấp hướng dẫn cho Claude Code (claude.ai/code) khi làm việc trong repository này.

## Chạy dự án

Mở bằng **VS Code Live Server** trên port `5501` (cấu hình trong `.vscode/settings.json` và `.vscode/.vscode/settings.json`). Không có bước build, không có dependency, không có package manager — chỉ cần mở file HTML trực tiếp trên trình duyệt hoặc qua Live Server.

## Kiến trúc

Landing page tĩnh cho **LadyLogin** (Anti-Detect Browser). Dự án gồm hai trang:

### Trang chính: `.vscode/index.html`
- Toàn bộ markup + JavaScript inline ở cuối file
- Style từ `.vscode/style.css`

### Trang bảng giá: `.vscode/.vscode/banggia.html`
- Link cả hai CSS: `../style.css` (dùng chung CSS variables và component classes) + `../../banggia.css` (styles riêng)
- `banggia.css` nằm ở **root** (`c:\test\banggia.css`), không phải trong `.vscode/`

### Style chung: `.vscode/style.css`
CSS custom properties qua `:root`, không dùng preprocessor. Các biến quan trọng:
- `--brand: #7c3aed` (purple), `--brand-soft`, `--brand-muted`, `--brand-dark`
- `--bg`, `--bg-soft`, `--bg-muted`, `--text`, `--text-2`, `--muted`, `--line`
- `--radius-sm / --radius / --radius-lg / --radius-xl`
- `--shadow-sm / --shadow / --shadow-lg`

---

## Scroll Reveal Animation (quan trọng)

`style.css` set `opacity: 0; transform: translateY(28px)` cho nhiều elements (`.pricing-card`, `.feature-card`, `.dl-card`, `.section-head`, `details`, v.v.). Class **`.in-view`** mới làm chúng hiện ra.

Trong `index.html`, IntersectionObserver tự động xử lý. Trong **các trang con** (`banggia.html`, v.v.) phải tự thêm IntersectionObserver:

```js
const revealObserver = new IntersectionObserver((entries) => {
  entries.forEach(e => { if (e.isIntersecting) { e.target.classList.add('in-view'); revealObserver.unobserve(e.target); } });
}, { threshold: 0.08 });
document.querySelectorAll('.pricing-card, .section-head, .dl-card, details, ...').forEach(el => revealObserver.observe(el));
```

Nếu thêm element mới dùng class có animation trong `style.css` vào trang con → phải thêm selector vào observer.

---

## Cấu trúc sections của index.html

| Section | id / class | Mô tả |
|---|---|---|
| Announcement Bar | `.announcement-bar` | Dải thông báo trên cùng |
| Header | `.header` | Nav + dropdowns + language toggle + CTA |
| Hero | `.hero` | Headline trái + dashboard panel phải |
| Brands | `.brands-section` | Logo marquee tĩnh |
| Stats | `.stats-section` | 4 số liệu: 50k+ users, 2M+ profiles, 99.9% uptime, 100+ quốc gia |
| Features | `#features` | 6 feature card |
| How It Works | `#how` | 3 bước setup + 2 testimonial |
| Pricing | `#pricing` | 3 gói: Starter (0đ), Growth (690K/tháng), Scale (Liên hệ) |
| Partner Bar | `.partner-section` | Infinite scroll marquee 14 platform, nhân đôi item để tạo loop |
| Use Cases | `#usecases` | Layout tab dọc trái + iPad mockup phải, 9 tab |
| Fingerprint | `#fingerprint` | Danh sách tính năng trái + code panel phải |
| Security | `.security-section` | 4 card bảo mật |
| FAQ | `#faq` | `<details>` native HTML |
| CTA Banner | `.cta-section` | Call-to-action toàn chiều rộng |
| Download | `#download` | 3 card: Windows, macOS, Web App |
| Footer | `.footer` | Brand + contact, 3 cột link, payment badges |

---

## JavaScript của index.html

Tất cả JS trong một khối `<script>` cuối file:

- **`switchTab(btn, panelId)`** — 9 panel iPad trong Use Cases; toggle class `active` trên `.uc-tab` và `.ipad-panel`
- **Nav dropdowns** — `mouseenter`/`mouseleave` trên `.nav-dropdown` toggle class `.open` với delay 80ms; áp dụng cho 3 dropdown (Tính Năng, Giải Pháp, Tài Nguyên)
- **`applyLang(lang)` / `toggleLang()`** — query `[data-i18n]` và set `innerHTML` từ object `translations`; cũng xử lý `[data-i18n-html]`
- **Scroll progress bar** — inject `<div class="scroll-progress">` vào `document.body`, width theo `scrollY / scrollHeight`
- **Header `.scrolled`** — thêm class khi scroll > 60px
- **Section fade-in** — `IntersectionObserver` thêm class **`.in-view`** (không phải `visible`)
- **Nav active** — highlight link nav theo section đang scroll vào viewport

## JavaScript của banggia.html

- **`setBilling(period, btn)`** — toggle Tháng/Quý/Năm, gọi `updatePrices()`
- **`selectProfile(btn, plan, count)`** — chọn số profile cho gói Pro
- **`changeTeam(plan, delta)`** — stepper thành viên nhóm cho Pro và Biz
- **`changeBizProfile(delta)`** — stepper số profile cho gói Biz
- **`updatePrices()`** — tính lại giá theo period + profiles + team, format với `formatPrice()`
- **`renderComp()`** — render comparison table từ mảng `COMP_DATA`
- **`toggleComp()`** — animate show/hide bảng so sánh bằng `scrollHeight` (không dùng `display:none`); quản lý `overflow` inline để `position: sticky` hoạt động khi mở
- **FAQ animation** — intercept click trên `<details>`, animate `.faq-a` với `max-height` + `opacity`
- **Count-up** — `IntersectionObserver` trigger đếm số trong `.trust-stat` khi scroll đến
- **Scroll progress bar** — giống index.html
- **Scroll hints** — `.plan-scroll-hint`, `.dl-scroll-hint`: click cuộn sang card tiếp theo (cardWidth + 12px gap); khi đến cuối thì `scrollTo(0)` quay về đầu. Bảng comparison có `.comp-scroll-hint` (hiện tại ẩn vì bảng fit vừa màn hình mobile)

---

## Responsive Mobile — banggia.html

`banggia.css` **không override** `.pricing-grid` với `!important` — để `style.css` tự handle horizontal scroll ở `≤768px` (mỗi card `min(300px, 78vw)`, `scroll-snap-type: x mandatory`). Tương tự `.dl-grid` scroll ngang từ `≤860px`.

Breakpoints trong `banggia.css`:

| Breakpoint | Thay đổi chính |
|---|---|
| `≤ 900px` | `.config-row` stack dọc, `.profile-btns` wrap |
| `≤ 860px` | `.dl-scroll-hint` hiện |
| `≤ 768px` | `.plan-scroll-hint` hiện; trust stats → 2×2 grid; bảng comparison thu nhỏ font/padding để 4 cột vừa màn hình (không scroll ngang) |
| `≤ 640px` | Giảm section padding, billing toggle wrap, FAQ padding nhỏ hơn |
| `≤ 480px` | Price amount 32px, trust stat số 26px |

### Comparison table — scroll wrapper
Bảng được wrap trong `<div class="comp-table-scroll">` bên trong `comp-table-wrap`. Mục đích: `overflow-x: auto` chỉ ảnh hưởng bảng, không block `position: sticky` trên `thead th` ở desktop. Trên mobile (`≤768px`), bảng thu nhỏ font thay vì scroll ngang.

---

## Quy tắc i18n (chỉ áp dụng index.html)

Mọi chuỗi text **phải** có `data-i18n="key"` và entry trong **cả hai** `translations.vi` và `translations.en`.

Khi element chứa node con (ví dụ button có SVG), chỉ wrap text trong `<span data-i18n="key">` — đặt `data-i18n` trực tiếp trên element cha sẽ xóa mất SVG khi `innerHTML` được set.

`banggia.html` hiện chưa có i18n.

---

## Cấu trúc nav dropdown (index.html)

Ba loại dropdown, dùng chung hover JS và CSS `visibility/opacity` transition:

- **Tính Năng / Features** — danh sách dọc (`.nav-dropdown-item` với `.nav-di-icon` + `.nav-di-text`)
- **Giải Pháp / Solutions** — mega menu (`.nav-mega-menu`): header strip + grid 3 cột (`.nav-mega-grid` → `.nav-mega-item`)
- **Tài Nguyên / Resources** — danh sách dọc thông thường

Dùng `padding-top: 16px` thay vì `margin-top` trên dropdown menu để tránh gap làm mất hover state.
