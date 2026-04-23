name: app-api-integration
description: Flutter’da API entegrasyonunda DioClient + ApiEndpoints zorunluluğu; manuel url/header yazımı yasağı.

---

## Purpose

API çağrılarında **tek merkezden yönetim**, doğru header/token kullanımı ve endpoint karmaşasının önlenmesi.

## Rules

- **Zorunlu kullanım**: Tüm ağ çağrıları `DioClient` üzerinden yapılır.
- **Endpoint kaynağı**: Tüm path’ler `ApiEndpoints` üzerinden gelir; string literal endpoint yazımı **yasak**.
- **Manuel url yasak**: Repository/UI içinde tam URL birleştirme **yasak**.
- **Manuel header yasak**: Authorization/Content-Type vb. header’ları elle set etmek **yasak**; DioClient sorumluluğudur.
- **Response sözleşmesi**: Backend’in standart response şemasına göre parse edilir; ad-hoc parse **yasak**.
- **Tek hata stratejisi**: Ağ hataları tek tip yakalanır ve Provider state’ine taşınır; UI içinde try-catch ile ağ çağrısı **yasak**.

## References

- `skills/app-clean-architecture`
- `skills/backend-standard-response`