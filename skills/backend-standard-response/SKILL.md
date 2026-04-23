name: backend-standard-response
description: Tüm API’lerde zorunlu standart JSON response şeması: success, customMessage, data.

---

## Purpose

Tüm istemciler (app/admin) için **tahmin edilebilir** ve **sözleşmeli** API cevapları üretmek.

## Rules

- **Zorunlu şema**: Tüm endpoint’ler aşağıdaki alanları döner:
  - **success**: boolean (zorunlu)
  - **customMessage**: string (zorunlu; kullanıcıya/ekrana uygun kısa mesaj)
  - **data**: object | array | null (zorunlu)
- **Başarı senaryosu**: `success: true` iken `customMessage` boş string **olamaz**; anlamlı bir mesaj verilir.
- **Boş data**: Veri yoksa `data: null` kullanılır; key’i tamamen kaldırmak **yasak**.
- **Hata senaryosu**: Controller içinde hata response üretmek **yasak**; hata middleware’i aynı şemayı uygular.
- **Ek alanlar**: Şemaya key eklemek (örn. `message`, `result`, `payload`) **yasak**; yalnızca bu üçlü kullanılır.
- **HTTP status**: Status kodu senaryoya göre doğru seçilir; fakat body şeması değişmez.

## References

- `skills/backend-controller-service-flow`
- `skills/app-api-integration`
- `skills/admin-api-client-rule`