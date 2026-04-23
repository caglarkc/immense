name: admin-data-loading-policy
description: Admin panel veri yükleme politikası; otomatik refresh yasağı, manuel yenileme standardı, tab değişiminde fetch yapmama.

---

## Purpose

Admin panelde gereksiz API yükünü önlemek ve kullanıcı kontrolünde, tahmin edilebilir bir veri akışı sağlamak.

## Rules

- **Otomatik refresh yok**: Sayfalar periyodik/interval ile veri çekmez.
- **İlk açılışta 1 kez**: Sayfa ilk kez açıldığında yalnızca bir kere yüklenir (`useRef` guard ile).
- **Sekme/sayfa geri dönüşte fetch yok**: Kullanıcı başka sayfaya gidip geri gelince otomatik istek atılmaz.
- **Manuel yenileme zorunlu**: Liste ekranlarında “Yenile” (refresh) aksiyonu bulunur; refetch sadece bu aksiyonla yapılır (veya kullanıcı filtre/arama/pagination değiştirdiğinde).
- **Kullanıcı aksiyonu = 1 istek**: Arama, filtre, sayfa değişimi gibi her aksiyon en fazla bir istek tetikler; aynı aksiyonla çoklu istek **yasak**.
- **Boş ekran yasak**: Loading / empty / error her zaman görünür şekilde gösterilir.
- **API kanalı**: Tüm istekler `src/api/client.js` üzerinden geçer; page içinde `fetch/axios` **yasak**.

## References

- `skills/admin-page-state-pattern`
- `skills/admin-api-client-rule`
