# Training Service Audit

## Backend Yuzeyi

- Route: `backend/services/training-service/src/routes/training.routes.js`
- Modeller:
  - `routine.model.js`
  - `workout.model.js`
- Service: `training.service.js`
- Ana endpointler:
  - Rutin CRUD, clone
  - Workout save/list/detail/delete
  - Internal workout ve routine endpointleri
  - Calendar ve coach/member baglantili internal proxy endpointleri

## Frontend Baglantilari

- Data:
  - `frontend/lib/modules/workouts/data/routine_repository.dart`
  - `workout_repository.dart`
- Logic:
  - `routine_provider.dart`
  - `workout_provider.dart`
- Screens:
  - `workouts_screen.dart`
  - `active_workout_screen.dart`
  - `routine_detail_screen.dart`
  - `create_routine_details_screen.dart`
  - `edit_workout_screen.dart`
  - `workout_detail_screen.dart`
  - `share_workout_screen.dart`
  - `workout_history_screen.dart`
  - `history_hub_screen.dart`

## Veri ve State

- Rutinler Drift `Routines` tablosunda cacheleniyor.
- Aktif workout state'i servis + foreground lifecycle ile yonetiliyor.
- Workout listeleri agirlikla API + provider uzerinden geliyor.
- Calendar ve stats modulleri workout datasina bagli.

## Ana Akislar

- Workouts tab root:
  - Antrenmana basla
  - Programlar sheet'i
  - Egzersiz kutuphanesi gecisi
- Active workout:
  - set/reps/duration kaydi
  - dinlenme zamanlayici
  - antrenmani bitirme
- Workout save -> paylasim opsiyonu -> social media baglantisi
- Workout detail ve routine detail inceleme

## Buton ve CTA Mantigi

- `Antrenmana basla / devam et`
- Programlar sheet acma
- Rutin olusturma/duzenleme/silme
- Workout detayindan paylasma, duzenleme, silme

## Risk ve Notlar

- `edit_workout_screen.dart` ve `share_workout_screen.dart` tarafinda statik analyze warningleri mevcut.
- Training service sosyal medya, stats, coaching ve takvim ile capraz bagli; workout kayit zinciri urunun cekirdek akislarindan biri.
- Koctan uyee kopyalanan rutin/egzersiz akislarinda source id ve attribution tasinmasi ozel durum.
