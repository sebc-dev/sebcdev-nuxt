# Plugin Suffixes & Context

**Suffixes contextuels pour contrôle d'exécution :**

| Suffixe | Bundle inclus | Exécution | Cas d'usage |
|---------|---------------|-----------|-------------|
| `.client.ts` | Client uniquement | Après hydratation | Analytics, DOM, localStorage, animations |
| `.server.ts` | Serveur uniquement | SSR/SSG | Secrets, APIs Node.js, bindings Cloudflare |
| Sans suffixe | Client + Serveur | Universel | Helpers, composables génériques |

**Règle critique :** Les APIs navigateur (`window`, `document`, `localStorage`) doivent être dans des fichiers `.client.ts` ou protégées par `if (import.meta.client)`.

```typescript
// app/plugins/analytics.client.ts
// ✅ Plugin client-only - exclu du bundle serveur
export default defineNuxtPlugin({
  name: 'analytics',
  parallel: true,  // Ne bloque pas les autres plugins
  setup(nuxtApp) {
    const config = useRuntimeConfig()
    // APIs navigateur sécurisées ici
  }
})
```

**Pattern `dependsOn` pour plugins interdépendants :**

```typescript
// app/plugins/02.data-consumer.ts
export default defineNuxtPlugin({
  name: 'data-consumer',
  dependsOn: ['data-provider'],  // Attend que data-provider soit terminé
  parallel: true,                // Reste parallèle aux autres plugins
  async setup(nuxtApp) {
    const { $dataProvider } = nuxtApp
    // Accès garanti au provider
  }
})
```

**Contraintes statiques :** Les propriétés `name`, `parallel`, `enforce` et `dependsOn` doivent être **statiques**. Définir dynamiquement (ex: `parallel: import.meta.server`) empêche les optimisations de build.
