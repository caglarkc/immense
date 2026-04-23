# Sync Service Audit

## Backend Yuzeyi

- Route: `backend/services/sync-service/src/routes/sync.routes.js`
- Service: `backend/services/sync-service/src/services/sync.service.js`
- Endpointler:
  - `GET /sync/status`
  - `GET /sync/pending`
  - `POST /sync/internal/events`
  - `POST /sync/internal/follow`

## Frontend Baglantilari

- Data:
  - `frontend/lib/app/data/sync_repository.dart`
- Logic:
  - `frontend/lib/app/data/sync_handler.dart`
- App orchestrasyon:
  - `frontend/lib/app/services/app_data_manager.dart`
  - `frontend/lib/app/widgets/app_lifecycle_handler.dart`

## Veri ve State

- Sync eventleri bootstrapte `AppDataManager` tarafinda cekilip `SyncHandler` ile isleniyor.
- Event sonucu ilgili provider ya refresh oluyor ya da local override/invalidate uygulaniyor.
- Block, follow, routine/workout/stats, exercise, feedback ve post deletion gibi alanlara etkisi var.

## Ana Akislar

- App acilisinda pending event cekme
- Event tipine gore local refresh:
  - auth/user
  - location/routine/workout/exercise/stats/measurement
  - relationship state
  - post deletion
  - follower/following delta

## Risk ve Notlar

- Sync service dogrudan UI gostermiyor ama veri tazeliginin omurgasi.
- En kritik kisim: relationship eventlerinin message/profile/match cache'lerine tutarli yansimasi.
- Sync event yoksa cache-only startup moduna gecilmesi performans icin iyi, ama stale data riski bu servisin en hassas alani.
