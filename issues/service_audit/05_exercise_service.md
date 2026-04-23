# Exercise Service Audit

## Backend Yuzeyi

- Route: `backend/services/exercise-service/src/routes/exercise.routes.js`
- Modeller:
  - `exercise.model.js`
  - `exerciseFavorite.model.js`
- Service: `exercise.service.js`
- Ana endpointler:
  - `GET /exercises`
  - `GET /exercises/custom`
  - `POST/DELETE /exercises/custom/:exerciseId/favorite`
  - `POST /exercises/custom`
  - `PUT/DELETE /exercises/custom/:exerciseId`
  - cesitli internal copy/owned-custom/source-id endpointleri

## Frontend Baglantilari

- Data:
  - `exercise_repository.dart`
  - `personal_exercise_repository.dart`
- Logic:
  - `exercise_provider.dart`
  - `personal_exercise_provider.dart`
- Screens:
  - `exercises_screen.dart`
  - `exercise_detail_screen.dart`
  - `exercise_form_screen.dart`
- Widgetler:
  - `exercise_card.dart`
  - `library_filter_bar.dart`
  - `cached_media_image.dart`
  - `exercise_video_player.dart`

## Veri ve State

- Egzersizler Drift `Exercises` tablosunda tutuluyor.
- Sistem egzersizleri app-open, tam egzersiz listesi login bootstrapinde yukleniyor.
- Favori bilgisi ayni tabloda `isFavorite` kolonu ile persist ediliyor.
- Personal exercise stats ayri provider/repository ile stats-service tarafindan geliyor.

## Ana Akislar

- Egzersiz kutuphanesi: arama, ekipman filtresi, kas grubu filtresi, favoriler
- Egzersiz detail: ozet, gecmis, talimatlar, leaderboard
- Custom exercise olusturma ve duzenleme
- Workout ve coaching akislarindan egzersiz secimi

## Buton ve CTA Mantigi

- Kart tiklama -> detail
- Kalp ikonu -> favorite toggle
- Filtre sheet/chip acma
- Egzersiz ekle / duzenle / sil
- Coaching member ekranlarinda read-only veya attribution'li detail varyantlari

## Risk ve Notlar

- Favori akisi backend + drift + UI zincirine oturmus durumda.
- Media lifecycle burada hassas; scroll/tab rebuildleri tekrar yukleme hissi uretebiliyor.
- Egzersizler user-service medya token zincirine de bagli oldugu icin gorsel sorunlar capraz servis kaynakli olabilir.
