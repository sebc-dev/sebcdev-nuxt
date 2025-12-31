# Stratégies de Cache Avancées Cloudflare

## Headers Cache-Control multi-niveaux

Cloudflare propose trois headers distincts pour contrôler le cache à différents niveaux :

| Header | Contrôle | Passé downstream ? |
|--------|----------|-------------------|
| `Cloudflare-CDN-Cache-Control` | Edge Cloudflare uniquement | Non |
| `CDN-Cache-Control` | Tous les CDN | Oui |
| `Cache-Control` | Navigateurs + shared caches | Oui |

**Cas d'usage pratique** : Quand le navigateur doit revalider fréquemment mais l'edge peut cacher plus longtemps :

```
# API data : browser 60s, edge 1h
/api/data.json
  Cache-Control: max-age=60
  Cloudflare-CDN-Cache-Control: max-age=3600
```

Le navigateur voit un TTL de 60 secondes ; l'edge Cloudflare cache pendant 1 heure, réduisant la charge sur l'origin.

## Pattern stale-while-revalidate pour HTML (Alternative)

Pour les sites statiques mis à jour rarement, `stale-while-revalidate` offre un excellent compromis — réponses instantanées avec vérification de fraîcheur en arrière-plan :

```
/*.html
  Cache-Control: max-age=600, stale-while-revalidate=86400
```

**Comportement :**
- **0-10 minutes** : Servi depuis le cache immédiatement (frais)
- **10 min à 24h** : Servi immédiatement (stale), revalidation en background
- **Après 24h** : Requête réseau complète obligatoire

**Trade-off** : Les utilisateurs peuvent voir du contenu jusqu'à 10 minutes obsolète au premier chargement. Acceptable pour blogs et documentation, à éviter pour e-commerce ou contenu temps-réel.

**Alternative stricte (configuration actuelle)** : `max-age=0, must-revalidate` force la revalidation à chaque requête mais exploite les ETags pour des réponses 304 (~200 bytes vs HTML complet).

## Directive `immutable` expliquée

La directive `immutable` dans `Cache-Control` empêche le navigateur de revalider même lors d'un Shift+Refresh :

```
Cache-Control: public, max-age=31536000, immutable
```

**Support navigateurs :**
- ✅ Firefox 49+
- ✅ Safari 11+
- ✅ Chrome/Edge modernes (ignoré mais inoffensif)

**Quand utiliser** : Uniquement pour les assets avec hash dans le nom de fichier (`/_nuxt/*.js`, fonts WOFF2). Le changement d'URL garantit le cache-busting automatique.

## CF-Cache-Status (Debugging)

Valeurs du header `CF-Cache-Status` retourné par Cloudflare :

| Status | Signification |
|--------|---------------|
| `HIT` | Servi depuis l'edge cache Cloudflare |
| `MISS` | Récupéré de l'origin, maintenant en cache |
| `BYPASS` | Réponse avec `no-cache` ou `private` |
| `DYNAMIC` | HTML (non caché par défaut) |
| `REVALIDATED` | Contenu stale revalidé via ETag |
| `EXPIRED` | Contenu expiré, nouvelle requête origin |
| `UPDATING` | stale-while-revalidate en cours |

**Note importante** : HTML n'est pas caché à l'edge par défaut (statut `DYNAMIC`). Pour un site SSG pur, c'est acceptable car l'origin est Cloudflare Pages lui-même (rapide).

## Validation du cache en production

Commandes curl pour vérifier le comportement du cache après déploiement :

```bash
# Vérifier les headers d'un asset fingerprinted
curl -I https://sebc.dev/_nuxt/DC5HVSK5.js

# Réponse attendue :
# cache-control: public, max-age=31536000, immutable
# cf-cache-status: HIT (après première requête)

# Vérifier les headers HTML
curl -I https://sebc.dev/

# Réponse attendue :
# cache-control: public, max-age=0, must-revalidate
# cf-cache-status: DYNAMIC

# Vérifier les fonts avec CORS
curl -I https://sebc.dev/fonts/Satoshi-Variable.woff2

# Réponse attendue :
# cache-control: public, max-age=31536000, immutable
# access-control-allow-origin: *
# cf-cache-status: HIT
```

**Impact Core Web Vitals** : Un caching correct améliore significativement le LCP pour les visiteurs récurrents — les assets fingerprinted se chargent depuis le disk cache (~1ms) au lieu du réseau (~50-200ms par ressource).

## Cache images non-fingerprinted

Pour les images dans `/public/images/` (non transformées par `@nuxt/image`) :

```
# Images statiques non-hashées - 30 jours
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
```

**Note** : Les images transformées par `@nuxt/image` dans `/_ipx/` ou `/_nuxt/` sont déjà hashées et couvertes par le cache immutable 1 an. Cette règle concerne uniquement les images statiques non-optimisées.

| Type d'image | Chemin | Cache recommandé |
|--------------|--------|------------------|
| Optimisées Nuxt | `/_ipx/*`, `/_nuxt/*.webp` | 1 an, immutable |
| Statiques | `/images/*` | 30 jours |
| Favicon | `/favicon.ico` | 1 jour (peut changer lors rebrand) |
