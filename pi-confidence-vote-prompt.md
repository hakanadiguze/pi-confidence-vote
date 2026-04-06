# Claude Code Prompt: PI Confidence Vote

## Görev
Aşağıdaki spesifikasyonlara birebir uyan, production-ready, tek dosyalı (`index.html`) bir **PI Confidence Vote** web uygulaması geliştir. Uygulama SAFe (Scaled Agile Framework) PI Planning toplantılarında kullanılmak üzere tasarlanmıştır.

---

## Tech Stack

- **Frontend:** Vanilla HTML + CSS + JavaScript (framework yok, build step yok)
- **Backend/Realtime DB:** Firebase Realtime Database (Firebase JS SDK v10, CDN üzerinden ES module import)
- **PDF Export:** jsPDF v2.5.1 (CDN)
- **Fonts:** Google Fonts — Nunito (weights: 400,600,700,800,900) + DM Sans (weights: 400,500,600)
- **Analytics:** Google Analytics 4 (gtag.js)

---

## Firebase Yapılandırması

Aşağıdaki Firebase config'i kullan (placeholder — entegrasyon yapan kişi kendi config'ini girecek):

```js
const firebaseConfig = {
  apiKey: "FIREBASE_API_KEY",
  authDomain: "PROJECT_ID.firebaseapp.com",
  databaseURL: "https://PROJECT_ID-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "PROJECT_ID",
  storageBucket: "PROJECT_ID.firebasestorage.app",
  messagingSenderId: "SENDER_ID",
  appId: "APP_ID"
};
```

Firebase SDK importları CDN üzerinden ES module olarak yapılacak:
```js
import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js';
import { getDatabase, ref, set, get, onValue } from 'https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js';
```

### Firebase Realtime Database Veri Yapısı

```
sessions/
  {SID}/
    art: string           // ART adı
    teams: [              // takım listesi
      { id: string, name: string }
    ]
    votes/
      {teamId}/
        {voterKey}: { voter: string, score: number }  // voterKey = anonim ID
      ART/
        {voterKey}: { voter: string, score: number }
    createdAt: number     // timestamp
```

---

## Uygulama Akışı

### Ekran 1 — Home Screen (`#homeScreen`)
- Logo (🗳️ emoji, teal gradient arka plan, rounded)
- Başlık: "PI Confidence Vote"
- Alt başlık: "Real-time PI Planning confidence votes for agile teams"
- Tek CTA butonu: "Configure & Create Session →" (turuncu)
- Collapsible "How does it work?" bölümü (açıklamalı 5 adım)
- Footer: "Built by Hakan"
- **Katılım SADECE link ile olur. Ana ekranda join formu, ID girişi veya isim girişi YOKTUR.**

### Ekran 2 — Config Screen (`#configScreen`)
- "← Back" butonu
- **Sadece 2 input:** ART Name, Teams listesi (dinamik ekle/çıkar)
- Varsayılan 2 takım: "Team 1", "Team 2"
- "🚀 Create Session" butonu (turuncu)
- **İsim veya host adı girişi YOKTUR**

### Ekran 3 — Quick Join Screen (`#quickJoinScreen`)
- Davet linki ile gelen kullanıcılar bu ekrana düşer
- ART adını gösterir: "You've been invited to vote in '{ART}'. Votes are anonymous."
- "Join & Vote →" butonu — isme gerek yok, direkt katılır
- Hata mesajı alanı

### Ekran 4 — Board Screen (`#boardScreen`)
Board ekranı herkese aynı görünür (host ve katılımcılar arası fark sadece share box görünürlüğüdür).

**Header:**
- Sol: ART adı + "Voting in progress…"
- Sağ (sadece host görür): Share box — davet linkini gösterir + "🔗 Copy Invite Link" butonu

**Banner (teal arka plan):**
- Sol: "🗳️ Voting in progress" + "Click any score button on a card to cast your vote — votes are anonymous"
- Sağ (sadece host görür): "⬇️ Download PDF" butonu

**Cards Grid:**
- CSS Grid: `repeat(auto-fill, minmax(280px, 1fr))`, gap: 16px
- Her takım için bir kart
- ART kartı: `grid-column: 1 / -1` (tam genişlik)

---

## Kart Yapısı (Her Takım Kartı)

```
┌─────────────────────────────────┐
│ TEAM NAME            N votes    │  ← turuncu header (#f5a623)
├─────────────────────────────────┤
│  [bar chart: 5 sütun, 1-5]      │
│  Team Average              3.8  │  ← renk kodlu ortalama badge
├─────────────────────────────────┤
│  Your vote for this team:       │
│  [1😟] [2😕] [3😐] [4🙂] [5😄]  │  ← inline vote butonları
│  ✓ You voted 4                  │
└─────────────────────────────────┘
```

**ART Kartı** aynı yapıda ama:
- Header: gradient (linear-gradient(135deg, #e8a020, #f5c842))
- Bar etiketleri: NO CONF. / LITTLE / GOOD / HIGH / V.HIGH
- Büyük ortalama badge (font-size: 32px)
- Label: "Your vote for the overall PI plan (ART level):"

---

## Renk Sistemi

### CSS Değişkenleri
```css
:root {
  --bg: #dff0eb;
  --card: #fff;
  --teal: #2d7a6e;
  --tl: #e0f2ef;
  --tm: #4a9e90;
  --or: #e8a96a;
  --ord: #d4895a;
  --tx: #1a2e2b;
  --mu: #6b8c87;
  --bo: #c8e0db;
  --r: 18px;
  --sh: 0 4px 24px rgba(45,122,110,.10);
  --v1: #e05252;
  --v2: #e88c4a;
  --v3: #c8a000;
  --v4: #7ec87a;
  --v5: #3aab6d;
}
```

### Bar Chart Renkleri
| Skor | Renk | Anlam |
|------|------|-------|
| 1 | #e05252 | No Confidence |
| 2 | #e88c4a | Little |
| 3 | #c8a000 | Good |
| 4 | #7ec87a | High |
| 5 | #3aab6d | Very High |

### Ortalama Badge Renk Kodlaması
```js
function sStyle(v) {
  const n = parseFloat(v);
  if (n < 1.5) return { bg: '#fde8e8', c: '#e05252' };
  if (n < 2.5) return { bg: '#fdf0e0', c: '#e88c4a' };
  if (n < 3.5) return { bg: '#fefae0', c: '#c8a000' };
  if (n < 4.5) return { bg: '#e8f8e8', c: '#7ec87a' };
  return { bg: '#d8f5e8', c: '#3aab6d' };
}
```

### Vote Buton Renkleri
```css
.ivb.s1 { background: #fde8e8; color: #e05252; }
.ivb.s2 { background: #fdf0e0; color: #e88c4a; }
.ivb.s3 { background: #fefae0; color: #c8a000; }
.ivb.s4 { background: #e8f8e8; color: #7ec87a; }
.ivb.s5 { background: #d8f5e8; color: #3aab6d; }
```

---

## JavaScript — Temel Fonksiyonlar

### State Değişkenleri
```js
let activeSid    = null;  // aktif session ID
let currentUser  = null;  // anonim voter ID (örn: "anon_AB12CD")
let isHost       = false;
let sessionData  = null;  // Firebase'den gelen son veri
let unsubscribe  = null;  // Firebase onValue unsubscribe fonksiyonu
```

### Session Oluşturma
```js
async function createSession() {
  // 1. cfgArt input'undan ART adını al
  // 2. teamList'teki tüm dolu input'lardan teams array'i oluştur
  // 3. uid() ile 6 karakterli session ID üret
  // 4. Firebase'e yaz: { art, teams, votes: {[teamId]: {}, ART: {}}, createdAt }
  // 5. currentUser = 'anon_' + uid() (anonim host)
  // 6. isHost = true
  // 7. openBoard(sid) çağır
}
```

### Davet Linki ile Katılım (URL Parsing)
```js
// DOMContentLoaded'da:
const params = new URLSearchParams(location.search);
let joinId = params.get('join');  // ?join=XXXXXX

// Session varsa → quickJoinScreen'e yönlendir, activeSid'i set et
// Session yoksa → homeScreen'e yönlendir, toast göster
```

### Oy Kullanma — KRİTİK
```js
async function castVote(tid, score) {
  // voterKey = currentUser (anonim ID)
  // Firebase path: sessions/{activeSid}/votes/{tid}/{voterKey}
  // Her kullanıcının her hedef için TEK oyu olur (key üzerine yazar)
  const voterKey = currentUser.replace(/[.#$/\[\]]/g, '_');
  await set(ref(db, `sessions/${activeSid}/votes/${tid}/${voterKey}`), { voter: currentUser, score });
}
```

### Realtime Güncelleme
```js
// openBoard içinde Firebase onValue listener kurulur
// Her değişiklikte renderBoard(sessionData) çağrılır
// Bar chart yükseklikleri: (count / maxCount) * 100%
// Animasyon: transition: height .5s cubic-bezier(.34,1.56,.64,1)
```

### UID Üretici
```js
function uid() {
  return Math.random().toString(36).slice(2,8).toUpperCase();
}
```

### Davet Linki Oluşturma
```js
function buildShareUrl(sid) {
  return `${location.origin}${location.pathname}?join=${sid}`;
}
```

### Firebase Votes → Array Dönüştürme
Firebase object olarak saklar, array'e çevirmek gerekir:
```js
function votesArray(obj) {
  if (!obj) return [];
  if (Array.isArray(obj)) return obj;
  return Object.values(obj);
}
```

---

## PDF Export (jsPDF — Programatik, html2canvas KULLANMA)

PDF tamamen jsPDF ile programatik olarak çizilir. `html2canvas` veya DOM screenshot kullanılmaz.

### PDF Yapısı: 2 Sayfa, A4 Landscape

**Sayfa 1 — Takım Kartları:**
- Mint yeşili arka plan (#dff0eb)
- Üstte teal başlık şeridi: ART adı + tarih + session ID
- Takım kartları grid halinde (max 3 sütun, otomatik satır)
- Her kart: turuncu header, 5 sütunlu bar chart, ortalama badge

**Sayfa 2 — ART Level:**
- Mint yeşili arka plan
- Sarı-turuncu gradient başlık
- Tam sayfa genişliğinde ART kartı
- Büyük bar chart, büyük ortalama badge

### jsPDF Yardımcı Fonksiyonları
```js
// Renkli yuvarlak köşeli dikdörtgen
const rr = (x, y, w, h, r, fill) => { /* roundedRect */ };

// Metin yazma (konum, boyut, renk, hizalama, kalınlık)
const txt = (text, x, y, size, color, align='left', bold=false) => { /* ... */ };

// Bar çizme (arka plan + dolgu)
const bar = (x, y, w, h, pct, col) => { /* ... */ };
```

---

## Anonimlik Kuralları

1. **Hiçbir yerden isim istenmez** — ne host'tan ne katılımcıdan
2. Her session oluşturmada: `currentUser = 'anon_' + uid()`
3. Her link ile katılımda: `currentUser = 'anon_' + uid()`
4. Kartlarda oy dağılımı gösterilir ama kim ne oy verdi gösterilmez
5. Kendi oyunu gösteren butona dark border eklenir (sadece o kullanıcının tarayıcısında)

---

## URL Routing

- Ana sayfa: `/` → homeScreen
- Davet linki: `/?join=XXXXXX` → quickJoinScreen (session varsa)
- Hash routing kullanılmaz, sadece query param `?join=`

---

## Responsive

```css
@media (max-width: 640px) {
  .bgrid { grid-template-columns: 1fr; }
  .vc-card.art-card-full { grid-column: auto; }
  .bhdr { flex-direction: column; }
  .share-box { width: 100%; }
}
```

---

## Dosya Çıktısı

- **Tek dosya:** `index.html` (CSS ve JS inline, external dependency yok — sadece CDN linkleri)
- **`vercel.json`** (SPA routing için):
```json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```
- **`README.md`** (kurulum ve deploy adımları)

---

## Dikkat Edilecek Noktalar

1. Tüm `window.*` global fonksiyon açıklamaları `<script type="module">` içinde yapılmalı (ES module scope sorunu)
2. Firebase votes nesnesini array'e çevirmek için `votesArray()` fonksiyonu kullanılmalı
3. Bar chart yüksekliği: `count / Math.max(...counts, 1) * 100%` (sıfıra bölünme önlenir)
4. Voted butonu highlight: sadece `currentUser`'a ait oy, `highlightMyVote()` ile işaretlenir
5. `html2canvas` kullanılmayacak — PDF tamamen programatik
6. Firebase `onValue` subscription `openBoard` çağrıldığında kurulur; önceki subscription varsa önce `unsubscribe()` çağrılır
