name: observability-logging-metrics
description: Backend servislerinde standart log/metric kuralları; request correlation, hata sınıflama ve temel metrik seti.

---

## Purpose

Üretimde “neden bozuldu?” sorusuna hızlı cevap verebilmek için servislerde **tutarlı log**, **ölçülebilir metrik** ve **correlation** standardı sağlamak.

## Rules

- **Structured logging**: Loglar JSON/structured formatta üretilir; serbest metin log standardı **yasak** (debug hariç).
- **Correlation ID zorunlu**:
  - Her HTTP isteğinde `requestId` üretilir/propagate edilir.
  - Gateway → servisler arası çağrılarda `requestId` forward edilir.
- **Log alanları (minimum)**:
  - `service`, `env`, `requestId`, `route`, `method`, `statusCode`, `durationMs`
  - Hata varsa: `errorType`, `customMessage`, `stack` (prod’da stack exposure politikası ayrıca belirlenir)
- **PII/secret yasağı**: Token, şifre, OTP, tam e-posta, belge URL’leri vb. log’a yazılmaz.
- **Hata sınıflama**:
  - Validation, auth, notFound, conflict, internal gibi sınıflar tutarlı kodlanır.
  - Aynı hata türü farklı servislerde farklı isimlendirme ile log’lanmaz.
- **Temel metrik seti** (minimum):
  - HTTP request count (route/status/method label’ları ile)
  - HTTP duration histogram (P95/P99 izlenebilir)
  - Error count (type label’ı ile)
  - DB query duration (varsa)
  - Cache hit ratio (varsa)
- **Admin kritik aksiyon logu**:
  - Admin panel üzerinden yapılan kritik aksiyonlar (onay/red/silme/ban) audit log olarak kaydedilir (PII minimize).

## References

- `skills/backend-controller-service-flow`
- `skills/backend-standard-response`
