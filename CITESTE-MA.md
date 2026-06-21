# Nuntă Ovidiu & Ionela — ghid

## Rulare locală (Windows)

Proiectul e livrat fără `node_modules`. O singură dată:

```bash
npm install
npm run dev      # http://localhost:3000
```

Build de producție: `npm run build` apoi `npm start`.

---

## Deploy pe Netlify + domeniu de la RoTLD

### 1. Stocarea confirmărilor (deja rezolvată)

Pe Netlify rutele API rulează ca **funcții serverless cu disc read-only**, deci
nu se poate scrie în `data/rsvp.json`. Am înlocuit stocarea cu **Netlify Blobs**
(persistentă, gratuită, zero-config). Codul alege automat:
- pe Netlify → Netlify Blobs;
- local cu `npm run dev` → fișierul `data/rsvp.json` (ca înainte).

Nu trebuie să configurezi nimic pentru Blobs — Netlify populează singur contextul.

### 2. Urcă pe GitHub și conectează la Netlify

1. Pune proiectul într-un repo GitHub (push).
2. Netlify → **Add new site → Import from GitHub** → alege repo-ul.
3. Build command: `npm run build` (e deja în `netlify.toml`). Runtime-ul Next.js
   (`@netlify/plugin-nextjs`) se instalează automat — nu seta manual „publish".
4. Înainte de primul deploy, la **Site settings → Environment variables**, adaugă:

   ```
   EMAIL_HOST=smtp.gmail.com
   EMAIL_PORT=587
   EMAIL_USER=adresa-care-trimite@gmail.com
   EMAIL_PASS=parola-de-aplicatie-gmail
   EMAIL_TO=unde-vor-ajunge-confirmarile@gmail.com
   ADMIN_PASSWORD=alege-o-parola
   NEXT_PUBLIC_WEDDING_DATE=2026-10-17T18:00:00
   NEXT_PUBLIC_SITE_URL=https://domeniul-tau.ro
   ```

   (`.env.local` NU se urcă — e în `.gitignore`. Pe Netlify variabilele se pun aici.)
5. **Deploy site.** La final ai o adresă de test `nume-aleator.netlify.app`.

### 3. Leagă domeniul RoTLD

RoTLD îți permite din panou doar setarea **nameserver-elor** (nu are editor de
înregistrări A/CNAME), deci cel mai curat e să delegi DNS-ul către Netlify:

1. Netlify → site-ul tău → **Domain management → Add custom domain** → scrie
   domeniul tău (fără `www`). Adaugă și apex-ul, și `www`.
2. Alege **„Set up Netlify DNS"**. Netlify îți dă 4 nameservere, de forma:
   `dns1.p0X.nsone.net`, `dns2.p0X.nsone.net`, `dns3...`, `dns4...`
3. Intră la **rotld.ro/domadmin** (cu numele domeniului + parola; dacă n-o ai,
   o resetezi de acolo). La secțiunea **Nameservers** înlocuiești nameserverele
   cu cele 4 de la Netlify și apeși **Update**.
4. RoTLD trimite un email de confirmare — deschizi linkul și confirmi (fii logat
   în RoTLD când deschizi linkul).
5. Propagarea durează până la ~24h. SSL-ul (HTTPS) se emite automat de Netlify.

**Alternativă** (dacă vrei panou DNS complet, util și pentru email pe domeniu):
delegi nameserverele RoTLD către host și folosești înregistrări manuale —
apex `A → 75.2.60.5`, iar `www CNAME → nume-site.netlify.app`. Dar pentru RoTLD,
varianta „Netlify DNS" de mai sus e cea mai simplă.

---

## Cum schimbi adresele de email de confirmare

- **`EMAIL_USER` / `EMAIL_PASS`** = contul Gmail care **trimite**. `EMAIL_PASS`
  nu e parola normală, ci o **parolă de aplicație** (Gmail cu verificare în 2 pași
  → „Parole de aplicație" → 16 caractere).
- **`EMAIL_TO`** = unde **ajung** confirmările. Poți pune mai multe adrese separate
  prin virgulă: `EMAIL_TO=ovidiu@gmail.com, ionela@gmail.com`.

Local le schimbi în `.env.local`; pe Netlify în **Environment variables** (apoi
re-deploy). Numele expeditorului și subiectul se schimbă în `lib/email.ts`.

Fără `EMAIL_USER`/`EMAIL_PASS`, site-ul merge normal, doar că nu trimite email —
confirmările se salvează oricum și apar în `/admin`.

---

## Pozele (reparate)

Trei probleme rezolvate: extensii `.JPG` cu litere mari redenumite în `.jpg`;
cache vechi `.next` șters (de aici „thumbnail-ul" vechi); fișiere PNG cu nume
`.jpg` convertite în JPEG real. Plus `og-image.jpg` creat și data corectată.

> Când înlocuiești o poză: păstrează același nume cu extensia `.jpg` litere mici.
> Dacă apare versiunea veche, șterge folderul `.next` și repornește.

---

## Confirmările (RSVP) cu meniuri

Contoare **+ / −** pentru: adult, copil, vegetarian, alt meniu (cu câmp de detalii
pentru ovo-lacto / fără gluten etc.). Totalul de invitați se calculează automat.
O familie confirmă o singură dată, cu meniul potrivit pentru fiecare persoană.

Tipurile de meniu se editează în `app/components/RSVPSection.tsx` (lista
`MENU_TYPES`); dacă adaugi/scoți categorii, actualizează și `app/api/rsvp/route.ts`,
`lib/email.ts` și `app/admin/page.tsx`.

## Admin

`/admin` (parola din `ADMIN_PASSWORD`). Vezi statistici, cardul
**„Necesar meniuri (catering)"** cu totaluri pe categorii și fiecare confirmare
cu defalcarea pe meniuri.
