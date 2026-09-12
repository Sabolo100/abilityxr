# abilityxr.hu – OpenClaw Eszter legal pages

Statikus oldalak a Google OAuth verifikációhoz (OAuth consent screen / Branding).
Nincs build step, nincs függőség, nincs külső erőforrás (nincs web font, nincs analytics).

Deploy lánc: **GitHub → Coolify → Hetzner**, DNS a Rackhost-nál.

## Repo felépítés

```
public/                             ← ez a doksigyökér, ez kerül kiszolgálásra
├── index.html                      → /                        (neutrális, csak "abilityxr.hu")
└── openclaw-eszter/
    ├── index.html                  → /openclaw-eszter/
    ├── privacy/index.html          → /openclaw-eszter/privacy/
    └── terms/index.html            → /openclaw-eszter/terms/
Dockerfile                          ← nginx:alpine, port 80
nginx.conf                          ← /etc/nginx/conf.d/default.conf
```

A `README.md` és az `nginx.conf` **nincs** kiszolgálva (a `public/`-on kívül vannak) – 404-et adnak.
Mappánként `index.html`, így a záró perjeles URL-ek rewrite nélkül működnek; a perjel nélküli
`/openclaw-eszter` relatív `Location`-nal 301-el a perjeles változatra (`absolute_redirect off`,
hogy a reverse proxy mögül ne a konténer belső host:port-ja szivárogjon ki).

Healthcheck végpont: `/healthz` → `200 ok`. A `/` is 200, szóval a Coolify default
healthcheckje is jó.

## Coolify beállítás

1. **New Resource → Public Repository** → `https://github.com/Sabolo100/abilityxr`, branch `main`.
   (A repo public, szóval nem kell hozzá GitHub App integráció.)
2. **Build Pack: Dockerfile** – a repo gyökerében lévő `Dockerfile`-t találja meg magától.
3. **Ports Exposes: `80`** ← ezt át kell írni, a Coolify default `3000`, az nginx viszont 80-on hallgat.
4. **Domains: `https://www.abilityxr.hu`** – a `https://` előtaggal add meg, ettől kér
   Let's Encrypt certet a proxy.
5. **Healthcheck path: `/healthz`** (opcionális).
6. Deploy.

A cert kiállítása HTTP-01 challenge-el megy, ehhez kell:
- a DNS már a Hetzner szerverre mutasson (lásd lentebb) – **ezért a DNS-t érdemes előbb átállítani**,
- a Hetzner cloud firewall és a szerver saját tűzfala engedje a **80** és **443** portot.

## Rackhost DNS

A Hetzner szerver **IPv4 címe** kell ide (nem a szerver ID-ja), sima A rekordként:

| Típus | Név | Érték |
|---|---|---|
| A | `www` | a Hetzner szerver IPv4 címe |
| AAAA | `www` | a Hetzner szerver IPv6 címe (opcionális) |

- Ezzel a `www.abilityxr.hu` lekerül a Rackhost parkoló oldaláról a Hetzner szerverre.
- Az apex (`abilityxr.hu`) marad a Rackhost-on, amíg nem nyúlsz hozzá. Ha azt is átvinnéd,
  vedd fel az apexre is az A rekordot, és a Coolify-ban add meg második domainként
  (vagy állíts be apex → www átirányítást).
- **MX és a többi TXT rekordhoz ne nyúlj** – a levelezést nem érinti az A rekord csere.
- A Google Search Console DNS TXT igazolás is a Rackhost DNS-be megy, és független attól,
  hogy az A rekord hova mutat.

## Ellenőrzés deploy után

```bash
curl -sS -o /dev/null -w "%{http_code} %{url_effective}\n" -L https://www.abilityxr.hu/openclaw-eszter/ https://www.abilityxr.hu/openclaw-eszter/privacy/ https://www.abilityxr.hu/openclaw-eszter/terms/
```

Mindháromnak `200`-at kell adnia, valódi tartalommal (nem a Rackhost parkoló oldalával),
érvényes HTTPS certtel, login nélkül.

## Google Cloud → OAuth consent screen → Branding

```
App name:                      OpenClaw Eszter
User support email:            eszterclaw@gmail.com
Application home page:         https://www.abilityxr.hu/openclaw-eszter/
Application privacy policy:    https://www.abilityxr.hu/openclaw-eszter/privacy/
Application terms of service:  https://www.abilityxr.hu/openclaw-eszter/terms/
Authorized domain:             abilityxr.hu
Developer contact:             eszterclaw@gmail.com
```

App logo: **egyelőre nincs** – nem szükséges, és extra verification-kört hozhat.

## Sorrend

1. Rackhost DNS: `www` A rekord → Hetzner IP.
2. Coolify: repo behúzása, Ports Exposes `80`, domain `https://www.abilityxr.hu`, deploy.
3. A fenti `curl` mindhárom URL-re 200 OK, valódi tartalommal.
4. `abilityxr.hu` igazolása Google Search Console-ban DNS TXT rekorddal (azzal a
   Google-fiókkal, ami a Cloud projekt ownere).
5. Branding mezők kitöltése.
6. Csak ezután: `Testing → In production`.

## Helyi teszt

```bash
docker build -t abilityxr . && docker run --rm -p 8080:80 abilityxr
```

Aztán http://localhost:8080/openclaw-eszter/
