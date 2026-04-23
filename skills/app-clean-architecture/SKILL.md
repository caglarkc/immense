name: app-clean-architecture
description: Flutter’da UI -> Provider -> Repository -> DioClient katman ayrımı ve bağımlılık yönü kuralları.

---

## Purpose

Flutter uygulamasında **katmanlı mimari** ile test edilebilirlik, sürdürülebilirlik ve ağ/iş mantığı ayrımını garanti etmek.

## Rules

- **Katman zinciri zorunlu**: `UI -> Provider -> Repository -> DioClient` dışında doğrudan çağrı **yasak**.
- **UI sorumluluğu**: Sadece render + kullanıcı etkileşimi; HTTP/parse/iş kuralı **yasak**.
- **Provider sorumluluğu**: State yönetimi + use-case orchestration; DioClient’a doğrudan erişim **yasak**.
- **Repository sorumluluğu**: Data kaynaklarını birleştirir/soyutlar; endpoint bilgisi `ApiEndpoints` ile kullanılır.
- **DioClient sorumluluğu**: Ağ çağrısı; header/url manuel yazımı repository/UI katmanında **yasak**.
- **Bağımlılık yönü**: Alt katman üst katmanı import edemez; ters bağımlılık **yasak**.

## References

- `skills/app-state-management`
- `skills/app-api-integration`