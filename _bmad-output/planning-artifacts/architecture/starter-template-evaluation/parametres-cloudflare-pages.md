# Paramètres Cloudflare Pages

**À désactiver dans le dashboard Cloudflare** (causent problèmes d'hydratation) :
- ❌ Rocket Loader™
- ❌ Mirage (Image Optimization)
- ❌ Email Address Obfuscation
- ❌ Auto-minification (déprécié août 2024 → minifier au build)

**À activer manuellement pour domaines custom** :
- ✅ HTTP/3 (activé par défaut sur `*.pages.dev` uniquement)

**Configuration Build** :
- Framework preset: `Nuxt.js`
- Build command: `pnpm run build`
- Build output directory: `.output/public`
- Node version: `22 LTS` (version stable recommandée)

**Avantages vérifiés (tier gratuit)** :
- Bande passante illimitée (pas de frais d'egress)
- Early Hints activé par défaut → **+30% LCP** pour nouveaux visiteurs

## Fichier public/_headers

Configuration cache, sécurité et Early Hints optimisée pour Cloudflare Pages :

```
# public/_headers

# ═══════════════════════════════════════════════════════════════════
# SÉCURITÉ GLOBALE
# ═══════════════════════════════════════════════════════════════════
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  X-XSS-Protection: 0
  Referrer-Policy: strict-origin-when-cross-origin
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=(), bluetooth=(), accelerometer=(), gyroscope=()
  Cross-Origin-Opener-Policy: same-origin-allow-popups
  Cross-Origin-Resource-Policy: same-origin
  Content-Security-Policy: default-src 'self'; script-src 'self' 'strict-dynamic'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob: https:; font-src 'self' data:; connect-src 'self'; frame-ancestors 'none'; base-uri 'none'; form-action 'self'; object-src 'none'; upgrade-insecure-requests

# ═══════════════════════════════════════════════════════════════════
# HTML STATIQUES - Cache court avec revalidation
# ═══════════════════════════════════════════════════════════════════
/*.html
  Cache-Control: public, max-age=0, must-revalidate

/
  Cache-Control: public, max-age=0, must-revalidate

# ═══════════════════════════════════════════════════════════════════
# ASSETS JS/CSS HASHÉS - Cache agressif immutable (1 an)
# ═══════════════════════════════════════════════════════════════════
/_nuxt/*.js
  Cache-Control: public, max-age=31536000, immutable

/_nuxt/*.css
  Cache-Control: public, max-age=31536000, immutable

# ═══════════════════════════════════════════════════════════════════
# IMAGES OPTIMISÉES - Immutable car hashées
# ═══════════════════════════════════════════════════════════════════
/_nuxt/*.webp
  Cache-Control: public, max-age=31536000, immutable

/_nuxt/*.avif
  Cache-Control: public, max-age=31536000, immutable

# ═══════════════════════════════════════════════════════════════════
# IMAGES NON-FINGERPRINTED - 30 jours
# ═══════════════════════════════════════════════════════════════════
/images/*
  Cache-Control: public, max-age=2592000

/*.webp
  Cache-Control: public, max-age=2592000

/*.avif
  Cache-Control: public, max-age=2592000

/*.svg
  Cache-Control: public, max-age=2592000

/*.png
  Cache-Control: public, max-age=2592000

/*.jpg
  Cache-Control: public, max-age=2592000

/favicon.ico
  Cache-Control: public, max-age=86400

# ═══════════════════════════════════════════════════════════════════
# FONTS - CORS obligatoire même en self-hosting
# ═══════════════════════════════════════════════════════════════════
/fonts/*
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

/*.woff2
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

# ═══════════════════════════════════════════════════════════════════
# EARLY HINTS (103) - Preload ressources critiques pour LCP
# Active uniquement si Early Hints activé dans CF Dashboard
# ═══════════════════════════════════════════════════════════════════
/
  Link: </fonts/Satoshi-Variable.woff2>; rel=preload; as=font; type=font/woff2; crossorigin

# ═══════════════════════════════════════════════════════════════════
# SEO & CRAWLERS
# ═══════════════════════════════════════════════════════════════════
/sitemap.xml
  Cache-Control: public, max-age=3600

/robots.txt
  Cache-Control: public, max-age=3600

# llms.txt pour assistants IA (nuxt-llms)
/llms.txt
  Content-Type: text/plain; charset=utf-8
  Cache-Control: public, max-age=3600
  X-Robots-Tag: noindex

/llms-full.txt
  Content-Type: text/plain; charset=utf-8
  Cache-Control: public, max-age=3600
  X-Robots-Tag: noindex

# ═══════════════════════════════════════════════════════════════════
# PREVIEW DEPLOYMENTS - Bloquer indexation
# ═══════════════════════════════════════════════════════════════════
https://:project.pages.dev/*
  X-Robots-Tag: noindex

https://*.:project.pages.dev/*
  X-Robots-Tag: noindex
```

**Security Headers expliqués :**

| Header | Fonction | Valeur |
|--------|----------|--------|
| `X-Content-Type-Options` | Empêche MIME-sniffing | `nosniff` |
| `X-Frame-Options` | Protection clickjacking | `DENY` (bloque tout iframe) |
| `X-XSS-Protection` | Désactive filtre XSS navigateur (déprécié, peut introduire failles) | `0` |
| `Referrer-Policy` | Contrôle info envoyée en referer | `strict-origin-when-cross-origin` |
| `Strict-Transport-Security` | Force HTTPS (2 ans) | `max-age=63072000; includeSubDomains; preload` |
| `Permissions-Policy` | Désactive APIs sensibles non utilisées | Liste des APIs désactivées |
| `Cross-Origin-Opener-Policy` | Protection cross-origin | `same-origin-allow-popups` (OAuth compatible) |
| `Cross-Origin-Resource-Policy` | Protège ressources d'embedding externe | `same-origin` |
| `Cross-Origin-Embedder-Policy` | Isolation cross-origin | `unsafe-none` (défaut recommandé SSG) |
| `Content-Security-Policy` | Protection XSS majeure | Voir détail ci-dessous |

**Notes importantes :**
- **`interest-cohort=()` obsolète** : Google a abandonné FLoC en 2022. Chrome v109+ ignore cette directive Permissions-Policy — ne pas l'inclure.
- **COEP `unsafe-none`** : Utiliser `require-corp` uniquement si SharedArrayBuffer nécessaire (jeux, audio processing). Pour un blog SSG, `unsafe-none` évite de casser les embeds externes.

**CSP (Content-Security-Policy) détaillé :**

| Directive | Valeur | Explication |
|-----------|--------|-------------|
| `default-src` | `'self'` | Défaut restrictif |
| `script-src` | `'self' 'strict-dynamic'` | Scripts locaux + chargement en chaîne |
| `style-src` | `'self' 'unsafe-inline'` | ⚠️ `unsafe-inline` requis pour shadcn-vue/Reka UI |
| `img-src` | `'self' data: blob: https:` | Images locales, data URIs, blobs, HTTPS externes |
| `font-src` | `'self' data:` | Fonts self-hosted uniquement |
| `connect-src` | `'self'` | XHR/Fetch locaux uniquement |
| `frame-ancestors` | `'none'` | Équivalent moderne de X-Frame-Options DENY |
| `base-uri` | `'none'` | Protection base tag injection |
| `form-action` | `'self'` | Formulaires soumis localement uniquement |
| `object-src` | `'none'` | Bloque plugins (Flash, etc.) |
| `upgrade-insecure-requests` | activé | Force HTTPS pour ressources HTTP |

**⚠️ Limitation CSP avec shadcn-vue :**

Reka UI (primitives de shadcn-vue) utilise des styles inline pour le positionnement, nécessitant `style-src 'unsafe-inline'`. Composants affectés :

| Composant | CSP Compatible | Raison |
|-----------|---------------|--------|
| Button, Input, Card | ✅ Oui | Classes CSS uniquement |
| Toast/Sonner | ❌ Non | Styles inline positionnement |
| Dialog, Sheet | ❌ Non | Overlay positioning |
| ScrollArea | ❌ Non | Comportement scroll Reka UI |
| Tabs | ⚠️ Partiel | Positionnement indicateur |

**Autres headers :**

| Header | Fonction | Impact |
|--------|----------|--------|
| `Link: rel=preload` | Early Hints 103 pour fonts | LCP -30% |
| `Access-Control-Allow-Origin: *` | CORS pour fonts cross-origin | Obligatoire |
| `immutable` | Jamais revalidé par le CDN | Performance |

**⚠️ Notes importantes :**
- **HSTS preload** : Ne pas activer si pas prêt à s'engager HTTPS sur tous les sous-domaines (retrait prend des mois). Soumettre à hstspreload.org après déploiement.
- **HSTS + Cloudflare** : Si HSTS activé dans CF Dashboard (SSL/TLS → Edge Certificates), il écrase le header origin. Configurer à un seul endroit.
- **Early Hints** : Doit être activé dans Cloudflare Dashboard (Speed → Optimization → Protocol Optimization)
- **CORS fonts** : `crossorigin` obligatoire même en self-hosting pour preload
- **Preview indexation** : Remplacer `:project` par le nom réel du projet CF Pages
- **CSP testing** : Utiliser `Content-Security-Policy-Report-Only` en dev pour identifier les violations avant d'enforcer
