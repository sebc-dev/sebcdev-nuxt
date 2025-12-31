# Content Validation & Testing

## Pattern 17 : Script de validation pré-build

**Problème critique** : Nuxt Content 3 ne fait **pas échouer le build** sur erreurs de validation frontmatter. Les champs invalides sont silencieusement omis.

**Solution** : Script autonome exécuté avant le build avec exit code 1 sur erreur.

```typescript
// scripts/validate-content.ts
import { z } from 'zod/v4'
import matter from 'gray-matter'
import { glob } from 'glob'
import { readFileSync } from 'fs'

// Importer les schemas depuis content.config.ts ou les redéfinir
const articleSchema = z.object({
  title: z.string().min(1),
  description: z.string().max(300),
  publishedAt: z.iso.date(),
  pillar: z.enum(['ai', 'engineering', 'ux']),
  category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
  level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
  tags: z.array(z.string()).default([]),
  draft: z.boolean().default(false),
})

interface ValidationError {
  file: string
  issues: z.ZodIssue[]
}

async function validateCollection(pattern: string, schema: z.ZodSchema): Promise<ValidationError[]> {
  const files = await glob(pattern)
  const errors: ValidationError[] = []

  for (const file of files) {
    const content = readFileSync(file, 'utf-8')
    const { data: frontmatter } = matter(content)
    const result = schema.safeParse(frontmatter)

    if (!result.success) {
      errors.push({ file, issues: result.error.issues })
    }
  }

  return errors
}

async function main() {
  console.log('🔍 Validation du contenu...\n')

  const collections = [
    { pattern: 'content/fr/**/*.md', schema: articleSchema, name: 'articles_fr' },
    { pattern: 'content/en/**/*.md', schema: articleSchema, name: 'articles_en' },
  ]

  let hasErrors = false

  for (const { pattern, schema, name } of collections) {
    const errors = await validateCollection(pattern, schema)

    if (errors.length > 0) {
      hasErrors = true
      console.error(`❌ Collection "${name}" - ${errors.length} erreur(s):\n`)

      for (const { file, issues } of errors) {
        console.error(`  📄 ${file}`)
        for (const issue of issues) {
          console.error(`     → ${issue.path.join('.')}: ${issue.message}`)
        }
        console.error('')
      }
    } else {
      console.log(`✅ Collection "${name}" - valide`)
    }
  }

  if (hasErrors) {
    console.error('\n💥 Validation échouée. Corrigez les erreurs avant le build.')
    process.exit(1)
  }

  console.log('\n✨ Tout le contenu est valide!')
}

main().catch((err) => {
  console.error('Erreur inattendue:', err)
  process.exit(1)
})
```

**Scripts npm :**

```json
{
  "scripts": {
    "validate:content": "tsx scripts/validate-content.ts",
    "prebuild": "pnpm validate:content",
    "build": "nuxt build --preset=cloudflare_pages"
  }
}
```

**Installation :**

```bash
pnpm add -D tsx gray-matter glob
```

## Pattern 18 : Configuration nuxt-link-checker

Le module `nuxt-link-checker` (inclus dans `@nuxtjs/seo`) doit être configuré explicitement pour faire échouer le build :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  linkChecker: {
    // Fail le build sur liens cassés
    failOnError: true,

    // Activer pendant le build SSG
    runOnBuild: true,

    // Générer des rapports
    report: {
      html: true,
      markdown: true,
    },

    // Exclure les patterns problématiques
    exclude: [
      '/api/**',           // Routes API (pas de HTML)
      '/__nuxt_island/**', // Internals Nuxt
    ],

    // Timeout pour liens externes (ms)
    fetchTimeout: 5000,

    // Retries sur erreurs réseau
    retries: 2,
  },

  nitro: {
    prerender: {
      // REQUIS : découvre les liens à vérifier
      crawlLinks: true,
      routes: ['/'],
    },
  },
})
```

| Option | Défaut | Recommandé | Effet |
|--------|--------|------------|-------|
| `failOnError` | `false` | `true` | Bloque le build sur lien cassé |
| `runOnBuild` | `false` | `true` | Vérifie pendant `nuxt generate` |
| `fetchTimeout` | `10000` | `5000` | Timeout liens externes |

## Pattern 19 : lychee en CI pour liens externes

Complément à nuxt-link-checker pour une vérification exhaustive post-build des liens externes :

```yaml
# .github/workflows/deploy.yml
jobs:
  check-links:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: .output/public

      - name: Link Checker
        uses: lycheeverse/lychee-action@v2
        with:
          args: >-
            --verbose
            --cache
            --max-cache-age 1d
            --exclude linkedin\.com
            --exclude twitter\.com
            --exclude x\.com
            --timeout 30
            --max-retries 3
            '.output/public/**/*.html'
          fail: true
```

**Avantages lychee vs nuxt-link-checker seul :**

| Aspect | nuxt-link-checker | lychee |
|--------|-------------------|--------|
| **Vitesse** | Moyen | Très rapide (Rust) |
| **Cache** | Non | Oui (1 jour) |
| **Parallélisme** | Limité | Massif |
| **Liens externes** | Timeout fréquents | Robuste |

**Exclusions recommandées** (rate limiting agressif) :
- `linkedin.com` - bloque les bots
- `twitter.com` / `x.com` - bloque les bots
- `github.com/*/edit/*` - liens dynamiques

## Pattern 20 : markdownlint-cli2 pour MDC

**Aucun linter MDC natif n'existe**. Adapter markdownlint avec le parser `markdown-it-mdc` :

**Installation :**

```bash
pnpm add -D markdownlint-cli2 markdown-it-mdc
```

**Configuration `.markdownlint-cli2.jsonc` :**

```jsonc
{
  // Parser MDC pour comprendre la syntaxe ::component{}
  "markdownItPlugins": [["markdown-it-mdc"]],

  // Pattern frontmatter YAML
  "frontMatter": "^---\\s*$[\\s\\S]*?^---\\s*$",

  "config": {
    // Désactiver les règles incompatibles MDC
    "MD033": false,  // Allow inline HTML (composants Vue)
    "MD041": false,  // First line heading (frontmatter interfère)

    // Règles adaptées
    "MD013": {
      "line_length": 120,
      "code_blocks": false,
      "tables": false
    },

    // Headings
    "MD022": { "lines_above": 1, "lines_below": 1 },
    "MD024": { "siblings_only": true },

    // Listes
    "MD004": { "style": "dash" },
    "MD007": { "indent": 2 }
  },

  // Fichiers à linter
  "globs": ["content/**/*.{md,mdc}"],

  // Ignorer certains fichiers
  "ignores": ["content/**/drafts/**"]
}
```

**Script npm :**

```json
{
  "scripts": {
    "lint:md": "markdownlint-cli2 'content/**/*.{md,mdc}'",
    "lint:md:fix": "markdownlint-cli2 --fix 'content/**/*.{md,mdc}'"
  }
}
```

**Règles désactivées expliquées :**

| Règle | Raison désactivation |
|-------|---------------------|
| `MD033` | Composants Vue inline `<Badge>`, `<Alert>` |
| `MD041` | Frontmatter YAML avant le premier heading |

## Pattern 21 : CI séparé validate → build

Structure de jobs pour fail-fast sans gaspiller le build :

```yaml
# .github/workflows/deploy.yml
jobs:
  # ════════════════════════════════════════════════
  # VALIDATION CONTENU - Fail fast
  # ════════════════════════════════════════════════
  validate-content:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: '10'
          cache: true

      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - run: pnpm install --frozen-lockfile

      # Validation frontmatter Zod
      - run: pnpm validate:content

      # Linting markdown MDC
      - run: pnpm lint:md

  # ════════════════════════════════════════════════
  # LINT & TYPECHECK - Parallèle
  # ════════════════════════════════════════════════
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '10', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '10', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck

  # ════════════════════════════════════════════════
  # BUILD - Après validations
  # ════════════════════════════════════════════════
  build:
    needs: [validate-content, lint, typecheck]
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '10', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      # ... upload artifact, deploy
```

**Avantages :**

| Sans séparation | Avec séparation |
|-----------------|-----------------|
| Erreur contenu → build complet gaspillé | Erreur contenu → fail en ~30s |
| Feedback lent | Feedback rapide |
| CI coûteux | CI économique |

## Pattern 22 : experimental.nativeSqlite (Node 22)

Éviter les erreurs de compilation `better-sqlite3` en CI avec le module SQLite natif de Node 22 :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  content: {
    experimental: {
      // Utilise le module sqlite natif de Node.js 22+
      // Évite la compilation native de better-sqlite3
      nativeSqlite: true,
    },
  },
})
```

**Quand l'activer :**

| Contexte | Activer ? | Raison |
|----------|-----------|--------|
| Node 22 LTS | ✅ Oui | Module sqlite intégré |
| Node 20 | ❌ Non | Pas de module sqlite natif |
| CI GitHub Actions | ✅ Recommandé | Évite compilation native |
| Docker Alpine | ✅ Recommandé | Évite dépendances build |

**Note** : Cette option est expérimentale (décembre 2025). Tester en local avant d'activer en production.
