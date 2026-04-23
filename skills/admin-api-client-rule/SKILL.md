name: admin-api-client-rule
description: Admin Panel’de yalnızca src/api/client.js üzerinden istek atma; direkt fetch/axios kullanım yasağı.

---

## Purpose

React admin panelde isteklerin **tek merkezden** yönetilmesi (baseURL, auth, interceptors, error handling) ve kod tekrarının önlenmesi.

## Rules

- **Tek giriş noktası**: Tüm HTTP istekleri sadece `src/api/client.js` üzerinden yapılır.
- **Direkt fetch yasak**: Bileşen/page içinde `fetch(...)` kullanımı **yasak**.
- **Direkt axios yasak**: `axios.get/post/...` doğrudan kullanım **yasak** (client wrapper dışında).
- **Tutarlı hata yönetimi**: Hatalar client katmanında normalize edilir; page içinde dağınık error parse **yasak**.
- **Auth/headers**: Token/header set etme sadece client katmanında yapılır; page içinde header eklemek **yasak**.

## References

- `skills/backend-standard-response`
- `skills/general-naming-language`