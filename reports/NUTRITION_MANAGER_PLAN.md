# Nutrition Manager Plan

> Bu doküman yönetici bakışıyla hazırlanmış iç yürütme planıdır. Amaç, nutrition entegrasyonunu backend-first yaklaşımıyla 3 fazda kontrollü şekilde tamamlamaktır.

---

## 1. Ana Hedef

Nutrition modülünü:

- mevcut proje kurallarına uyumlu
- sade ama genişlemeye açık
- backend ve frontend arasında net contract üreten
- AI ve tracker için sağlam temel bırakan

bir yapıyla sisteme entegre etmek.

---

## 2. Yönetim Kararları

- Final referans doküman: `NUTRITION_FINAL_ARCHITECTURE.md`
- Eski nutrition planları doğrudan uygulanmayacak; sadece karşılaştırma referansı olarak kalacak
- Yeni yürütme sırası backend-first olacaktır
- Önce nutrition core backend oturtulacak
- Sonra AI entegrasyonu eklenecek
- En son tracker entegrasyonu yapılacak
- Frontend implementation, backend core contract netleştikten sonra hızlanacaktır

---

## 3. Fazlar

### Faz 1 - Backend Core Nutrition

Amaç:

- AI ve tracker olmadan nutrition çekirdeğini ayağa kaldırmak
- veri modeli, ilişkiler ve endpoint contract’ını sabitlemek

Kapsam:

- `Target`
- `Food`
- `Meal`
- `MealFood`
- `Supplement`
- `SupplementLog`
- `WaterLog`
- `Program`
- `ProgramMeal`
- `ProgramMealFood`
- `FoodFavorite`
- `SupplementFavorite`

Çıktılar:

- servis sınırı ve klasör yapısı
- model dosyaları
- relation kurgusu
- CRUD endpointleri
- summary endpointleri
- hesaplama servisleri
- programdan gerçek meal üretme akışı
- user/system ownership kuralları

Kapsam dışı:

- AI parse ve create akışları
- tracker / reminder mantığı
- push veya cron/job akışları

Risk:

- eski backend planındaki gereksiz karmaşıklığın yeni core’a sızması
- frontend’in ihtiyaç duyduğu contract’ın eksik bırakılması

Başarı ölçütü:

- AI ve tracker olmadan app tarafı gerçek nutrition kullanımını yapabilecek contract’ın oluşması

### Faz 2 - AI Entegrasyonu

Amaç:

- nutrition core üstüne AI meal giriş katmanını güvenli biçimde eklemek

Çıktılar:

- AI parse endpointleri
- geçici öneri veri yapısı
- kullanıcı onayı sonrası gerçek meal create akışı
- prompt ve parser tasarımı
- food eşleme mantığı

Risk:

- AI çıktısının doğrudan güvenilir veri kabul edilmesi

Başarı ölçütü:

- AI çıktısı sadece öneri olur, gerçek kayıt kontrollü şekilde açılır

### Faz 3 - Tracker Entegrasyonu

Amaç:

- water ve supplement reminder yapısını generic `Tracker` üstünden sisteme eklemek

Çıktılar:

- tracker veri modeli
- tracker CRUD endpointleri
- trigger rule yapısı
- job/worker planı
- notification tetikleme akışı

Risk:

- reminder mantığının nutrition çekirdeğini gereksiz yavaşlatması

Başarı ölçütü:

- tracker eklendiğinde nutrition core ve AI akışı bozulmadan genişleyebilmesi

---

## 4. Alt AI Görev Dağıtım Sırası

1. Backend plan AI - Faz 1
Görev: final mimariye göre nutrition backend core planını çıkarmak.

2. Backend implementation AI - Faz 1
Görev: onaylı backend core planına göre kod yazmak.

3. Frontend plan AI
Görev: backend core contract’a göre frontend entegrasyon planını çıkarmak.

4. Frontend implementation AI
Görev: onaylı frontend planına göre kod yazmak.

5. AI integration AI - Faz 2
Görev: nutrition AI akışını planlayıp entegre etmek.

6. Tracker AI - Faz 3
Görev: generic tracker mimarisini sisteme oturtmak.

Not:

- Bu sıra bozulmamalı
- Backend core netleşmeden frontend implementation prompt’u üretilmemeli
- AI ve tracker prompt’ları, Faz 1 tamamlanmadan yazılmamalı

---

## 5. Faz 1 İçin Kritik Kararlar

Faz 1 backend core içinde korunması gereken ana kararlar:

- `Program` gerçek tüketim değildir, template’tir
- gerçek tüketim `Meal` ve `MealFood` üzerinden tutulur
- program içinden tek öğün bazlı gerçek meal oluşturulur
- water ve supplement günlük JSON row değil, satır bazlı log olur
- `Food` ve `Supplement` hem sistem hem user scoped olabilir
- favoriler ayrı relation tablolarında tutulur
- AI alanları Faz 1’de modele veya endpoint setine zorla sokulmaz
- tracker mantığı Faz 1 contract’ını kirletmez

---

## 6. Yöneticinin Uygulama Stratejisi

- Önce final mimariyi sabit tut
- Ardından backend core’u minimal ama eksiksiz şekilde netleştir
- Frontend’i backend contract’a göre hizala
- AI ve tracker’ı çekirdeğin üstüne ekle
- Her faz sonunda prompt üretmeden önce kapsam sızıntısı kontrolü yap

---

## 7. Sonraki Adım

Sonraki çalışma:

- Faz 1 backend core planını üretmek

Bu tamamlandıktan sonra:

- Faz 1 backend implementation prompt’u
- ardından frontend planı

sırasıyla üretilecektir.
