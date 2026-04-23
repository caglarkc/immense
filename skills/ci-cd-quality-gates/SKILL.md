name: ci-cd-quality-gates
description: PR/branch ve kalite kapıları standardı; lint/typecheck/build zorunlulukları ve merge kriterleri.

---

## Purpose

Testleri sen manuel yapıyor olsan bile, entegrasyon kalitesini korumak için minimum “kalite kapıları” ve PR disiplini tanımlamak.

## Rules

- **PR zorunlu**: `main/master` doğrudan commit ile değiştirilemez; PR üzerinden ilerlenir.
- **Kapsam kontrolü**: PR yalnızca tek iş/konu kapsar; “bir PR’da her şey” **yasak**.
- **Kırmızı çizgi**: Lint/typecheck/build başarısızsa merge **yasak**.
- **Geriye dönük uyumluluk**: API sözleşmesini bozacak değişiklikler (response şeması, pagination) PR açıklamasında açıkça işaretlenir.
- **Secrets yasağı**: `.env`, token, credential, private key commit’lenmez.
- **Changelog disiplini**: Kullanıcıya/operasyona etki eden değişiklikler için kısa not bırakılır (format projede ayrıca belirlenir).
- **Bağımlılık ekleme**: Yeni dependency eklemek gerekirse gerekçesi PR açıklamasında yazılır; “rastgele ekleme” **yasak**.

## References

- `skills/general-naming-language`
- `skills/backend-standard-response`
