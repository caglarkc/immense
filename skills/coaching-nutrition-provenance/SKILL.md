---
name: coaching-nutrition-provenance
description: Coaching ile nutrition entegrasyonunda member-owned veri, coach write/read yetkisi, provenance alanları ve workspace tab organizasyonunu koruma skill'i.
---

## Purpose

Nutrition entegrasyonunda:
- member-owned veri modelini korumak
- coach write/read sınırlarını net tutmak
- provenance izini kaybetmemek
- coach/member workspace içinde nutrition sekmesini kontrollü entegre etmek

## Rules

- Nutrition domain doğrusu `nutrition-service` içinde kalır; coach yetkisi ayrı bir bypass mantığına dönmemelidir.
- Member-owned veri korunur; coach write açılıyorsa provenance zorunludur.
- Target ve program değişikliklerinde en az son editör rolü ve coach izi tutulmalıdır.
- Meal / water / supplement günlük logları ile target / program yönetimi aynı davranış gibi ele alınmamalıdır.
- Coach workspace ve member workspace nutrition entegrasyonu 3-tab yapıyı bozmayacak şekilde kurulmalıdır:
  - `İletişim`
  - `Antrenman`
  - `Beslenme`
- Mevcut nutrition ekranları kör kopyalanmaz; coaching context gerekiyorsa wrapper veya context-aware varyant kullanılır.
- UI katmanında doğrudan endpoint veya context hack'i yapılmaz; repository ve provider zinciri korunur.

## Workflow

1. Nutrition verisinin owner’ı kim, editor’ı kim, viewer’ı kim çıkar.
2. Read ve write yüzeylerini ayrı listele.
3. Provenance hangi model/alanlarla tutulacak sabitle.
4. Coach tabı ile member tabında hangi nutrition yüzeyleri reuse edilecek belirle.
5. Snapshot / load politikasını mevcut nutrition ve coaching kurallarıyla çakıştırma.

## References

- `son_immense/.agent/rules.md`
- `son_immense/.agent/app.md`
- `son_immense/.agent/backend.md`
- `son_immense/backend/services/nutrition-service/`
- `son_immense/backend/services/coaching-service/`
- `son_immense/frontend/lib/modules/nutrition/`
- `son_immense/frontend/lib/modules/coaching/`
