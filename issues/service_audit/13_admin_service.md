# Admin Service Audit

## Backend Yuzeyi

- Route: `backend/services/admin-service/src/routes/admin.routes.js`
- Modeller:
  - admin-user
  - beta feedback, block, comment, follow, gym, city/district
  - exercise, routine, workout, stats, measurement
  - post, post-like, post-save, report
- Servisler:
  - `admin-auth.service.js`
  - `sync-client.js`

## Frontend Baglantisi

- Bu repo icinde mobil Flutter tarafinda admin panel UI yok.
- Proje kurallari admin panelin React oldugunu soyluyor, ancak bu audit kapsaminda admin frontend kaynaklari gorunur degil.

## Islevsel Kapsam

- Founder bootstrap
- Admin login / me
- Muhtemel moderasyon ve yonetim yuzeylerine veri saglama
- Sync ile diger servisleri tetikleme veya veri denetimi

## Not

- Backend modeli genis; admin tarafi diger tum urun alanlarinin capraz gorunumu gibi duruyor.
- Admin frontend kodu bu workspace'te olmadigi icin "button/view/component" seviyesi audit bu servis icin backend-merkezli kaldi.
