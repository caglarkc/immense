name: admin-2fa-mfa
description: Admin panel giriş güvenliği için 2FA/MFA standardı; TOTP zorunluluğu, recovery kodları ve oturum politikası.

---

## Purpose

Admin panelin yüksek yetkili doğası nedeniyle tek faktörlü giriş riskini azaltıp hesap ele geçirme senaryolarını engellemek.

## Rules

- **2FA zorunlu**: Admin kullanıcıları için TOTP tabanlı 2FA zorunludur (istisna yok).
- **Enrollment akışı**:
  - Admin ilk girişte 2FA kurulumuna yönlendirilir.
  - QR/secret sadece kurulum ekranında gösterilir; sonrasında tekrar gösterilmez.
- **Recovery codes**:
  - Tek kullanımlık recovery kodları üretilir ve kullanıcıya verilir.
  - Recovery kodları sunucuda hash’li saklanır; plain saklamak **yasak**.
- **Session politikası**:
  - 2FA doğrulanmadan “admin yetkisi gerektiren” endpoint’lere erişim **yasak**.
  - Oturum süresi ve yenileme politikası net tanımlıdır (idle timeout vs absolute).
- **Brute force koruması**:
  - 2FA kod denemeleri rate limit’e tabidir.
  - Yanlış kod mesajı uniform olur; bilgi sızdıran farklı mesajlar **yasak**.
- **Audit log**:
  - Başarılı/başarısız 2FA denemeleri (PII minimize) audit log’a yazılır.

## References

- `skills/backend-rate-limiting`
- `skills/observability-logging-metrics`
