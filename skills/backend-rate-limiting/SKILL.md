name: backend-rate-limiting
description: Gateway ve servislerde rate limiting standardı; brute-force ve abuse’a karşı Redis-backed limit politikaları.

---

## Purpose

Public endpoint’leri brute-force, scraping ve abuse senaryolarına karşı koruyup sistem stabilitesini artırmak.

## Rules

- **Zorunlu katman**: Rate limiting en az gateway seviyesinde uygulanır; kritik endpoint’lerde (login/register/OTP) servis seviyesinde ek limit uygulanır.
- **Redis-backed zorunlu**: Tek instance memory limiter **yasak** (container scale’da tutarsız olur).
- **Limit boyutları**:
  - **IP bazlı** + **fingerprint/device-id** (varsa) + **userId** (auth sonrası) kombinasyonu tercih edilir.
  - Sadece IP’ye bağlı limiter (NAT/kurumsal ağlarda) tek başına yeterli kabul edilmez.
- **Politika tablolaştırma**: Her limiter için `windowMs`, `max`, kapsam (route) ve anahtar stratejisi (ip/userId/deviceId) açıkça tanımlanır; “rastgele limit” **yasak**.
- **Login koruması**:
  - Başarısız denemelerde adaptive/backoff uygulanması tercih edilir.
  - Username/email enumeration’a yol açacak farklı hata mesajları **yasak**.
- **Response standardı**: Limit aşımlarında da backend standart response şeması bozulmaz; hata `next(error)` ile middleware’e gider.
- **Retry/abuse sinyali**: Rate limit tetiklendiğinde log’lanır (PII olmadan) ve mümkünse metrik üretilir.

## References

- `skills/backend-standard-response`
- `skills/observability-logging-metrics`
