# MD-2: Ürün ve Özellik Analizi Raporu
## Immense Fitness Platform — Ürün Derinlik Değerlendirmesi

> **Rapor Tarihi:** 2026-04-20 *(önceki tam tarama: 2026-03-24)*
> **Analiz Kapsamı:** Core özellikler, UX akışları, teknik karşılıklar, özellik boşlukları
> **Metodoloji:** Kaynak kodu, route/controller/screen analizi, alan haritalaması

---

## 1. ÜRÜN KİMLİĞİ

### 1.1 Ürün Nedir?

**Immense**, bireysel fitness kullanıcıları için tasarlanmış, antrenman takibi + sosyal paylaşım + AI hareketi öğrenme yeteneklerini bir arada sunan kapsamlı bir mobil fitness platformudur.

```
Immense = Antrenman Takipçisi
        + Sosyal Fitness Ağı
        + Kişiselleştirilmiş Egzersiz Kütüphanesi
        + İstatistik & Ölçüm Takibi
        + Beslenme Takibi (öğün/gıda/hedef; AI ile fotoğraftan öğün — devam eden)
        + [Planlanan] AI Hareket Analizi
```

### 1.2 Hedef Kullanıcı (Ürün Perspektifinden)

| Kullanıcı Tipi | Tanım | Birincil İhtiyaç |
|---|---|---|
| Aktif Sporcular | Düzenli spor yapan, ilerlemeyi takip etmek isteyen | Antrenman kayıt + istatistik |
| Sosyal Fitness Kullanıcıları | Antrenmanlarını paylaşmak isteyen | Feed + beğeni + yorum |
| Yeni Başlayanlar | Egzersiz öğrenmek isteyen | Hareket kütüphanesi + form rehberliği |
| Koçlar/PT'ler | Müşterilere program hazırlayan | Koçluk modülü (MVP): başvuru, admin onayı, davet, panel, üye detayı — program atama / etkileşim geçmişi sırada; beslenme tarafı üye hedefleriyle tam entegrasyon henüz ürünleşmedi |

---

## 2. CORE ÖZELLİKLER HARİTASI

### 2.1 Kimlik Doğrulama & Onboarding

**Backend:** `auth-service` (port 3001)
**Frontend:** `auth` modülü (splash, login, register, email_verification)

```
Kullanıcı Akışı:
SplashScreen → [Token var?]
    ├── Evet → Ana Sayfa
    └── Hayır → LoginScreen
                ├── Kayıtlı → Giriş → Email Doğrulama → Ana Sayfa
                └── Yeni → RegisterScreen → Email OTP → Ana Sayfa
```

**Teknik Karşılıklar:**
- JWT (access + refresh token) — `token.js` shared utility
- Email doğrulama: OTP tabanlı — `emailService.js` SMTP
- Device ID takibi: `device_manager.dart` → her istekte `x-device-id` header
- Güvenli token saklama: `flutter_secure_storage`

**Ürün Değerlendirmesi:**
- ✅ E-posta OTP doğrulama mevcut — güvenli onboarding
- ❌ Social login (Google/Apple Sign-in) yok — kullanıcı edinme sürtüşmesi
- ❌ Onboarding wizard (seviye/hedef/tercih toplama) yok — kişiselleştirme fırsatı kaçırılıyor
- ❌ Şifre sıfırlama akışı backend'de muhtemelen var ama frontend durumu belirsiz

---

### 2.2 Antrenman Takip Sistemi

**Backend:** `training-service` (port 3003)
**Frontend:** `workouts` modülü (workout_provider, routine_provider)

#### 2.2.1 Veri Modeli

```
Routine (Antrenman Programı)
├── id, userId, name, description
├── workouts[] → Workout[]
│
Workout (Antrenman Seansı)
├── id, routineId, userId
├── exercises[] → WorkoutExercise[]
│   ├── exerciseId
│   ├── sets[]
│   │   ├── setNumber
│   │   ├── reps
│   │   ├── weight
│   │   └── restTime
│   └── notes
└── completedAt, duration
```

#### 2.2.2 Antrenman Akışı

```
RoutineListScreen
    │ Rutin seç veya oluştur
    ▼
WorkoutScreen (aktif antrenman)
    │ Egzersiz ekle → ExercisePicker
    │ Set/rep/ağırlık gir
    │ Rest timer
    │ Tamamla
    ▼
WorkoutCompleted
    │ Stats otomatik hesaplanır (RabbitMQ → stats-service)
    │ [İsteğe bağlı] Sosyal medyaya paylaş
    ▼
StatsDashboard (güncellendi)
```

**Teknik Karşılık:**
- `training-service` → workout CRUD
- `RabbitMQ event` → `stats-service` güncelleme tetiklenir
- `sync-service` (Socket.io) → gerçek zamanlı UI güncelleme

**Ürün Değerlendirmesi:**
- ✅ Rutin → Antrenman → Set hiyerarşisi doğru modellenmış
- ✅ Ağırlık/rep/set takibi mevcut
- ❌ **Rest timer** uygulaması belirsiz (kritik özellik)
- ❌ Süper set / drop set / piramit set türleri yok
- ❌ 1RM hesaplama (Epley formülü) yok — güç takipçileri için temel
- ❌ Antrenman şablonları / program kopyalama belirsiz
- ❌ Antrenman takvimi görünümü (kalender modülü var, ama entegrasyon?)

---

### 2.3 Egzersiz Kütüphanesi

**Backend:** `exercise-service` (port 3004)
**Frontend:** `exercises` modülü (exercise_detail, exercise_form, exercises screens)

#### 2.3.1 Egzersiz Modeli

```
Exercise
├── id, name, description
├── primaryMuscles[]    (target-matrix.js ile eşleştirilmiş)
├── secondaryMuscles[]
├── equipment
├── category (strength, cardio, flexibility, vb.)
├── mediaUrls[] (MinIO'da video/gif/image)
└── isCustom (sistem vs. kullanıcı egzersizi)
```

**Kişisel Egzersiz:**
`personal_exercises` (stats-service'te) — kullanıcının oluşturduğu özel egzersizler. Bu güçlü bir özellik; kullanıcıların kütüphaneyi kişiselleştirmesini sağlıyor.

**Ürün Değerlendirmesi:**
- ✅ System + custom egzersiz ayrımı
- ✅ Kas grubu haritalaması (target-matrix.js)
- ✅ Media (video/gif) desteği MinIO ile
- ✅ Admin panel üzerinden egzersiz yönetimi
- ❌ **Egzersiz arama / filtreleme** UX'i belirsiz
- ❌ Alternatif egzersiz önerisi yok (dumbbell bench press → barbell bench press)
- ❌ Zorluk seviyesi / deneyim etiketleme yok
- ❌ **AI form analizi** — planlı ama mevcut değil (projenin en büyük fırsatı)

---

### 2.4 İstatistik ve Ölçüm Takibi

**Backend:** `stats-service` (port 3005)
**Frontend:** `stats` + `measurements` modülleri

#### 2.4.1 Veri Yapısı

```
Stats (Antrenman İstatistikleri — otomatik)
├── userId
├── totalWorkouts
├── totalSets
├── totalReps
├── totalVolume (kg × reps toplamı)
├── personalRecords (egzersiz bazlı)
└── weeklyActivity

Measurement (Vücut Ölçümleri — manuel)
├── userId, date
├── weight, bodyFat, muscleMass
├── chest, waist, hip, arm, leg (cm)
└── photos[] (progress photos)
```

**Teknik Akış:**
```
Workout tamamlandı
    │ RabbitMQ event: "workout.completed"
    ▼
stats-service consumer
    │ Otomatik hesaplama
    │ - Toplam hacim güncelle
    │ - PR kontrolü ve güncelle
    │ - Haftalık aktivite güncelle
    ▼
sync-service
    │ Socket.io event: "stats.updated"
    ▼
Flutter UI güncellendi
```

**Görselleştirme:**
`fl_chart ^0.69.0` — güçlü Flutter chart kütüphanesi. Ağırlık ilerleme grafiği, hacim trendi, kas grubu dağılımı gibi grafikler desteklenebilir.

**Ürün Değerlendirmesi:**
- ✅ Otomatik stat hesaplama (tetikleyici tabanlı)
- ✅ Vücut ölçüm takibi
- ✅ Progress fotoğrafı desteği
- ❌ İlerleme raporları (PDF export) yok
- ❌ Zaman bazlı karşılaştırma ("bu ay vs. geçen ay") belirsiz
- ❌ Güç standartları karşılaştırması (kg bazında "bench press seviyeniz: orta") yok
- ✅ **Beslenme (2026-04):** `nutrition-service` + Flutter `modules/nutrition` — günlük öğünler, gıda veritabanı, hedefler, programlar, su ve takviye kaydı; AI ile fotoğraftan öğün analizi (Gemini) akışı kodda mevcut; admin panel ve koç–üye rapor entegrasyonu aşamalı geliştirilebilir

---

### 2.5 Sosyal Medya & Topluluk

**Backend:** `social-media-service` (port 3006)
**Frontend:** `feed` modülü

#### 2.5.1 Sosyal Model

```
Post (Gönderi)
├── userId, content, mediaUrls[]
├── workoutId? (antrenman paylaşımı)
├── gymId? (konum etiketi)
├── likes[], saves[], comments[]
└── visibility (public/private?)

Sosyal Graf:
├── Follow (takipçi/takip edilen)
├── Block (engel)
└── Report (şikayet)
```

**Güvenlik & Moderasyon:**
`block`, `report` ve admin panel üzerinden moderasyon — topluluk yönetimi düşünülmüş.

**Ürün Değerlendirmesi:**
- ✅ Temel sosyal medya döngüsü (post/like/yorum/kaydet/takip)
- ✅ Antrenman paylaşımı (workout → post bağlantısı)
- ✅ İçerik moderasyon altyapısı (blok/rapor)
- ⚠️ **Push bildirim** — `notification-service` içinde FCM ve token kaydı mevcut; tüm ürün yüzeyinde (sosyal, koçluk, hatırlatıcı) tutarlı kullanım henüz tamamlanmış değil
- ❌ Hikaye (Story/24h içerik) formatı yok
- ❌ Canlı antrenman özelliği yok
- ❌ Fitness challenge / yarışma özelliği yok
- ❌ Grup/topluluk oluşturma yok
- ❌ Hashtag sistemi yok
- ❌ Keşfet / Trending içerik yok

---

### 2.6 Lokasyon & Spor Salonu Sistemi

**Backend:** `user-service` (lokasyon routes)
**Frontend:** `profile/location` modülü

```
Lokasyon Hiyerarşisi:
Country → City → District → Gym

user.gymId → Gym (isim, koordinat, fotoğraf)
```

**Ürün Değerlendirmesi:**
- ✅ Spor salonu etiketleme — Türkiye'deki gym kültürüne uygun
- ❌ Harita entegrasyonu (Google Maps/Mapbox) belirsiz
- ❌ Yakındaki spor salonlarını göster
- ❌ Gym check-in / ziyaretçi akışı yok
- ❌ Gym tabanlı sosyal özellikler (aynı salondaki kişiler) yok

---

### 2.7 Senkronizasyon Mimarisi

**Backend:** `sync-service` (port 3007)
**Frontend:** `api_interceptor.dart` + Drift SQLite

```
Senkronizasyon Akışı:
Kullanıcı Aksiyon (Flutter)
    │ DioClient HTTP request
    ▼
Backend Service (işle)
    │ RabbitMQ event yayınla
    ▼
sync-service (Socket.io)
    │ İlgili kullanıcılara push
    ▼
Flutter Socket listener
    │ Provider güncelle
    ▼
UI yenilendi
```

**Offline Senaryosu:**
Drift SQLite ile yerel yazma → internet bağlantısı geldiğinde sync. Bu çok değerli bir özellik — özellikle spor salonlarında sinyal zayıf olabilir.

---

### 2.8 Beta Geri Bildirim Sistemi

`beta_feedback` modülü: sorulardan oluşan anket → kullanıcı yanıtları → admin panel görüntüleme.

Bu özellik beta sürecinde ürün geri bildirimi toplamak için doğru bir mekanizma. Ürünün henüz beta aşamasında olduğunu gösteriyor.

---

### 2.9 Takvim Modülü

`calendar` modülü — özel takvim bileşenleri (month_view, year_view, multi_year_view).

**Beklenen Kullanım:** Antrenman geçmişini takvimde göster, gelecek antrenmanları planla.

**Değerlendirme:** Özel takvim bileşeni yazmak ciddi bir yatırım. `table_calendar` gibi hazır kütüphane yerine bu tercih performans veya UX esnekliği gerektiriyordur.

---

### 2.10 Beslenme (Nutrition) — MVP Durumu

**Backend:** `nutrition-service` (port **3012**), gateway: `/nutrition` → `/api/nutrition`. PostgreSQL modelleri: öğün (`meal`), gıda (`food`), hedef (`target`), program (`program` / `programMeal`), su (`waterLog`), takviye (`supplement` / `supplementLog`), AI foto günlük kullanım ve geri bildirim tabloları. İsteğe bağlı **Google Gemini** (`@google/generative-ai`) ile görsel öğün analizi; `multer` ile yükleme.

**Mobil (Flutter):** `nutrition_home_screen`, öğün ekleme/detay, gıda listesi/form, hedef ve program ekranları, geçmiş, su/takviye, AI fotoğraf akışı (`nutrition_ai_photo_*`). Repository + provider katmanı diğer modüllerle uyumlu.

**Değerlendirme:** Antrenman + beslenme birleşimi için güçlü temel; ürün olgunluğu (barkod, üçüncü parti kalori API’leri, diyetisyen/koç paylaşımı) aşamalı genişletilebilir.

---

### 2.11 Koçluk (Coaching) — MVP Durumu

**Backend:** `coaching-service` (gateway: `/coaching` → `/api/coaching`), PostgreSQL modeller: koç kaydı (`coach_registration`), onaylı koç (`coach`), üyelik (`coach_membership`), davet (`coach_invite`). Dahili çağrılar: `user-service` (profil görüntüleme), MinIO (belge yükleme), RabbitMQ (olay/bildirim hattı).

**Admin:** `CoachesAdminPage`, `CoachDocumentsPage` — başvuru ve belge inceleme hattı.

**Mobil (Flutter):**
- **Üye / keşif:** Koçlar listesi, koç detay (avatar, bio; eğitim, başarı ve deneyim maddeleri; kullanıcı profilinden spor alanı ve deneyim yılı gibi alanlar).
- **Koç adayı:** Çok aşamalı kayıt; eğitim / başarı / deneyim satırları tekil alanlar olarak toplanır.
- **Koç oturumu:** Ana koçluk ekranı doğrudan **koç paneli** (genel hub sekmeleri yerine): aktif üyeler, gönderilmiş davetler, üye çıkarma (zorunlu onay metni + isteğe bağlı gerekçe), kısayollar mevcut rutin ve egzersiz yönetimine; toplu bildirim/mesaj henüz yer tutucu.
- **Üye detayı:** Foto, kullanıcı adı, isim, koçluk başlangıcı, spor alanı, deneyim; altta rutin/egzersiz/geçmiş antrenman/olaylar/ölçüm/istatistik/haftalık plan/bildirim için butonlar (çoğu şimdilik “yakında”).

**Henüz yok / planlanan:** Üyeye rutin kopyalama, koçun üyenin tüm rutinlerini yönetmesi, etkileşim geçmişi (tek model + enum `type` + JSON `payload`), koçluk aboneliği ve Google Play faturalandırması.

---

## 3. ÖZELLİK OLGUNLUK MATRİSİ

| Özellik | Durum | Teknik Olgunluk | UX Kalitesi |
|---|---|---|---|
| Kimlik Doğrulama | ✅ Tamamlandı | Yüksek | Orta |
| Antrenman Takibi | ✅ Core mevcut | Yüksek | Bilinmiyor |
| Egzersiz Kütüphanesi | ✅ Mevcut | Orta | Orta |
| İstatistik Takibi | ✅ Mevcut | Yüksek | Bilinmiyor |
| Vücut Ölçümleri | ✅ Mevcut | Orta | Bilinmiyor |
| Sosyal Feed | ✅ Core mevcut | Orta | Bilinmiyor |
| Takip/Engel/Rapor | ✅ Mevcut | Orta | — |
| Senkronizasyon | ✅ Mevcut | Yüksek | — |
| Offline Mod | ✅ Kısmi (Drift) | Orta | — |
| Push Bildirim | ⚠️ Kısmen | notification-service (FCM) + cihaz token API | Ürün senaryolarında tutarlı tetikleme / koçluk bildirimleri tamamlanmadı |
| Social Login | ❌ Yok | — | — |
| AI Form Analizi | ❌ Yok | — | — |
| Koç/PT Araçları | ⚠️ MVP | Orta-Yüksek | Panel + üye yönetimi iyi temel; program/ölçüm entegrasyonu eksik |
| Beslenme Takibi | ⚠️ MVP | nutrition-service + Flutter modül; AI foto | Akışlar kodda; ürün cilası ve entegrasyonlar sürüyor |
| Bildirim Sistemi | ⚠️ Altyapı var | — | — |
| Fitness Challenge | ❌ Yok | — | — |

---

## 4. KULLANICI DENEYİMİ (UX) DEĞERLENDİRMESİ

### 4.1 Güçlü UX Kararları

1. **Tema Sistemi:** AppTheme extensions ile tutarlı tasarım dili
2. **Yükleme Durumları:** LoadState enum ile her ekranda loading/error/success ayrımı
3. **Güvenli Depolama:** Hassas veriler secure storage'da
4. **Offline Capability:** Spor salonunda internet olmasa bile antrenman kaydedebilme
5. **cached_media_image:** Medya tekrar yükleme olmadan hızlı görüntüleme

### 4.2 UX Boşlukları

**Onboarding Deneyimi:**
Kayıt → OTP → Ana Sayfa akışı çalışıyor ama ilk kullanımda kişiselleştirme yok. Kullanıcıya "Amacın ne? (Kilo vermek / kas kazanmak / dayanıklılık)" sorusu sorulsa, öneri sistemi için zengin veri toplanır.

**Antrenman Sırasında UX:**
Aktif antrenman sırasında kullanıcının ihtiyaç duyduğu şeyler: büyük set/rep giriş butonları, otomatik rest timer, bir sonraki egzersiz önizlemesi. Bu detaylar ekran implementasyonuna bağlı — kod analizi sınırlı olduğundan değerlendirme yapılamadı.

**Sosyal Keşif:**
Kullanıcıların birbirini bulmasını sağlayan keşfet akışı kritik. Hashtag, konum bazlı keşif, önerilen kullanıcılar gibi özellikler sosyal ağın büyümesi için zorunlu.

### 4.3 Erişilebilirlik (Accessibility)

`flutter_svg` ve özel widget'lar kullanılıyor. Erişilebilirlik (Semantics widget, font scaling, dark mode) durumu belgelenmemiş.

---

## 5. PLATFORM STRATEJİSİ

### 5.1 Mevcut Durum

| Platform | Durum |
|---|---|
| iOS | ✅ Flutter cross-platform |
| Android | ✅ Flutter cross-platform |
| Web | ⚠️ Flutter Web mümkün ama optimize edilmemiş |
| Desktop (macOS/Win) | ❌ Hedef dışı |
| Apple Watch | ❌ Hedef dışı |
| Wear OS | ❌ Hedef dışı |

### 5.2 Eksik Platform Fırsatları

**Wearable Entegrasyonu:** Apple Watch ve Wear OS fitness verisi (kalp atışı, kalori) entegrasyonu olmadan, fitness app kategorisinde ciddi bir rekabet dezavantajı var. HealthKit (iOS) ve Health Connect (Android) entegrasyonu minimum beklenti.

---

## 6. VERİ MİMARİSİ VE GİZLİLİK

### 6.1 Toplanan Veriler

Immense aşağıdaki kişisel verileri topluyor:
- Kimlik: e-posta, cihaz ID
- Profil: isim, yaş, fotoğraf
- Sağlık: vücut ölçüleri, kilo, vücut yağ oranı, egzersiz geçmişi
- Lokasyon: şehir, ilçe, spor salonu
- Sosyal: gönderiler, beğeniler, takip ilişkileri

**KVKK/GDPR Uyumluluğu:**
Sağlık verisi özel kategori kişisel veri kapsamında. Açık rıza, veri saklama politikası, silme hakkı uygulaması — bunların hepsi beta öncesinde belgelenmeli.

---

## 7. ÖZELLİK ÖNCELIK MATRİSİ (Sonraki Geliştirme için)

```
                    DEĞER
                    Yüksek │
                           │  Push Bildirim  Social Login
                           │
                           │  AI Form        Fitness
                           │  Analizi        Challenge
                    Düşük  │  Koç Araçları   Beslenme (MVP→)
                           └─────────────────────────
                           Düşük            Yüksek
                                    EFOR
```

| Özellik | İş Değeri | Teknik Efor | Öncelik |
|---|---|---|---|
| Push Bildirim (FCM/APNs) | Yüksek | Düşük | 🔴 Hemen |
| Social Login (Google/Apple) | Yüksek | Düşük | 🔴 Hemen |
| HealthKit / Health Connect | Yüksek | Orta | 🟠 Kısa vadeli |
| Rest Timer (UI) | Yüksek | Düşük | 🔴 Hemen |
| Keşfet / Trending Feed | Yüksek | Orta | 🟠 Kısa vadeli |
| Fitness Challenge | Yüksek | Yüksek | 🟡 Orta vadeli |
| AI Form Analizi | Çok Yüksek | Çok Yüksek | 🟡 Orta vadeli |
| Koç/PT Araçları (program + faturalama) | Yüksek | Yüksek | 🟠 Devam eden MVP → kısa-orta vade |
| Beslenme Takibi (MVP → olgun) | Yüksek | Orta–Yüksek | 🟠 Kısa–orta vade (temel kod mevcut) |

---

## 8. SONUÇ

Immense'in ürün çekirdeği — antrenman takibi, egzersiz kütüphanesi, istatistik, sosyal paylaşım, **beslenme MVP** — sağlam bir temel üzerinde inşa edilmiş. Teknik mimari, bu özellikleri genişletmek için hazır. Ancak ürün henüz **"tamamlanmış bir fitness uygulaması" için minimum beklenti sınırının altında** birkaç eksiklik taşıyor:

1. **Push bildirim** altyapısı varken, **tetikleyiciler ve ürün akışı** (sosyal + koç + hatırlatma) tamamlanmalı; aksi halde elde tutma zayıflar.
2. **Social login** olmadan kullanıcı edinimi sürtüşmeli.
3. **HealthKit/Health Connect** olmadan fitness app değeri sınırlı.
4. **AI form analizi** olmadan rakiplerden ayrışma zor.
5. **Koçluk** tarafında ücretlendirme (ör. Google Play) ve derin antrenman entegrasyonu tamamlanmadan B2C koç modeli ölçeklenmez.
6. **Beslenme** tarafında gıda veritabanı zenginleştirme, AI maliyet/limit politikası ve koç/rapor entegrasyonu ürünleştirilmeli.

Bu başlıklar ilerledikçe Immense, **rekabetçi fitness + beslenme + koçluk** platformu konumuna yaklaşır.

---

## 9. RAPOR GÜNCELLEMESİ (2026-04-20)

**`nutrition-service` (3012)** ve Flutter **`modules/nutrition`** eklendi: öğün/gıda/hedef/program/su/takviye, AI fotoğraf ile öğün analizi akışı. İstatistik bölümündeki “beslenme yok” maddesi kaldırıldı; özellik matrisi, öncelik matrisi ve sonuç buna göre güncellendi. Koçluk MVP anlatımı önceki sürümlerle uyumludur.

---

*Sonraki Rapor: MD-3 — Pazar Analizi ve Büyüme Raporu*
