# Stats Service Audit

## Backend Yuzeyi

- Route: `backend/services/stats-service/src/routes/stats.routes.js`
- Modeller:
  - `stats.model.js`
  - `measurement.model.js`
  - `personal-exercise.model.js`
- Service: `stats.service.js`
- Ana endpointler:
  - `personal-exercises/list`
  - exercise history ve personal exercise detayi
  - measurements CRUD + photo upload
  - stats overview/summary/muscle-distribution/exercise-list/body-regions/leaderboard/calendar
  - internal workout PR endpointi

## Frontend Baglantilari

- Stats:
  - `frontend/lib/modules/stats/*`
- Measurements:
  - `frontend/lib/modules/measurements/*`
- Calendar:
  - `frontend/lib/modules/calendar/*`
- Providers:
  - `stats_provider.dart`
  - `measurement_provider.dart`
  - `calendar_provider.dart`

## Veri ve State

- Personal exercise verisi Drift `PersonalExercises` tablosunda cacheleniyor.
- Olcumler provider bazli tutuluyor; ekran bazli lazy-load da mevcut.
- Stats summary/overview verileri genelde bellek cache + refresh mantigi ile ilerliyor.

## Ana Akislar

- Stats ana ekrani
- Kas dagilimi, body region detail, egzersiz listesi, aylik rapor, leaderboard
- Calendar ekrani ve antrenman gun gorunumu
- Olcum ekleme, detay, foto goruntuleme, silme

## Buton ve CTA Mantigi

- Period filtreleri
- Body region secimleri
- Exercise leaderboard'a gecis
- Measurement ekle/guncelle/sil
- Photo viewer ve tarih listesi

## Risk ve Notlar

- Olcum upload ve stats ekranlari paylasim, workout, exercise ve coaching ile capraz bagli.
- Analyze sonucunda measurement ve stats widgetlari icinde agir yapisal hata yok, ama bazı import/underscore warningleri var.
- Coach-member stats proxy akisi coaching-service uzerinden de bu servise dayanir.
