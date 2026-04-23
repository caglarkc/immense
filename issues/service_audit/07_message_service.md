# Message Service Audit

## Backend Yuzeyi

- Route: `backend/services/message-service/src/routes/message.routes.js`
- Modeller:
  - `conversation.model.js`
  - `message.model.js`
- Servisler:
  - `conversation.service.js`
  - `message.service.js`
- Ana endpointler:
  - conversation list
  - conversation with user
  - first message send
  - conversation messages list/send
  - conversation close
  - internal create/reactivate/status/close-on-block

## Frontend Baglantilari

- Data:
  - `message_repository.dart`
  - `message_models.dart`
- Logic:
  - `message_provider.dart`
  - `message_socket_service.dart`
- Screens:
  - `conversations_screen.dart`
  - `chat_screen.dart`
- App baglantilari:
  - `app_lifecycle_handler.dart`
  - `notification_inbox_provider.dart`

## Veri ve State

- Drift `Conversations` + `Messages` tablolari ile kalici cache kullaniliyor.
- `MessageProvider` hem memory hem drift reconcile mantigi ile calisiyor.
- App resume sonrasi hafif reconcile var.
- Socket mesajlari memory + drift + heads-up banner akisini besliyor.

## Ana Akislar

- Konusmalar listesi acilisi
- Bir kullanici ile sohbet acma
- Ilk mesaj / mevcut conversation'a mesaj
- Pull-to-refresh ile reconcile
- Match-end veya block sonrasi composer kisitlama

## Buton ve CTA Mantigi

- Conversation tile -> chat ekranina gecis
- Chat composer -> optimistic send
- Pull-to-refresh
- Chat menu -> conversation close
- Kapatilmis veya blocked conversation durum metinleri

## Risk ve Notlar

- Messaging, match ve social profile CTA'lariyla yuksek derecede bagli.
- Socket, cache ve sync birlikte calistigi icin duplicate, stale status ve unread count en kritik alanlar.
- Message cache logoutta temizleniyor; match/profile cache temizligi sonradan eklenmis.
