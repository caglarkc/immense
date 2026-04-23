# Coaching Service Audit

## Backend Yuzeyi

- Route: `backend/services/coaching-service/src/routes/coaching.routes.js`
- Modeller:
  - `coach.model.js`
  - `coach_registration.model.js`
  - `coach_invite.model.js`
  - `coach_membership.model.js`
  - `coach_member_week_plan*.model.js`
  - `coach_push_batch.model.js`
  - `coach_push_delivery.model.js`
  - activity log modelleri
- Servisler:
  - `coaching.service.js`
  - `coach_notification.service.js`
- Kapsam:
  - coach registration
  - invitation/membership
  - coach panel/member detail
  - weekly plan
  - member stats/nutrition/training proxy akislari
  - coach notifications

## Frontend Baglantilari

- Data:
  - `coaching_repository.dart`
  - `coach_member_stats_repository.dart`
- Logic:
  - `coaching_provider.dart`
  - coach/member detail, stats, nutrition, weekly plan providerlari
- Screens:
  - `coaching_hub_screen.dart`
  - `coach_detail_screen.dart`
  - coach/member workspace ve detail ekranlari
  - assign routines / assign exercises / stats / nutrition / weekly plan ekranlari
- Widgets:
  - `coach_panel_view.dart`
  - `member_panel_view.dart`
  - coach detail kartlari ve notification sheet

## Veri ve State

- Koçluk verisi login sonrasi app bootstrapte yukleniyor.
- Hub ve coach panel memory cache agirlikli; ilk tam login'de prefetch davranisi var.
- Uye detay, atanmis rutin/egzersiz kaynak id'leri ve bazi nutrition/stats ekranlari coach proxy endpointleri kullanir.

## Ana Akislar

- Coach registration
- Coach panel:
  - uye listesi
  - davet gonderme / geri alma
  - toplu bildirim
  - antrenman programlari / egzersizler
- Member side:
  - gelen coach invite
  - coach detail inceleme
  - my coach overview
  - assigned routines/exercises
  - weekly plan
- Coach side member workspace:
  - routine/exercise assign
  - member stats
  - member nutrition
  - activity log

## Buton ve CTA Mantigi

- `Antrenman Programlarim`, `Egzersizler`, `Davet`, `Bildirim`, `Mesaj`
- Invite kartlari: `Profili incele`, `Kabul et`, `Reddet`, `Taahhudu geri cek`
- Member workspace: routines, exercises, nutrition, stats, measurements, workouts gecisleri

## Risk ve Notlar

- Coaching service urunun en capraz bagimli alani: training, exercise, nutrition, stats, notifications ve user profile ile bagli.
- UX tarafinda copy/navigasyon son donemde cok degisti; regression test burada ozellikle onemli.
- Shell route gecisi kaynakli `Antrenman Programlarim` assertion problemi bu turda duzeltildi.
