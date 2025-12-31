# Tree Shaking et Optimisation Bundle

## Tree Shaking Composables Nuxt 4

Nuxt 4 intègre un tree shaking natif des composables via l'option `optimization.treeShake`, permettant d'éliminer automatiquement le code serveur du bundle client et inversement :

```typescript
// nuxt.config.ts
optimization: {
  treeShake: {
    composables: {
      // Composables Vue éliminés du bundle client (server-only)
      client: {
        vue: ['onServerPrefetch', 'onRenderTracked', 'onRenderTriggered'],
        '#app': ['definePayloadReducer', 'onPrehydrate']
      },
      // Composables Vue éliminés du bundle serveur (client-only)
      server: {
        vue: ['onMounted', 'onUpdated', 'onUnmounted', 'onBeforeMount',
              'onBeforeUpdate', 'onBeforeUnmount', 'onActivated', 'onDeactivated'],
        '#app': ['definePayloadReviver']
      }
    }
  }
}
```

## Bonnes Pratiques Imports ESM

Les imports nommés sont essentiels pour un tree shaking efficace. Évitez les imports namespace et les bibliothèques CommonJS :

```typescript
// ✅ Tree-shakeable - imports nommés
import { ref, computed } from 'vue'
import { debounce } from 'lodash-es'
import { Home, Settings } from 'lucide-vue-next'

// ❌ Anti-patterns bloquant le tree shaking
import * as Vue from 'vue'           // Namespace import → bundle entier
import lodash from 'lodash'          // CJS → pas de tree shaking
import { icons } from 'lucide-vue-next'  // Objet entier → toutes les icônes
```

| Pattern | Résultat | Impact bundle |
|---------|----------|---------------|
| `import { fn } from 'lib'` | ✅ Tree-shakeable | Minimal |
| `import * as lib from 'lib'` | ❌ Namespace | Bundle entier |
| `import lib from 'cjs-lib'` | ❌ CommonJS | Bundle entier |
| `import { obj } from 'lib'` puis `obj.fn` | ⚠️ Dépend | Peut inclure tout `obj` |

**Bibliothèques ESM recommandées :**

| Éviter (CJS) | Utiliser (ESM) |
|--------------|----------------|
| `lodash` | `lodash-es` |
| `date-fns` (import global) | `date-fns` (imports nommés) |
| `@fortawesome/fontawesome-free` | `lucide-vue-next` (icons individuels) |

## Seuils de Taille Bundle

Objectifs pour un blog SSG performant :

| Métrique | 🎯 Objectif | ✅ Acceptable | ⚠️ À optimiser |
|----------|-------------|---------------|----------------|
| **JS initial** (gzip) | < 100 KB | 100-200 KB | > 200 KB |
| **JS total** (gzip) | < 300 KB | 300-400 KB | > 400 KB |
| **CSS** (gzip) | < 50 KB | 50-100 KB | > 100 KB |
| **Chunk par route** | < 50 KB | 50-100 KB | > 100 KB |

**Commande d'analyse :**

```bash
# Analyse interactive du bundle
npx nuxi analyze

# Ou avec variable d'environnement
ANALYZE=true pnpm run build
```
