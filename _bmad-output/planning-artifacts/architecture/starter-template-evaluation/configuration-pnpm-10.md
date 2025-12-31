# Configuration pnpm 10

**Option 1 - package.json (Recommandée) :**
```json
{
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild", "sharp"]
  }
}
```

**Option 2 - pnpm-workspace.yaml (Recommandée pour sécurité) :**
```yaml
# pnpm-workspace.yaml - Configuration pnpm 10 avec sécurité
packages:
  - '.'

onlyBuiltDependencies:
  - esbuild
  - sharp

# ══════════════════════════════════════════════════════════════
# SÉCURITÉ SUPPLY CHAIN (pnpm 10.16+)
# ══════════════════════════════════════════════════════════════

# Délai avant installation de nouveaux packages (protection supply chain)
# 1440 minutes = 24 heures - bloque packages malveillants récents
minimumReleaseAge: 1440

# Bloque les dépendances transitives depuis git URLs ou local paths
# Empêche injection de code malveillant via subdeps exotiques
blockExoticSubdeps: true  # pnpm 10.26.0+

# ══════════════════════════════════════════════════════════════
# AUDIT - Gestion des faux positifs
# ══════════════════════════════════════════════════════════════

auditConfig:
  # CVEs à ignorer (après évaluation manuelle du risque)
  ignoreCves: []
    # - CVE-2022-36313  # Exemple : vulnerability non exploitable dans notre contexte

  # GitHub Security Advisories à ignorer
  ignoreGhsas: []
    # - GHSA-42xw-2xvc-qx8m  # Exemple : dev dependency uniquement

# Force versions non-vulnérables des dépendances transitives
overrides:
  # "lodash@<4.17.21": "^4.17.21"  # Exemple : forcer version patchée
```

**Note importante :** La syntaxe `.npmrc` avec `pnpm.onlyBuiltDependencies[]` est **incorrecte pour pnpm 10**. Utiliser `package.json` (champ `pnpm`) ou `pnpm-workspace.yaml` à la place.

**Commandes pnpm audit essentielles :**

```bash
# Audit basique (fail si high/critical)
pnpm audit --audit-level=high

# Ignorer les CVEs sans fix disponible
pnpm audit --audit-level=high --ignore-unfixable

# Générer JSON pour CI
pnpm audit --json > audit.json

# ⚠️ IMPORTANT : --fix modifie pnpm.overrides, pas le lockfile
pnpm audit --fix && pnpm install  # Les deux commandes sont requises
```
