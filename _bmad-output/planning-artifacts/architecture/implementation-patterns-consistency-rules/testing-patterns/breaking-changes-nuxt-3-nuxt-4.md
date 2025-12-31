# Breaking Changes Nuxt 3 → Nuxt 4

| Aspect | Nuxt 3 | Nuxt 4 |
|--------|--------|--------|
| **Data fetching initial** | `data.value` = `null` | `data.value` = `undefined` |
| **Noms composants** | `'MyComponent'` | `'FolderMyComponent'` (normalisé) |
| **Vitest workspace** | `vitest.workspace.ts` | `projects` dans `vitest.config.ts` |

**Mise à jour des assertions :**

```typescript
// ❌ Nuxt 3
expect(data.value).toBe(null)

// ✅ Nuxt 4
expect(data.value).toBe(undefined)
```

**Mise à jour des sélecteurs :**

```typescript
// ❌ Nuxt 3
wrapper.findComponent({ name: 'MyComponent' })

// ✅ Nuxt 4
wrapper.findComponent({ name: 'ContentMyComponent' })  // Dossier + nom
```

**Codemods disponibles :**

```bash
npx codemod@latest nuxt/4/file-structure
npx codemod@latest nuxt/4/default-data-error-value
```

---
