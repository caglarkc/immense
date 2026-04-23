---
name: code-implementation-mode
description: Bu repoda kod yazan/uygulayan ajan modu; önce kısa etki analizi yapar, sonra rules.md ve ilgili katman kurallarına uyarak doğrudan kodu uygular, açık hata ve manuel kontrol notlarını kısa verir.
---

## Purpose

Bu skill, ajanı bu projede **uygulayıcı/kod yazan mühendis** gibi çalıştırmak için kullanılır.

Ana hedef:

- istenen işi doğru anlamak
- etkilenen katmanları ve riskleri kısa çıkarmak
- repo kurallarına uyarak doğrudan kodu yazmak
- gereksiz teori yerine uygulamaya odaklanmak
- sonuçta kısa doğrulama ve manuel kontrol notu vermek

## Rules

- **Önce kısa analiz zorunlu**: Koddan önce şu 4 başlık net verilir: ne istendi, hangi dosya/katmanlar etkilenecek, riskler neler, hangi güncellemeler yapılacak.
- **Anayasa zorunlu**: `son_immense/.agent/rules.md` ana referanstır.
- **Katman referansı zorunlu**: İşe göre `app.md`, `backend.md`, `admin.md` ve ilgili skill kuralları dikkate alınır.
- **Doğrudan uygulama**: Belirsizlik kritik değilse uzun plan yerine uygulanır; kullanıcı açıkça sadece plan istemedikçe iş kodla tamamlanır.
- **Varsayım disiplini**: Mimariyi etkileyen kritik boşlukta durup netleştir; küçük uygulama detaylarında makul karar al ama bunu sonuçta kısa not düş.
- **App kuralı**: Flutter işlerinde `UI -> Provider -> Repository -> DioClient` zinciri korunur; UI içinde API çağrısı ve `setState` ile state yönetimi yapılmaz.
- **Backend kuralı**: `Router -> Controller -> Service -> Model` zinciri korunur; controller içinde `try/catch` ve `next(error)` standardı bozulmaz.
- **Admin kuralı**: API çağrıları yalnız `src/api/client.js` üzerinden yapılır; otomatik refresh eklenmez, sayfa state pattern’i korunur.
- **Standart response ve naming**: Mevcut response yapısı, camelCase isimlendirme ve proje dili kuralları korunur.
- **Loading/error UX**: Kullanıcı hiçbir yerde boş veya tepkisiz ekranda bırakılmaz.
- **Test yaklaşımı**: Kullanıcı özellikle istemedikçe kapsamlı test mimarisi kurmaya yönelmez; mümkünse kısa manuel doğrulama notu bırakır.
- **Commit disiplini zorunlu**: Her genel değişiklik bloğu tamamlandığında kısa ve anlamlı bir commit atılır.
- **Push disiplini zorunlu**: Görevin tamamı bittikten sonra son durum remote'a push edilir.
- **Teslim standardı zorunlu**: İş bitince hangi dosyaların değiştiği, hangi yeni dosya/asset/env beklendiği ve hangi davranışın aktif olduğu açıkça listelenir.
- **Belirsiz kapanış yasak**: `gerekli düzenlemeler yapıldı` gibi muğlak kapanış yerine somut çıktı verilir.
- **Kurulum komutu yasağı**: Backend ve genel repo için `npm install`, `pnpm install`, `yarn install`, `flutter pub get` gibi bağımlılık kurulum komutlarını çalıştırma. Bu proje uzak makinede build aldığı için sadece ilgili manifest dosyalarını güncelle; kurulum kullanıcı tarafında yapılır.

## Workflow

1. İstenen işi kısa cümleyle özetle.
2. Etkilenen dosya/katmanları ve riskleri belirt.
3. İlgili repo kurallarını eşleştir.
4. Kodu doğrudan uygula.
5. Her genel değişiklik bloğu sonrası commit at.
6. Sonuçta ne yapıldığını, hangi dosyaların değiştiğini ve nasıl kontrol edileceğini net yaz.
7. Görev tamamen bittiğinde push at.

## Output Style

- Kısa ve net yazılır.
- Ön analiz kısa tutulur, kod işi uzatılmaz.
- Sonuç mesajında yapılan iş, risk kaldıysa not ve manuel kontrol özeti verilir.

## Required Output Format

Kod yazmaya başlamadan önce şu 4 kısa başlık zorunludur:

1. `Ne istendi?`
2. `Hangi dosya/katmanlar etkilenecek?`
3. `Riskler neler?`
4. `Hangi güncellemeler yapılacak?`

Kod işi bittikten sonraki final çıktı şu sırayla verilmelidir:

1. `Yapılan iş`
- Çok kısa özet

2. `Degisen dosyalar`
- Değiştirilen dosyaları listele
- Yeni oluşturulan dosyaları ayrıca belirt

3. `Aktif davranislar`
- Kullanıcı açısından artık ne çalışıyor, kısa kısa yaz

4. `Beklenen eklemeler`
- Kullanıcının ayrıca eklemesi gereken asset, env key, prompt metni veya config varsa açıkça listele
- Yoksa `Yok` yaz

5. `Manuel kontrol`
- Test yazılmadıysa bunu açıkça söyle
- Kullanıcının elle neyi kontrol etmesi gerektiğini kısa belirt

## Delivery Principles

- Dosya listesi atlanmaz.
- Commit mesajları kısa, anlamlı ve değişiklik bloğunu temsil eder.
- Push öncesi mevcut dal ve kullanıcı değişiklikleri ezilmeden korunur.
- Yeni env değişkeni gerekiyorsa adı açıkça yazılır.
- Yeni asset gerekiyorsa tam path önerisi verilir.
- Kullanıcının sonradan dolduracağı placeholder dosyalar varsa tam path ile belirtilir.
- Bir şey eksik bırakıldıysa saklama; doğrudan not düş.
- Bağımlılık eklendiyse `install edildi` deme; sadece hangi `package.json` veya `pubspec.yaml` alanının güncellendiğini söyle.

## References

- `son_immense/.agent/rules.md`
- `son_immense/.agent/app.md`
- `son_immense/.agent/backend.md`
- `son_immense/.agent/admin.md`
- `son_immense/skills/app-clean-architecture`
- `son_immense/skills/app-api-integration`
- `son_immense/skills/app-state-management`
- `son_immense/skills/backend-controller-service-flow`
- `son_immense/skills/backend-standard-response`
- `son_immense/skills/backend-model-definition`
- `son_immense/skills/backend-input-validation`
- `son_immense/skills/admin-api-client-rule`
- `son_immense/skills/admin-page-state-pattern`
- `son_immense/skills/admin-data-loading-policy`
