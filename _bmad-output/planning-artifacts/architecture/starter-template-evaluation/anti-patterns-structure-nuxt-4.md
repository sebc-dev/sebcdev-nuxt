# Anti-patterns Structure Nuxt 4

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| Placer `content/` dans `app/` | Collections non détectées | Garder `content/` à la racine |
| Placer `server/` dans `app/` | Routes API non détectées | Garder `server/` à la racine |
| Placer `public/` dans `app/` | Assets statiques non servis | Garder `public/` à la racine |
| Modifier `tsconfig.json` directement | Écrasé par Nuxt au build | Utiliser `alias` dans `nuxt.config.ts` |
| Composables sans préfixe `use` | Auto-import échoue | Toujours préfixer avec `use` |
| Code Vue/Nitro dans `shared/` | Erreurs d'import | Uniquement TypeScript isomorphe pur |
| Sous-dossiers dans `composables/` | Pas auto-scannés | Re-export dans `index.ts` ou `imports.dirs` |
| `tailwind.config.ts` avec TW4 | Ignoré | Utiliser `@theme` dans le CSS |
| Ignorer validation frontmatter | Erreurs runtime silencieuses | Toujours définir schémas Zod |

**Note:** L'initialisation du projet avec ces commandes sera la première story d'implémentation.