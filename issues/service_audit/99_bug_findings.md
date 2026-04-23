# Acik Bulgular ve Riskler

Bu dosya mevcut repo uzerinde gorunen acik hata, risk ve teknik borc notlarini toplar.

## Genel Statik Tarama Sonucu

- `cd frontend && flutter analyze` sonucunda 99 issue goruldu.
- `backend/services/*/src` icin `node --check` taramasi gecti.

## Yuksek Oncelikli Riskler

1. `lib/modules/profile/screens/user_profile_screen.dart`
   - Cok sayida `use_build_context_synchronously` info'su var.
   - Profil CTA, follow, block, match ve message akislarinda await sonrasi context kullanimi runtime risk tasiyor.

2. `lib/shared/api/api_interceptor.dart`
   - Async gap sonrasi context kullanimi var.
   - 401/logout yonetiminde navigation veya snack davranisinda edge-case uretebilir.

3. `lib/modules/workouts/screens/share_workout_screen.dart`
   - Birden fazla async-context warning var.
   - Paylasim ve post olusturma sirasinda unmounted ekran uzerinden UI aksiyonu riski mevcut.

4. `lib/app/data/post_data_manager.dart`
   - Async gap sonrasi context warningleri var.
   - Post bootstrap/refresh akislarinda lifecycle edge-case olusabilir.

## Orta Oncelikli Kod Kalitesi Bulgulari

5. `lib/modules/matching/data/match_repository.dart`
   - `void_checks` info'lari var; kod niyeti okunurlugunu dusuruyor.

6. `lib/modules/workouts/screens/edit_workout_screen.dart`
   - `dead_code` ve `dead_null_aware_expression` warningleri var.
   - Bu ekranin bir dali artik ulasilamaz veya beklenenden farkli durumda.

7. `lib/modules/feed/widgets/explore_post_sheet.dart`
   - Kullanilmayan import ve async-context warning mevcut.

8. `lib/modules/profile/screens/edit_profile_screen.dart`
   - Kullanilmayan field'ler ve deprecated `value` kullanimi var.

9. `lib/modules/profile/widgets/profile_info_section.dart`
   - Kullanilmayan element, gereksiz `!`, import warningleri var.

10. `lib/shared/services/media_url_service.dart`
    - Kullanilmayan importlar ve style warningleri var.
    - Medya cozumleme merkezi oldugu icin burada bir regression cok modulu etkiler.

## Uygulama Davranis Riski Notlari

- Sosyal profil, match ve messaging ayni kullanici iliskisi datasina dayaniyor; bu alanlarda stale cache tekrar test edilmeli.
- Coaching, training, nutrition ve stats capraz bagimli; koç-uye akislarinda regression test her release'te gerekli.
- Explore media-only UX backend datasina bagli olarak load-more ile telafi ediliyor; feed bos ama hasMore varsa davranis test edilmeli.

## Son Turda Duzeltilenler

- Coach panel `Antrenman Programlarim` route assertion'i giderildi.
- Splash token bootstrap geri hizlandirildi.
- Logout/account-switch cache cleanup guclendirildi.
- Exercise favorite failure UI tutarsizligi kapatildi.
