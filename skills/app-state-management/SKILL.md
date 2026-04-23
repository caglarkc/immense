name: app-state-management
description: Provider ile state yönetimi; LoadState enum zorunluluğu, context.mounted kontrolü ve boş ekran bırakmama kuralı.

---

## Purpose

Ekranların yükleme/hata/başarı durumlarında **tutarlı UX** ve güvenli async UI güncellemeleri sağlamak.

## Rules

- **State aracı**: Provider zorunlu; alternatif state management kullanımı **yasak** (aksi belirtilmedikçe).
- **LoadState zorunlu**: Her async veri akışı `LoadState` (örn. initial/loading/success/empty/error) ile yönetilir.
- **context.mounted**: Async işlem sonrası UI/Provider etkileşiminden önce `context.mounted` kontrolü zorunlu.
- **Boş ekran yasak**: Loading/empty/error durumlarında UI her zaman anlamlı bir görünüm sunar; “hiçbir şey göstermeme” **yasak**.
- **Tek kaynak**: UI, state’i Provider’dan okur; lokal kopya state ile paralel kaynak oluşturmak **yasak**.
- **Hata görünürlüğü**: Error state’de kullanıcıya okunabilir mesaj gösterilir; sessiz fail **yasak**.

## References

- `skills/app-clean-architecture`
- `skills/backend-standard-response`