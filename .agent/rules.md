# Proje Kuralları (Bağlayıcı Anayasa)

> Bu doküman tüm geliştirme süreçleri için bağlayıcıdır. Backend, App ve Admin Panel'e özel kurallar ilgili .md dosyalarındadır.

---
## KESİNLİKLE UYMAN GEREKEN VE ASLA UNUTMAMAN GEREKEN KURAL!!!!
- Her ne olursa olsun, öncelikle ne anladığını açıkla kısaca ve karışması mümkün olan konularda asla kendin karar verme.
- Kendin karar alabileceğin durumlarda dahi bana sorup en azından anladığın şeyi açıklamaktan çekinme. 
- Uygulayacagın değişiklik veya yeni özellik için gereken ve etkilenecek her yapıyı inceleyip olası riskleri göz önünde bulundurmayı asla unutma! 



---

## 1. Çalışma Modları

### Chat Modu
- Karşılıklı plan veya düşünce tartışması.
- Kısa, tek-iki cümlelik sohbet havasında yanıtlar.

### Develop Modu
- Detaylı planlama, etki analizi, teknik notlar ve kod entegrasyonu.
- Kod entegrasyon işleri varsayılan olarak bu modda ilerler.

---

## 2. Koddan Önce Zorunlu Planlama

Planlama modunda olmasan dahi, **her kod yazmadan önce** aşağıdaki 4 maddeyi net yaz:

1. **Ne istendi?** → Görevin özeti.
2. **Hangi dosya/katmanlar etkilenecek?** → Değişiklik haritası.
3. **Hangi riskler/yan etkiler var?** → Kırılabilecek noktalar.
4. **Hangi güncellemeler yapılacak?** → Adım adım yapılacaklar.

**Plan netleşmeden koda geçilmez.** Proje büyük; her özellik eklemek birçok yeri etkileyebilir. Bunu her zaman hesaba kat. 

---

## 3. Varsayım Yasağı (Sıfır Tolerans)

- Anlamadığın veya eksik kalan **her** noktayı SOR.
- Boşlukları tahminle doldurma.
- Teknik kararları kullanıcıyla netleşmeden finalize etme.
- Eksik endpoint, bilinmeyen model verisi veya tasarımsal eksiklik varsa kendi inisiyatifinle doldurmak yerine **DUR ve SOR**.

---

## 4. Bilişsel Süreç (Her Görev İçin)

```
ANLA → DÜŞÜN → PLANLA → KONTROL ET → UYGULA → DOĞRULA
```

1. **ANLA** → Eksik bilgi varsa DUR ve SOR.
2. **DÜŞÜN** → Mimariye uygun mu? Hangi katmanlar etkileniyor?
3. **PLANLA** → Değişiklikleri kısaca özetle (4 madde kuralı).
4. **KONTROL ET** → Kritik kısıtlamalar ve yasaklara bak.
5. **UYGULA** → Temiz, modüler kod yaz.
6. **DOĞRULA** → Çalışabilirlik, uyum ve yan etki kontrolü.

---

## 5. Ortak Geliştirme Kuralları

### Dil Kuralı
- Tüm açıklamalar ve yorumlar → **Türkçe**.
- Değişken, fonksiyon, sınıf isimleri → **İngilizce**.

### Async/Await Güvenliği
- `await` sonrası UI güncellenecek veya sayfa değiştirilecekse → **`context.mounted`** (veya karşılığı) kontrolü zorunlu.

### Hata Yönetimi ve UX
- Loading durumları mutlaka gösterilir.
- Hatalarda anlamlı mesajlar çıkarılır.
- Kullanıcı **hiçbir senaryoda** boş/donuk ekranda bırakılmaz.

### Veritabanı Kolon İsimlendirmesi
- **Tüm kolonlar camelCase:** `userId`, `requestId`, `isNotificationSent`, `createdAt`.
- **snake_case yasak:** `user_id`, `request_id` kullanılmaz.
- Sequelize modelde `field: 'x_y'` kullanılmaz; attribute adı = kolon adı.
- Mevcut snake_case tablolar `scripts/*-camelcase-migration.sql` ile normalize edilir.

### Ortam: Docker ve Uzak Makine
- Proje **Docker** içinde çalışır; backend servisleri container'larda ayağa kalkar.
- Geliştirme **uzak makinede** yapılıyor; lokal `npm install` veya `flutter pub get` çalıştırılmıyor.
- Bağımlılık eklerken sadece `package.json` / `pubspec.yaml` güncellenir; kurulum uzak makinede build sırasında yapılır.
- Terminal komutları (install, build, run) kullanıcıya bırakılır; uzak ortamda çalıştırılacak.

---

## 6. Kategori Bazlı Kural Referansları

### Backend (Node.js + Express)
- Akış: `Router → Controller → Service → Model`.
- Controller: `try/catch` + `return next(error)` standart.
- Service sonucu doğrudan `return res.json(result)` ile publish edilir.
- Response formatı (`{ success, customMessage, data }`) asla bozulmaz.
- Detay: **`backend.md`**

### App (Flutter)
- Mimari: `UI (Screen) → Provider → Repository → DioClient`.
- UI katmanında direkt API çağrısı yapılmaz.
- Endpoint'ler sadece `ApiEndpoints` üzerinden; token/device-id interceptor yönetir.
- `context.mounted` kontrolü zorunlu.
- Tema extension'ları kullanılır; hardcode renk/stil yasak.
- Detay: **`app.md`**

### Admin Panel (React)
- Tüm API istekleri `src/api/client.js` üzerinden geçer.
- Veri yükleme politikası: otomatik refresh yok, kullanıcı aksiyonu ile fetch.
- Sayfa state ve yükleme akışı mevcut pattern ile uyumlu kalır.
- Desktop-only tasarım.
- Detay: **`admin.md`**

---

## 7. Dosya Haritası

| Dosya | İçerik |
|-------|--------|
| `backend.md` | Backend (Node.js/Express) mimari ve kodlama kuralları |
| `app.md` | Mobil Uygulama (Flutter) Clean Architecture ve kuralları |
| `admin.md` | Admin Panel (React) yapı ve kuralları |
| `rules.md` | Bu dosya — tüm projeyi bağlayan genel kurallar |
