name: backend-controller-service-flow
description: Router -> Controller -> Service -> Model zinciri; try-catch, next(error) delegasyonu ve standart başarılı dönüş formatı kuralları.

---

## Purpose

Backend’te **tutarlı istek akışı** ve **merkezi hata yönetimi** sağlayarak okunabilirlik, test edilebilirlik ve standart API çıktısı üretmek.

## Rules

- **Zincir zorunlu**: `Router -> Controller -> Service -> Model` dışında iş akışı kurulmaz.
- **Controller sorumluluğu**: Sadece `req`’den veri alır, service çağırır, response döner; iş kuralı yazılmaz.
- **Service sorumluluğu**: İş kuralları + orchestration; doğrudan `res`/`req` kullanılmaz.
- **Model/DB erişimi**: Sadece service (veya repository katmanı varsa orası) üzerinden yapılır; controller’dan DB çağrısı **yasak**.
- **Controller try-catch zorunlu**: Tüm async controller fonksiyonları `try { ... } catch (err) { next(err) }` kullanır.
- **Hata delegasyonu**: Controller’da `res.status(...).json(...)` ile hata dönmek **yasak**; her hata `next(error)` ile middleware’e aktarılır.
- **Başarılı dönüş**: Controller, success response’u tek formatla döner; dağınık `res.json` şemaları **yasak**.
- **Early return**: Response döndükten sonra akış devam ettirilmez; çoklu `res.*` çağrısı **yasak**.

## References

- `skills/backend-standard-response`
- `skills/general-naming-language`