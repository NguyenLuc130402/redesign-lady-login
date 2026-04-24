# CLAUDE.md

File này cung cấp hướng dẫn cho Claude Code (claude.ai/code) khi làm việc trong repository này.

## Chạy dự án

Mở bằng **VS Code Live Server** trên port `5501` (cấu hình trong `.vscode/settings.json`). Không có bước build, không có dependency, không có package manager — chỉ cần mở `.vscode/index.html` trực tiếp trên trình duyệt hoặc qua Live Server.

## Kiến trúc

Landing page tĩnh một trang cho **LadyLogin** (Anti-Detect Browser). Toàn bộ code nằm trong hai file:

- `.vscode/index.html` — toàn bộ markup + tất cả JavaScript inline ở cuối file
- `.vscode/style.css` — toàn bộ style (CSS custom properties qua `:root`, không dùng preprocessor)

---

## Cấu trúc các section theo thứ tự

| Section | id / class | Mô tả |
|---|---|---|
| Announcement Bar | `.announcement-bar` | Dải thông báo trên cùng |
| Header | `.header` | Nav + dropdowns + language toggle + CTA |
| Hero | `.hero` | Headline trái + dashboard panel phải |
| Brands | `.brands-section` | Logo marquee tĩnh |
| Stats | `.stats-section` | 4 số liệu: 50k+ users, 2M+ profiles, 99.9% uptime, 100+ quốc gia |
| Features | `#features` | 6 feature card (Multi-Profile, Fingerprint Spoofing, Proxy, Team Permission, Automation & API, Analytics) |
| How It Works | `#how` | 3 bước setup + 2 testimonial |
| Pricing | `#pricing` | 3 gói: Starter (0đ), Growth (690K/tháng), Scale (Liên hệ) |
| Partner Bar | `.partner-section` | Infinite scroll marquee 14 platform (Facebook → Temu), nhân đôi item để tạo loop |
| Use Cases | `#usecases` | Layout tab dọc trái + iPad mockup phải, 9 tab: eCommerce, Affiliate, Social Media, Crypto, Web Scraping, Quảng cáo, Agency, Game & Airdrop, SEO & SERP |
| Fingerprint | `#fingerprint` | Danh sách tính năng trái + code panel giả lập fingerprint phải |
| Security | `.security-section` | 4 card: Quyền hồ sơ, Mã hóa, Extension Protection, Bug Bounty |
| FAQ | `#faq` | 5 câu hỏi dạng `<details>` native HTML |
| CTA Banner | `.cta-section` | Call-to-action toàn chiều rộng |
| Download | `#download` | 3 card: Windows, macOS, Web App |
| Footer | `.footer` | Brand + contact, 3 cột link, payment badges, copyright |

---

## JavaScript inline (cuối index.html)

Tất cả JS trong một khối `<script>`. Các hệ thống chính:

- **`switchTab(btn, panelId)`** — điều khiển 9 panel iPad trong section Use Cases; toggle class `active` trên `.uc-tab` và `.ipad-panel`
- **Nav dropdowns** — `mouseenter`/`mouseleave` trên `.nav-dropdown` toggle class `.open` với delay 80ms để tránh nhấp nháy khi di chuột từ button sang menu; áp dụng cho cả 3 dropdown (Tính Năng, Giải Pháp, Tài Nguyên)
- **`applyLang(lang)`** — query tất cả `[data-i18n]` và set `innerHTML` từ object `translations`; cũng xử lý `[data-i18n-html]` tương tự
- **`toggleLang()`** — chuyển đổi giữa `vi` và `en`, gọi `applyLang`
- **Scroll progress bar** — `<div class="scroll-progress">` inject vào `document.body`, width theo `scrollY`
- **Header `.scrolled`** — thêm class khi scroll > 20px
- **Section fade-in** — `IntersectionObserver` thêm class `visible` vào các `.section`
- **Nav active** — highlight link nav theo section đang scroll vào viewport

---

## Quy tắc i18n (bắt buộc)

Mọi chuỗi text hiển thị cho người dùng **phải** có `data-i18n="key"` và entry trong **cả hai** `translations.vi` và `translations.en`.

Khi element chứa node con (ví dụ button có SVG chevron), chỉ wrap phần text trong `<span data-i18n="key">` thay vì đặt `data-i18n` trực tiếp trên element cha — nếu không `innerHTML` sẽ xóa mất SVG.

---

## Cấu trúc nav dropdown

Ba loại dropdown trong header, đều dùng chung hover JS và CSS transition visibility/opacity:

- **Tính Năng / Features** — danh sách dọc thông thường (`.nav-dropdown-item` với `.nav-di-icon` + `.nav-di-text`)
- **Giải Pháp / Solutions** — mega menu (`.nav-mega-menu`) với header strip (`.nav-mega-header` gồm `.nav-mega-tag` + `.nav-mega-sub`) và grid 3 cột (`.nav-mega-grid` → `.nav-mega-item`)
- **Tài Nguyên / Resources** — danh sách dọc thông thường

Dropdown menu dùng `padding-top: 16px` thay vì `margin-top` để tránh gap làm mất hover state khi di chuột từ button xuống menu.
