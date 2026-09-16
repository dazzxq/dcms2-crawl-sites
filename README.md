# dcms2-crawl-sites

Danh sách CSS selectors để crawl bài viết từ các trang báo điện tử Việt Nam.

DCMS2 instances (zmag, plcs, banquyen, ...) đồng bộ list này qua admin button **"Đồng bộ với upstream"** trong page Crawl Sites. Pull-only, manual trigger — không tự động sync.

## Source URL

```
https://raw.githubusercontent.com/dazzxq/dcms2-crawl-sites/main/crawl-sites.json
```

(Public, no auth required, GitHub CDN-cached.)

## Schema

```json
{
  "version": 1,
  "updated_at": "ISO-8601 timestamp",
  "sites": [
    {
      "domain": "example.vn",
      "name": "Hiển thị name",
      "selectors": {
        "title":   "CSS selector cho tiêu đề",
        "content": "CSS selector cho body content",
        "excerpt": "CSS selector cho sapo / mô tả ngắn",
        "author":  "CSS selector cho tên tác giả",
        "date":    "CSS selector cho ngày đăng",
        "exclude": "Comma-separated CSS selectors để LOẠI BỎ khỏi content (vd quảng cáo, related news)"
      }
    }
  ]
}
```

### Field rules

- **domain**: hostname KHÔNG có scheme (`tuoitre.vn`, KHÔNG `https://tuoitre.vn`). Unique key — DCMS2 upserts theo domain.
- **name**: tên hiển thị trong admin UI. Tiếng Việt có dấu OK.
- **selectors**: mọi key đều OPTIONAL trừ `title` + `content`. Nếu site không có `excerpt`/`author`/`date` rõ ràng, bỏ qua.
- **exclude**: comma-separated CSS selectors (advanced — multiple targets). Dùng để strip ads, related-news boxes, share buttons.
- **note** (optional, cấp site): ghi chú tự do cho người bảo trì registry. `CrawlSitesSyncService` chỉ đọc
  `domain` / `name` / `selectors` nên key này được bỏ qua khi sync — dùng để đánh dấu site có caveat đã biết.

## Cập nhật site mới

**Cách 1 — GitHub web UI (nhanh nhất):**
1. Mở https://github.com/dazzxq/dcms2-crawl-sites/blob/main/crawl-sites.json
2. Click pencil icon (Edit)
3. Thêm site mới vào array `sites`, bump `version` + `updated_at`
4. Commit message: `feat: add <domain>`
5. Commit to main
6. Trong DCMS2 admin: click **"Đồng bộ với upstream"** → site mới xuất hiện

**Cách 2 — Local git:**
```bash
git clone https://github.com/dazzxq/dcms2-crawl-sites.git
cd dcms2-crawl-sites
# Edit crawl-sites.json
git commit -am "feat: add <domain>"
git push origin main
# Trigger DCMS2 sync via admin UI
```

## Test selectors trước khi commit

DCMS2 admin UI có button **"Test crawl"** — paste 1 URL bài viết bất kỳ thuộc domain → xem selector extract đúng chưa. Recommend test ít nhất 3 URL bài khác nhau (article cũ + mới + có/không author).

## Convention

- Bump `version` integer khi schema thay đổi (breaking change). DCMS2 sync warn nếu `version > supported_version`.
- `updated_at` ISO 8601 UTC — dùng để DCMS2 hiển thị "lần update cuối" trong sync UI.
- 1 commit = 1 logical change (1 site mới, hoặc fix 1 selector). Atomic + dễ revert.
- KHÔNG commit selectors chưa test thực tế trên ≥3 URL bài viết.

## Sites hiện có

Nguồn: *Danh sách các đơn vị báo chí được trích, dẫn tin bài* (hiệu lực từ tháng 6/2026).
Toàn bộ selector dưới đây được verify offline bằng đúng pipeline của `ArticleCrawlerService`
(curl không JS, UA `DCMS CrawlBot/1.0`, Symfony DomCrawler/CssSelector, exclude chạy TRƯỚC khi extract),
mỗi domain 3 URL bài chi tiết khác nhau.

Cột **STT** giữ nguyên số thứ tự trong văn bản gốc để đối chiếu khi rà soát — vì vậy không có số 2
(văn bản gốc không có dòng số 2) và số 3 xuất hiện 2 lần (Thông tấn xã Việt Nam vận hành 2 trang tin).

| STT | Domain | Tên | Trạng thái |
|-----|--------|-----|------------|
| — | tuoitre.vn | Tuổi Trẻ | ✅ Tested zmag prod |
| 1 | baovanhoa.vn | Báo Văn hóa | ✅ 3 URL |
| 3 | vietnamplus.vn | VietnamPlus (TTXVN) | ✅ 3 URL |
| 3 | baotintuc.vn | Báo Tin tức (TTXVN) | ✅ 3 URL |
| 4 | daidoanket.vn | Báo Đại Đoàn Kết | ✅ 3 URL |
| 5 | congthuong.vn | Báo Công Thương | ✅ 3 URL |
| 6 | baophapluat.vn | Báo Pháp luật Việt Nam | ✅ 3 URL |
| 7 | vnexpress.net | VnExpress | ✅ 3 URL |
| 8 | vietnamnet.vn | VietNamNet | ✅ 3 URL (2 template) |
| 9 | dantri.com.vn | Báo Dân trí | ✅ 3 URL |
| 10 | doanhnghiepvn.vn | Tạp chí Doanh nghiệp Việt Nam | ✅ 3 URL |
| 11 | vjst.vn | Tạp chí Khoa học và Công nghệ Việt Nam | ✅ 3 URL |
| 12 | thanhnien.vn | Báo Thanh Niên | ✅ 3 URL |
| 13 | laodong.vn | Báo Lao Động | ⚠️ selector OK, fetch bị chặn — xem *Known issues* |
| 14 | tienphong.vn | Báo Tiền Phong | ✅ 3 URL |
| 15 | phunuvietnam.vn | Báo Phụ nữ Việt Nam | ✅ 3 URL |
| 16 | baovephapluat.vn | Báo Bảo vệ pháp luật | ✅ 3 URL |
| 17 | thethaovanhoa.vn | Báo Thể thao & Văn hóa | ✅ 3 URL |
| 18 | lsvn.vn | Tạp chí điện tử Luật sư Việt Nam | ✅ 3 URL |
| 19 | moitruonggiaothong.vn | Tạp chí Môi trường Giao thông | ✅ 3 URL |
| 20 | sohuutritue.net.vn | Tạp chí Sở hữu trí tuệ và Sáng tạo | ✅ 3 URL |
| 21 | abei.gov.vn | Cục Phát thanh, truyền hình và thông tin điện tử | ✅ 3 URL |
| 22 | bvhttdl.gov.vn | Cổng TTĐT Bộ VHTTDL | ✅ 3 URL |
| 23 | timc.vn | Trung tâm Thông tin (TIMC) | ✅ 3 URL |
| — | game.gov.vn | Cổng thông tin chính thức về Game Online | ⚠️ 9 URL — field đúng, đầu content dính header, xem *Known issues* |
| — | kol.gov.vn | Cổng thông tin nhà sáng tạo nội dung số | ✅ 9 URL |

## Ghi chú thiết kế selector

**`exclude` chạy TRƯỚC mọi field extraction** (`extractWithSelectors()`), trên cùng một DOM.
Nên nếu exclude một node thì field trỏ tới node đó cũng mất theo. Có 3 site mà `<h1>` / sapo
nằm *bên trong* container body, buộc phải chọn: để lại rác trong `content`, hoặc cắt và cho
title/sapo rơi xuống tier 2 (JSON-LD) / tier 3 (OpenGraph). Đã chọn cắt, vì tier 2/3 của các
site này cho chuỗi sạch hơn:

| Site | Cắt gì khỏi body | Title/sapo lấy từ |
|------|------------------|-------------------|
| vnexpress.net | `article.fck_detail > h1.title-detail` + `> p.description` | JSON-LD `NewsArticle` (headline/description sạch, không dính hậu tố "- Báo VnExpress") |
| vjst.vn | `.sc-longform-header-cate/-title/-sapo` | JSON-LD. Giữ lại `.sc-longform-header-meta` để `author`/`date` lấy bằng CSS — JSON-LD `author` của vjst bị escape HTML entity (`Xu&#226;n B&#236;nh`) |
| baovanhoa.vn | `.detail__summary` (sapo nằm trong `#abody`) | `og:description` (bằng đúng full sapo) |

**Không dùng JSON-LD cho `thethaovanhoa.vn`**: `headline`/`description` trong JSON-LD của site
này bị double-escape entity, nên `title` + `excerpt` bắt buộc trỏ CSS (`h1.title`,
`.detail-content > p:first-child`).

**`date` không thể trỏ `meta[...]`**: field `date` extract bằng `->text()`, thẻ `<meta>` không có
text node ⇒ luôn rỗng. Site nào không có node ngày hiển thị sạch thì bỏ hẳn key `date`, để
tier 2 lấy `datePublished` (ISO) — áp dụng cho daidoanket, baophapluat, dantri, lsvn.

**Comma trong selector = fallback chain**, không phải "match nhiều node":
`extractWithSelectors()` split theo `,` rồi lấy candidate ĐẦU TIÊN có match. Dùng cho
`vietnamnet.vn` (2 template detail) và `phunuvietnam.vn`.

**Chỉ dùng CSS3**: Symfony CssSelector là bản port của `cssselect`, không hỗ trợ `:has()`.
Toàn bộ selector trong file đã được gate qua `GenericTranslator().css_to_xpath()`.

## Known issues

- **laodong.vn** — ⚠️ **chưa dùng được trên prod.** Mọi request đầu tiên bị chặn bằng JS challenge (trả 177 byte
  `document.cookie="D1N=..."; location.reload()`). `fetchHtml()` của DCMS2 dùng curl thuần,
  không chạy JS và không giữ cookie ⇒ crawl sẽ fail ở tầng fetch, **không phải do selector**.
  Selector trong file đã verify trên 3 URL bằng cách set sẵn cookie `D1N`. Muốn chạy được
  trên prod thì `ArticleCrawlerService::fetchHtml()` cần: phát hiện body < ~500 byte có
  `document.cookie=`, parse cặp cookie đó rồi retry 1 lần với `CURLOPT_COOKIE`.
  Entry vẫn nằm trong registry (kèm key `note` in-band) để không mất công verify lại khi fetch layer được vá;
  hiện tại nếu ai bấm "Test crawl" sẽ nhận lỗi *"Không thể trích xuất nội dung từ URL này"* — fail an toàn,
  không sinh dữ liệu sai. Nếu muốn ẩn khỏi UI thì tắt `is_active` của row `laodong.vn` trong DCMS2 sau lần sync đầu
  (sync chỉ set `is_active = true` lúc INSERT, các lần update sau giữ nguyên cờ này).
- **abei.gov.vn / timc.vn** — caption ảnh nằm ở `<em>` hoặc `<p>` không class ngay sau ảnh.
  `cleanContent()` unwrap `<em>` trước khi `detectCaption()` chạy, nên caption không gắn được
  vào `dcms-object`; chữ vẫn còn trong body dưới dạng text thường.
- **abei.gov.vn** — không có `og:image`, `avatar` fallback sang ảnh đầu tiên trong bài.
- **game.gov.vn** — ⚠️ **content dính header bài.** Trong `main article`, thân bài bị Next.js render thành
  N khối `div.prose-vn` *ngang hàng* với `h1` / byline / sapo (giữa các khối là `div.my-6` rỗng).
  `extractWithSelectors()` chỉ lấy `->first()->html()` nên không có selector nào gom đủ các khối.
  Đã cân nhắc 3 hướng (debate với Codex, đồng thuận):
  - `content: .prose-vn` → **mất ngầm** phần thân thứ 2 trở đi ⇒ loại.
  - `content: main article` + exclude cả `h1`/byline/sapo → content sạch nhưng site không có JSON-LD,
    OG cho `excerpt` bị cắt "…" (hoặc là câu giới thiệu chung của portal), `meta[name=author]` là tên portal
    (editor tự thêm làm tác giả bài), `date` = null ⇒ **sai ngầm** ⇒ loại.
  - **Đang dùng:** `content: main article`, chỉ exclude breadcrumb/chip/share/khối rỗng/tin cùng chuyên mục.
    title/excerpt/author/date lấy đúng bằng CSS; đổi lại đầu content còn *tiêu đề + "Tác giả: … giờ đăng … lượt xem" + sapo*,
    editor phải xoá tay sau khi crawl. `.source-note` ("Nguồn: …") cuối bài được giữ nguyên.
  Fix triệt để phải vá DCMS2, ví dụ: option cho `content` nối HTML của *mọi* node match, hoặc chạy `exclude`
  trên bản clone chỉ dùng cho content (sau khi đã extract các field metadata).
  Thêm: giờ hiển thị trong byline SSR là giờ **UTC** (khớp `article:published_time` có hậu tố `Z`), chậm 7 tiếng so với giờ VN.
- **kol.gov.vn** — `content` trỏ class CSS-module có hash (`post-content_content__yKTZY`) nên dùng
  `[class*="post-content_content"]` để không vỡ khi site build lại. Bài không có sapo thì `excerpt` rơi xuống
  `og:description` (một số bài là đoạn đầu thân bài bị cắt "…"). Bài chỉ có infographic (vd *Bộ quy tắc ứng xử…*) cho content toàn ảnh, 0 ký tự text.
  `date` dạng `dd/mm/yyyy`: màn preview "Test crawl" format bằng `Carbon::parse()` vốn hiểu `11/09/2026` là 9/11 —
  lỗi có sẵn của DCMS2 (chỉ ảnh hưởng hiển thị preview, editor không dùng field `date`).
- **moitruonggiaothong.vn** — node byline gộp cả tên tác giả lẫn giờ đăng
  (`Đỗ Khuyễn - 07:15 15/06/2026 GMT+7`), không tách được ⇒ bỏ key `author`.

## Schema versioning

| Version | Changes |
|---------|---------|
| 1 | Initial schema (domain, name, selectors{title, content, excerpt, author, date, exclude}) |

> `selectors` còn nhận thêm 2 key mà `CrawlSiteService` validate nhưng README v1 chưa ghi:
> `avatar` (featured image — hỗ trợ `<img>`, `<meta>`, hoặc element bọc ảnh) và
> `image_caption` (chỉ áp dụng khi ảnh nằm trong `<figure>`).
