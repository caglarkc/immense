# Social Media Service Audit

## Backend Yuzeyi

- Route: `backend/services/social-media-service/src/routes/social-media.routes.js`
- Modeller:
  - `post.model.js`, `comment.model.js`, `post-like.model.js`, `post-save.model.js`
  - `follow.model.js`, `block.model.js`, `report.model.js`
- Servisler:
  - `post.service.js`, `comment.service.js`, `follow.service.js`
  - `block.service.js`, `report.service.js`, `search.service.js`, `user-profile.service.js`
- Ana endpointler:
  - Post CRUD, upload, like/save, comments, report
  - Following feed ve explore feed
  - Follow/follower/follow stats/check-mutual
  - User search
  - Diger kullanici profil ozeti
  - Block/unblock/blocked list
  - Report mine

## Frontend Baglantilari

- Data:
  - `frontend/lib/modules/feed/data/post_repository.dart`
  - `follow_repository.dart`
  - `search_repository.dart`
- Logic:
  - `post_provider.dart`
  - `follow_provider.dart`
  - `search_history_provider.dart`
- Screens:
  - `feed_screen.dart`
  - `explore_screen.dart`
  - `search_screen.dart`
  - `user_profile_screen.dart`
- Widgetler:
  - `feed_header.dart`
  - `explore_post_sheet.dart`
  - `comment_list_section.dart`
  - `report_dialog.dart`
  - `workout_post_card.dart` (shared)

## Veri ve State

- Kendi postlari Drift `MyPosts` tablosunda hibrit cache ile tutuluyor.
- Following ve explore feed'ler Provider belleginde tutuluyor.
- Search history local repository ile saklaniyor.
- Diger kullanici profili `OtherUserProfileProvider` icinde oturum ici cache kullaniyor.

## Ana Kullanici Akislari

- Feed sekmeleri: Following / Explore
- Explore media-only kurali iki yuzeyde de uygulanmis
- Post detayi alt sheet, yorumlar, begenenler, report akislari
- Search -> user profile
- Follow / unfollow / remove follower
- Block / unblock
- Kullanici raporlama ve rapor gecmisi

## Buton ve Etkiler

- Post karti: like, comment, save, report, profile goruntuleme
- Comment satiri: guncelle/sil/report akislari
- Search sonuclari: profile navigasyonu
- User profile CTA'lari: follow, message, match, block/report

## Risk ve Notlar

- Feed ve profile diger servislerle yogun bagimli: match, message, coaching, stats.
- `flutter analyze` sonucunda post/follow katmaninda bazi gereksiz cast ve async context warningleri var.
- Explore media-only UX mantigi urun bazli karar; backend explore datasinda medyasiz icerik geldiginde frontend load-more ile telafi ediyor.
