# Code Splitting et Chunks Vite

## Configuration manualChunks pour vendor splitting

Séparer les vendors en chunks distincts améliore le caching lors des mises à jour :

```typescript
// nuxt.config.ts
vite: {
  build: {
    cssCodeSplit: true,
    rollupOptions: {
      output: {
        manualChunks(id) {
          if (id.includes('node_modules')) {
            // Core framework séparé
            if (id.includes('vue') || id.includes('pinia')) {
              return 'vendor-core'
            }
            // UI components (shadcn-vue, Reka UI)
            if (id.includes('reka-ui') || id.includes('radix-vue')) {
              return 'vendor-ui'
            }
            // Utilitaires
            if (id.includes('@vueuse') || id.includes('clsx')) {
              return 'vendor-utils'
            }
            return 'vendor'
          }
        },
        // Nommage hash-only pour éviter blocage adblockers
        chunkFileNames: '_nuxt/[hash].js',
        entryFileNames: '_nuxt/[hash].js',
        assetFileNames: '_nuxt/[hash].[ext]'
      }
    }
  }
}
```

**Tailles cibles :** chunks de **100-250 KB** (30-80 KB gzippé), warning Vite à 500 KB.

| Chunk | Contenu | Mise à jour |
|-------|---------|-------------|
| `vendor-core` | Vue, Pinia | Rare (version majeure) |
| `vendor-ui` | Reka UI, shadcn | Occasionnel |
| `vendor-utils` | VueUse, clsx | Rare |
| `vendor` | Autres dépendances | Variable |

**Avantage caching :** Quand le code applicatif change, seuls les chunks concernés sont invalidés. Les vendors restent en cache.

## Configuration Rollup Tree Shaking

Configurer Rollup pour un tree shaking plus agressif tout en préservant les fichiers CSS :

```typescript
// nuxt.config.ts
vite: {
  build: {
    target: 'esnext',
    cssCodeSplit: true,
    rollupOptions: {
      treeshake: {
        preset: 'recommended',  // 'safest' | 'recommended' | 'smallest'
        moduleSideEffects: (id) => id.endsWith('.css'),  // Préserve les imports CSS
        propertyReadSideEffects: false,  // Élimination plus agressive
        annotations: true  // Respecte /* #__PURE__ */ et @__NO_SIDE_EFFECTS__
      },
      output: {
        minifyInternalExports: true,
        // ... manualChunks config
      }
    }
  },
  // Pre-bundle des dépendances pour accélérer le dev server
  optimizeDeps: {
    include: ['lodash-es', '@vueuse/core']
  }
}
```

**Presets Rollup treeshake :**

| Preset | Comportement | Cas d'usage |
|--------|--------------|-------------|
| `safest` | Conserve tous les side effects possibles | Dépendances legacy problématiques |
| `recommended` | Équilibre sécurité/taille (**défaut**) | ✅ **95% des projets** |
| `smallest` | Élimination agressive | Projets sans dépendances CJS |

**Options clés :**

| Option | Effet | Recommandation |
|--------|-------|----------------|
| `moduleSideEffects: (id) => id.endsWith('.css')` | Préserve les imports CSS purs | ✅ Obligatoire |
| `propertyReadSideEffects: false` | Élimine les lectures de propriétés inutilisées | ✅ Recommandé |
| `minifyInternalExports: true` | Réduit les noms d'exports internes | ✅ Recommandé |

**⚠️ Attention :** Si une dépendance casse avec `recommended`, tester d'abord avec `safest` avant de la signaler comme problématique.

## Prefetching stratégique NuxtLink

Contrôler le prefetching pour éviter la surcharge réseau :

```vue
<template>
  <!-- Default : prefetch à la visibilité (recommandé pour navigation principale) -->
  <NuxtLink to="/about">À propos</NuxtLink>

  <!-- Désactivé pour pages lourdes ou rarement visitées -->
  <NuxtLink to="/heavy-dashboard" :prefetch="false">Dashboard</NuxtLink>

  <!-- Prefetch au hover uniquement (économise bande passante) -->
  <NuxtLink to="/contact" prefetch-on="interaction">Contact</NuxtLink>
</template>
```

**Configuration globale dans nuxt.config.ts :**

```typescript
experimental: {
  defaults: {
    nuxtLink: {
      prefetchOn: {
        visibility: true,    // Prefetch quand le lien est visible
        interaction: false   // Pas de prefetch au hover (économie)
      }
    }
  }
}
```

| Stratégie | Comportement | Cas d'usage |
|-----------|--------------|-------------|
| `visibility: true` | Prefetch dès que visible | Navigation principale, liens fréquents |
| `interaction: true` | Prefetch au hover/focus | Liens secondaires |
| `prefetch: false` | Pas de prefetch | Pages lourdes, liens rares |

---
