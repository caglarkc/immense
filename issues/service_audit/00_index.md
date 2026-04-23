# Servis Bazli Audit Index

Bu klasor, repo icindeki backend servisleri ile bunlara bagli Flutter katmanlari icin hazirlanmis yonetsel audit notlarini icerir.

## Ortak Mimari Ozeti

- Backend mimarisi mikroservis duzeninde: `gateway -> service -> controller -> service -> model`.
- Flutter mimarisi agirlikla `Screen -> Provider -> Repository -> DioClient` akisi ile calisiyor.
- Uygulama seviye bootstrap ve veri yukleme omurgasi:
  - `frontend/lib/main.dart`
  - `frontend/lib/app/services/app_data_manager.dart`
  - `frontend/lib/app/data/*_data_manager.dart`
  - `frontend/lib/app/router/app_router.dart`
- Flutter local persistence:
  - Drift: `frontend/lib/shared/database/app_database.dart`
  - Tablolar: `Exercises`, `Routines`, `PersonalExercises`, `MyPosts`, `Conversations`, `Messages`
- Runtime cache ve state:
  - `Provider` tabanli oturum ici bellek cache
  - Bazı alanlarda Drift + bellek hibrit kullanim
  - FCM + foreground heads-up + app lifecycle refreshleri mevcut

## Dosyalar

- `01_auth_service.md`
- `02_user_service.md`
- `03_social_media_service.md`
- `04_training_service.md`
- `05_exercise_service.md`
- `06_stats_service.md`
- `07_message_service.md`
- `08_match_service.md`
- `09_coaching_service.md`
- `10_nutrition_service.md`
- `11_notification_service.md`
- `12_sync_service.md`
- `13_admin_service.md`
- `99_bug_findings.md`

## Not

- Bu audit, kod incelemesi, route haritasi, provider/repository akisları ve statik analiz sonucuna dayanir.
- "Her butonun runtime davranisi" bolumleri ekran amaci, route, provider aksiyonlari ve repository endpointleri uzerinden cikartilmistir; tam davranis dogrulamasi icin manuel QA yine gerekir.
