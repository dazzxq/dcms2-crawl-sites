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

| Domain | Tên | Trạng thái |
|--------|-----|------------|
| tuoitre.vn | Tuổi Trẻ | ✅ Tested zmag prod |

## Schema versioning

| Version | Changes |
|---------|---------|
| 1 | Initial schema (domain, name, selectors{title, content, excerpt, author, date, exclude}) |
