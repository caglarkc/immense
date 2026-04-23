name: general-naming-language
description: Kodda İngilizce isimlendirme; yorum/açıklamalarda Türkçe; veritabanında camelCase zorunluluğu.

---

## Purpose

Ekip genelinde okunabilirlik ve sürdürülebilirlik için **tek dil ve isimlendirme standardı** oluşturmak.

## Rules

- **Kod dili**: Değişken, fonksiyon, class, dosya ve klasör adları **İngilizce** olmalı.
- **Yorum/açıklama dili**: Kod yorumları ve kullanıcıya dönük açıklamalar **Türkçe** olmalı (aksi belirtilmedikçe).
- **DB alanları**: Veritabanı tablo/kolon alanları **camelCase** olmalı; `snake_case` **yasak**.
- **Tutarlılık**: Aynı kavram için birden fazla isim kullanmak **yasak**; tek terim seçilir ve her yerde uygulanır.
- **Kısaltma disiplini**: Belirsiz kısaltmalar (örn. `dt`, `tmp`, `misc`) **yasak**; anlamlı isim zorunlu.

## References

- `skills/backend-model-definition`
- `skills/backend-standard-response`
- `skills/app-clean-architecture`
- `skills/admin-api-client-rule`