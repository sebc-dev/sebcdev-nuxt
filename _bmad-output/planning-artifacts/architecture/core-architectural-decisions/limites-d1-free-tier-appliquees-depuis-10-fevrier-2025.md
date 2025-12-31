# Limites D1 Free Tier (appliquées depuis 10 février 2025)

| Ressource | Free Tier | Workers Paid ($5/mois) |
|-----------|-----------|------------------------|
| **Rows read** | 5 millions/jour | 25 milliards/mois |
| **Rows written** | 100 000/jour | 50 millions/mois |
| **Stockage total** | 5 GB | 1 TB |
| **Taille max par DB** | **500 MB** | 10 GB |
| **Nombre de bases** | 10 | 50 000 |
| **Time Travel** | 7 jours | 30 jours |

**Ce qui compte dans le quota :**
- ✅ Requêtes depuis Workers/Pages déployés
- ✅ `wrangler d1 execute --remote`
- ✅ Requêtes via Dashboard Cloudflare
- ❌ Développement local (`wrangler dev` sans `--remote`)
- ❌ Build-time (D1 n'est pas accessible pendant le build)

**⚠️ Piège majeur** : D1 facture les **rows_read**, pas les rows_returned. Un `SELECT * FROM table` sur 5000 lignes sans index = 5000 rows_read même avec `LIMIT 1`.
