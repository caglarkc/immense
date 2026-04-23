# Backend Geliştirme Kuralları

> Node.js + Express.js · Mikroservis Mimarisi · Sequelize + PostgreSQL

---

## Mimari Genel Bakış

```
gateway/          → API Gateway (proxy, CORS, health-check)
services/         → Domain servisleri (auth, user, exercise, training, stats, social-media, admin, sync)
shared/           → Ortak config, middleware, factory, service, util katmanı
```

Her servis kendi içinde şu yapıyı barındırır:

```
services/<servis>/
├── server.js          → BaseServer extend
└── src/
    ├── app.js
    ├── config/
    ├── controllers/
    ├── models/
    ├── routes/
    ├── services/
    └── utils/
```

Servisler `createApp(...)` + `BaseServer` ile standart şekilde ayağa kalkar.

---

## İstek Akış Zinciri

```
Router  →  Controller  →  Service  →  Model
                ↓ (hata)
           next(error) → errorHandler middleware
```

1. **Router:** İsteği alır, ilgili Controller metoduna yönlendirir.
2. **Controller:** Parametre parse/kontrol yapar, Service katmanını çağırır. Başarılı sonucu `return res.json(result)` ile döner. Hata durumunda `return next(error)` ile middleware'e iletir.
3. **Service:** İş mantığı, veritabanı işlemleri ve dış kaynak haberleşmesi burada yapılır. Standart response objeleri döner.
4. **errorHandler Middleware:** Yakalanan hatayı standart JSON formatına çevirip client'a iletir.

---

## Controller Kuralları

- **Class-based + singleton** export edilir.
- Route bağlamada: `controller.methodName.bind(controller)`.
- Her metodta **try-catch zorunlu**; hata `return next(error)` ile middleware'e gider.
- Başarılı akışta servis sonucu **doğrudan** `return res.json(result)` ile publish edilir.
- Bazı özel durumlarda (validation, media, internal) `res.status(...).json(...)` istisnası olabilir.
- Zorunlu alan kontrolü: `checkRequiredFields(...)` (shared/utils/validation).
- Auth bilgisi: `req.context.userId`, `req.context.deviceId`.
- Device ID gerektiğinde: `req.headers['x-device-id']`.

---

## Service Kuralları

- **Class-based + singleton** export edilir.
- **Lazy loading:** Model `getModel()` ile yüklenir.
- Standart response pattern'i:
  - `_successResponse(customMessage, data?)` → Başarılı sonuç döner.
  - `_throwError(customMessage, errorMessage?, data?)` → Hata fırlatır.
- Hata nesnesi: `customMessage` (opsiyonel `statusCode`, `errorMessage`, `details`, `data`).
- Hata zincirinde **re-throw:** `if (error.customMessage) throw error;`
- Hata mesajları: `shared/config/errorMessages.js` üzerinden çağrılır.

---

## Response Formatı (Standart JSON Şeması)

```jsonc
// Başarı
{ "success": true, "customMessage": "...", "data": { ... } }

// Hata
{ "success": false, "customMessage": "...", "errorMessage": "...", "details": "..." }
```

`errorHandler` middleware tüm hataları bu formata çevirir.

---

## Model Kuralları

- Sequelize `define` ile tanımlanır.
- **UUID** primary key.
- `timestamps: true`, `paranoid: true` (soft delete).
- Singleton pattern: `getXyzModel(sequelize)`.
- **Kolon isimlendirmesi:** camelCase zorunlu; `field: 'snake_case'` yasak.

---

## Route Tipleri

| Tip | Middleware | Açıklama |
|-----|-----------|----------|
| Public | — | Auth gerektirmeyen |
| Protected | `authMiddleware` | Kullanıcı doğrulaması gerekli |
| Internal | `internalAuthMiddleware` | Servisler arası iletişim |

---

## Shared Import Örnekleri

```js
const { checkRequiredFields } = require('../../../../shared/utils/validation');
const errorMessages = require('../../../../shared/config/errorMessages');
const authMiddleware = require('../../../../shared/middlewares/authMiddleware');
```

---

## Yapılacaklar Kontrol Listesi

- [ ] Yeni endpoint/model eklerken mimari bütünlüğü koru.
- [ ] Gateway tanımlamasını güncelle.
- [ ] Ortak bileşenler `shared/` altından kullanılsın/güncellensin.
- [ ] Admin backend için `services/admin-service/src/modules/*` yapısına uy.
- [ ] Response formatını hiçbir durumda bozma.

