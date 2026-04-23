# Match Service Audit

## Backend Yuzeyi

- Route: `backend/services/match-service/src/routes/match.routes.js`
- Modeller:
  - `broadcast.model.js`
  - `broadcast_response.model.js`
  - `one_by_one.model.js`
  - `match_user_pair.model.js`
- Servisler:
  - `broadcast.service.js`
  - `one_by_one.service.js`
  - `inbox.service.js`
  - `status.service.js`
  - `pair_relationship.service.js`

## Frontend Baglantilari

- Data:
  - `match_repository.dart`
  - `match_models.dart`
- Logic:
  - `broadcast_provider.dart`
  - `match_inbox_provider.dart`
  - `match_status_provider.dart`
- Screens/widgets:
  - `gym_match_screen.dart`
  - `broadcast_card.dart`
  - `broadcast_form_sheet.dart`
  - `broadcast_detail_sheet.dart`
  - `match_inbox_sheet.dart`
  - `match_request_card.dart`
- Dolayli baglantilar:
  - `user_profile_screen.dart`
  - `message_provider.dart`
  - `sync_handler.dart`

## Veri ve State

- Match state oturum ici `MatchStatusProvider` cache'i ile tutuluyor.
- Profil CTA'lari `matchStatus` sonucuna gore sekilleniyor.
- Sync event'leri block ve conversation close durumlarinda local override uyguluyor.

## Ana Akislar

- Broadcast olusturma, listeleme, cevap verme, geri cekme, kabul/reddetme
- One-by-one request gonderme, geri cekme, kabul/reddetme
- Inbox ekraninda gelen/giden istekleri gorme
- Profilde aktif match/pending/blocked durumuna gore CTA degisimi

## Buton ve CTA Mantigi

- Gym match ekraninda ilan kartlari uzerinden basvuru
- Inbox sheet'te kabul/red/geri cek
- User profile'da:
  - `Birebir Istek`
  - `Istek Geri Cek`
  - `Tekrar Baglanti`
  - `Mesajlas`

## Risk ve Notlar

- Match service tek basina degil; message-service ve social profile UI ile birlikte calisiyor.
- Local override mantigi stale state'i azaltir ama logout/account-switch temizligi kritik oldugu icin buna ayrica dikkat edildi.
- Profil ekranindaki async/context warningleri match CTA davranisini da etkileyebilir.
