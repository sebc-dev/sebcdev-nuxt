# Cloudflare D1 - Requis pour Nuxt Content 3

**⚠️ CRITIQUE : D1 est OBLIGATOIRE avec le preset `cloudflare_pages`**

| Contexte | Accès fichiers | Base de données |
|----------|----------------|-----------------|
| Build (Node.js) | ✅ Filesystem disponible | SQLite local généré |
| Runtime (CF Worker) | ❌ Pas d'accès filesystem | D1 obligatoire |

**Pourquoi D1 est requis :**
- Le preset `cloudflare_pages` déploie un Worker (serveur) pour les routes API et le contenu dynamique
- Le runtime Worker n'a **pas accès au système de fichiers**
- Il ne peut donc pas lire le fichier SQLite généré lors du build
- Sans D1 configuré : **Erreur 500 au runtime** dès qu'une requête interroge l'API de contenu

**Configuration wrangler.toml :**

```toml
name = "sebc-dev"
compatibility_date = "2024-09-19"

[[d1_databases]]
binding = "DB"
database_name = "content-db"
database_id = "VOTRE_DATABASE_ID"  # Généré par: wrangler d1 create content-db
```

**Configuration nuxt.config.ts :**

```typescript
content: {
  database: {
    type: 'd1',
    bindingName: 'DB'  // Doit correspondre au binding dans wrangler.toml
  }
}
```

Voir [Limites Cloudflare Free Tier](#limites-cloudflare-free-tier) pour les quotas D1 et Pages.
