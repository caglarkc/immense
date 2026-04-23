# Admin Panel Geliştirme Kuralları

> React 18 · Vite · SPA · State-Based Sayfa Geçişi · Desktop-Only

---

## Klasör Yapısı

```
src/
├── api/           → client.js (tüm API fonksiyonları)
├── components/    → AdminAvatar, AdminExerciseMedia, ImageLightbox, ExerciseAssetPicker
├── context/       → AdminContext (token, lightbox)
├── layout/        → AdminLayout, MenuBar
├── modules/       → auth, users, training, exercises, social-media, stats, gyms
│   └── [modül]/   → *Page.jsx
├── pages/         → HomePage
└── App.jsx, main.jsx
```

---

## Uygulama Akışı

- Giriş noktası: `App.jsx`.
- Token var → `AdminLayout` (ana sayfa), Token yok → `AuthPage` (login).
- SPA yapısı: route yerine `currentPage` state ile modül göster/gizle.
- Sayfalar `display: none/block` ile gösterilir.
- Her sayfa `token` ve `onNavigateTo*` prop'ları alır.

---

## API Client

- Tüm API istekleri **`src/api/client.js`** içindeki `request(...)` wrapper'ı ile yapılır.
- Base URL: `VITE_ADMIN_API_BASE_URL` (fallback: `http://localhost:3000/admin`).
- Header: `Authorization: Bearer ${token}`.
- Hata: `data?.customMessage || resp.statusText` → Error fırlatılır.

**YASAK:** Sayfa bileşenlerinde doğrudan `fetch()` veya `axios` çağrısı yapılmaz. Her istek `client.js` üzerinden geçer.

---

## Auth ve Token Yönetimi

- Token `localStorage` anahtarı: `immense_admin_token`.
- `getAdminProfile(token)` ile token doğrulama.
- `AdminContext` → `token`, `openImageLightbox`, `closeImageLightbox`, `lightboxImage` sağlar.

---

## Veri Yükleme Politikası

Bu kurallar `DATA_LOADING_POLICY.md` ile uyumludur ve tüm sayfalarda geçerlidir:

- **Otomatik/periyodik refresh YOK.**
- Sayfa ilk açıldığında **bir kez** veri çekilir.
- Sekme değiştirip geri gelince otomatik istek atılmaz.
- **Yenile** butonu ile kullanıcı manuel yeniler.
- Ara, filtre, sayfa değiştirme → kullanıcı aksiyonu = 1 istek.

---

## Sayfa State Pattern

Her sayfa aşağıdaki standart state yapısını kullanır:

```jsx
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);
const [page, setPage] = useState(1);
const [selectedId, setSelectedId] = useState(null);
const [loading, setLoading] = useState(false);
const [saving, setSaving] = useState(false);
const [error, setError] = useState(null);
const [info, setInfo] = useState(null);

const initialLoadDone = useRef(false);

useEffect(() => {
  if (token && !initialLoadDone.current) {
    initialLoadDone.current = true;
    load();
  }
}, [token]);
```

Mevcut modüller: UsersPage, RoutinesPage, WorkoutsPage, ExercisesPage, PostsPage, ReportsPage, StatsPage, GymsPage.

---

## Medya URL

- `getMediaUrl(path)` → `API_BASE_URL/media/file?path=...`
- Path prefix'leri: `users/`, `posts/`, `exercises/`, `measurements/`.

---

## UI/UX Kuralları

- **Desktop-only** platform: Tasarımlar masaüstü öncelikli UX ile yapılır.
- **Loading ve hata mesajları zorunlu:** Kullanıcı hiçbir senaryoda tepkisiz/boş ekranda bırakılmaz.
- Bileşenler sade ve temiz kod standartlarına uygun olmalıdır.
- Değişken/fonksiyon/bileşen isimleri → **İngilizce**; yorumlar → **Türkçe**.
- Backend ve App ile tutarlı tema/renk/font standartları korunur.

---

## Yapılacak / Yapılmayacak Özet

| ✅ YAP | ❌ YAPMA |
|--------|----------|
| `client.js` üzerinden API çağır | Sayfa içinde doğrudan fetch/axios |
| Standart state pattern kullan | Farklı state yapıları icat etme |
| `useRef` ile ilk yükleme kontrolü | Otomatik/periyodik refresh |
| Loading + error state göster | Kullanıcıyı boş ekranda bırakma |
| Kullanıcı aksiyonunda veri çek | Sekme geçişinde otomatik istek |
| `AdminContext` üzerinden token al | Token'ı prop drilling ile taşıma |