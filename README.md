# abilityxr.hu – OpenClaw Eszter legal pages

Statikus oldalak a Google OAuth verifikációhoz (OAuth consent screen / Branding).
Nincs build, nincs függőség, nincs külső erőforrás (nincs Google Fonts, nincs analytics) –
három önálló HTML fájl.

## Fájlok → URL-ek

| Fájl | Publikus URL |
|---|---|
| `openclaw-eszter/index.html` | https://www.abilityxr.hu/openclaw-eszter/ |
| `openclaw-eszter/privacy/index.html` | https://www.abilityxr.hu/openclaw-eszter/privacy/ |
| `openclaw-eszter/terms/index.html` | https://www.abilityxr.hu/openclaw-eszter/terms/ |

A záró `/` miatt mindegyik mappa `index.html`-ként van kitéve – így bármelyik statikus
hoston (Apache, nginx, Caddy, cPanel) a fenti URL-ek működnek átirányítás nélkül.

## Telepítés (Rackhost / cPanel)

A domain jelenleg a Rackhost parkoló oldalát adja vissza, ami minden útvonalra
200-at válaszol, de nem a lenti tartalmat. A fájlokat ki kell tenni a webtárhelyre:

1. FTP / File Manager → a webtár dokumentumgyökere (jellemzően `public_html/`).
2. Töltsd fel az `openclaw-eszter/` mappát a teljes tartalmával (alkönyvtárakkal együtt).
3. Ellenőrzés:

```bash
curl -sS -o /dev/null -w "%{http_code} %{url_effective}\n" -L https://www.abilityxr.hu/openclaw-eszter/ https://www.abilityxr.hu/openclaw-eszter/privacy/ https://www.abilityxr.hu/openclaw-eszter/terms/
```

Mindháromnak `200`-at kell adnia, login nélkül, HTTPS-en, és a valódi tartalmat kell
visszaadnia (nem a parkoló oldalt).

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

App logo: **egyelőre nincs feltöltve** – nem szükséges, és extra verification-kört hozhat.

## Hátralévő lépések

1. Fájlok kitéve, mindhárom URL 200 OK, publikus, login nélkül nyitható.
2. `abilityxr.hu` igazolása Google Search Console-ban DNS TXT rekorddal
   (ugyanazzal a Google-fiókkal, ami a Cloud projekt ownere).
3. Branding mezők kitöltése a fenti értékekkel.
4. Csak ezután: `Testing → In production` váltás.
