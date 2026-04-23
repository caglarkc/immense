# App (Flutter) Geliştirme Kuralları

> Flutter · Provider · go_router · Dio · Clean Architecture

---

## Klasör Yapısı

```
lib/
├── app/           → router (go_router), theme
├── shared/        → api, storage, models, services, utils, widgets
└── modules/       → auth, user, exercises, workouts, feed, stats, measurements, calendar, beta_feedback
    └── [modül]/
        ├── data/      → *_repository.dart
        ├── logic/     → *_provider.dart
        ├── screens/
        └── widgets/
```

Her özellik (`auth`, `user`, `training` vb.) kendi modülü içinde **tam izole** olmalıdır.
Modüller arası ortak kullanımlar (modeller, global servisler, Dio utils, ortak widget'lar) `shared/` altındadır.

---

## Katmanlı Mimari

```
UI (Screen)  →  Provider (Logic)  →  Repository (Data)  →  DioClient (API)
```

- **UI katmanında HTTP isteği (API çağrısı) ASLA yapılmaz.**
- Provider iş mantığını ve state'i yönetir.
- Repository, DioClient aracılığıyla API haberleşmesini yapar.
- DioClient tek HTTP istemcisidir.

---

## API Katmanı

| Bileşen | Konum | Görevi |
|---------|-------|--------|
| DioClient | `shared/api/` | Tek HTTP istemcisi |
| ApiEndpoints | `shared/api/api_endpoints.dart` | Tüm endpoint'ler merkezi |
| ApiInterceptor | `shared/api/` | Otomatik `x-device-id` ve `Authorization` header ekler, `X-New-Access-Token` yakalar |
| ApiResponse\<T\> | `shared/api/` | `when(success:, error:)` pattern ile standart akış |

**YASAKLAR:**
- URL hardcode verilmez → Sadece `ApiEndpoints.xxx` kullanılır.
- Token/device-id manuel eklenmez → Interceptor otomatik yönetir.

---

## State Management (Provider)

- `main.dart` içinde singleton repository'ler ve `MultiProvider` kurulumu yapılır.
- Provider'lar `ChangeNotifier` tabanlıdır.
- **Constructor Injection zorunlu:** `AuthProvider(this._repository)` → Hard instantiation (`= AuthRepository()`) **YASAK**.
- UI tarafında `Consumer` veya `context.watch/read` kullanılır.
- **`setState` KULLANILMAZ.**
- `LoadState` enum: `initial`, `loading`, `success`, `error`.
- Loading, error ve empty state açıkça yönetilir; kullanıcı **asla** boş/donuk ekranda bırakılmaz.

---

## DİKKAT — Frontend’de veri nasıl tutulmalı? (zorunlu netleştirme)

> **Yeni özellik, yeni ekran veya yeni veri akışı** eklerken sadece API ve UI yeterli değildir: **veri istemcide nerede, ne kadar süre ve hangi yenileme kuralıyla duracak** ayrı ve bilinçli bir karardır. Bu dosyadaki “Login / App-open / önbellek” tabloları **örnektir**; her özellik için aynı varsayılanı kullanmak zorunda değilsin.

**Ajan ve geliştirici için kural:** Bu görevde veya `app.md` içinde bu veri için **açık bir tutma stratejisi yoksa**, kod yazmadan önce **mutlaka** şu başlıklarda netleştirme iste (gerekirse ürün sahibine / kullanıcıya sor):

| Başlık | Netleştirilecek soru |
|--------|----------------------|
| **Konum** | Sadece bellek (Provider, oturum boyu) mı, Drift/SQLite mı, `SharedPreferences` özet mi, yoksa her seferinde API mi? |
| **Ömür** | Uygulama kapanınca / çıkışta silinsin mi, cihazda kalsın mı? |
| **Tazelik** | Giriş bootstrap’ta mı yüklensin, sync’e mi bağlansın, ekran açılışında mı, sadece kullanıcı yenilemesinde mi? |
| **Tutarlılık** | Sunucu değişince istemci nasıl haberdar olacak (push, periyodik çekme, manuel yenileme)? |

**Yapılmaması gereken:** Belgede veya görevde yazmadan “hep bellekte tutarız”, “her girişte çekeriz” gibi **sessiz varsayılanlar** seçmek. Karar verildikten sonra ilgili modül yorumunda veya bu dosyada kısa bir cümleyle **sabitlenmeli**; böylece sonraki değişikliklerde aynı hata tekrarlanmaz.

---

## Tema ve Stil

- `context.appTheme.primaryColor` (AppThemeExtension) kullanılır.
- İlgili dosyalar: `app_colors.dart`, `app_theme.dart`, `app_spacing.dart`.
- **Hardcode renk, text style YASAK.** Sadece `context.theme`, `context.textTheme` veya AppTheme referansları kullanılır.

---

## Router

- `go_router` + `AppRouter.router`.
- `buildPageWithTransition` ile animasyon (fade, slide).
- `initialLocation: '/splash'`.

---

## Uygulama Açılışında Yüklenen Veriler (App-open)

Token yoksa ve ilk açılışsa `loadAppOpenData` çalışır:

| Veri | Açıklama | Konum |
|------|----------|-------|
| Şehirler, ilçeler | Lokasyon verileri | `LocationDataManager` |
| Sistem egzersizleri | API'den alınan standart egzersiz listesi | `ExerciseDataManager.loadSystemExercises` |
| Feedback soruları | Beta geri bildirim soruları | `BetaFeedbackQuestionsProvider` |

Bu veriler çıkışta silinmez, uygulama kapatılsa bile kalır.

---

## Kullanıcı Giriş Yaptığında Yüklenen Veriler (Login)

Token varsa veya giriş sonrası `loadAfterLogin` → `loadLoginData` → `loadProfileAndFollow` sırasıyla çalışır:

| Veri | Açıklama |
|------|----------|
| Auth | Token doğrulama, kullanıcı oturum bilgisi |
| Sync | Bekleyen sync event'leri işlenir |
| Egzersizler | Custom + sistem egzersizleri |
| Rutinler | Kullanıcının antrenman rutinleri |
| Antrenmanlar | Geçmiş antrenmanlar |
| Kişisel istatistikler | Kişisel egzersiz istatistikleri |
| Ölçümler | Vücut ölçümleri |
| İstatistikler | Genel istatistik özeti |
| Feedback cevapları | Kullanıcının beta feedback cevapları |
| Postlar | Kullanıcının kendi postları |
| Profil | Kullanıcı profil bilgisi |
| Takip istatistikleri | Takipçi / takip edilen sayıları |
| Koçluk (hub) | `CoachingProvider.loadHub` — kayıt özeti, `isCoach` vb. |
| Koç paneli + üye önbellek | Koç ise `loadCoachPanel` sonrası her üye için detay + atanmış şablon rutin id'leri bellekte prefetch |

Bu veriler çıkışta silinir (`clearAppData`).

### Veri ne sıklıkla çekilsin? (Koçluk)

- **Sync portu** koçluk için ayrıca kullanılmaz; admin panel bu verileri sync’e bağlamıyor.
- **Giriş (`loadAfterLogin`)**: Hub yüklenir; kullanıcı koç ise panel listesi çekilir ve paneldeki her üye için üye detayı + atanmış kaynak rutin id’leri **bir kez** prefetch edilir (bellek önbelleği).
- **Üye detay / rutin atama ekranı**: Öncelik bellek önbelleği; **ekran her açıldığında tekrar API yok**. İstek yalnızca **Yenile / Tekrar dene** ile veya önbellekte veri yokken (kenar durum) yapılır.
- Rutin atama ekranındaki rutin listesi, login’de yüklenen `RoutineProvider` belleğinden üretilir; ayrıca `GET /training/routines` atılmaz (yenilemede `refreshFromApi`).

---

## Özel Durumlar

- `fromCacheOnly = true` (sync boş, daha önce giriş yapılmış): Antrenmanlar, ölçümler, istatistikler ve beta cevapları API'den çekilmez; ilgili ekranlara girildiğinde lazy-load ile yüklenir.
- **Lazy-load ekranları:** `MeasurementsScreen`, `CalendarScreen`, `StatsScreen` vb. kendi verilerini gerektiğinde yükleyebilir.
- **`main.dart`:** Sadece Firebase, FCM token, veritabanı ve repository singleton'ları başlatılır; veri çekimi yapılmaz.

---

## Kritik Kurallar (İhlal Edilemez)

### 1. context.mounted Kontrolü
`await` sonrası `BuildContext` kullanılacaksa:
```dart
await someAsyncOperation();
if (!context.mounted) return; // ZORUNLU
// Artık context güvenle kullanılabilir
```

### 2. Hata Yönetimi
- Cihaz offline, timeout, HTTP hataları (401, 403 vb.) → Provider üzerinden UI'a anlaşılır mesajlar/durumlar dönülür.
- **Hiçbir senaryoda boş ekran gösterilmez.**

### 3. Dil Kuralı
- Tüm açıklamalar ve yorumlar → **Türkçe**
- Değişken, fonksiyon, sınıf isimleri → **İngilizce**

### 4. Repository Pattern Örneği
```dart
final response = await _client.post(
  ApiEndpoints.login,
  data: {...},
);
return ApiResponse.fromJson(response.data, fromJsonT);
```

---

## Yapılacak / Yapılmayacak Özet Tablosu

| ✅ YAP | ❌ YAPMA |
|--------|----------|
| Katmanlı mimari uygula | UI'da direkt API çağrısı |
| Provider + constructor injection | setState ile state yönetimi |
| `context.mounted` kontrol et | await sonrası kontrolsüz context |
| `ApiEndpoints.xxx` kullan | Hardcode endpoint/URL |
| Tema extension'ları kullan | Hardcode renk/stil |
| Loading + error state göster | Kullanıcıyı boş ekranda bırakma |
| Interceptor'a güven | Manuel token/device-id ekleme |
| Yeni provider'ı `main.dart` MultiProvider'a ekle | Provider kayıt etmeyi unutma |
| Veri tutma stratejisini sor / `app.md` veya modülde sabitle | Varsayılan önbellek veya çekme politikasıyla sessizce ilerlemek |