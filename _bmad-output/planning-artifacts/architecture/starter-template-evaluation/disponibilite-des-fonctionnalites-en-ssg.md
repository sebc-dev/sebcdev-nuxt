# Disponibilité des fonctionnalités en SSG

En mode SSG (`nuxt generate` ou `nuxt build --preset=cloudflare_pages`), certaines fonctionnalités ne sont disponibles qu'au build-time :

| Fonctionnalité | Build-time | Runtime | Notes |
|----------------|:----------:|:-------:|-------|
| Routes `server/api/` | ✅ | ❌ | Exécutées au build, résultats sauvegardés en payload |
| Pré-rendu via `useFetch` | ✅ | ❌ | Données intégrées dans le HTML généré |
| Server middleware | ✅ | ❌ | Exécuté au build uniquement |
| Génération sitemap/robots | ✅ | ✅ | Fichiers statiques générés |
| Requêtes Nuxt Content | ✅ | ✅ | Via SQLite WASM côté client (D1 non requis pour navigation) |
| Routes API dynamiques | ❌ | ❌ | **Non supporté en SSG** |
| Server-sent events | ❌ | ❌ | Requiert serveur runtime |

**Point critique** : Les appels `useFetch('/api/...')` dans les pages fonctionnent car ils s'exécutent au build-time. Les résultats sont sérialisés dans les payloads statiques. Mais ces routes API ne répondront **pas** à des requêtes runtime.
