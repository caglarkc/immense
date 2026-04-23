name: admin-page-state-pattern
description: React sayfalarında ilk yüklemede veriyi useRef ile bir kez çekme ve standart state setini kullanma kuralı.

---

## Purpose

Sayfalarda gereksiz tekrar fetch’leri engellemek ve loading/empty/error/success durumlarında **tutarlı** bir state akışı oluşturmak.

## Rules

- **İlk yükleme**: Sayfa açılış fetch’i `useEffect` içinde `useRef` guard ile **bir kez** çalıştırılır.
- **Standart state seti**: Her sayfa en az şu state’leri taşır: `isLoading`, `error`, `data` (ve gerekiyorsa `isEmpty`).
- **Re-render fetch yasağı**: Dependency değişimleri yüzünden istemeden tekrar fetch etmek **yasak**; bilinçli refetch ayrı aksiyon olmalı.
- **API çağrısı kuralı**: Fetch sadece `src/api/client.js` üzerinden yapılır; farklı kanal **yasak**.
- **Boş ekran yasak**: Loading/empty/error durumları UI’da her zaman görünür bir state ile gösterilir.

## References

- `skills/admin-api-client-rule`
- `skills/backend-standard-response`