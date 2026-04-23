# Auth Service Audit

## Backend Yuzeyi

- Route: `backend/services/auth-service/src/routes/auth.routes.js`
- Model: `backend/services/auth-service/src/models/auth.model.js`
- Service: `backend/services/auth-service/src/services/auth.service.js`
- Ana endpointler:
  - `POST /auth/register`
  - `POST /auth/send-verification-email`
  - `POST /auth/verify-email`
  - `POST /auth/login`
  - `POST /auth/forgot-password`
  - `POST /auth/reset-password`
  - `GET /auth/profile`
  - `POST /auth/change-password`
  - `POST /auth/change-email`
  - `POST /auth/change-phone`
  - `POST /auth/logout`
  - `POST /auth/logout-all`
  - `GET /auth/check-access`

## Frontend Baglantilari

- Data: `frontend/lib/modules/auth/data/auth_repository.dart`
- Logic: `frontend/lib/modules/auth/logic/auth_provider.dart`
- Screens:
  - `splash_screen.dart`
  - `login_screen.dart`
  - `register_screen.dart`
  - `email_verification_screen.dart`
- App orchestrasyon:
  - `frontend/lib/app/services/app_data_manager.dart`
  - `frontend/lib/app/data/auth_data_manager.dart`

## Veri ve State

- Token/device id ag katmaninda interceptor ve storage ile yonetiliyor.
- Login basarili oldugunda `AuthProvider` local login-veri cache temizligini tetikliyor.
- Splash akisi token yoksa login, token varsa arka plan bootstrap mantigina gecti.
- Auth state oturumun ana belirleyicisi; lifecycle ve socket baglantisi buna gore acilip kapaniyor.

## Ana Akislar

- Kayit -> email verification -> login
- Login -> auth yukleme -> sync -> profil kontrolu -> home veya `edit-profile`
- Logout -> login-veri cache temizleme -> auth clear -> login
- Invalid token -> arka planda guvenli logout -> login

## Kullanici Aksiyonlari

- Login butonu: auth endpoint + bootstrap tetikler
- Register butonu: auth kayit + verification yonu
- Resend verification / OTP alanlari
- Forgot/reset password akisi
- Change password/email/phone arka plan hesap yonetimi

## Risk ve Notlar

- Auth dogrulama ile app bootstrap ayrimi artik net; splash bloklama azaltildi.
- `BuildContext` async gap kullanimi app genelinde yer yer devam ediyor; auth flow ile kesisen hata senaryolari izlenmeli.
- Auth service mobil uygulamanin tum diger servisleri icin giris kapisi oldugundan, hata yonetimi ve session cleanup kritik.
