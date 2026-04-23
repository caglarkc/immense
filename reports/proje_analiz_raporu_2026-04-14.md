# PROJE ANALİZ RAPORU
**Tarih:** 2026-04-14  
**Proje:** son_immense — Fitness Sosyal Platform  
**Analiz Kapsamı:** Backend (Node.js mikroservisler) + Frontend (Flutter) — Auth'tan başlayarak tüm servisler ve ekranlar

---

## İÇİNDEKİLER

1. [Genel Mimari Özet](#1-genel-mimari-özet)
2. [Backend Analizi](#2-backend-analizi)
   - 2.1 Auth Servisi
   - 2.2 Gateway
   - 2.3 User Servisi
   - 2.4 Social-Media Servisi (Post)
   - 2.5 Match Servisi
   - 2.6 Notification Servisi
   - 2.7 Message Servisi
   - 2.8 Sync Servisi
   - 2.9 Stats Servisi
   - 2.10 Coaching Servisi
   - 2.11 Nutrition Servisi
   - 2.12 Training Servisi
3. [Frontend Analizi](#3-frontend-analizi)
   - 3.1 Auth Modülü
   - 3.2 API Interceptor
   - 3.3 DioClient
   - 3.4 ExploreScreen
   - 3.5 GymMatchScreen
   - 3.6 ChatScreen
   - 3.7 ActiveWorkoutScreen
   - 3.8 NutritionHomeScreen
   - 3.9 MainTabShellScaffold
   - 3.10 AppModalSheet
   - 3.11 ForegroundHeadsUpController
4. [Frontend UI/UX Analizi](#4-frontend-uiux-analizi)
5. [Kritik Sorunlar](#5-kritik-sorunlar-öncelikli-düzeltilmeli)
6. [Orta Öncelikli Sorunlar](#6-orta-öncelikli-sorunlar)
7. [İyileştirme Önerileri](#7-iyileştirme-önerileri)
8. [Özet Skor Kartı](#8-özet-skor-kartı)

---

## 1. GENEL MİMARİ ÖZET

**Son İmmense**, fitness odaklı bir mobil sosyal platform. Mimari katmanlar:

- **Backend**: Node.js tabanlı mikroservis mimarisi. API Gateway (Express + http-proxy-middleware) üzerinden 13 servis:
  `auth`, `user`, `training`, `exercise`, `stats`, `social-media`, `sync`, `notification`, `match`, `message`, `coaching`, `nutrition`, `admin`
- **Veri katmanı**:
  - PostgreSQL (Sequelize ORM)
  - Redis (session + sync event'leri)
  - RabbitMQ (async event bus)
  - MinIO (medya storage)
- **Frontend**: Flutter, Provider + Repository + DioClient zinciri, go_router navigasyon, socket.io (mesajlaşma), Firebase FCM.
- **Gateway**: Reverse proxy. Her path bir servise yönlendiriliyor. API key middleware var ama üretimde devre dışı bırakılmış (comment'li).

---

## 2. BACKEND ANALİZİ

---

### 2.1 Auth Servisi

#### Akış

```
register
  └─ email doğrulama (Redis OTP, 5dk TTL)
       └─ login (device binding)
            └─ JWT access token + Redis session
                 └─ middleware auto-refresh
                      └─ logout
```

Token mimarisi doğru kurulmuş: JWT sadece `userId + deviceId` taşıyor, asıl session Redis'te tutuluyor. Token expire olunca middleware session'ı kontrol edip yeni access token üretiyor ve `X-New-Access-Token` header'ında dönüyor.

#### Sorunlar

**[GÜVENLİK - KRİTİK] `changeEmail` e-posta normalize etmiyor**
- **Dosya:** `auth.service.js:781`
- **Sorun:** `email: newEmail` satırında `toLowerCase()` çağrısı yok. `register`'da `email.toLowerCase()` var ama `changeEmail`'de yok.
- **Etki:** `User@Gmail.com` ve `user@gmail.com` aynı kullanıcı olabilir; email duplicate bypass açığı mevcut.
- **Düzeltme:** `newEmail.toLowerCase().trim()` kullanılmalı.

```js
// Mevcut (hatalı):
email: newEmail

// Olması gereken:
email: newEmail.toLowerCase().trim()
```

---

**[GÜVENLİK - KRİTİK] Debug endpoint production'da açık**
- **Dosya:** `auth.routes.js:35`
- **Sorun:**
```js
router.get('/debug/redis', authController.debugRedis.bind(authController));
```
Auth kontrolsüz. Tüm Redis verileri (session token'ları, OTP kodları) düz HTTP GET ile okunabilir. Auth middleware yok, IP kısıtlaması yok.
- **Etki:** Aktif kullanıcıların session token'larına ve geçerli OTP kodlarına yetkisiz erişim.
- **Düzeltme:** Endpoint production build'de tamamen kaldırılmalı veya en azından `adminAuthMiddleware` + IP whitelist eklenmeli.

---

**[GÜVENLİK - KRİTİK] `internalAuthMiddleware` fallback key**
- **Dosya:** `internalAuthMiddleware.js:12`
- **Sorun:**
```js
const DEFAULT_INTERNAL_API_KEY = 'internal-api-key-change-in-production';
```
`.env`'de `INTERNAL_API_KEY` tanımlanmazsa bu sabit key ile tüm servisler arası internal endpoint'ler erişilebilir olur.
- **Etki:** Herhangi biri bu anahtarı bilirse tüm internal API'ları çağırabilir (kullanıcı oluşturma, silme, ID sorgulama vb.)
- **Düzeltme:** Fallback kaldırılmalı; key yoksa servis startup'ta hata verip çıkmalı.

```js
// Olması gereken:
if (!process.env.INTERNAL_API_KEY) {
  throw new Error('INTERNAL_API_KEY must be set in environment');
}
```

---

**[GÜVENLİK - YÜKSEK] Login rate limiting eksik**
- **Dosya:** `auth.service.js:230-321`
- **Sorun:** Başarısız login denemeleri sayılmıyor. Email OTP için 1 dk cooldown var ama şifre tahmin saldırısına (brute-force) karşı hiçbir önlem yok.
- **Etki:** Sonsuz şifre denemesi mümkün.
- **Düzeltme:** `express-rate-limit` veya Redis tabanlı attempt counter eklenebilir:
  - 5 başarısız denemede 15 dk bekleme
  - IP bazlı ve userId bazlı limit

---

**[GÜVENLİK - YÜKSEK] `changeEmail` re-verification yok**
- **Dosya:** `auth.service.js:765-795`
- **Sorun:** Email değişince `isVerified` flag'i `false`'a çekilmiyor, yeni email doğrulama yapılmıyor.
- **Etki:** Kullanıcı unverified (hatta var olmayan) bir email adresine geçebilir ve hesap aktif kalmaya devam eder.
- **Düzeltme:** Email değişiminde `isVerified: false` set edilmeli, yeni adrese OTP gönderilmeli ve kullanıcı yeni emaili doğrulayana kadar bazı özellikler kısıtlanmalı.

---

**[ORTA] `resetPassword`: hash order verimsiz**
- **Dosya:** `auth.service.js:539-543`
- **Sorun:** Önce `hashPassword(newPassword)` çağrılıyor, sonra "eski şifreyle aynı mı?" kontrolü yapılıyor. Kontrol başarısız olunca önceden yapılmış hash işlemi boşa gidiyor.
- **Etki:** Gereksiz bcrypt CPU harcaması.
- **Düzeltme:** Önce kontrol, sonra hash.

---

**[DÜŞÜK] Typo: `UNKOWN`**
- **Dosya:** `shared/config/errorMessages.js:88`
- **Sorun:** `UNKOWN: "Beklenmeyen hata oluştu"` — `UNKNOWN` olmalı.
- **Etki:** Tüm servislerde bu key kullanılıyor (en az 15+ yerde). Tutarsızlık ve kod kalitesi sorunu.
- **Düzeltme:** Global search-replace ile düzeltilmeli.

---

**[DÜŞÜK] `forgotPassword` hardcoded Türkçe mesaj**
- **Dosya:** `auth.service.js:437`
- **Sorun:** `this._throwError("Mail henüz onaylanmamis")` — merkezi `errorMessages` sisteminin dışında, hem yazım hatası var (`onaylanmamış`).
- **Düzeltme:** `errorMessages` objesinden kullanılmalı.

---

### 2.2 Gateway

#### Sorunlar

**[GÜVENLİK - KRİTİK] API key middleware üretimde kapalı**
- **Dosya:** `gateway/src/app.js:61-80`
- **Sorun:** Tüm middleware bloğu comment içinde. Gateway tamamen açık; herhangi bir IP istediği servise istek atabilir.
- **Etki:** Dışarıdan direkt servis erişimi mümkün, rate limiting yok, IP kısıtlaması yok.
- **Düzeltme:** Middleware aktifleştirilmeli. Üretim ortamında comment kaldırılmalı.

---

**[ORTA] Body parsing yeri yanlış**
- **Dosya:** `gateway/src/app.js:249-251`
- **Sorun:** Proxy tanımlamalarından sonra `express.json()` ekleniyor. Comment'te de belirtilmiş ama bazı fallback durumlar için JSON parsing hiç çalışmayabilir.
- **Düzeltme:** `express.json()` proxy tanımlamalarından önce middleware stack'e alınmalı.

---

**[DÜŞÜK] `onProxyReq`'de body re-serialize'ı işlevsiz**
- **Dosya:** `gateway/src/app.js:100-105`
- **Sorun:** Yalnızca `Object.keys(req.body).length > 0` ise re-serialize yapılıyor. Gateway'de body parse edilmediği için `req.body` her zaman `undefined`/boş. Bu code path pratikte hiç tetiklenmiyor.

---

### 2.3 User Servisi

Genel mimari temiz. `registerUser`, `getAuthUser` ile auth servisini internal API üzerinden çekiyor. Profil oluştururken `isVerified` kontrolü yapılıyor.

#### Sorunlar

**[DÜŞÜK] Hardcoded İngilizce error mesajı**
- **Dosya:** `user.service.js:89`
- **Sorun:** `this._throwError('User profile already exists')` — merkezi error mesaj sisteminin dışında ve İngilizce.
- **Düzeltme:** `errorMessages` objesine eklenmeli.

---

**[DÜŞÜK] `_enrichUserWithLocation`'da 3 ayrı DB sorgusu**
- **Dosya:** `user.service.js:33-43`
- **Sorun:** City, District, Gym için `findByPk` ayrı ayrı çağrılıyor. Her kullanıcı için 3 DB round-trip.
- **Düzeltme:** Tek `include` ile JOIN yapılabilir veya sonuçlar cache'lenebilir.

```js
// Mevcut: 3 ayrı sorgu
const city = await City.findByPk(user.cityId);
const district = await District.findByPk(user.districtId);
const gym = await Gym.findByPk(user.gymId);

// Olması gereken: tek sorgu
const user = await User.findByPk(userId, {
  include: [{ model: City }, { model: District }, { model: Gym }]
});
```

---

### 2.4 Social-Media Servisi (Post)

#### Sorunlar

**[YÜKSEK] `createPost` — workout + post atomic değil**
- **Dosya:** `post.service.js:113-195`
- **Sorun:** Workout training-service'de oluşturulur, sonra post bu serviste create edilir. Training başarılı ama post fail ederse orphan workout oluşur. Tersi durumda da tutarsızlık olabilir.
- **Etki:** Veritabanında sarkan (orphan) workout kayıtları birikir.
- **Düzeltme:** Distributed transaction veya saga/compensating action pattern uygulanmalı. Post fail ederse training-service'e `DELETE /workouts/:id` gönderilmeli.

---

**[YÜKSEK] `getExploreFeed` — yeni kullanıcı içerik göremez**
- **Dosya:** `post.service.js:366-372`
- **Sorun:** `excludeArr.length === 0` kontrolünde (followingIds ve blockedIds boşsa) boş array dönüyor. Yeni kullanıcı (hiç takip etmeyen) explore'da hiç içerik göremez.
- **Düzeltme:** Boş excludeArr durumunda sadece viewerUserId exclude edilerek sorgu yapılmalı.

```js
// Mevcut (hatalı):
if (excludeArr.length === 0) return [];

// Olması gereken:
const finalExclude = excludeArr.length === 0 ? [viewerUserId] : excludeArr;
```

---

**[ORTA] `findAndCountAll` ile offset pagination**
- **Dosya:** `post.service.js:376`
- **Sorun:** Her sorguda `COUNT(*)` yapılıyor. Büyük tablolarda maliyetli.
- **Düzeltme:** Cursor-based pagination'a geçilmeli (createdAt + id ile).

---

**[DÜŞÜK] `_enrichPostsWithAuthor` batch sorgu belirsiz**
- **Dosya:** `post.service.js`
- **Sorun:** `getUsersByIds` çağrısının N adet ayrı istek mi yoksa tek batch mi attığı netleştirilmeli. N+1 riski var.

---

### 2.5 Match Servisi

OneByOne ve Broadcast akışları iyi tasarlanmış. `pairRelationship` servisi ikili ilişki durumunu tutuyor, block kontrolü yapılıyor.

#### Sorunlar

**[ORTA] Expired istek'lerin background temizliği yok**
- **Dosya:** `one_by_one.service.js:107-110`
- **Sorun:** `expiresAt` geçmişse `updateStatus` sırasında `rejected` set ediliyor ama arka planda expired kayıtlar temizlenmiyor.
- **Etki:** DB'de expired kayıtlar birikerek şişer; sorgu performansı düşer.
- **Düzeltme:** Günlük/saatlik cron job ile `expiresAt < NOW()` kayıtlar temizlenmeli.

---

**[DÜŞÜK] Notification hatası loglanmıyor**
- **Dosya:** `one_by_one.service.js:86-88`
- **Sorun:**
```js
} catch (e) { /* push hatası istek oluşturmayı bozmaz */ }
```
Sessiz hata yutma var, log bile yok.
- **Düzeltme:** `logger.warn('Push notification failed', { error: e.message })` eklenmeli.

---

### 2.6 Notification Servisi

`normalizeUserIdForDb` ile UUID formatı doğrulanıyor — iyi pratik.

#### Sorunlar

**[ORTA] `persistInAppNotification` fire-and-forget**
- **Dosya:** `notification.service.js:59-87`
- **Sorun:** Hata loglanıyor ama caller hiç bilmiyor. Kritik bildirimler (eşleşme bildirimi, mesaj bildirimi) kaybolabilir.
- **Düzeltme:** En azından hata loglanmalı ve monitoring alert'i tetiklenmeli.

---

### 2.7 Message Servisi

Optimistic message, socket.io entegrasyonu, conversation modeli var.

#### Sorunlar

**[YÜKSEK] `getMessages` — `before` param validasyonu yok**
- **Dosya:** `message.service.js:71-73`
- **Sorun:** `new Date(before)` kullanılıyor. Geçersiz tarih stringinde (`"abc"`, `"undefined"`) `Invalid Date` oluşur ve Sequelize hata fırlatır.
- **Etki:** 500 hatası, kötü UX.
- **Düzeltme:**
```js
if (before) {
  const parsedDate = new Date(before);
  if (isNaN(parsedDate.getTime())) {
    throw new ValidationError('Invalid "before" date parameter');
  }
}
```

---

**[DÜŞÜK] Mesaj içerik uzunluk limiti belirsiz**
- **Sorun:** `content` alanına model veya controller katmanında karakter sınırı uygulanıyor mu belirsiz.
- **Düzeltme:** DB migration + controller validasyonuyla max 2000 karakter sınırı konulmalı.

---

### 2.8 Sync Servisi

Blok kontrolü her auth'lu istekte Redis'ten okunuyor (syncMiddleware). Immediate event'ler header üzerinden iletiliyor. Mimari sağlam.

#### Sorunlar

**[ORTA] `MAX_PENDING_EVENTS = 50` ile event kaybı**
- **Dosya:** `sync.service.js:17`
- **Sorun:** Kullanıcı 50'den fazla event kaçırdıysa eski event'ler düşebilir. TTL 7 gün ama sınır 50.
- **Etki:** Aktif uygulamalarda (sosyal bildirimler, eşleşme güncellemeleri) event kaybı.
- **Düzeltme:** Limit artırılmalı (150-200) veya event tiplerine göre öncelik sıralanmalı.

---

**[ORTA] Global broadcast key**
- **Dosya:** `sync.service.js:65-75`
- **Sorun:** Broadcast event'ler global bir Redis key üzerinden tüm aktif kullanıcılara atılıyor.
- **Etki:** Binlerce eş zamanlı kullanıcıda Redis'e yük bindirme.
- **Düzeltme:** Fan-out pattern veya Redis Pub/Sub channel'ları kullanılmalı.

---

### 2.9 Stats Servisi

RabbitMQ event consumer'ı ile `workout.completed` event'lerini alıyor, PR (kişisel rekoru) hesaplıyor. Genel yapı iyi, belirgin sorun tespit edilmedi.

---

### 2.10 Coaching Servisi

Çok sayıda model (registration, invite, coach, membership, week plan, activity log). İç tutarlılık var. `lookupUserIdByEmail` ile auth servisine danışıyor. Belirgin kritik sorun tespit edilmedi.

---

### 2.11 Nutrition Servisi

Gemini AI entegrasyonu var (`nutritionAiGemini.service.js`). 12 ayrı service dosyası — sorumluluklar ayrışık, iyi.

#### Sorunlar

**[YÜKSEK] `_loadFeedbackLevelCodes` her çağrıda `fs.readFileSync`**
- **Dosya:** `nutritionAi.service.js:21-24`
- **Sorun:** Dosya okuma senkron ve her request sırasında yapılıyor. Node.js event loop'u bloke eder.
- **Etki:** AI analiz endpoint'i çağrıldığında tüm diğer istekler bekler.
- **Düzeltme:** Module-level veya singleton cache ile tek seferlik okuma:
```js
// Module seviyesinde cache:
let _feedbackLevelCodes = null;
function getFeedbackLevelCodes() {
  if (!_feedbackLevelCodes) {
    _feedbackLevelCodes = JSON.parse(fs.readFileSync(path, 'utf8'));
  }
  return _feedbackLevelCodes;
}
```

---

### 2.12 Training Servisi

RabbitMQ'ya `workout.completed` eventi publish ediyor. Routine ve workout CRUD temiz. Belirgin kritik sorun tespit edilmedi.

---

## 3. FRONTEND ANALİZİ

---

### 3.1 Auth Modülü

**Mimari:** `AuthProvider → AuthRepository → DioClient` zinciri doğru uygulanmış.

#### İyi Noktalar
- `mounted` kontrolleri dikkatli yapılmış (`login_screen.dart`, `splash_screen.dart`)
- `forceUnauthenticated()` ile 401 interceptor'u state'i temizliyor
- `onLoginSuccess` callback ile cross-provider temizlik yapılıyor

#### Sorunlar

**[YÜKSEK] `verifyEmail` başarı sonrası yanlış state geçişi**
- **Dosya:** `auth_provider.dart:155`
- **Sorun:** Başarılı email doğrulamada state `unauthenticated` oluyor. Kullanıcı verified olunca direkt `authenticated` olması beklenir; `unauthenticated`'a düşürüp tekrar login sayfasına yönlendirmek kötü UX.
- **Düzeltme:** Doğrulama başarılıysa token hâlâ geçerliyse `authenticated` state'ine geçilmeli.

---

**[ORTA] `resendVerificationEmail` loading state yok**
- **Dosya:** `auth_provider.dart:171-196`
- **Sorun:** Resend işlemi sırasında state `loading` olmuyor. Kullanıcı art arda birden çok kez tıklayabilir, birden fazla istek gönderilir.
- **Düzeltme:** İşlem başında `_setLoading(true)`, bitişinde `_setLoading(false)` eklenip buton disable edilmeli.

---

**[ORTA] "Şifremi Unuttum" linki yok**
- **Dosya:** `login_screen.dart`
- **Sorun:** `forgotPassword` endpoint'i ve `AuthRepository.forgotPassword()` metodu var ama login screen'de UI bağlantısı görünmüyor.
- **Etki:** Kullanıcılar şifrelerini sıfırlayamaz.
- **Düzeltme:** Login formuna `TextButton` ile şifre sıfırlama sayfasına yönlendirme eklenebilir.

---

### 3.2 API Interceptor

#### İyi Noktalar
- Token refresh otomatik (`X-New-Access-Token` header'ı algılanıyor)
- 401'de token temizleme + state reset + login yönlendirme doğru uygulanmış
- Socket service reconnect, token refresh sonrası yapılıyor

#### Sorunlar

**[ORTA] `onResponse`'da token kaydetme hatası yutulur**
- **Dosya:** `api_interceptor.dart:71-74`
- **Sorun:** `AuthStorage.saveTokens(...).then(...)` — hata handle edilmiyor. Token kaydetme veya socket reconnect hata verirse sessizce yutulur.
- **Düzeltme:**
```dart
unawaited(AuthStorage.saveTokens(newToken).catchError((e) {
  logger.e('Token save failed', error: e);
}));
```

---

**[DÜŞÜK] `_processSyncEventsFromHeader` her response'da JSON decode**
- **Sorun:** Header yoksa erken çıkış var ama decode işlemi tüm response'larda deneniyor. Sadece sync servisinden gelen response'larda beklenmeli.

---

### 3.3 DioClient

Yapı basit ve doğru. Her method için ayrı wrapper var.

#### Sorunlar

**[DÜŞÜK] Timeout override sadece `postMultipart`'ta**
- **Sorun:** AI foto upload gibi yavaş işlemler için sadece `postMultipart`'ta override var. Normal `post`/`put` için custom timeout imkânı yok.
- **Düzeltme:** Diğer metodlara da opsiyonel `timeout` parametresi eklenmeli.

---

### 3.4 ExploreScreen

#### İyi Noktalar
- Scroll listener ile lazy loading doğru uygulanmış
- `_pendingAutoLoadMore` ile infinite loop koruması var
- Error, loading, empty state'ler handle ediliyor

#### Sorunlar

**[ORTA] Scroll event'lerde throttle/debounce yok**
- **Dosya:** `explore_screen.dart:43`
- **Sorun:** `_onScroll` her scroll pixel değişiminde tetikleniyor. `context.read<PostProvider>()` her callback'te çağrılıyor.
- **Düzeltme:** Belirli bir eşik (ör. 200px) aşıldığında tetiklenecek şekilde `debounce` uygulanmalı.

---

**[ORTA] `_scheduleExploreLoadIfNeeded` her rebuild'de çağrılıyor**
- **Dosya:** `explore_screen.dart:57-72`
- **Sorun:** `Consumer` builder içinde bu metod çağrılıyor. Her rebuild'de `addPostFrameCallback` birikimi olabilir.
- **Düzeltme:** `initState` veya `didChangeDependencies`'e taşınmalı; builder içinde side-effect olmamalı.

---

### 3.5 GymMatchScreen

#### İyi Noktalar
- `TabController` dispose ediliyor
- `Consumer2` ile iki provider izleniyor
- `_ListStateView` yeniden kullanılabilir component olarak iyi tasarlanmış
- Error, loading, empty state'ler için `_ListStateView` widget var
- `context.mounted` kontrolleri var

#### Sorunlar

**[YÜKSEK] Türkçe karakter eksiklikleri**
- **Dosya:** `gym_match_screen.dart:122-214`
- **Sorun:** `'Spor Arkadasi'`, `'cagri ac'`, `'talepleri tek yerden yonet'`, `'Tum Cagrilar'` gibi Türkçe karaktersiz yazılar.
- **Etki:** Üretim ortamında yanlış görünüm, profesyonellik eksikliği.
- **Düzeltme:** `'Spor Arkadaşı'`, `'çağrı aç'`, `'talepleri tek yerden yönet'`, `'Tüm Çağrılar'` olarak güncellenmeli.

---

**[ORTA] `_showBroadcastDetail` içinde context karışıklığı**
- **Dosya:** `gym_match_screen.dart:422`
- **Sorun:** Sheet içinden ana ekranın `context`'i capture ediliyor. Sheet kapatıldıktan sonra `context.mounted` kontrolü var ancak `showModalBottomSheet` builder içindeki `ctx` ile `context` karıştırılabilir.

---

### 3.6 ChatScreen

#### İyi Noktalar
- `_messageProvider` ve `_inboxProvider` `didChangeDependencies`'te cache'lenmiş, `dispose`'da `context.read` kullanılmıyor — doğru pattern
- Socket join/leave lifecycle'a bağlı
- Optimistic message gönderme mevcut

#### Sorunlar

**[YÜKSEK] Listener ekleme race condition**
- **Dosya:** `chat_screen.dart:59`
- **Sorun:** `provider.addListener(_onProviderUpdate)` `addPostFrameCallback` içinde ekleniyor. Widget unmount edilmeden önce callback tetiklenmemiş olabilir; listener hiç eklenmez.
- **Düzeltme:** `initState` içinde `WidgetsBinding.instance.addPostFrameCallback` değil, `didChangeDependencies` içinde listener eklenmeli.

---

**[ORTA] `dispose`'da async işlem**
- **Dosya:** `chat_screen.dart:103`
- **Sorun:** `Future.microtask(() => mp.loadConversations())` dispose içinde çağrılıyor. Dispose sonrası async çalışma güvensiz; `mp` (provider) bu sırada invalidate olmuş olabilir.
- **Düzeltme:** Bu refresh işlemi `dispose` yerine bir üst widget'ın `RouteAware.didPopNext` callback'ine taşınmalı.

---

### 3.7 ActiveWorkoutScreen

650 satırlık büyük dosya. `ActiveWorkoutSessionController` ayrı bir sınıfa taşınmış, bu iyi.

#### Sorunlar

**[ORTA] Çift kaynak problemi**
- **Dosya:** `active_workout_screen.dart:43`
- **Sorun:** `_coachMemberRoutineSnapshot` state'i hem `_ActiveWorkoutScreenState`'te local state olarak tutulurken hem `_session.manager` üzerinden paylaşılıyor. Tek doğru kaynak belirsiz; stale data riski var.
- **Düzeltme:** State tek bir yerden (session manager veya provider) yönetilmeli.

---

### 3.8 NutritionHomeScreen

993 satır — büyük bir dosya.

#### İyi Noktalar
- `NutritionScreenScaffold`, `NutritionViewState` ile kısmen bölünmüş
- Loading, error, empty state üçlüsü mevcut
- `loadIfNeeded` pattern — gereksiz yeniden yükleme önleniyor

#### Sorunlar

**[ORTA] Dosya boyutu aşırı büyük (993 satır)**
- **Sorun:** Widget ağaçları derin, okunabilirlik düşük, test edilebilirlik zor.
- **Düzeltme:** Alt widget'lar ayrı dosyalara taşınmalı:
  - `NutritionDailySummaryCard`
  - `NutritionMealsList`
  - `NutritionProgressChart`
  - vb.

---

### 3.9 MainTabShellScaffold

#### Sorunlar

**[ORTA] `BouncingScrollPhysics` Android'de yanlış**
- **Dosya:** `main_tab_shell_scaffold.dart:103`
- **Sorun:** Android'de `BouncingScrollPhysics` beklenmedik görünüm yaratır (iOS'a özgü bounce efekti).
- **Düzeltme:**
```dart
// Platform adaptive kullanım:
physics: Platform.isIOS
  ? const BouncingScrollPhysics()
  : const ClampingScrollPhysics(),
```

---

**[DÜŞÜK] `didUpdateWidget`'ta `addPostFrameCallback` birikimi**
- **Dosya:** `main_tab_shell_scaffold.dart:41-48`
- **Sorun:** Her widget güncellenmesinde `addPostFrameCallback` ile `jumpToPage` çağrısı ekleniyor. `_isAnimatingToPage` koruma var ama hızlı tab değişimlerinde callback'ler birikebilir.

---

### 3.10 AppModalSheet

Temiz ve yeniden kullanılabilir widget. Sorun tespit edilmedi.

---

### 3.11 ForegroundHeadsUpController

Sofistike bir singleton. Dedupe, chat grouping, queue yönetimi, navigation delay ile frame çakışması önleniyor. Genel yapı etkileyici.

#### Sorunlar

**[YÜKSEK] `_navigationInFlight` race condition**
- **Dosya:** `foreground_heads_up.dart:289-305`
- **Sorun:** `_navigationInFlight = true` set ediliyor, `finally` ile sıfırlanıyor. Ancak `router.push(path)` await edilmeden çağrılıyor (`void` döndürüyor). Bayrak `true` olarak kalsa da sonraki push için garanti yok; eş zamanlı birden fazla navigation push mümkün.
- **Düzeltme:** `router.push` await edilebilir değilse en azından `Future.delayed` ile minimum bekletme uygulanmalı.

---

**[DÜŞÜK] `_dedupeAt` map temizlenmiyor**
- **Dosya:** `foreground_heads_up.dart:120-128`
- **Sorun:** `_dedupeWindow` geçmiş key'ler map'te kalıyor. Uzun süre çalışan uygulamada hafıza birikimi.
- **Düzeltme:** Periyodik temizlik (ör. her 5 dk bir `_dedupeAt.removeWhere(...)`) veya TTL kontrollü temizlik.

---

## 4. FRONTEND UI/UX ANALİZİ

---

### 4.1 LoginScreen

| Özellik | Değerlendirme |
|---------|---------------|
| Loading feedback | İyi — `_postLoginBootstrap` overlay ile |
| Başarılı giriş | İyi — `AppColors.success` snackbar var |
| Form validasyonu | Zayıf — Sadece boş kontrol; email format, min uzunluk yok |
| "Şifremi unuttum" | **Eksik** — Backend'de var ama UI'da yok |
| Controller dispose | İyi — Yapılıyor |
| FocusNode dispose | İyi — Yapılıyor |

**Öneriler:**
- Email format regex validasyonu ekle (`RegExp(r'^[^@]+@[^@]+\.[^@]+$')`)
- Şifre minimum uzunluk kontrolü (ör. 8 karakter)
- "Şifremi Unuttum" `TextButton` ile `ForgotPasswordScreen`'e yönlendirme

---

### 4.2 ExploreScreen

| Özellik | Değerlendirme |
|---------|---------------|
| Lazy loading | İyi — Scroll ile tetikleniyor |
| Infinite loop koruması | İyi — `_pendingAutoLoadMore` flag |
| Empty state | İyi — Özel widget var |
| Arama | Yüzeysel — "Fake input" (gerçek input değil, navigate ediyor) |
| Grid tiling | Potansiyel sorun — Tinder tile her zaman index 2'de |

**Öneriler:**
- Arama input'u gerçek `TextField` olmalı veya tıklandığında arama sayfasına navigation'ın görsel göstergesi olmalı (chevron icon vb.)
- Scroll throttle ekle
- `Consumer` builder içinden side-effect kaldır

---

### 4.3 GymMatchScreen

| Özellik | Değerlendirme |
|---------|---------------|
| Header metrikler | İyi — Açık çağrı, Aktif ilan, Bekleyen talep |
| Badge sistemi | İyi — Görsel göstergeler var |
| Türkçe karakterler | **Kritik eksik** — `Spor Arkadasi`, `cagri`, `yonet`, vb. |
| Loading/error/empty | İyi — `_ListStateView` ile handle ediliyor |
| Tab yapısı | İyi — 3 sekme mantıklı ayrışmış |

**Düzeltilmesi Gereken Metinler:**
```
'Spor Arkadasi'     → 'Spor Arkadaşı'
'cagri ac'          → 'Çağrı Aç'
'Tum Cagrilar'      → 'Tüm Çağrılar'
'talepleri yonet'   → 'Talepleri Yönet'
'musait'            → 'müsait'
```

---

### 4.4 NutritionHomeScreen

| Özellik | Değerlendirme |
|---------|---------------|
| Loading/error/empty | İyi — Üçlü handle ediliyor |
| Dosya boyutu | **Kötü** — 993 satır, bakımı zor |
| Widget organizasyonu | Orta — Kısmen bölünmüş ama yetersiz |
| State yönetimi | İyi — `loadIfNeeded` pattern |
| Navigasyon | İyi |

**Öneriler:**
- `NutritionDailySummaryCard`, `NutritionMealsList`, `NutritionProgressChart` gibi widget'lara böl
- Her sub-widget kendi dosyasında olsun

---

### 4.5 ChatScreen

| Özellik | Değerlendirme |
|---------|---------------|
| Kapalı conversation uyarısı | İyi — Özel mesaj gösteriliyor |
| Eşleşme sonlandırma | İyi — `PopupMenuButton` ile |
| AppBar profil fotoğrafı | **Eksik** — Sadece isim var |
| Mesaj pagination | İyi — `before` parametresiyle |
| Optimistic send | İyi — Anında UI'a ekleniyor |

**Öneriler:**
- AppBar'a kullanıcı avatar'ı ekle (zaten diğer ekranlarda var)
- Mesaj gönderm başarısız olunca retry button'u ekle

---

### 4.6 ActiveWorkoutScreen

| Özellik | Değerlendirme |
|---------|---------------|
| Dosya boyutu | Orta — 650 satır, kabul edilebilir |
| Controller ayrımı | İyi — `ActiveWorkoutSessionController` ayrı sınıf |
| State yönetimi | Karmaşık — Çift kaynak sorunu var |

---

### 4.7 Genel UI/UX Değerlendirmesi

**İyi Pratikler:**
- Loading/error/empty üçlüsü çoğu ekranda mevcut
- `context.mounted` kontrolleri yapılmış
- Provider + Repository zinciri tutarlı
- go_router ile tip-güvenli navigasyon

**Sistematik Eksiklikler:**
1. Form validasyonları minimal (sadece boş kontrol)
2. Büyük ekranlar parçalanmamış (NutritionHome: 993 satır)
3. Türkçe karakter tutarsızlıkları (özellikle GymMatch ekranı)
4. Bazı eksik UX bağlantıları (Şifremi Unuttum)
5. Android/iOS platform uyumu eksik (BouncingScrollPhysics)

---

## 5. KRİTİK SORUNLAR (Öncelikli Düzeltilmeli)

| # | Sorun | Dosya | Neden Kritik |
|---|-------|-------|--------------|
| 1 | `/debug/redis` endpoint korumasız | `auth.routes.js:35` | Tüm session/OTP verilerine yetkisiz erişim |
| 2 | `internalAuthMiddleware` fallback default key | `internalAuthMiddleware.js:12` | Servisler arası tüm internal endpoint'ler açık kalabilir |
| 3 | `changeEmail` lowercase normalize eksik | `auth.service.js:781` | Email duplicate bypass, güvenlik açığı |
| 4 | `changeEmail` sonrası `isVerified` sıfırlanmıyor | `auth.service.js:781-795` | Unverified email ile aktif hesap |
| 5 | Login brute-force koruması yok | `auth.service.js:230` | Şifre tahmin saldırısı |
| 6 | Gateway API key devre dışı | `gateway/src/app.js:61-80` | Herhangi IP gateway'e istek atabiliyor |

---

## 6. ORTA ÖNCELİKLİ SORUNLAR

| # | Sorun | Dosya | Etki |
|---|-------|-------|------|
| 1 | `createPost` — workout + post non-atomic | `post.service.js:113` | Orphan workout kayıtları |
| 2 | Explore feed `findAndCountAll` — offset pagination | `post.service.js:376` | Büyük tabloda performans |
| 3 | `getExploreFeed` — boş excludeArr'da yanlış dönüş | `post.service.js:366` | Yeni kullanıcı içerik göremez |
| 4 | `nutritionAi` — `fs.readFileSync` her çağrıda | `nutritionAi.service.js:21` | Senkron I/O, request blocking |
| 5 | `resendVerificationEmail` loading state yok | `auth_provider.dart:171` | Çift istek gönderilebilir |
| 6 | ChatScreen listener race condition | `chat_screen.dart:59` | Listener eklenmeden unmount olabilir |
| 7 | `message.getMessages` — `before` param validasyonu yok | `message.service.js:71` | Invalid Date → Sequelize error |
| 8 | `verifyEmail` başarı sonrası `unauthenticated` state | `auth_provider.dart:155` | Kullanıcı tekrar login zorunda |
| 9 | Sync global broadcast key | `sync.service.js:65-75` | Çok kullanıcılı ortamda Redis yükü |
| 10 | `MAX_PENDING_EVENTS = 50` ile event kaybı | `sync.service.js:17` | Yoğun güncellemelerde bildirim kaybı |
| 11 | `ForegroundHeadsUpController` navigation race | `foreground_heads_up.dart:289` | Eş zamanlı push mümkün |
| 12 | ChatScreen `dispose`'da async işlem | `chat_screen.dart:103` | Dispose sonrası güvensiz async |
| 13 | `BouncingScrollPhysics` Android'de yanlış | `main_tab_shell_scaffold.dart:103` | Platform uyumsuzluğu |
| 14 | Expired OneByOne request'lerin temizliği yok | `one_by_one.service.js:107` | DB şişmesi, performans |

---

## 7. İYİLEŞTİRME ÖNERİLERİ

### Backend

1. **Güvenlik:**
   - `/debug/redis` endpoint'i production build'den tamamen kaldırılmalı
   - `INTERNAL_API_KEY` env'de yoksa servis başlamamalı
   - Login için `express-rate-limit` (5 başarısız deneme → 15 dk bekleme)
   - `changeEmail`: `toLowerCase()` + `isVerified: false` + yeni email doğrulama

2. **Veri tutarlılığı:**
   - `createPost` için saga pattern: post fail → workout rollback
   - `changeEmail` sonrası re-verification akışı

3. **Performans:**
   - Offset → cursor-based pagination (social feed)
   - `fs.readFileSync` → module-level cache (nutrition AI)
   - `_enrichUserWithLocation` → tek JOIN sorgusu
   - Expired match kayıtları için günlük cron job

4. **Kod kalitesi:**
   - `UNKOWN` → `UNKNOWN` global replace
   - Hardcoded Türkçe/İngilizce error mesajları → `errorMessages` objesine taşı
   - `message.getMessages` `before` param validasyonu

### Frontend

1. **UX eksiklikleri:**
   - LoginScreen: "Şifremi Unuttum" bağlantısı ekle
   - GymMatchScreen: Türkçe karakterleri düzelt
   - ChatScreen AppBar: Kullanıcı avatar'ı ekle
   - Form validasyonlarını zenginleştir (email format, şifre uzunluğu)

2. **Güvenilirlik:**
   - `AuthStorage.saveTokens` → `unawaited(...)` + error catch
   - ChatScreen listener: `addPostFrameCallback` → `didChangeDependencies`
   - ChatScreen `dispose`: async işlemi `RouteAware.didPopNext`'e taşı
   - `_dedupeAt` map: periyodik temizlik

3. **Platform uyumu:**
   - `BouncingScrollPhysics` → `Platform.isIOS ? BouncingScrollPhysics() : ClampingScrollPhysics()`

4. **Kod organizasyonu:**
   - `NutritionHomeScreen` (993 satır) alt widget'lara böl
   - `Consumer` builder içinden side-effect'leri kaldır (`_scheduleExploreLoadIfNeeded`)

---

## 8. ÖZET SKOR KARTI

| Kategori | Puan | Notlar |
|----------|------|--------|
| Backend Mimari | **8/10** | Mikroservis yapısı temiz, sorumluluklar ayrışık |
| Auth Güvenliği | **5/10** | Debug endpoint açık, brute-force koruması yok, changeEmail açıkları |
| Backend Error Handling | **7/10** | Tutarlı pattern var, birkaç kaçak nokta |
| DB Sorgu Verimliliği | **6/10** | Offset pagination, N adet sorgu sorunları |
| API Response Tutarlılığı | **8/10** | `{success, customMessage, data}` formatı genel tutarlı |
| Frontend Mimari | **8/10** | Provider + Repository + DioClient zinciri doğru |
| Flutter State Management | **7/10** | `mounted` kontrolü iyi, 2-3 race condition var |
| Flutter UI/UX | **6/10** | Türkçe karakter, eksik navigasyon, büyük dosyalar |
| Memory Leak Riski | **7/10** | Dispose'lar genel doğru, minor sorunlar |
| Genel Olgunluk | **7/10** | Beta için hazır, production öncesi güvenlik düzeltmeleri şart |

---

## 9. ÖNCELİK SIRASI EYLEM PLANI

### Sprint 1 — Güvenlik (Hemen)
- [ ] `auth.routes.js:35` — `/debug/redis` endpoint kaldır
- [ ] `gateway/src/app.js:61-80` — API key middleware aktifleştir
- [ ] `internalAuthMiddleware.js:12` — Fallback key kaldır, startup'ta hata ver
- [ ] `auth.service.js:781` — `changeEmail` lowercase + isVerified reset
- [ ] `auth.service.js:230` — Login rate limiting (express-rate-limit)

### Sprint 2 — Kritik Bug Fix
- [ ] `post.service.js:366` — `getExploreFeed` yeni kullanıcı bug
- [ ] `message.service.js:71` — `before` param validasyonu
- [ ] `auth_provider.dart:155` — `verifyEmail` başarı sonrası state
- [ ] `gym_match_screen.dart` — Türkçe karakterler

### Sprint 3 — UX ve Stabilite
- [ ] LoginScreen — "Şifremi Unuttum" linki
- [ ] `chat_screen.dart:59` — Listener race condition fix
- [ ] `auth_provider.dart:171` — `resendVerificationEmail` loading state
- [ ] `main_tab_shell_scaffold.dart:103` — Platform adaptive scroll physics

### Sprint 4 — Performans ve Kod Kalitesi
- [ ] Cursor-based pagination (explore feed)
- [ ] `nutritionAi.service.js:21` — `fs.readFileSync` cache
- [ ] `user.service.js:33` — `_enrichUserWithLocation` JOIN optimizasyonu
- [ ] `NutritionHomeScreen` widget bölünmesi
- [ ] `UNKOWN` → `UNKNOWN` global replace
- [ ] Expired match kayıtları cron job

---

*Rapor oluşturulma tarihi: 2026-04-14 | Analiz eden: Claude Sonnet 4.6 (project-manager-mode agent)*
