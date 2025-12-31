# E2E Testing Playwright pour Nuxt 4 SSG : Guide complet

Playwright combiné à `@nuxt/test-utils` offre une intégration native avec Nuxt 4.2.x, permettant des tests E2E robustes pour les sites SSG déployés sur Cloudflare Pages. Ce guide couvre la configuration optimisée, les patterns de test pour blogs multilingues, le visual regression testing et l'intégration CI/CD — le tout adapté à votre stack technique exact (TailwindCSS 4.1.x, @nuxtjs/i18n v10.2.1+, shadcn-vue 2.4.3+, MiniSearch 7.x, Node.js 22 LTS, pnpm 10.26+).

---

## Configuration Playwright pour Nuxt 4

### Installation et setup initial

Nuxt 4 requiert une installation spécifique avec `@nuxt/test-utils/playwright` qui fournit des fixtures personnalisées pour l'intégration native :

```bash
pnpm add -D @nuxt/test-utils @playwright/test vitest @vue/test-utils happy-dom playwright-core
npx playwright install --with-deps
```

La structure de fichiers recommandée respecte la nouvelle organisation `app/` de Nuxt 4 :

```
my-nuxt-app/
├── app/                      # Code applicatif Nuxt 4
│   ├── components/
│   ├── pages/
│   └── app.vue
├── tests/
│   ├── e2e/                  # Tests Playwright
│   │   ├── fixtures/         # Fixtures personnalisées
│   │   ├── pages/            # Page Object Models
│   │   └── *.spec.ts
│   └── __screenshots__/      # Snapshots visuels
├── playwright.config.ts
└── nuxt.config.ts
```

### playwright.config.ts optimisé pour SSG

Pour les sites SSG, deux stratégies existent : tester les fichiers statiques pré-générés (recommandé pour CI) ou utiliser le dev server. Voici la configuration production-ready :

```typescript
// playwright.config.ts
import { fileURLToPath } from 'node:url'
import { defineConfig, devices } from '@playwright/test'
import type { ConfigOptions } from '@nuxt/test-utils/playwright'

export default defineConfig<ConfigOptions>({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 2 : undefined,
  timeout: 30000,
  
  reporter: process.env.CI 
    ? [['blob'], ['github']] 
    : [['html'], ['list']],
  
  expect: {
    timeout: 5000,
    toHaveScreenshot: {
      maxDiffPixelRatio: 0.025,
      threshold: 0.2,
      animations: 'disabled',
    },
  },

  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    
    // Intégration Nuxt
    nuxt: {
      rootDir: fileURLToPath(new URL('.', import.meta.url)),
    },
  },

  // Servir les fichiers SSG pré-générés
  webServer: {
    command: 'pnpm exec serve .output/public -p 3000',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },

  projects: [
    // Desktop browsers
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    
    // Mobile testing
    { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
    { name: 'mobile-safari', use: { ...devices['iPhone 12'] } },
    
    // Theme testing
    { name: 'dark-mode', use: { ...devices['Desktop Chrome'], colorScheme: 'dark' } },
  ],
})
```

**Point critique** : L'alias `~` de Nuxt 4 pointe désormais vers `app/` et non plus la racine. Les imports comme `~/components` résolvent vers `app/components/`.

### Scripts package.json

```json
{
  "scripts": {
    "generate": "nuxt generate",
    "preview": "serve .output/public -p 3000",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:debug": "playwright test --debug",
    "test:e2e:update": "playwright test --update-snapshots",
    "test:e2e:ci": "pnpm generate && playwright test"
  }
}
```

---

## Critical paths testing pour blog multilingue

### Navigation entre pages

```typescript
// tests/e2e/navigation.spec.ts
import { expect, test } from '@nuxt/test-utils/playwright'

test.describe('Navigation', () => {
  test('navigates between main pages', async ({ page, goto }) => {
    await goto('/', { waitUntil: 'hydration' })
    
    // Utiliser les rôles ARIA pour des sélecteurs résilients
    await page.getByRole('link', { name: 'Blog' }).click()
    await expect(page).toHaveURL(/\/blog$/)
    await expect(page.getByRole('heading', { level: 1 })).toBeVisible()
  })

  test('all internal links return 200', async ({ page }) => {
    await page.goto('/')
    const links = await page.getByRole('link').all()
    
    for (const link of links) {
      const href = await link.getAttribute('href')
      if (href?.startsWith('/') && !href.includes('#')) {
        const response = await page.request.get(href)
        expect(response.status(), `${href} should return 200`).toBe(200)
      }
    }
  })

  test('SSG content is pre-rendered', async ({ page }) => {
    await page.goto('/')
    
    // Vérifier que le contenu existe dans le HTML initial (avant hydratation JS)
    const initialHTML = await page.content()
    expect(initialHTML).toContain('data-nuxt-hydration')
    
    // Attendre l'hydratation complète
    await page.waitForFunction(() => window.__NUXT__?.isHydrated === true)
  })
})
```

### Tests fonctionnalité recherche MiniSearch

MiniSearch étant client-side, il faut gérer le debounce et les temps de traitement :

```typescript
// tests/e2e/search.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Search (MiniSearch)', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/')
  })

  test('search returns relevant results', async ({ page }) => {
    // Ouvrir le dialogue de recherche (pattern Command de shadcn)
    await page.getByRole('button', { name: /search|rechercher/i }).click()
    await expect(page.getByRole('dialog')).toBeVisible()
    
    // Saisir la requête
    const searchInput = page.getByRole('combobox').or(page.getByPlaceholder(/search/i))
    await searchInput.fill('nuxt')
    
    // Attendre le debounce MiniSearch (~300ms typique)
    await page.waitForTimeout(350)
    
    // Vérifier les résultats
    const results = page.getByTestId('search-result')
    await expect(results.first()).toBeVisible()
    await expect(results.first()).toContainText(/nuxt/i)
  })

  test('clicking result navigates to article', async ({ page }) => {
    await page.getByRole('button', { name: /search/i }).click()
    await page.getByRole('combobox').fill('tutorial')
    await page.waitForTimeout(350)
    
    await page.getByTestId('search-result').first().click()
    await expect(page).toHaveURL(/\/blog\//)
  })

  test('empty results show appropriate message', async ({ page }) => {
    await page.getByRole('button', { name: /search/i }).click()
    await page.getByRole('combobox').fill('xyznonexistent123')
    await page.waitForTimeout(350)
    
    await expect(page.getByText(/no results|aucun résultat/i)).toBeVisible()
  })

  test('handles special characters gracefully', async ({ page }) => {
    await page.getByRole('button', { name: /search/i }).click()
    const searchInput = page.getByRole('combobox')
    
    const specialQueries = ['vue.js', 'C++', 'test & demo', '"exact phrase"']
    for (const query of specialQueries) {
      await searchInput.fill(query)
      await page.waitForTimeout(350)
      // Ne doit pas crash - le champ doit conserver la valeur
      await expect(searchInput).toHaveValue(query)
    }
  })
})
```

### Tests language switch i18n (prefix_except_default)

Avec la stratégie `prefix_except_default`, la locale par défaut n'a pas de préfixe URL (`/blog`) tandis que les autres en ont (`/fr/blog`) :

```typescript
// tests/e2e/i18n.spec.ts
import { test, expect } from '@playwright/test'

test.describe('i18n Language Switching', () => {
  test('default locale (en) has no URL prefix', async ({ page }) => {
    await page.goto('/')
    await expect(page).not.toHaveURL(/\/en\//)
    
    await page.goto('/blog')
    await expect(page).toHaveURL(/\/blog$/)
  })

  test('non-default locale (fr) has URL prefix', async ({ page }) => {
    await page.goto('/fr')
    await expect(page).toHaveURL(/\/fr\/?$/)
    
    await page.goto('/fr/blog')
    await expect(page).toHaveURL(/\/fr\/blog/)
  })

  test('language switcher changes locale correctly', async ({ page }) => {
    await page.goto('/blog')
    
    // Cliquer sur le switcher (dropdown shadcn pattern)
    await page.getByRole('button', { name: /language|english/i }).click()
    await page.getByRole('menuitem', { name: /français/i }).click()
    
    // L'URL doit maintenant avoir le préfixe /fr
    await expect(page).toHaveURL(/\/fr\/blog/)
  })

  test('switching to default locale removes prefix', async ({ page }) => {
    await page.goto('/fr/blog')
    
    await page.getByRole('button', { name: /langue|français/i }).click()
    await page.getByRole('menuitem', { name: /english/i }).click()
    
    await expect(page).toHaveURL(/\/blog$/)
    await expect(page).not.toHaveURL(/\/en\//)
  })

  test('page content is translated', async ({ page }) => {
    await page.goto('/')
    const englishTitle = await page.getByRole('heading', { level: 1 }).textContent()
    
    await page.goto('/fr')
    const frenchTitle = await page.getByRole('heading', { level: 1 }).textContent()
    
    expect(englishTitle).not.toBe(frenchTitle)
  })

  test('locale persists via cookie', async ({ page, context }) => {
    await page.goto('/')
    await page.getByRole('button', { name: /language/i }).click()
    await page.getByRole('menuitem', { name: /français/i }).click()
    
    const cookies = await context.cookies()
    const localeCookie = cookies.find(c => c.name === 'i18n_redirected')
    expect(localeCookie?.value).toBe('fr')
  })
})

// Fixture pour tests i18n systématiques
const translations = {
  en: { home: 'Home', blog: 'Blog', about: 'About' },
  fr: { home: 'Accueil', blog: 'Blog', about: 'À propos' },
}

test.describe('Translation verification', () => {
  for (const [locale, t] of Object.entries(translations)) {
    test(`navigation labels in ${locale}`, async ({ page }) => {
      const prefix = locale === 'en' ? '' : `/${locale}`
      await page.goto(`${prefix}/`)
      await expect(page.getByRole('link', { name: t.blog })).toBeVisible()
    })
  }
})
```

### Tests composants UI shadcn-vue/Reka UI

Les composants Radix-based (Reka UI) utilisent des portals et des états ARIA. Voici les patterns clés :

```typescript
// tests/e2e/components.spec.ts
import { test, expect } from '@playwright/test'

test.describe('UI Components (shadcn-vue/Reka UI)', () => {
  test.describe('Dialog', () => {
    test('opens, displays content, and closes', async ({ page }) => {
      await page.goto('/components')
      
      await page.getByRole('button', { name: 'Open Dialog' }).click()
      
      const dialog = page.getByRole('dialog')
      await expect(dialog).toBeVisible()
      await expect(dialog.getByRole('heading')).toBeVisible()
      
      // Fermer avec le bouton X
      await dialog.getByRole('button', { name: /close|fermer/i }).click()
      await expect(dialog).toBeHidden()
    })

    test('closes on Escape key', async ({ page }) => {
      await page.goto('/components')
      await page.getByRole('button', { name: 'Open Dialog' }).click()
      
      await expect(page.getByRole('dialog')).toBeVisible()
      await page.keyboard.press('Escape')
      await expect(page.getByRole('dialog')).toBeHidden()
    })

    test('traps focus within dialog', async ({ page }) => {
      await page.goto('/components')
      await page.getByRole('button', { name: 'Open Dialog' }).click()
      
      const dialog = page.getByRole('dialog')
      await expect(dialog).toBeVisible()
      
      // Tab plusieurs fois et vérifier que le focus reste dans le dialog
      for (let i = 0; i < 5; i++) {
        await page.keyboard.press('Tab')
      }
      
      const focusedInDialog = await page.evaluate(() => 
        document.activeElement?.closest('[role="dialog"]') !== null
      )
      expect(focusedInDialog).toBe(true)
    })
  })

  test.describe('Select/Dropdown', () => {
    test('select component opens and selects value', async ({ page }) => {
      await page.goto('/form')
      
      // shadcn Select utilise le rôle combobox
      await page.getByRole('combobox').first().click()
      
      // Le listbox est rendu dans un portal à la racine
      await expect(page.getByRole('listbox')).toBeVisible()
      
      await page.getByRole('option', { name: 'Option 2' }).click()
      await expect(page.getByRole('combobox').first()).toHaveText(/Option 2/)
    })

    test('dropdown menu with keyboard navigation', async ({ page }) => {
      await page.goto('/')
      await page.getByRole('button', { name: 'Menu' }).click()
      
      const menu = page.getByRole('menu')
      await expect(menu).toBeVisible()
      
      await page.keyboard.press('ArrowDown')
      await expect(menu.getByRole('menuitem').first()).toBeFocused()
      
      await page.keyboard.press('Enter')
      await expect(menu).toBeHidden()
    })
  })
})
```

**Piège courant** : Les composants portal (Dialog, Popover, DropdownMenu) rendent leur contenu hors du parent. Utilisez des sélecteurs ARIA globaux (`page.getByRole('dialog')`) plutôt que des sélecteurs scopés.

---

## Visual regression testing

### Configuration screenshots et comparaison

```typescript
// tests/e2e/visual.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Visual Regression', () => {
  test('homepage matches snapshot', async ({ page }) => {
    await page.goto('/')
    await page.waitForLoadState('networkidle')
    
    await expect(page).toHaveScreenshot('homepage.png', {
      fullPage: true,
      mask: [
        page.locator('[data-testid="timestamp"]'),
        page.locator('.animate-pulse'),
      ],
    })
  })

  test('blog card component', async ({ page }) => {
    await page.goto('/blog')
    
    const blogCard = page.getByTestId('blog-card').first()
    await expect(blogCard).toHaveScreenshot('blog-card.png')
  })

  test('responsive homepage', async ({ page }) => {
    const viewports = [
      { width: 1920, height: 1080, name: 'desktop' },
      { width: 768, height: 1024, name: 'tablet' },
      { width: 375, height: 667, name: 'mobile' },
    ]
    
    for (const { width, height, name } of viewports) {
      await page.setViewportSize({ width, height })
      await page.goto('/')
      await expect(page).toHaveScreenshot(`homepage-${name}.png`, {
        fullPage: true,
      })
    }
  })
})
```

### Dark mode / light mode testing

```typescript
// tests/e2e/themes.spec.ts
import { test, expect } from '@playwright/test'

// Tests light mode par défaut
test.describe('Light Mode', () => {
  test.use({ colorScheme: 'light' })
  
  test('homepage light', async ({ page }) => {
    await page.goto('/')
    await expect(page.locator('html')).not.toHaveClass(/dark/)
    await expect(page).toHaveScreenshot('homepage-light.png')
  })
})

// Tests dark mode
test.describe('Dark Mode', () => {
  test.use({ colorScheme: 'dark' })
  
  test('homepage dark', async ({ page }) => {
    await page.goto('/')
    // Tailwind dark mode class-based
    await expect(page.locator('html')).toHaveClass(/dark/)
    await expect(page).toHaveScreenshot('homepage-dark.png')
  })
})

// Test du toggle de thème
test('theme toggle switches correctly', async ({ page }) => {
  await page.goto('/')
  
  // Cliquer sur le toggle de thème
  await page.getByRole('button', { name: /theme|toggle dark/i }).click()
  await expect(page.locator('html')).toHaveClass(/dark/)
  
  await page.getByRole('button', { name: /theme|toggle light/i }).click()
  await expect(page.locator('html')).not.toHaveClass(/dark/)
})
```

### Gestion des snapshots en CI/CD

Pour éviter les différences de rendu entre environnements :

```typescript
// playwright.config.ts - section expect
expect: {
  toHaveScreenshot: {
    maxDiffPixelRatio: 0.025,
    threshold: 0.2,
    animations: 'disabled',
    caret: 'hide',
  },
},

// Utiliser un stylePath pour masquer les éléments dynamiques
use: {
  ...
},
```

```css
/* tests/e2e/screenshot.css */
iframe { visibility: hidden !important; }
.animate-spin, .animate-pulse { animation: none !important; }
[data-dynamic="true"] { opacity: 0 !important; }
video, .video-player { visibility: hidden !important; }
```

**Règle d'or** : Générez les baselines dans l'environnement CI (Docker), jamais localement, pour garantir la cohérence.

---

## Bonnes pratiques Playwright 2024-2025

### Page Object Model pour Vue/Nuxt

```typescript
// tests/e2e/pages/blog.page.ts
import { expect, type Locator, type Page } from '@playwright/test'

export class BlogPage {
  readonly page: Page
  readonly searchButton: Locator
  readonly searchDialog: Locator
  readonly searchInput: Locator
  readonly articleCards: Locator
  readonly languageSwitcher: Locator

  constructor(page: Page) {
    this.page = page
    this.searchButton = page.getByRole('button', { name: /search/i })
    this.searchDialog = page.getByRole('dialog')
    this.searchInput = page.getByRole('combobox')
    this.articleCards = page.getByRole('article')
    this.languageSwitcher = page.getByTestId('language-switcher')
  }

  async goto(locale?: string) {
    const path = locale ? `/${locale}/blog` : '/blog'
    await this.page.goto(path)
  }

  async search(query: string) {
    await this.searchButton.click()
    await expect(this.searchDialog).toBeVisible()
    await this.searchInput.fill(query)
    await this.page.waitForTimeout(350) // MiniSearch debounce
  }

  async switchLocale(locale: 'en' | 'fr') {
    await this.languageSwitcher.click()
    await this.page.getByRole('menuitem', { name: locale === 'fr' ? /français/i : /english/i }).click()
  }

  async getArticleCount(): Promise<number> {
    return this.articleCards.count()
  }
}
```

**Utilisation** :

```typescript
// tests/e2e/blog.spec.ts
import { test, expect } from '@playwright/test'
import { BlogPage } from './pages/blog.page'

test('blog search functionality', async ({ page }) => {
  const blogPage = new BlogPage(page)
  await blogPage.goto()
  await blogPage.search('nuxt')
  
  await expect(page.getByTestId('search-result')).toHaveCount({ minimum: 1 })
})
```

### Fixtures personnalisées

```typescript
// tests/e2e/fixtures/test-fixtures.ts
import { test as base, expect } from '@playwright/test'
import { BlogPage } from '../pages/blog.page'

type MyFixtures = {
  blogPage: BlogPage
  authenticatedContext: BrowserContext
}

export const test = base.extend<MyFixtures>({
  blogPage: async ({ page }, use) => {
    const blogPage = new BlogPage(page)
    await use(blogPage)
  },
  
  // Fixture pour contexte authentifié (si nécessaire)
  authenticatedContext: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: '.auth/user.json',
    })
    await use(context)
    await context.close()
  },
})

export { expect }
```

### Locators recommandés (priorité décroissante)

| Priorité | Stratégie | Exemple | Usage |
|----------|-----------|---------|-------|
| 1 | `getByRole()` | `page.getByRole('button', { name: 'Submit' })` | Éléments interactifs |
| 2 | `getByLabel()` | `page.getByLabel('Email')` | Champs de formulaire |
| 3 | `getByText()` | `page.getByText('Welcome')` | Contenu textuel |
| 4 | `getByTestId()` | `page.getByTestId('blog-card')` | Composants complexes |
| 5 | CSS selectors | `page.locator('.card')` | Dernier recours |

### Configuration parallel execution

```typescript
// playwright.config.ts
export default defineConfig({
  fullyParallel: true,
  workers: process.env.CI ? 2 : undefined,
  
  // Sharding pour CI
  // Exécuter avec: npx playwright test --shard=1/4
})
```

### Prévention des flaky tests

```typescript
// ✅ CORRECT : Utiliser les assertions web-first (auto-wait)
await expect(page.getByText('Success')).toBeVisible()
await expect(page).toHaveURL('/dashboard')

// ❌ INCORRECT : Assertions manuelles sans attente
expect(await page.getByText('Success').isVisible()).toBe(true)

// ✅ CORRECT : Attendre une réponse réseau spécifique
const responsePromise = page.waitForResponse('**/api/save')
await page.getByRole('button', { name: 'Save' }).click()
await responsePromise

// ❌ INCORRECT : Délai arbitraire
await page.waitForTimeout(5000)

// ✅ CORRECT : Mocker les APIs tierces
await page.route('**/analytics.google.com/**', route => route.abort())
await page.route('**/api/users', route => route.fulfill({
  status: 200,
  body: JSON.stringify([{ id: 1, name: 'Test User' }]),
}))
```

---

## Intégration CI/CD Cloudflare Pages

### Workflow GitHub Actions complet

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '22'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: pnpm/action-setup@v4
        with:
          version: 10
      - uses: actions/setup-node@v6
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm generate
      - uses: actions/upload-artifact@v4
        with:
          name: nuxt-build
          path: .output/public
          retention-days: 1

  deploy-preview:
    needs: build
    runs-on: ubuntu-latest
    outputs:
      preview_url: ${{ steps.deploy.outputs.deployment-url }}
    steps:
      - uses: actions/checkout@v5
      - uses: actions/download-artifact@v4
        with:
          name: nuxt-build
          path: .output/public
      - name: Deploy to Cloudflare Pages
        id: deploy
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy .output/public --project-name=my-blog --branch=${{ github.head_ref || github.ref_name }}

  test:
    needs: deploy-preview
    timeout-minutes: 60
    runs-on: ubuntu-latest
    container:
      image: mcr.microsoft.com/playwright:v1.50.0-noble
      options: --user 1001
    strategy:
      fail-fast: false
      matrix:
        shardIndex: [1, 2, 3, 4]
        shardTotal: [4]
    steps:
      - uses: actions/checkout@v5
      - uses: pnpm/action-setup@v4
        with:
          version: 10
      - uses: actions/setup-node@v6
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      
      - name: Run Playwright tests
        run: pnpm exec playwright test --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}
        env:
          BASE_URL: ${{ needs.deploy-preview.outputs.preview_url }}
          HOME: /root
      
      - uses: actions/upload-artifact@v4
        if: ${{ !cancelled() }}
        with:
          name: blob-report-${{ matrix.shardIndex }}
          path: blob-report
          retention-days: 1
      
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: test-traces-${{ matrix.shardIndex }}
          path: test-results/
          retention-days: 7

  merge-reports:
    if: ${{ !cancelled() }}
    needs: [test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: pnpm/action-setup@v4
        with:
          version: 10
      - uses: actions/setup-node@v6
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      
      - uses: actions/download-artifact@v4
        with:
          path: all-blob-reports
          pattern: blob-report-*
          merge-multiple: true
      
      - run: pnpm exec playwright merge-reports --reporter html ./all-blob-reports
      
      - uses: actions/upload-artifact@v4
        with:
          name: html-report
          path: playwright-report
          retention-days: 14
```

**Note sur le caching des browsers** : Playwright recommande officiellement d'utiliser l'image Docker plutôt que le caching manuel, car le temps de restauration du cache est comparable au temps de téléchargement.

---

## Anti-patterns à éviter

### Configuration

- ❌ Générer les snapshots visuels localement, exécuter en CI → Différences de rendu garanties
- ❌ Utiliser `workers: 1` en CI → Temps d'exécution excessif
- ❌ Omettre `forbidOnly: !!process.env.CI` → `.only` oublié bloque la CI
- ❌ `reuseExistingServer: true` en CI → Tests contre mauvais serveur possible

### Tests

- ❌ `await page.waitForTimeout(5000)` → Source de flakiness, utiliser les assertions auto-wait
- ❌ `page.locator('div.container > form > button:nth-child(2)')` → Sélecteur fragile
- ❌ `expect(await element.isVisible()).toBe(true)` → Pas d'auto-wait, utiliser `await expect(element).toBeVisible()`
- ❌ Tests qui dépendent de l'ordre d'exécution → Utiliser `test.describe.configure({ mode: 'serial' })` si vraiment nécessaire
- ❌ Partager des données entre tests parallèles → Utiliser `testInfo.parallelIndex` pour l'isolation

### shadcn-vue/Reka UI

- ❌ `await trigger.locator('..').getByRole('menu')` → Les portals ne sont pas enfants du trigger
- ❌ Tester immédiatement après `click()` sur un dialog → Attendre `await expect(dialog).toBeVisible()` pour les transitions
- ❌ Ignorer les tests d'accessibilité clavier → Les composants headless supportent la navigation clavier

---

## Checklist pour Claude Code

### Setup initial
- [ ] Installer `@nuxt/test-utils @playwright/test playwright-core`
- [ ] Créer `playwright.config.ts` avec configuration SSG
- [ ] Créer structure `tests/e2e/` avec fixtures et pages
- [ ] Ajouter scripts npm/pnpm pour les tests
- [ ] Configurer `.gitignore` pour `test-results/`, `playwright-report/`, `.auth/`

### Tests critiques à implémenter
- [ ] Navigation entre pages principales
- [ ] Fonctionnalité recherche MiniSearch
- [ ] Language switch i18n (en → fr, fr → en)
- [ ] Composants Dialog, Dropdown, Select shadcn-vue
- [ ] Visual regression homepage (light + dark)
- [ ] Tests responsive (desktop, tablet, mobile)

### CI/CD
- [ ] Créer workflow GitHub Actions avec sharding
- [ ] Configurer déploiement preview Cloudflare Pages
- [ ] Tester contre URL de preview
- [ ] Upload artifacts (traces, screenshots, report HTML)
- [ ] Merge des reports shardés

### Qualité
- [ ] Utiliser `getByRole()` comme stratégie de locator principale
- [ ] Masquer contenu dynamique dans les screenshots
- [ ] Mocker les APIs tierces (analytics, etc.)
- [ ] Configurer retries en CI (`retries: 2`)
- [ ] Activer traces sur premier retry (`trace: 'on-first-retry'`)

---

## Conclusion

L'intégration Playwright + Nuxt 4 via `@nuxt/test-utils` simplifie considérablement le setup E2E pour les sites SSG. Les points clés pour une configuration production-ready sont : utiliser l'image Docker Playwright en CI pour éliminer les problèmes de cache browsers, tester contre les fichiers statiques pré-générés plutôt que le dev server, et exploiter les fixtures personnalisées pour le POM et l'isolation des tests. Pour les blogs multilingues avec shadcn-vue, privilégiez les sélecteurs ARIA (`getByRole`) et n'oubliez pas que les composants portal rendent leur contenu hors du DOM parent. Le sharding sur 4 workers en CI avec merge des reports blob offre le meilleur équilibre performance/lisibilité.