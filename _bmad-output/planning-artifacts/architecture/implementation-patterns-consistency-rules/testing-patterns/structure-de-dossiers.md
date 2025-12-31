# Structure de Dossiers

Les tests restent à la **racine du projet**, pas dans `app/` :

```
project/
├── app/                     # srcDir Nuxt 4
│   ├── components/
│   ├── composables/
│   └── pages/
├── test/                    # Tests à la racine
│   ├── unit/               # Tests purs (Node) - rapides
│   │   ├── utils.test.ts
│   │   └── formatters.test.ts
│   └── nuxt/               # Tests composants (Nuxt env)
│       ├── ArticleCard.test.ts
│       └── useReadingTime.test.ts
├── vitest.config.ts
└── nuxt.config.ts
```

| Dossier | Environnement | Usage | Vitesse |
|---------|---------------|-------|---------|
| `test/unit/` | Node | Fonctions pures, utils, validations | Rapide |
| `test/nuxt/` | Nuxt | Composants, composables, pages | Plus lent |

---
