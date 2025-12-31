# Checklist d'implémentation

## Phase 1 : Configuration de base

- [ ] Installer les dépendances : `pnpm add minisearch`
- [ ] Optionnel stemming : `pnpm add snowball-stemmers`
- [ ] Créer `app/lib/minisearch-config.ts` avec les options
- [ ] Créer `app/lib/search-utils.ts` (removeAccents, STOP_WORDS)
- [ ] Définir les types dans `app/types/search.ts`

## Phase 2 : Génération d'index SSG

- [ ] Créer `scripts/generate-search-index.mjs`
- [ ] Ajouter `"postgenerate": "node scripts/generate-search-index.mjs"` dans package.json
- [ ] Vérifier que l'index généré est < 25 MiB (contrainte Cloudflare)
- [ ] Tester la génération : `pnpm generate && ls -la .output/public/search-index.json`

## Phase 3 : Composables client

- [ ] Créer `app/composables/useSearch.ts` avec lazy loading
- [ ] Implémenter debounce 250ms + maxWait 1000ms
- [ ] Utiliser `loadJSONAsync()` pour le parsing
- [ ] Tester le temps de chargement d'index (< 500ms)

## Phase 4 : Interface utilisateur

- [ ] Créer `app/components/search/SearchCommand.vue` (⌘K palette)
- [ ] Implémenter raccourci clavier ⌘K / Ctrl+K
- [ ] Ajouter highlighting des résultats
- [ ] Ajouter tous les attributs ARIA requis
- [ ] Tester navigation clavier (↑↓ Enter Escape)

## Phase 5 : Multilingue (si applicable)

- [ ] Implémenter gestion des élisions françaises dans tokenizer
- [ ] Ajouter cache d'index avec `Map` pour éviter rechargements
- [ ] Filtrer par locale dans la fonction de recherche
- [ ] Ajouter traductions i18n pour les messages

## Phase 6 : Tests et optimisation

- [ ] Vérifier bundle size de MiniSearch (~7KB gzipped)
- [ ] Ajouter `manualChunks` Vite pour isoler MiniSearch
- [ ] Tester sur mobile (index < 200KB recommandé)
- [ ] Valider accessibilité avec screen reader
- [ ] Profiler performance avec DevTools
