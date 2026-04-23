# Nutrition Final Architecture

> Bu doküman nutrition entegrasyonu için bağlayıcı final mimari çerçevedir. Sonraki frontend, backend, AI ve tracker planları bunun üstüne kurulmalıdır. Kod yazımı öncesi `rules.md` ve ilgili katman kuralları ayrıca geçerlidir.

---

## 1. Amaç

Nutrition modülü ilk fazda:

- kullanıcının günlük hedeflerini takip etmesini
- öğün, su ve supplement girişlerini hızlıca yapmasını
- tekrar eden öğünlerini şablon halinde saklamasını
- sistem ve kullanıcı bazlı nutrition kataloglarını kullanmasını

sağlar.

Temel yaklaşım:

- sistem günlük kullanım odaklı kalır
- gerçek tüketim verisi ile şablon veri ayrılır
- ilk kurulum gereksiz kompleksliğe girmez
- AI ve tracker çekirdek nutrition akışının önüne geçmez

---

## 2. Final Domain Kararları

### 2.1 Gerçek tüketim ve şablon ayrımı

- `Meal` gerçek tüketim kaydıdır.
- `Program` gerçek tüketim değildir, sadece şablondur.
- Program seçilip toplu uygulanmaz.
- Kullanıcı program içindeki tek bir öğün için "bunu yaptım" aksiyonu verdiğinde yeni bir gerçek `Meal` oluşturulur.

### 2.2 Katalog mantığı

- `Food` hem sistem hem kullanıcı bazlı olabilir.
- `Supplement` hem sistem hem kullanıcı bazlı olabilir.
- `userId = null` ise sistem kaydıdır.
- `userId` dolu ise kullanıcıya ait kayıttır.

### 2.3 Log mantığı

- Su tüketimi günlük tek JSON satırda tutulmaz.
- Supplement kullanımı günlük tek JSON satırda tutulmaz.
- Her kullanım ayrı satır olarak loglanır.

### 2.4 Hedef mantığı

- İlk fazda hedef yapısı sadece günlük hedef odaklıdır.
- Aynı anda kullanıcı için tek aktif hedef mantığı uygulanır.
- Dönemsel hedef geçmişi bu fazın parçası değildir.

### 2.5 Zaman mantığı

- `Meal` tam zaman bilgisi taşır.
- Sadece tarih değil, timestamp tutulur.
- Günlük özet ve geçmiş ekranları bu timestamp üzerinden gruplanır.

### 2.6 Hesaplama mantığı

- Makro ve toplam hesapları backend’de yapılır.
- Frontend sadece sonucu gösterir.
- Snapshot alanları tutulur; geçmiş kayıtlar katalog değişiminden etkilenmez.

### 2.7 AI ve tracker sıralaması

- İlk core nutrition yapısı AI olmadan kurulacaktır.
- AI entegrasyonu ayrı fazda eklenecektir.
- `Tracker` modeli reminder için düşünülür, fakat entegrasyonu en sona bırakılır.

---

## 3. Final Veri Modelleri

Tüm modeller için ortak kurallar:

- UUID primary key
- camelCase alan isimleri
- `timestamps: true`
- `paranoid: true`

### 3.1 Target

Kullanıcının günlük nutrition hedefi.

Alanlar:

```text
id
userId
targetCal
targetCarb
targetPro
targetFat
targetWater
createdAt
updatedAt
deletedAt
```

Kurallar:

- kullanıcı için tek aktif günlük hedef gibi davranır
- yeni kayıt geldiğinde eski kayıt güncellenebilir veya tek kayıt mantığıyla yönetilebilir

### 3.2 Food

Sistem veya kullanıcı food kataloğu.

Alanlar:

```text
id
userId
name
cal
carb
pro
fat
bestFor
type
defaultAmount
defaultUnit
createdAt
updatedAt
deletedAt
```

Kurallar:

- `userId = null` ise sistem food
- `type`: `solid | liquid`
- nutrition değerleri standart bazda saklanır
- `defaultAmount` ve `defaultUnit` giriş kolaylığı içindir, hesap standardı değildir

### 3.3 Meal

Gerçek tüketim kaydı.

Alanlar:

```text
id
userId
consumedAt
mealType
note
totalCal
totalCarb
totalPro
totalFat
source
createdAt
updatedAt
deletedAt
```

Kurallar:

- `mealType`: `breakfast | lunch | dinner | snack | other`
- `source`: `manual | program | aiText | aiPhoto`

### 3.4 MealFood

Meal içindeki tekil satırlar.

Alanlar:

```text
id
mealId
foodId
consumedAmount
consumedUnit
normalizedAmount
normalizedUnit
foodNameSnapshot
calSnapshot
carbSnapshot
proSnapshot
fatSnapshot
createdAt
updatedAt
deletedAt
```

Kurallar:

- hesaplama `normalizedAmount` üzerinden yapılır
- snapshot alanları geçmiş veriyi korur
- `consumedUnit` kullanıcı girişini, `normalizedUnit` sistem standardını temsil eder

### 3.5 Supplement

Sistem veya kullanıcı supplement kataloğu.

Alanlar:

```text
id
userId
name
purpose
description
recommendedAmount
recommendedUnit
createdAt
updatedAt
deletedAt
```

### 3.6 SupplementLog

Her supplement kullanımı için ayrı satır.

Alanlar:

```text
id
userId
supplementId
consumedAmount
consumedUnit
loggedAt
note
createdAt
updatedAt
deletedAt
```

### 3.7 WaterLog

Her su tüketimi için ayrı satır.

Alanlar:

```text
id
userId
amountMl
loggedAt
source
createdAt
updatedAt
deletedAt
```

Kurallar:

- UI bardak/şişe gibi girişler sunabilir
- backend sonucu `amountMl` olarak normalize eder

### 3.8 Program

Şablon kapsayıcı kayıt.

Alanlar:

```text
id
userId
name
notes
isActive
createdAt
updatedAt
deletedAt
```

### 3.9 ProgramMeal

Program içindeki öğün şablonu.

Alanlar:

```text
id
programId
mealType
title
note
totalCal
totalCarb
totalPro
totalFat
orderIndex
createdAt
updatedAt
deletedAt
```

### 3.10 ProgramMealFood

Program öğünündeki food satırları.

Alanlar:

```text
id
programMealId
foodId
consumedAmount
consumedUnit
normalizedAmount
normalizedUnit
foodNameSnapshot
calSnapshot
carbSnapshot
proSnapshot
fatSnapshot
createdAt
updatedAt
deletedAt
```

### 3.11 FoodFavorite

Kullanıcının favorilediği food kayıtları.

Alanlar:

```text
id
userId
foodId
createdAt
updatedAt
deletedAt
```

### 3.12 SupplementFavorite

Kullanıcının favorilediği supplement kayıtları.

Alanlar:

```text
id
userId
supplementId
createdAt
updatedAt
deletedAt
```

### 3.13 Tracker

Tek reminder modeli.

Alanlar:

```text
id
userId
type
description
scheduleType
triggerRules
isActive
createdAt
updatedAt
deletedAt
```

Not:

- `Tracker` final model olarak düşünülür
- gerçek entegrasyon 4. fazda yapılacaktır

---

## 4. İlişkiler

```text
user 1---N target
user 1---N food
user 1---N meal
user 1---N supplement
user 1---N supplementLog
user 1---N waterLog
user 1---N program
user 1---N foodFavorite
user 1---N supplementFavorite
user 1---N tracker

meal 1---N mealFood
food 1---N mealFood

program 1---N programMeal
programMeal 1---N programMealFood
food 1---N programMealFood

food 1---N foodFavorite
supplement 1---N supplementLog
supplement 1---N supplementFavorite
```

---

## 5. Uygulama Davranışı

### 5.1 Günlük kullanım

- kullanıcı ana nutrition ekranında bugünkü toplamları görür
- hedefe göre kalan kalori, makro ve su durumu hesaplanır
- öğün, su ve supplement girişleri hızlı aksiyonlarla yapılır

### 5.2 Program davranışı

- program bir template’tir
- gerçek meal yerine geçmez
- kullanıcı program içindeki bir öğünü tamamlandığında seçer
- sistem ilgili `programMealId` üzerinden yeni gerçek `Meal + MealFood` üretir

### 5.3 Favori davranışı

- kullanıcı sistem veya kendi food kayıtlarını favorileyebilir
- kullanıcı sistem veya kendi supplement kayıtlarını favorileyebilir
- favori kayıtları ayrı relation tablolarıyla yönetilir

### 5.4 AI davranışı

- AI ilk core fazda gerçek entegrasyon olmadan planlanır
- daha sonra text/photo parse sonucu kullanıcıya öneri döner
- kullanıcı onaylarsa gerçek meal oluşur

### 5.5 Tracker davranışı

- water ve supplement reminder mantığı tek `Tracker` modeli üzerinden yönetilecektir
- job/notification entegrasyonu çekirdek nutrition tesliminden sonra gelir

---

## 6. Faz Planı

### Faz 1 - Frontend Mimari ve UX

- bu final modele uygun ekran ve akış tasarlanır
- today odaklı bilgi mimarisi netleştirilir
- provider/repository yapısı belirlenir

### Faz 2 - Backend Core Nutrition

- tüm çekirdek modeller, ilişkiler ve endpointler yazılır
- hedef, food, meal, supplement, water, program ve favoriler tamamlanır
- AI ve tracker bu fazın içinde zorunlu değildir

### Faz 3 - AI Entegrasyonu

- text ve photo parse akışı eklenir
- meal create onay akışı bağlanır
- prompt ve parser yapıları eklenir

### Faz 4 - Tracker Entegrasyonu

- generic `Tracker` CRUD tamamlanır
- reminder tetikleme/job akışı planlanır
- push entegrasyonu bağlanır

---

## 7. Bu Dokümanın Kullanımı

Bu doküman sonrasında:

- frontend planı bu veri yapısına göre revize edilir
- backend planı bu model setine göre sadeleştirilir
- AI promptları bu akışa göre üretilir
- tracker planı en sona bırakılır

Bu yapı nutrition modülünü gereksiz karmaşıklığa sokmadan, kontrollü şekilde 4 fazda sisteme entegre etmek için final referans olarak kullanılacaktır.
