# abilityxr.hu – OpenClaw legal pages

Statikus oldalak a Google OAuth verifikációhoz (OAuth consent screen / Branding).
Két külön OpenClaw példányhoz, két külön oldalkészlet ugyanazon a domainen.
Nincs build step, nincs függőség, nincs külső erőforrás (nincs web font, nincs analytics).

Deploy lánc: **GitHub → Coolify → Hetzner**, DNS a Rackhost-nál.

## Repo felépítés

```
public/                             ← ez a doksigyökér, ez kerül kiszolgálásra
├── index.html                      → /                        (neutrális, csak "abilityxr.hu")
├── openclaw-eszter/
│   ├── index.html                  → /openclaw-eszter/
│   ├── privacy/index.html          → /openclaw-eszter/privacy/
│   └── terms/index.html            → /openclaw-eszter/terms/
└── openclaw-sia/
    ├── index.html                  → /openclaw-sia/
    ├── privacy/index.html          → /openclaw-sia/privacy/
    └── terms/index.html            → /openclaw-sia/terms/
Dockerfile                          ← nginx:alpine, port 80
nginx.conf                          ← /etc/nginx/conf.d/default.conf
```

A két készlet szövegében csak az app neve és a kapcsolattartó e-mail tér el:

| | OpenClaw Eszter | OpenClaw Sia |
|---|---|---|
| Útvonal | `/openclaw-eszter/` | `/openclaw-sia/` |
| Kontakt | eszterclaw@gmail.com | sialevelezes@outlook.hu |

A `README.md` és az `nginx.conf` **nincs** kiszolgálva (a `public/`-on kívül vannak) – 404-et adnak.
Mappánként `index.html`, így a záró perjeles URL-ek rewrite nélkül működnek; a perjel nélküli
`/openclaw-sia` relatív `Location`-nal 301-el a perjeles változatra (`absolute_redirect off`,
hogy a reverse proxy mögül ne a konténer belső host:port-ja szivárogjon ki).

Healthcheck végpont: `/healthz` → `200 ok`. A `/` is 200, szóval a Coolify default
healthcheckje is jó. A domain gyökere szándékosan semleges – nem foglalja le egyik
OpenClaw példány sem.

## Coolify beállítás

1. **New Resource → Public Repository** → `https://github.com/Sabolo100/abilityxr`, branch `main`.
2. **Build Pack: Dockerfile**, Base Directory `/`, Dockerfile Location `/Dockerfile`.
3. **Ports Exposes: `80`** ← a Coolify default `3000`, az nginx viszont 80-on hallgat.
4. **Domains: `https://www.abilityxr.hu,https://abilityxr.hu`** – a `https://` előtaggal.
5. **Healthcheck path: `/healthz`** (opcionális).

Új commit után a deploy nem indul magától (public repónál nincs GitHub App integráció):
vagy kattints **Deploy**-t a Coolify-ban, vagy vedd fel az app **Webhooks** fülén lévő
deploy webhook URL-t a GitHub repo → Settings → Webhooks alá.

## Rackhost DNS

| Típus | Név | Érték |
|---|---|---|
| A | `@` | 23.88.123.18 (Hetzner szerver) |
| A | `www` | 23.88.123.18 |

Az MX és a többi TXT rekordhoz nem kell nyúlni.

## Ellenőrzés deploy után

```bash
curl -sS -o /dev/null -w "%{http_code} %{url_effective}\n" -L https://www.abilityxr.hu/openclaw-eszter/ https://www.abilityxr.hu/openclaw-eszter/privacy/ https://www.abilityxr.hu/openclaw-eszter/terms/ https://www.abilityxr.hu/openclaw-sia/ https://www.abilityxr.hu/openclaw-sia/privacy/ https://www.abilityxr.hu/openclaw-sia/terms/
```

Mind a hatnak `200`-at kell adnia, valódi tartalommal, érvényes HTTPS certtel, login nélkül.

## Google Cloud → OAuth consent screen → Branding

**OpenClaw Eszter**

```
App name:                      OpenClaw Eszter
User support email:            eszterclaw@gmail.com
Application home page:         https://www.abilityxr.hu/openclaw-eszter/
Application privacy policy:    https://www.abilityxr.hu/openclaw-eszter/privacy/
Application terms of service:  https://www.abilityxr.hu/openclaw-eszter/terms/
Authorized domain:             abilityxr.hu
Developer contact:             eszterclaw@gmail.com
```

**OpenClaw Sia**

```
App name:                      OpenClaw Sia
User support email:            sialevelezes@outlook.hu
Application home page:         https://www.abilityxr.hu/openclaw-sia/
Application privacy policy:    https://www.abilityxr.hu/openclaw-sia/privacy/
Application terms of service:  https://www.abilityxr.hu/openclaw-sia/terms/
Authorized domain:             abilityxr.hu
Developer contact:             sialevelezes@outlook.hu
```

App logo egyikhez sem – nem szükséges, és extra verification-kört hozhat.

## Search Console domain igazolás

Az `Authorized domain: abilityxr.hu` mindkét apphoz ugyanaz, de a Search Console-os
igazolás **fiókhoz kötött**, nem domainhez. Ha a két Cloud projekt két külön Google-fiók
alatt van, akkor vagy:

- a második fiókot is fel kell venni a Search Console property **ownereként**
  (Search Console → Settings → Users and permissions), vagy
- fel kell venni egy **második** `google-site-verification=...` TXT rekordot az apexre.

Több TXT rekord egyszerre nyugodtan megfér az apexen; a meglévőket ne írd felül.
A TXT rekordokat az igazolás után is hagyd bent – a Google időnként újraellenőriz.

## Sorrend

1. Rackhost DNS: `@` és `www` A rekord → Hetzner IP.
2. Coolify: deploy, mind a hat URL 200 OK.
3. `abilityxr.hu` igazolása Search Console-ban DNS TXT-vel (appenként lásd fent).
4. Branding mezők kitöltése mindkét projektben.
5. Csak ezután: `Testing → In production`.

## Helyi teszt

```bash
docker build -t abilityxr . && docker run --rm -p 8080:80 abilityxr
```

Aztán http://localhost:8080/openclaw-eszter/ és http://localhost:8080/openclaw-sia/
