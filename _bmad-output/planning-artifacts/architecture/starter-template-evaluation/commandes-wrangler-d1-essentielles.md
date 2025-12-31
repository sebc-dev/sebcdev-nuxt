# Commandes Wrangler D1 Essentielles

**Diagnostic :**
```bash
# Lister toutes les bases D1 du compte
wrangler d1 list

# Infos sur une base spécifique
wrangler d1 info content-db

# Vérifier les tables locales
wrangler d1 execute DB --local --command "SELECT name FROM sqlite_master WHERE type='table'"

# Vérifier les tables en production
wrangler d1 execute DB --remote --command "SELECT * FROM _content_info LIMIT 5"
```

**Backup et Time Travel :**
```bash
# Exporter pour backup
wrangler d1 export DB --remote --output backup.sql

# Time Travel (récupération point-in-time, 7 jours free tier)
wrangler d1 time-travel info DB --timestamp "2025-12-25T10:00:00Z"
```

**Localisation des données locales :**
```
.wrangler/state/v3/d1/miniflare-D1DatabaseObject/*.sqlite
```
Inspectable avec n'importe quel client SQLite ou l'extension VS Code SQLite.
