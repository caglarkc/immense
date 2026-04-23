# Nutrition Service Audit

## Backend Yuzeyi

- Route klasoru: `backend/services/nutrition-service/src/routes/*.js`
- Modeller:
  - `food`, `foodFavorite`
  - `meal`, `mealFood`
  - `program`, `programMeal`, `programMealFood`
  - `supplement`, `supplementFavorite`, `supplementLog`
  - `target`, `waterLog`
  - `nutritionAi*`
- Servisler:
  - `food.service.js`, `meal.service.js`, `program.service.js`
  - `supplement.service.js`, `target.service.js`, `water.service.js`
  - `nutrition-summary.service.js`
  - `nutritionAi.service.js`, `nutritionAiGemini.service.js`

## Frontend Baglantilari

- Data:
  - `nutrition_repository.dart`
  - AI optimizer/model dosyalari
- Logic:
  - `nutrition_home_provider.dart`
  - `nutrition_food_provider.dart`
  - `nutrition_meal_provider.dart`
  - `nutrition_program_provider.dart`
  - `nutrition_target_provider.dart`
  - `nutrition_supplement_provider.dart`
  - `nutrition_water_provider.dart`
  - `nutrition_history_provider.dart`
  - `nutrition_ai_photo_provider.dart`
- Screens:
  - `nutrition_home_screen.dart`
  - foods, meals, programs, supplements, targets, water, history ekranlari
  - coach member nutrition varyantlari

## Veri ve State

- Nutrition master data ve home summary bootstrapte hazirlaniyor.
- `NutritionHomeProvider` bugun verisini memory state + active request dedupe ile tutuyor.
- Coach-member nutrition akislari ayri proxy endpoint seti ile ilerliyor.
- AI photo flow gecici dosya/optimizasyon ve onayli meal yaratma adimlari iceriyor.

## Ana Akislar

- Home summary
- Hedef olusturma/guncelleme
- Food CRUD + favorite
- Meal CRUD + meal food yonetimi
- Program CRUD + program meal/food yonetimi + meal create
- Supplement CRUD + favorite + log
- Water log + daily summary
- AI photo -> preview -> confirm meal -> feedback

## Buton ve CTA Mantigi

- Home quick action chips
- Quick add water / supplement
- Food/program/meal/supplement ekleme-duzenleme-silme
- Pull-to-refresh
- Programdan gercek meal olusturma

## Risk ve Notlar

- Nutrition service hem normal kullanici hem coaching member proxy varyantina sahip oldugu icin iki farkli navigation yuzeyi var.
- Home preload ve ekran ic load dedupe son donemde optimize edildi.
- AI akislarinda media, network ve parse hata yonetimi urun kalitesi acisindan kritik.
