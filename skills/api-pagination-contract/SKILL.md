name: api-pagination-contract
description: Liste endpoint’leri için pagination sözleşmesi; response shape, limit/offset veya cursor standardı ve istemci kullanım kuralları.

---

## Purpose

Feed/liste ekranlarında performans ve tutarlılık için tüm servislerde **tek pagination sözleşmesi** kullanmak.

## Rules

- **Liste endpoint standardı**: Liste dönen her endpoint pagination destekler; “tümünü dön” yaklaşımı **yasak** (admin özel durumları hariç açıkça belirtilir).
- **Parametre standardı**:
  - `page` + `limit` (offset-based) veya `cursor` + `limit` (cursor-based) seçilir ve endpoint bazında keyfi karıştırılmaz.
  - `limit` için üst sınır (max) zorunludur.
- **Response standardı** (minimum):
  - `items`: array
  - `pagination`: object (örn. `{ page, limit, total }` veya `{ nextCursor, limit }`)
  - Body yine backend standart response şemasının `data` alanında taşınır.
- **Total politikası**: `total` pahalı ise endpoint’te opsiyonel hale getirilebilir; fakat karar dokümante edilir ve istemci buna göre tasarlanır.
- **Sıralama deterministik**: Pagination yapılan listeler deterministik sort ile döner (örn. `createdAt desc, id desc`).
- **Cursor tercih kuralı**: Büyük veri seti/akış (feed, posts) için cursor-based yaklaşım tercih edilir; offset-based sadece küçük admin listeleri için varsayılan değildir.

## References

- `skills/backend-standard-response`
- `skills/app-api-integration`
- `skills/admin-api-client-rule`
