## #82 — Uygulama Girişinde 3 Saniyelik Gecikme

Uygulama her açılışta 3 saniye bekletiyor.

**Çözüm:**
Kullanıcıyı direkt uygulamaya al, veri güncellemelerini kullanıcı uygulama içindeyken arka planda yap.
Sync zaten kontrol ediliyor ve duruma göre yeni data çekiliyor, bu yüzden kullanıcıyı direkt uygulamaya almak mantıklı.

**Davranış:**
- Uygulama açıldığında önce access token kontrolü yapılır.
- Token **geçerli** → kullanıcı direkt uygulamaya alınır, veriler arka planda çekilir.
- Token **yok** → kullanıcı login ekranına yönlendirilir.
- Token **geçersiz / hata** → kullanıcı uygulamadan çıkarılır (logout).

---

## #81 — Eşleşme Sonlandırma Sorunu

Eşleşme sonlandırıldıktan sonra:

- Diğer tarafın sohbetinde eşleşmenin bittiği açıkça gösterilmiyor.
- Mesaj atmaya çalışınca hata veriyor ancak bu hata kullanıcıya net şekilde iletilmiyor.
- Sağ üst köşede hâlâ "Eşleşmeyi Sonlandır" butonu görünüyor (olmamalı).
- Eşleşme biten kullanıcının profiline girildiğinde "Birebir" butonu hâlâ görünüyor ve mesajlaşma sarı (aktif) yanıyor — bunlar gözükmemeli.

**Ek:**
- Engelleme durumlarında da eşleşme otomatik olarak sonlandırılmalı.

---

## #80 — Yorumlar Görünümü

Yorumlar bölümü sıkışık görünüyor, düzeltilecek.

---

## #79 — Beslenme Sayfası Yüklenme Gecikmesi

Beslenme bilgileri, kullanıcı uygulamayı açtığında çekilmediği için beslenme sayfası ilk açıldığında data geç yükleniyor.

**Çözüm:**
- Kullanıcı giriş yaptığında tüm beslenme verileri çekilmeli.
- Beslenme sayfası açıldığında yalnızca kullanıcı sayfa içinde manuel yenileme yaparsa tekrar çekilmeli; aksi hâlde login sonrası çekilen data kullanılmaya devam etmeli.
- Frontend'de kullanıcı yeni bir şey eklediğinde, o data frontend tarafında mevcut data ile sync edilmeli (yeniden istek atmadan).

---

## #78 — Navbar Sayfa Geçiş Animasyonu

Navbar üzerinden sayfalara geçişte animasyon yok. Açılış animasyonu eklenmeli.

---

## #77 — Profilde "null" Link Gösterimi

Profildeki link alanı boş veya null olduğunda ekranda "null" yazıyor.
Link alanı null ise profilde hiçbir şey gösterilmemeli.

---

## #76 — Keşfet: Fotoğrafsız Postlar

Keşfet sayfasında fotoğrafı olmayan postlar gösterilmeyecek.
Takip edilenler sekmesinde ise gösterilebilir.

---

## #67 — Member'lara "Eşleşmeyi Sonlandır" Butonu Kaldırılacak

Mesajlaşma ekranında, eşleşmeden kaynaklanan "Eşleşmeyi Sonlandır" butonu görünüyor.
Bu buton yalnızca coach–member ilişkisinde geçerli değil; **member'ların mesajlaşma ekranında bu buton görünmemeli.**