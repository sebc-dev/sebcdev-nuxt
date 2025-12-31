# Résumé des patterns

| Pattern | Cas d'usage | Avantage collections séparées |
|---------|-------------|-------------------------------|
| Locale dynamique typée | Requêtes multilingues | `articles_${locale}` - table isolée |
| Watch locale | Listes d'articles | Re-fetch sur collection différente |
| Fallback locale | Articles non traduits | Requête fallback explicite |
| Composable complet | Réutilisation projet-wide | Abstraction complète |
| Helper collection | DRY | Évite répétition du cast |
