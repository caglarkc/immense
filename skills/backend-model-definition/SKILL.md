name: backend-model-definition
description: Sequelize model standartları; UUID PK, soft delete (paranoid: true) ve camelCase alan isimleri.

---

## Purpose

Veritabanı modellerinde **tutarlılık**, **kolay migrasyon**, **silme güvenliği** (soft delete) ve kod tarafında **tek isimlendirme standardı** sağlamak.

## Rules

- **ORM**: Sequelize kullanılır; model tanımı Sequelize standartlarına uygun olmalıdır.
- **Primary Key**: Tüm tabloların PK’sı **UUID** olmalı; integer/auto-increment PK **yasak**.
- **Soft delete zorunlu**: `paranoid: true` kullanılacak; hard delete default akış **yasak**.
- **camelCase zorunlu**: Tüm alan adları (DB kolonları dahil) **camelCase** olmalı; `snake_case` **yasak**.
- **İsim standardı**: Model/field isimleri İngilizce; kısaltma/yerel isimlendirme **yasak**.
- **Timestamp tutarlılığı**: Sequelize’in timestamp alanları proje standardına göre tek tip kullanılmalı; model bazında keyfi farklılaştırma **yasak**.

## References

- `skills/general-naming-language`
- `skills/backend-controller-service-flow`