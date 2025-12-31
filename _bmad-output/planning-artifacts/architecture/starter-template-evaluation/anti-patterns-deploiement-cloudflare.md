# Anti-patterns Déploiement Cloudflare

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| `ssr: false` | Pages vides, pas de prerendering | Garder `ssr: true` (défaut) |
| Cacher `node_modules` avec pnpm | Cache invalide, builds lents | Cacher le pnpm store à la place |
| Fallback SPA `/* /index.html 200` | Inutile pour SSG (chaque route a son HTML) | Ne pas ajouter |
| `NITRO_PRESET` en variable d'environnement | Détecté automatiquement | Ne pas définir |
| Oublier de désactiver Rocket Loader | Erreurs d'hydratation Vue | Désactiver dans CF Dashboard |
| Pas de dimensions sur images | CLS élevé | Toujours `width` + `height` |
| `wrangler.toml` sans binding D1 | Erreur 500 au runtime | Configurer `[[d1_databases]]` |
