# Architecture Dump/Restore D1

Nuxt Content 3 utilise une **architecture dump/restore** plutôt que des migrations traditionnelles :

```
Build Time                          Runtime (Cold Start)
    ↓                                      ↓
Parse contenu MDC                   Détection du dump
    ↓                                      ↓
Génère dump SQL                     Restauration dans D1
    ↓                                      ↓
Inclus dans .output/public          Tables _content_* créées
```

**Processus en 4 étapes :**
1. `nuxt build --preset=cloudflare_pages` parse le contenu et génère le dump
2. Le build produit `.output/public` avec le dump intégré
3. Déploiement via `wrangler pages deploy .output/public` ou Git push
4. Premier accès : Nuxt Content détecte le dump et le restaure dans D1

**Tables créées automatiquement :** `_content_info` et `_content_content` — aucune migration manuelle nécessaire.
