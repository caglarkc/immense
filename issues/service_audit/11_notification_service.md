# Notification Service Audit

## Backend Yuzeyi

- Route: `backend/services/notification-service/src/routes/notification.routes.js`
- Modeller:
  - `notification.model.js`
  - `device_token.model.js`
- Servisler:
  - `notification.service.js`
  - `notification_inbox.service.js`
  - `device_token.service.js`
  - `notification.consumer.js`
  - `chat_push_throttle.js`

## Frontend Baglantilari

- Data:
  - `frontend/lib/shared/data/notification_repository.dart`
- Logic:
  - `notification_inbox_provider.dart`
- Screens/widgets:
  - `notifications_screen.dart`
  - `fcm_foreground_host.dart`
  - `foreground_heads_up.dart`
  - `app_lifecycle_handler.dart`
- Main init:
  - `main.dart`
  - `shared/services/notification_service.dart`

## Veri ve State

- FCM token `main.dart` icinde aliniyor ve notification servis repo uzerinden register ediliyor.
- Inbox provider bellek state tutuyor; preload ve mark-as-read akislarini yonetiyor.
- Chat mesajlari foreground heads-up ile gecici banner uretebiliyor.

## Ana Akislar

- Token register/update/delete
- Inbox listeleme
- Tekli ve toplu read islemleri
- Belirli conversation'a ait chat bildirimlerini temizleme
- Internal send akisi diger servislerden tetikleniyor

## Buton ve CTA Mantigi

- Bildirim listesinde tiklama -> ilgili ekrana gecis
- `read all` veya tek bildirim okuma
- Chat acildiginda ilgili bildirimleri dusurme

## Risk ve Notlar

- Notification urunun capraz keseni: message, coaching, social ve auth eventleriyle bagli.
- Foreground ve inbox davranislarinin ayrik olmasi dogru, fakat duplicate ve stale notification temizligi kritik.
- App resume/feed tab girisinde controlled preload davranisi mevcut.
