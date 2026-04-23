# User Service Audit

## Backend Yuzeyi

- Route: `backend/services/user-service/src/routes/user.routes.js`
- Modeller:
  - `user.model.js`
  - `city.model.js`
  - `district.model.js`
  - `gym.model.js`
  - `beta_feedback*.model.js`
- Servisler:
  - `user.service.js`
  - `location.service.js`
  - `media.service.js`
- Ana endpointler:
  - `POST /user/register`
  - `GET/PUT /user/users/me`
  - `DELETE /user/users/me/photo`
  - `DELETE /user/users/me`
  - `DELETE /user/users/me/permanent`
  - `POST /user/users/me/restore`
  - `GET /user/users/by-gym/:gymId`
  - `GET /user/cities`
  - `GET /user/districts`
  - `GET /user/gyms/search`
  - `POST /user/gyms`
  - `GET /user/beta-feedback/questions`
  - `GET/POST /user/beta-feedback/answers`
  - `GET /media/token`
  - `GET /media/presigned-url`

## Frontend Baglantilari

- Profile data/logic:
  - `frontend/lib/modules/profile/data/user_repository.dart`
  - `frontend/lib/modules/profile/logic/user_provider.dart`
  - `frontend/lib/modules/profile/data/location_repository.dart`
  - `frontend/lib/modules/profile/logic/location_provider.dart`
- Screens:
  - `profile_screen.dart`
  - `edit_profile_screen.dart`
  - `settings_screen.dart`
  - `blocked_users_screen.dart`
  - `reported_users_screen.dart`
- Beta feedback:
  - `frontend/lib/modules/beta_feedback/*`

## Veri ve State

- Kullanici profili bellek icinde `UserProvider` ile tutuluyor.
- Lokasyon, gym ve beta feedback sorulari app-open/login bootstrap fazlarina bagli.
- Medya URL cozumleme `MediaRepository` ve `MediaUrlService` ile ortak servis olarak kullaniliyor.

## Ana Akislar

- Kullanici detayli profil tamamlama
- Profil guncelleme, foto silme/yukleme
- Gym secme/arama/yeni gym ekleme
- Ayarlar icinden hesap silme, engellenenler, raporlananlar
- Beta feedback sorulari ve cevaplarinin ilerlemeli kaydi

## Buton ve Ekran Davranis Mantigi

- Edit profile ekraninda form alanlari, lokasyon secicileri, gym secme, foto yonetimi, kaydet CTA'si var.
- Settings ekrani diger hesap/rapor/block ekranlarina gecis dugumu.
- Profile screen kullanicinin kendi post, istatistik ve takip ozetini gosteren merkez ekran.

## Risk ve Notlar

- `edit_profile_screen.dart` icinde kullanilmayan alanlar ve birkac warning var.
- Medya/presigned URL zinciri birden fazla modulu etkiliyor; user-service kaynakli medya sorunlari feed, profile, exercise ve coaching'i da vurur.
- Beta feedback kullanici servisine bagli ama uygulama bootstrapi icinde oldugu icin acilis performansina etkisi olabilir.
