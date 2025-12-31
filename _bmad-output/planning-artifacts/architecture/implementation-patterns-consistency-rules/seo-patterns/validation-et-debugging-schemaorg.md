# Validation et Debugging Schema.org

## Outils de validation

| Outil | URL | Usage |
|-------|-----|-------|
| **Google Rich Results Test** | `search.google.com/test/rich-results` | Validation des rich snippets Google |
| **Schema Markup Validator** | `validator.schema.org` | Validation syntaxique Schema.org |
| **Nuxt DevTools** | Onglet Schema.org | Visualisation en développement |

## Endpoint de debug

En développement, accédez à `/__schema-org__/debug.json` pour visualiser le schéma JSON-LD généré.

```bash
# Exemple en développement local
curl http://localhost:3000/__schema-org__/debug.json | jq
```

## Validation conditionnelle en dev

```typescript
<script setup lang="ts">
// Afficher le schéma dans la console en développement
if (import.meta.dev) {
  const schemaData = useSchemaOrg()
  console.log('Schema.org:', JSON.stringify(schemaData, null, 2))
}
</script>
```
