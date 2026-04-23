name: backend-input-validation
description: Backend’te input validation standardı; controller’da doğrulama zorunluluğu, tip/format/limit kuralları ve standart hata üretimi.

---

## Purpose

Backend endpoint’lerinde eksik/yanlış/zararlı input’u daha içeri girmeden yakalayıp **tutarlı**, **güvenli** ve **öngörülebilir** bir hata akışı üretmek.

## Rules

- **Validation zorunlu**: Tüm public/protected endpoint’lerde controller, service çağırmadan önce input doğrulaması yapar.
- **Validation kapsamı**: required kontrolü tek başına yeterli değildir; **tip**, **format**, **min/max**, **enum**, **string trim**, **array limit** kuralları uygulanır.
- **Kaynaklar**:
  - `req.body`, `req.query`, `req.params` ve gerekli ise `req.headers` (örn. `x-device-id`) doğrulanır.
- **Tek mekanizma**: Endpoint bazlı doğrulama şeması tek yerde tutulur; controller içinde dağınık `if (...)` bloklarıyla “yarım doğrulama” **yasak**.
- **Bilinmeyen alanlar**: Beklenmeyen alanlar (unknown keys) default olarak **reddedilir** (explicit allow-list).
- **Hata üretimi**: Validation hatalarında controller `next(error)` kullanır; controller içinde ad-hoc `res.status(...).json(...)` ile hata dönmek **yasak**.
- **Hata içeriği**:
  - `customMessage`: Kullanıcıya/ekrana uygun kısa mesaj (örn. “Geçersiz istek”).
  - `details`: Hangi alanların neden geçersiz olduğu (örn. `email: invalid_format`).
  - PII/secret sızdırmak **yasak** (şifre, token, tam e-posta vb. log/hata detayına basılmaz).
- **Sanitization**: String alanlarında trim uygulanır; XSS riskli alanlarda HTML/JS içerik kabul politikası açıkça belirlenmeden “serbest metin” geniş izin verilmez.
- **DTO disiplini**: Service’e yalnızca doğrulanmış ve normalize edilmiş DTO geçirilir; service’in `req` üzerinden veri çekmesi **yasak**.

## References

- `skills/backend-controller-service-flow`
- `skills/backend-standard-response`
- `skills/general-naming-language`
