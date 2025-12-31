# Notes Importantes

1. **TailwindCSS 4 intégration** : `@nuxtjs/tailwindcss@6.14.0` supporte TW4, mais `@tailwindcss/vite` est recommandé pour les nouveaux projets TW4 (Config Viewer moins utile avec CSS-first).

2. **Zod 4 migration** :
   - Import API moderne : `import { z } from 'zod/v4'` (top-level validators, `z.iso.date()`, etc.)
   - Import mini : `import * as z from 'zod/mini'` (pas `@zod/mini` package)
   - Codemod automatique : `npx zod-v3-to-v4 path/to/tsconfig.json`
   - JSON Schema export natif : `z.toJSONSchema(schema)` remplace `zod-to-json-schema` (déprécié nov. 2025)
   - ⚠️ `z.date()` → `z.iso.date()` pour compatibilité JSON Schema
   - ⚠️ `.default()` s'applique maintenant DANS `.optional()` (breaking change comportemental)
   - **zod/mini syntaxe fonctionnelle** (tree-shakable) :
     ```typescript
     // Regular zod (chaining)
     z.string().min(5).max(100).trim()

     // zod/mini (functional, ~64% plus léger)
     z.string().check(z.minLength(5), z.maxLength(100), z.trim())
     ```
   - zod/mini nécessite `z.config(z.locales.en())` pour messages localisés (défaut: "Invalid input")

3. **nuxt-delay-hydration obsolète** : Hydratation lazy native depuis Nuxt 3.16+ (`hydrate-on-visible`, `hydrate-on-idle`, `hydrate-on-interaction`, `hydrate-on-media-query`, `hydrate-when`, `hydrate-never`).

4. **Reka UI est le nouveau standard** : Rebrand de Radix Vue (février 2025). shadcn-vue utilise Reka UI par défaut.

5. **Node.js 22 LTS recommandé** : Version stable pour Cloudflare Pages. Node 24 existe mais non recommandé pour CF (support souvent en retard).

6. **pnpm 10 sécurité** : Lifecycle scripts désactivés par défaut. Configuration via `package.json` (champ `pnpm`) ou `pnpm-workspace.yaml` (pas `.npmrc`).

7. **llms.txt** : Module `nuxt-llms` génère automatiquement `/llms.txt` avec @nuxt/content ^3.2.0. Remplace le server route personnalisé.

8. **@nuxt/content v3 API** : Changements majeurs par rapport à v2 :
   - ❌ Composants supprimés : `<ContentDoc>`, `<ContentList>`, `<ContentQuery>`
   - ✅ Utiliser `<ContentRenderer>` pour tout le rendu
   - ❌ `queryContent()` (v2) → ✅ `queryCollection()` (v3)
   - Mode document-driven supprimé - créer les pages manuellement
   - Composants prose personnalisés dans `components/prose/`

9. **Reading time** : 200 wpm standard, considérer 180 wpm pour contenu technique avec code.

10. **Wrangler 3.33.0+** : les commandes D1 utilisent **local par défaut**. Toujours spécifier `--local` ou `--remote` explicitement pour éviter les confusions.

11. **Wrangler 4.2.0 cassé** : Cette version a des bugs avec D1. Épingler à `"wrangler": "4.0.0"` dans package.json si problèmes.

12. **Tests Nuxt 4 avec @nuxt/test-utils** : Configuration complète dans `testing-patterns.md`. Installation :
    ```bash
    pnpm add -D @nuxt/test-utils vitest vitest-axe @vue/test-utils happy-dom @vitest/coverage-v8
    ```
    Helpers essentiels : `mountSuspended()` (composants async), `mockNuxtImport()` (auto-imports), `registerEndpoint()` (API mock).

13. **Tests a11y vitest-axe** : Pour les tests unitaires d'accessibilité des composants :
    ```typescript
    import { axe, toHaveNoViolations } from 'vitest-axe'
    expect.extend(toHaveNoViolations)

    it('composant accessible', async () => {
      const wrapper = await mountSuspended(MonComposant)
      expect(await axe(wrapper.element)).toHaveNoViolations()
    })
    ```
    Complète @axe-core/playwright (tests E2E) avec validation au niveau composant.
