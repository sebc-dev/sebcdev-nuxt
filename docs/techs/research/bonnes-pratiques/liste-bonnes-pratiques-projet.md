# Guide exhaustif des bonnes pratiques - Blog technique bilingue Nuxt 4 (Décembre 2025)

Cette cartographie complète identifie **17 domaines** et **127 sous-catégories** de bonnes pratiques pour un blog technique bilingue FR/EN avec la stack Nuxt 4.2.2, TailwindCSS 4, @nuxt/content 3.10 et déploiement Cloudflare Pages. L'objectif est de permettre à Claude Code de générer du code optimal respectant les standards 2025.

---

## 1. Développement Nuxt 4 / Vue 3

### 1.1 Configuration SSG (`nuxt.config.ts`)
- **Mode génération statique** : `ssr: true` + `nitro.prerender.crawlLinks: true`
- **Route Rules** : `prerender: true` par route, `noScripts` pour pages statiques pures
- **Preset Cloudflare** : `nitro.preset: 'cloudflare_pages'`
- **Payload extraction** : `experimental.payloadExtraction: true` pour optimiser les données

### 1.2 Vue 3 Composition API
- **`<script setup lang="ts">`** comme standard obligatoire
- **Composables** : préfixe `use`, retour d'objets avec refs, `MaybeRef<T>` pour flexibilité
- **Réactivité avancée** : `shallowRef` pour grandes listes (10k+ items), `computed` avec cache intelligent
- **Auto-imports Nuxt** : Vue utilities, composables, components automatiquement disponibles

### 1.3 Hydratation & Performance
- **Lazy hydration directives** (Nuxt 3.16+) : `hydrate-on-visible`, `hydrate-on-idle`, `hydrate-on-interaction`, `hydrate-never`
- **`defineLazyHydrationComponent()`** pour contrôle programmatique
- **Server Components** : `.server.vue` pour contenu zéro-JS côté client
- **ClientOnly** : `<ClientOnly>` avec `#fallback` slot pour widgets tiers

### 1.4 Routing & Middleware
- **File-based routing** : `pages/`, paramètres `[slug].vue`, catch-all `[...slug].vue`
- **Route validation** : `definePageMeta({ validate })` pour valider paramètres
- **Middleware patterns** : global (`.global.ts`), named, anonymous inline
- **Navigation guards** : retourner `navigateTo()` ou `abortNavigation()`, jamais `next()`

### 1.5 Plugins & Modules
- **Suffixes contextuels** : `.client.ts` / `.server.ts` pour exécution ciblée
- **Parallel loading** : `parallel: true` pour plugins indépendants
- **Module development** : `defineNuxtModule()` avec `@nuxt/kit`
- **Runtime config** : `runtimeConfig.public` pour variables client-safe

---

## 2. Gestion de contenu (@nuxt/content 3.10)

### 2.1 Configuration Collections
- **Définition collections** : `defineCollection()` dans `content.config.ts`
- **Sources séparées par langue** : `source: { include: 'fr/blog/**/*.md' }`
- **Indexes SQLite** : colonnes `date`, `tags`, `draft` pour queries optimisées
- **Database mode** : `type: 'sqlite'`, `:memory:` pour serverless

### 2.2 MDC (Markdown Components)
- **Inline components** : `:badge[Beta]{type="warning"}`
- **Block components** : `::card{title="Titre"} contenu ::`
- **Named slots** : `#title`, `#description`, `#default`
- **YAML props** : block `---` pour configs complexes
- **Data binding** : `{{ $doc.title }}` dans le contenu

### 2.3 Frontmatter & Validation
- **Zod schemas** : validation typée du frontmatter
- **Champs essentiels** : `title`, `description`, `date`, `tags`, `draft`, `author`
- **SEO metadata** : intégrer `seo: { title, description, image }`
- **Property editor** : `property(z.string()).editor({ input: 'media' })` pour Studio

### 2.4 Code Blocks (Shiki)
- **Multi-thèmes** : `theme: { default: 'github-light', dark: 'github-dark' }`
- **Line highlighting** : ` ```ts {1,3-5} `
- **Filename display** : ` ```ts [nuxt.config.ts] `
- **Code groups** : `::code-group` pour alternatives pnpm/npm/yarn

### 2.5 Content Queries
- **API queryCollection** : `.all()`, `.first()`, `.path()`, `.where()`, `.order()`
- **Pagination** : `.skip()` + `.limit()` + `.count()`
- **Select fields** : `.select('path', 'title', 'date')` pour réduire payload
- **Surroundings** : `queryCollectionItemSurroundings()` pour prev/next

### 2.6 Prose Components
- **Customisation** : créer dans `components/content/ProsePre.vue`
- **Copy button** : implémenter avec `navigator.clipboard.writeText()`
- **External links** : ajouter `target="_blank"` + `rel="noopener"` automatiquement
- **Alias** : `content.renderer.alias` pour remplacer composants

---

## 3. Styling & UI (TailwindCSS 4 + shadcn-vue)

### 3.1 TailwindCSS 4 Configuration
- **CSS-first config** : `@import "tailwindcss"` + directive `@theme {}`
- **Design tokens** : `--color-*`, `--font-*`, `--spacing-*` en CSS variables
- **OKLCH colors** : espace colorimétrique moderne pour couleurs vibrantes
- **Container queries natifs** : `@container` + `@sm:`, `@md:`, `@lg:`
- **Lightning CSS** : préfixes vendeurs, transpilation automatiques

### 3.2 Design System
- **Color tokens** : `:root` + `.dark` avec variables HSL/OKLCH
- **Typography scale** : Inter/JetBrains Mono, échelle `--text-xs` à `--text-4xl`
- **Spacing sémantique** : `--spacing-page`, `--spacing-section`
- **CVA (Class Variance Authority)** : variants de composants + `twMerge`

### 3.3 shadcn-vue & Reka UI
- **Installation** : `npx shadcn-vue@latest init` + module `shadcn-nuxt`
- **Composants blog** : Dialog, Tabs, DropdownMenu, Accordion, Toast, Tooltip
- **Reka UI** : primitives accessibles WAI-ARIA compliant
- **Transpile requis** : `build.transpile: ['reka-ui', 'vee-validate']`

### 3.4 Dark Mode
- **Tailwind v4** : `@custom-variant dark (&:where(.dark, .dark *))`
- **useColorMode** : gestion light/dark/system avec persistence localStorage
- **No-flash** : script inline dans `<head>` pour éviter FOUC
- **System preference** : `prefers-color-scheme` comme défaut

### 3.5 Responsive & Animation
- **Mobile-first** : classes de base → extension `md:`, `lg:`
- **Fluid typography** : `clamp(1.5rem, 4vw, 3rem)`
- **Reduced motion** : `motion-safe:animate-*`, `motion-reduce:transition-none`
- **Vue transitions** : classes Tailwind dans `enter-active-class`, etc.

---

## 4. Accessibilité WCAG 2.2

### 4.1 Contrastes & Focus
- **Ratios AA** : 4.5:1 texte normal, 3:1 texte large, 3:1 UI
- **Focus indicators** : `:focus-visible` avec outline 2px, offset 2px
- **Double-ring technique** : outline blanc + box-shadow coloré pour contraste universel

### 4.2 Navigation Clavier
- **Tabindex logique** : ordre séquentiel naturel
- **Skip links** : lien "Aller au contenu" en premier élément
- **Focus trap** : modales avec gestion du focus (Reka UI automatique)
- **Échappement** : `Escape` pour fermer modales/popovers

### 4.3 Screen Readers
- **ARIA** : `aria-label`, `aria-labelledby`, `aria-describedby` appropriés
- **Live regions** : `aria-live="polite"` pour contenus dynamiques
- **Structure headings** : hiérarchie h1→h6 logique sans sauts
- **Alt text** : descriptions significatives sur toutes images

### 4.4 Reka UI Patterns
- **Dialogs** : `role="dialog"`, focus trap automatique, `aria-modal`
- **Menus** : navigation flèches, `role="menu"` + `menuitem`
- **Tabs** : `role="tablist"`, `tab`, `tabpanel` avec navigation ←→
- **Accordions** : `aria-expanded`, `aria-controls` automatiques

---

## 5. SEO Technique Classique

### 5.1 Meta Tags Essentiels
- **Title** : `<title>{titre} | {site_name}</title>` (50-60 chars)
- **Description** : 150-160 caractères, `useSeoMeta()`
- **Canonical** : URL absolue unique par page, `<link rel="canonical">`
- **Robots** : `index, follow` par défaut, `noindex` pour previews

### 5.2 Open Graph & Twitter Cards
- **OG essentiels** : `og:title`, `og:description`, `og:image` (1200x630px), `og:url`
- **OG article** : `article:published_time`, `article:modified_time`, `article:author`
- **Twitter Card** : `twitter:card="summary_large_image"`, image 1200x600px
- **Locales** : `og:locale`, `og:locale:alternate` pour multilingue

### 5.3 Schema.org (nuxt-schema-org)
- **WebSite** : `defineWebSite()` avec `inLanguage: ['fr-FR', 'en-US']`
- **Article/TechArticle** : `defineArticle()` avec author, dates, publisher
- **BreadcrumbList** : `defineBreadcrumb()` avec itemListElement
- **Person** : `definePerson()` pour auteur avec sameAs, knowsAbout
- **FAQ** : `defineQuestion()` pour articles avec FAQ intégrée

### 5.4 Sitemap (@nuxtjs/sitemap)
- **Auto-i18n** : `autoI18n: true` génère hreflang automatiquement
- **Dynamic sources** : `sources: ['/api/__sitemap__/urls']`
- **Image discovery** : `discoverImages: true`
- **XSL visualization** : `xsl: '/sitemap.xsl'` pour debug

---

## 6. GEO (Generative Engine Optimization)

### 6.1 llms.txt Specification
- **Format Markdown** : titre, description, sections avec liens
- **Sections structurées** : Documentation, Tutorials, Optional
- **nuxt-llms module** : génération automatique depuis config
- **llms-full.txt** : version complète avec contenu Markdown

### 6.2 Optimisation pour LLMs
- **Fact density** : statistiques, données, citations avec sources
- **Answer capsules** : réponses 40-60 mots sous chaque H2-question
- **Structure claire** : TL;DR en intro, H2 = questions utilisateurs
- **E-E-A-T** : bio auteur visible, expertise démontrée, dates mises à jour

### 6.3 Citation Optimization
- **Schema enrichi** : `citation` array, `about` avec Things, `keywords`
- **Sources référencées** : liens vers sources primaires dans contenu
- **Fraîcheur** : dates publication/mise à jour visibles et récentes
- **Contenu unique** : recherche originale, perspectives nouvelles

### 6.4 Hooks nuxt-llms
- **`llms:generate`** : modifier sections dynamiquement
- **`llms:generate:full`** : ajouter contenu au llms-full.txt
- **Integration Content** : `contentCollection: 'blog'` dans sections

---

## 7. Internationalisation (@nuxtjs/i18n 10.2)

### 7.1 Configuration Bilingue
- **Locales definition** : `{ code: 'fr', language: 'fr-FR' }` (BCP 47)
- **Strategy recommandée** : `prefix_except_default` pour SEO
- **Lazy loading** : `lazy: true` + `langDir` pour performance
- **Browser detection** : `detectBrowserLanguage` avec cookie persistence

### 7.2 Hreflang Automatique
- **useLocaleHead()** : génère hreflang links, og:locale, canonical
- **x-default** : vers locale par défaut pour crawlers
- **Sitemap intégration** : hreflang dans sitemap.xml automatique
- **Cross-language canonical** : évite contenu dupliqué

### 7.3 Content Bilingue
- **Structure miroir** : `content/fr/`, `content/en/` identiques
- **Collections séparées** : `blog_fr`, `blog_en` avec schema partagé
- **Fallback content** : requête locale défaut si traduction manquante
- **Watch locale** : `{ watch: [locale] }` dans useAsyncData

### 7.4 Language Switcher
- **useSwitchLocalePath()** : génère URLs localisées
- **Custom routes** : `pages: { about: { fr: '/a-propos', en: '/about' } }`
- **Aria labels** : `aria-label="Switch to {language}"`

---

## 8. Recherche Full-Text (MiniSearch 7.x + Nuxt Content 3)
### 8.1 Installation & Setup - Recherche Full-Text (MiniSearch 7.x + Nuxt Content 3)
- **Package** : `pnpm add minisearch`
- **Data source** : `queryCollectionSearchSections('collection')` natif Nuxt Content
- **Index creation** : côté client, dans un composable ou component
- **Fields config** : `fields: ['title', 'content']` pour indexation, `storeFields` pour retour

### 8.2 Configuration Index - Recherche Full-Text (MiniSearch 7.x + Nuxt Content 3)
- **Options clés** :
```typescript
  {
    fields: ['title', 'content'],      // Champs indexés (searchable)
    storeFields: ['title', 'id'],      // Champs retournés dans résultats
    idField: 'id',                     // Identifiant unique (défaut: 'id')
    tokenize: (text) => text.split(/\s+/),  // Custom tokenizer si besoin
  }
```
- **Boosting** : `boost: { title: 2, content: 1 }` dans searchOptions
- **Stemming FR** : via `stemmer` option (snowball-stemmers ou custom)

### 8.3 Search Options - Recherche Full-Text (MiniSearch 7.x + Nuxt Content 3)
- **Fuzzy search** : `fuzzy: 0.2` (distance Levenshtein, 0-1)
- **Prefix search** : `prefix: true` pour "auto" → "autocomplete"
- **Combinaison** : `combineWith: 'AND' | 'OR'` (défaut: OR)
- **Limite résultats** : `{ limit: 10 }` dans `.search(query, options)`
- **Filtres** : `filter: (result) => result.category === 'tutorial'`

### 8.4 Indexation Multilingue (i18n) - Recherche Full-Text (MiniSearch 7.x + Nuxt Content 3)
- **Index séparés par locale** :
```typescript
  const { locale } = useI18n()
  const { data } = await useAsyncData`search-${locale.value}`, 
    () => queryCollectionSearchSections`blog-${locale.value}`)
  )
```
- **Réindexation au changement** : `watch(locale, () => rebuildIndex())`
- **Stemmer par langue** : configurer `processTerm` selon locale
- **Collections Nuxt Content** : une collection par langue ou filtrage

### 8.5 Gestion Index Dynamique - Recherche Full-Text (MiniSearch 7.x + Nuxt Content 3)
- **Ajout** : `miniSearch.add(document)` / `addAll(documents)`
- **Suppression** : `miniSearch.remove(document)` / `removeAll()`
- **Update** : `remove()` puis `add()` (pas d'update natif)
- **Serialization** : `JSON.stringify(miniSearch)` / `MiniSearch.loadJSON()`

### 8.6 Search UX - Recherche Full-Text (MiniSearch 7.x + Nuxt Content 3)
- **Modal pattern** : Dialog shadcn-vue + CommandPalette
- **Raccourci ⌘K** : 
```typescript
  useEventListener('keydown', (e) => {
    if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
      e.preventDefault()
      isOpen.value = true
    }
  })
```
- **Debounce** : `refDebounced(query, 200)` via VueUse
- **Highlighting** : utiliser `result.match` pour positions des termes
- **Empty state** : gérer `results.length === 0` avec message localisé

### 8.7 Performance - Recherche Full-Text (MiniSearch 7.x + Nuxt Content 3)
- **Index size** : ~50% plus compact que Lunr.js
- **Lazy loading** : indexer après hydration `onMounted`)
- **Limite documents** : performant jusqu'à ~10k documents
- **Pre-build index** : possible via `JSON.stringify()` at build time

---

## 9. Validation & Type Safety (Zod 4)

### 9.1 Nouveautés Zod 4
- **Performance** : 14x plus rapide strings, bundle 57% plus petit
- **Formats top-level** : `z.email()`, `z.url()`, `z.uuidv4()`, `z.iso.date()`
- **Recursive objects** : getters natifs, plus besoin de `z.lazy()`
- **File schemas** : `z.file().min().max().mime([])`
- **JSON Schema export** : `z.toJSONSchema(schema)` natif

### 9.2 Patterns Frontmatter
- **Coercion dates** : `z.coerce.date()` pour YAML → Date
- **Enums catégories** : `z.enum(['tutorial', 'news', 'opinion'])`
- **Optional avec default** : `z.boolean().default(false)`
- **Array constraints** : `z.array(z.string()).min(1).max(10)`

### 9.3 Schema Composition
- **Extend** : `BaseSchema.extend({ newField })`
- **Merge** : `SchemaA.merge(SchemaB)`
- **Pick/Omit** : `Schema.pick({ title: true })`
- **Discriminated unions** : `z.discriminatedUnion('type', [...])`

### 9.4 Error Handling
- **safeParse()** : préférer à parse() (pas d'exception)
- **prettifyError()** : formatage user-friendly natif Zod 4
- **format()** : structure pour formulaires `{ field: { _errors: [] } }`
- **Localisation** : `z.config(z.locales.fr())`

### 9.5 TypeScript Integration
- **Type inference** : `type Article = z.infer<typeof ArticleSchema>`
- **Input vs Output** : `z.input<>` avant transform, `z.output<>` après
- **Vue props** : `defineProps<z.infer<typeof Schema>>()`
- **Error types** : `z.inferFormattedError<typeof Schema>`

---

## 10. Performance & Core Web Vitals

### 10.1 Métriques Cibles
- **LCP** : ≤ 2.5s (Largest Contentful Paint)
- **INP** : ≤ 200ms (Interaction to Next Paint, remplace FID)
- **CLS** : ≤ 0.1 (Cumulative Layout Shift)

### 10.2 Optimisation LCP
- **Preload LCP image** : `<NuxtImg preload fetchpriority="high">`
- **Formats modernes** : WebP, AVIF avec @nuxt/image
- **Critical CSS** : inline automatique Nuxt
- **Font display** : `display: 'swap'` avec @nuxt/fonts

### 10.3 Optimisation INP
- **Lazy hydration** : `hydrate-on-visible`, `hydrate-on-idle`
- **Code splitting** : dynamic imports, `defineAsyncComponent()`
- **requestIdleCallback** : tâches non-critiques
- **Event handlers** : éviter opérations synchrones lourdes

### 10.4 Optimisation CLS
- **Dimensions explicites** : `width` + `height` sur toutes images
- **Aspect-ratio** : `aspect-ratio: 16/9` pour conteneurs dynamiques
- **Placeholders LQIP** : `:placeholder` prop sur NuxtImg
- **Font preload** : éviter reflow au chargement fonts

### 10.5 Assets Optimization
- **Tree shaking** : automatique avec Vite/Rollup
- **CSS purging** : JIT TailwindCSS v4 (automatique)
- **Compression Brotli** : `nitro.compressPublicAssets.brotli: true`
- **Bundle analysis** : `build.analyze: true` pour audit

---

## 11. Déploiement Cloudflare Pages

### 11.1 Configuration Build - Déploiement Cloudflare Pages
- **Build command** : `nuxt generate` (SSG)
- **Output directory** : `.output/public`
- **Variables** : `NODE_VERSION=22`, `NODE_ENV=production`
- **Désactiver** : Rocket Loader, Mirage, Email Obfuscation

### 11.2 Headers (_headers file) - Déploiement Cloudflare Pages
- **Assets hashés** : `Cache-Control: public, max-age=31536000, immutable`
- **HTML** : `Cache-Control: public, max-age=0, must-revalidate`
- **Security** : CSP, X-Frame-Options, X-Content-Type-Options
- **Preview noindex** : `X-Robots-Tag: noindex` sur `.pages.dev`

### 11.3 Caching Strategies - Déploiement Cloudflare Pages
- **Static assets** : 1 an immutable (fingerprinted)
- **HTML SSG** : stale-while-revalidate pour fraîcheur
- **Fonts/Images** : 30 jours à 1 an selon cas
- **CDN-Cache-Control** : directive Cloudflare spécifique

### 11.4 CI/CD GitHub Actions - Déploiement Cloudflare Pages
- **pnpm cache** : `pnpm store path` + actions/cache
- **Build cache** : .nuxt, .output, node_modules/.vite
- **Deploy action** : `cloudflare/pages-action@v1`
- **Preview environments** : automatique sur chaque PR

---

## 12. Architecture Projet

### 12.1 Structure Dossiers Nuxt 4
- **`app/`** : nouveau dossier application (components, pages, composables)
- **`content/`** : articles Markdown, structure bilingue
- **`server/`** : API Nitro, middleware serveur
- **`shared/`** : code partagé app/server
- **`layers/`** : architecture extensible pour marketplace futur

### 12.2 Conventions Nommage
- **Components** : PascalCase (`ArticleCard.vue`)
- **Composables** : camelCase + `use` (`useArticle.ts`)
- **Pages** : kebab-case avec paramètres (`blog-[slug].vue`)

### 12.3 Layers Architecture
- **Base Layer** : config commune, composables génériques
- **Blog Layer** : fonctionnalités blog spécifiques
- **Marketplace Layer** (futur) : e-commerce, produits, paiements

---

## 13. Testing

### 13.1 Configuration Vitest - Testing
- **@nuxt/test-utils** : environnement Nuxt intégré
- **Projects** : séparer unit (node) et nuxt (environment nuxt)
- **Helpers** : `mountSuspended()`, `mockNuxtImport()`, `registerEndpoint()`

### 13.2 Component Testing - Testing
- **@vue/test-utils** : pour tests composants isolés
- **happy-dom** : environnement DOM léger
- **Snapshot testing** : pour composants UI stables
- **Coverage** : > 60% sur composants clés

### 13.3 E2E Testing - Testing
- **Playwright** : support officiel, cross-browser
- **@nuxt/test-utils/playwright** : helpers intégration
- **Critical paths** : navigation, search, language switch
- **Visual regression** : captures pour UI consistency

### 13.4 Content Testing - Testing
- **Frontmatter validation** : vérifier champs requis
- **Links checking** : valider URLs internes/externes
- **Schema compliance** : tester contre Zod schemas

---

## 14. Sécurité

### 14.1 nuxt-security Module - Sécurité
- **CSP** : Content-Security-Policy avec nonces/hashes
- **HSTS** : Strict-Transport-Security automatique
- **Rate limiting** : protection API endpoints
- **XSS validator** : sanitization automatique

### 14.2 Headers Sécurité - Sécurité
- **X-Content-Type-Options** : nosniff
- **X-Frame-Options** : SAMEORIGIN (anti-clickjacking)
- **Referrer-Policy** : strict-origin-when-cross-origin
- **Permissions-Policy** : désactiver caméra, micro, géoloc

### 14.3 Dependency Security - Sécurité
- **pnpm audit** : audit régulier dépendances
- **npm-check-updates** : mise à jour sécurisée
- **Renovate Bot** : automatisation PRs sécurité
- **CI audit** : `pnpm audit --audit-level=high` dans pipeline

---

## 15. Maintenabilité

### 15.1 Code Organization - Maintenabilité
- **Feature-based** : regrouper par fonctionnalité vs type
- **Composables extraction** : logique réutilisable → `composables/`
- **Component splitting** : max ~200 lignes par composant
- **Type safety** : éliminer `any`, typage strict

### 15.2 Refactoring Patterns - Maintenabilité
- **Extract composable** : répétition → composable partagé
- **Component composition** : props drilling → provide/inject ou composables
- **Lazy loading** : composants lourds → dynamic imports

### 15.3 Technical Debt - Maintenabilité
- **Registry** : `docs/TECH_DEBT.md` avec priorités
- **ADR** : documenter décisions majeures
- **Regular review** : révision trimestrielle dette technique

---

## 16. Git & Versioning

### 16.1 Conventional Commits - Git & Versioning
- **Format** : `type(scope): description`
- **Types** : feat, fix, docs, style, refactor, perf, test, build, ci, chore
- **Commitlint** : validation automatique format
- **Husky** : hooks pre-commit, commit-msg

### 16.2 Branching Strategy (Solo Dev) - Git & Versioning
- **main** : production, toujours déployable
- **develop** : intégration, PRs depuis feature branches
- **feat/**, **fix/**, **docs/** : branches courtes durée

### 16.3 Changelog Automation - Git & Versioning
- **standard-version** : génération CHANGELOG.md
- **Semantic versioning** : MAJOR.MINOR.PATCH
- **Release scripts** : `pnpm release`, `pnpm release:minor`

---

## 17. Préparation Marketplace

### 17.1 Architecture Extensible
- **Layers** : structure prête pour layer e-commerce
- **Content products** : collection `products/` dans Nuxt Content
- **API routes** : `server/api/products/`, `server/api/checkout/`

### 17.2 Payment Integration (Stripe)
- **Module** : `@unlok-co/nuxt-stripe` (client + server)
- **Runtime config** : `STRIPE_SECRET_KEY`, `STRIPE_PUBLIC_KEY`
- **Webhooks** : endpoint events Stripe sécurisé

### 17.3 Authentication Prep
- **Options légères** : nuxt-auth-utils (OAuth ready)
- **Options complètes** : @sidebase/nuxt-auth (NextAuth-like)
- **Supabase Auth** : si Supabase backend prévu

### 17.4 E-commerce Patterns
- **Cart state** : Pinia store + persistence localStorage
- **Product schema** : Zod validation avec Nuxt Content
- **Checkout flow** : API routes server-side pour sécurité

---

## Sources officielles et documentation

| Domaine | Sources principales |
|---------|---------------------|
| **Nuxt 4** | nuxt.com/docs/4.x, github.com/nuxt/nuxt |
| **Vue 3** | vuejs.org/guide, vuejs.org/api |
| **@nuxt/content** | content.nuxt.com/docs |
| **TailwindCSS 4** | tailwindcss.com/docs, tailwindcss.com/blog/tailwindcss-v4 |
| **shadcn-vue** | shadcn-vue.com, reka-ui.com/docs |
| **Zod 4** | zod.dev/v4, github.com/colinhacks/zod |
| **i18n** | i18n.nuxtjs.org/docs |
| **Pagefind** | pagefind.app/docs |
| **SEO** | nuxtseo.com, developers.google.com/search |
| **GEO** | llmstxt.org, nuxt.com/modules/llms |
| **Cloudflare** | developers.cloudflare.com/pages |
| **Web Vitals** | web.dev/vitals, web.dev/articles/vitals |
| **WCAG** | w3.org/TR/WCAG22, webaim.org |
| **Security** | nuxt-security.vercel.app, owasp.org |

---

## Checklist récapitulative pour Claude Code

Cette cartographie identifie **17 domaines majeurs** avec leurs sous-catégories de bonnes pratiques. Pour chaque génération de code, Claude Code devrait vérifier :

1. ✅ **SSG optimal** : prerender routes, lazy hydration, no-JS où possible
2. ✅ **Type safety** : Zod 4 schemas, TypeScript strict, inférence types
3. ✅ **Accessibilité** : WCAG 2.2 AA, Reka UI patterns, focus management
4. ✅ **Performance** : Core Web Vitals targets, image optimization, bundle size
5. ✅ **SEO/GEO** : Schema.org complet, llms.txt, content structuré
6. ✅ **i18n** : hreflang auto, collections bilingues, fallback content
7. ✅ **Sécurité** : CSP headers, input validation, dependency audit
8. ✅ **Maintenabilité** : conventions nommage, composables, documentation

Ce guide constitue la référence exhaustive pour générer du code optimal avec cette stack technique en décembre 2025.