---
name: coaching-member-boundary
description: Coach-member akışlarında aktif ilişki doğrulaması, stale snapshot riski, role boundary ve read-only/member-safe ekran davranışlarını koruma skill'i.
---

## Purpose

Coach-member özellikleri geliştirirken:
- aktif ilişki kontrolünü tek doğruda tutmak
- coach / member / discovery workspace ayrımını bozmamak
- stale snapshot nedeniyle yanlış workspace açılmasını önlemek
- member-safe read-only davranışı korumak

## Rules

- `acceptedCoachCommitment` ile gerçek aktif membership aynı şey değildir; kararlar `hasActiveCoachMembership` üstünden verilir.
- Disk snapshot, aktif ilişkiyi doğrulayan server cevabının yerine geçemez; kritik workspace kararları stale snapshot ile finalize edilmez.
- Inactive coach ilişkisi aktif workspace sayılmaz.
- Member route’ları aktif ilişki yoksa güvenli fallback vermelidir.
- Coach-only CTA’lar member-safe ekranlarda görünmemelidir.
- Read-only reuse yapılacaksa flag zorunlu ve merkezi olmalıdır; “UI'da tesadüfen gizlendi” yaklaşımı yeterli değildir.
- Chat / profile / training / nutrition yüzeyleri coach-member context’i kaybetmeden bağlanmalıdır.

## Workflow

1. Workspace kararının hangi state ile verildiğini kontrol et.
2. Aynı veri snapshot’tan hydrate oluyorsa server revalidation gerekip gerekmediğini netleştir.
3. Member-safe ekranlarda edit/delete/assign gibi coach-only aksiyonların kapandığını doğrula.
4. Route cold-start senaryosunda ekranın sadece cache/extra ile kırılmadığını kontrol et.

## References

- `son_immense/.agent/rules.md`
- `son_immense/.agent/app.md`
- `son_immense/.agent/backend.md`
- `son_immense/frontend/lib/modules/coaching/`
- `son_immense/backend/services/coaching-service/`
