# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Chạy dự án

Mở bằng **VS Code Live Server** trên port `5501`. Không có build step, không có package manager — HTML/CSS/JS thuần, mở trực tiếp trên trình duyệt.

## Cấu trúc file

```
c:\test\
├── index.html              ← Trang chính (landing page)
├── banggia.html            ← Bảng giá
├── blog.html               ← Blog
├── taixuong.html           ← Trang tải xuống
├── trungtamdoitac.html     ← Trung tâm đối tác
├── trungtamhotro.html      ← Trung tâm hỗ trợ (có sidebar nav)
├── vechungtoi.html         ← Về chúng tôi
├── gioithieu.html          ← Chương trình giới thiệu/referral
└── css/
    ├── style.css           ← CSS chính (chỉ dùng cho index + banggia)
    ├── banggia.css
    ├── blog.css
    ├── taixuong.css
    ├── trungtamdoitac.css
    ├── trungtamhotro.css
    ├── vechungtoi.css
    └── gioithieu.css
```

## Quy tắc CSS linking (quan trọng)

- **`index.html`** → chỉ `css/style.css`
- **`banggia.html`** → link cả `css/style.css` **và** `css/banggia.css` (dùng shared component classes từ style.css như `.pricing-card`, `.dl-card`, `.btn`)
- **Tất cả trang còn lại** → chỉ link CSS riêng của trang đó, **không link** `css/style.css`

Lý do: `style.css` có scroll reveal rules (`opacity: 0` cho nhiều selectors) gây ẩn content trên trang không có IntersectionObserver tương ứng. Mỗi CSS trang con tự chứa `:root` variables (copy từ `style.css`) và tự định nghĩa mọi thứ cần thiết.

## CSS Variables dùng chung

Mọi CSS file đều copy cùng bộ `:root` này từ `style.css`:

```css
--brand: #7c3aed;  --brand-light: #8b5cf6;  --brand-soft: #f5f3ff;
--brand-muted: #ede9fe;  --brand-dark: #6d28d9;
--text: #0f172a;  --text-2: #334155;  --muted: #64748b;
--bg: #ffffff;  --bg-soft: #f8fafc;  --line: #e2e8f0;
--radius: 12px;  --radius-lg: 16px;  --radius-xl: 24px;
--shadow-sm / --shadow / --shadow-lg / --shadow-brand
```

## Scroll Reveal — pattern chuẩn cho trang con

Thêm vào cuối `<script>` của mỗi trang con:

```js
// Scroll progress bar
const progressBar = document.createElement("div");
progressBar.className = "scroll-progress";
document.body.prepend(progressBar);
window.addEventListener(
  "scroll",
  () => {
    const total = document.documentElement.scrollHeight - window.innerHeight;
    progressBar.style.setProperty(
      "--pct",
      (total > 0 ? (window.scrollY / total) * 100 : 0).toFixed(2) + "%",
    );
  },
  { passive: true },
);

// Scroll reveal
const revealIO = new IntersectionObserver(
  (entries) => {
    entries.forEach((e) => {
      if (e.isIntersecting) {
        e.target.classList.add("in-view");
        revealIO.unobserve(e.target);
      }
    });
  },
  { threshold: 0.08 },
);
document
  .querySelectorAll(".section-head, /* ...selectors... */")
  .forEach((el) => revealIO.observe(el));
```

CSS tương ứng phải có:

```css
.selector {
  opacity: 0;
  transform: translateY(22px);
  transition:
    opacity 0.5s ease,
    transform 0.5s ease;
}
.selector.in-view {
  opacity: 1;
  transform: none;
}
.selector.in-view:hover {
  transform: translateY(-5px);
} /* nếu card có hover */
```

Class reveal dùng `.in-view` (không phải `.visible`). Stagger delay: set `el.style.transitionDelay` trong JS khi observe (không dùng `:nth-child` trong CSS).

## Hero pattern cho trang con

Tất cả trang con dùng hero sáng cùng pattern:

```css
background: linear-gradient(160deg, #fdfcff 0%, #f0ebff 45%, #faf8ff 100%);
/* + ::before radial gradients tím nhạt + ::after grid dot pattern 60px */
```

Shimmer animation trên `h1 span`:

```css
background: linear-gradient(90deg, var(--brand), #a78bfa, var(--brand));
background-size: 200% auto;
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;
animation: shimmer 3s linear infinite;
```

## JavaScript — index.html

Toàn bộ JS inline cuối file. Các function chính:

- `switchTab(btn, panelId)` — 9 panel iPad trong Use Cases
- `applyLang(lang)` / `toggleLang()` — i18n qua `[data-i18n]` attributes
- Nav dropdowns — `mouseenter/mouseleave` với delay 80ms, toggle class `.open`
- Scroll progress, header `.scrolled` (>60px), IntersectionObserver `.in-view`

## JavaScript — banggia.html

- `setBilling(period, btn)` + `updatePrices()` — toggle Tháng/Quý/Năm, tính giá
- `selectProfile` / `changeTeam` / `changeBizProfile` — steppers cấu hình gói
- `renderComp()` / `toggleComp()` — bảng so sánh (dùng `scrollHeight` animate, không `display:none`)
- FAQ animation: intercept `<details>` click, animate `.faq-a` bằng `max-height + opacity`
- Count-up cho `.trust-stat` khi scroll đến

## JavaScript — trungtamhotro.html

SPA-style sidebar navigation:

- `toggleGroup(el)` / `toggleChild(el)` — mở/đóng sidebar accordion
- `showContent(id, title, parent)` — load nội dung từ object `CONTENT`, update breadcrumb, trigger animation bằng `void panel.offsetWidth` trước khi add class `active`
- `triggerSection(group, contentId, title, parent)` — mở group + show content (dùng cho quick links)
- Trên mobile (≤768px), `showContent` tự `scrollIntoView` xuống content area
- Content animation: `.hc-content-inner.active { animation: hcFadeIn .32s ... }`

## Quy tắc i18n (chỉ index.html)

Mọi text phải có `data-i18n="key"` + entry trong cả `translations.vi` và `translations.en`. Khi element chứa SVG con, wrap text trong `<span data-i18n="key">` — không đặt `data-i18n` trực tiếp trên element cha (sẽ xóa mất SVG).

## taixuong.html — đặc điểm riêng

- Dùng **Font Awesome CDN** cho OS icons: `<i class="fa-brands fa-linux">`, `<i class="fab fa-windows">`
- Tab OS chọn hệ điều hành (Windows/macOS/Linux), panel hiển thị tương ứng
- `.features-strip-inner` dùng `repeat(2, 1fr)` — 2 cột ở mọi kích thước

## Responsive Mobile — banggia.html

`banggia.css` không override `.pricing-grid` — để `style.css` tự xử lý horizontal scroll ở ≤768px (`scroll-snap-type: x mandatory`). Tương tự `.dl-grid` từ ≤860px.
