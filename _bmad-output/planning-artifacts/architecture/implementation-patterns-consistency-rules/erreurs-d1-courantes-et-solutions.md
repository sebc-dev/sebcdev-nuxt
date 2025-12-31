# Erreurs D1 Courantes et Solutions

| Erreur | Cause | Solution |
|--------|-------|----------|
| "Missing Cloudflare DB binding (D1)" | Binding manquant ou mal nommé | Vérifier que `DB` est identique dans nuxt.config.ts, wrangler.toml ET Dashboard Cloudflare |
| "no such table: _content_info" | Dump non restauré | Redéployer, vérifier que le build inclut le dump |
| "binding DB must have a database that already exists" | database_id incorrect | `wrangler d1 list` pour vérifier l'UUID correct |
| Tables vides après deploy | Preview vs Production confondus | Dans Dashboard, scroller vers le haut pour vérifier l'environnement sélectionné (bug UI) |
| Erreur 500 au runtime | D1 non configuré | Ajouter `content.database.type: 'd1'` dans nuxt.config.ts |

**Bug UI Dashboard Cloudflare :** L'onglet Production/Preview scroll hors de vue. Toujours scroller vers le haut pour vérifier quel environnement est sélectionné.
