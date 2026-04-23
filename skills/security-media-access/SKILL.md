name: security-media-access
description: Media erişim güvenliği standardı; path doğrulama, presigned URL politikası, yetkilendirme ve sızıntı önleme.

---

## Purpose

MinIO/medya dosyalarının yetkisiz erişimini engelleyip path traversal ve link sızıntısı risklerini azaltmak.

## Rules

- **Path allow-list**: `path` parametresi allow-list prefix’leriyle sınırlandırılır (örn. `users/`, `posts/`, `exercises/`, `measurements/`); serbest dosya yolu **yasak**.
- **Path traversal koruması**: `..`, `//`, URL-encoded traversal varyantları reddedilir.
- **Yetkilendirme zorunlu**:
  - Private kullanıcı medyası sadece sahibi/izinli roller tarafından erişilebilir.
  - Admin erişimleri ayrı audit log ile izlenir.
- **Presigned politika**:
  - Presigned URL süresi sınırlıdır (kısa TTL).
  - Presigned URL üretimi, kullanıcı kimliği ve erişim kontrolünden sonra yapılır.
- **Response standardı**: Media endpoint’lerinde hata akışı da standart hata mekanizmasına uyar (next(error)).
- **Log hijyeni**: Presigned URL’ler log’a yazılmaz.

## References

- `skills/backend-input-validation`
- `skills/observability-logging-metrics`
