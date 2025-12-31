# System-Level Test Design - sebc.dev

**Date:** 2025-12-31
**Author:** Negus
**Status:** Draft
**Mode:** System-Level (Phase 3 - Testability Review)

---

## Executive Summary

**Projet:** sebc.dev - Blog technique bilingue Nuxt 4 SSG
**Stack:** Nuxt 4.2 + Nuxt Content 3 + TailwindCSS 4 + shadcn-vue/Reka UI
**Déploiement:** Cloudflare Pages + D1

**Évaluation de testabilité:**
- Controllability: **PASS**
- Observability: **PASS**
- Reliability: **PASS avec recommandations**

**Recommandation gate:** **PASS** - Prêt pour implémentation avec recommandations Sprint 0

---

## 1. Testability Assessment

### 1.1 Controllability

| Critère | Status | Détails |
|---------|--------|---------|
| Contrôle de l'état système | ✅ PASS | SSG = fichiers Markdown source de vérité, D1 pour runtime |
| Dépendances mockables | ✅ PASS | Nuxt Content 3 avec composables testables, MiniSearch isolable |
| Injection de dépendances | ✅ PASS | Composables Nuxt = fonctions pures auto-importées |
| Trigger conditions d'erreur | ✅ PASS | API routes mockables via MSW, offline simulation possible |

**Points forts:**
- **SSG pur** = contenu 100% statique, reproductibilité garantie
- **MiniSearch client-side** = recherche isolée du backend, testable unitairement
- **Composables Nuxt** = logique métier extraite en fonctions testables
- **shadcn-vue** = composants copiés localement, ownership total pour tests

**Recommandations:**
1. Créer des fixtures de contenu Markdown pour tests (`test/fixtures/content/`)
2. Implémenter mock MiniSearch pour tests unitaires composables search

### 1.2 Observability

| Critère | Status | Détails |
|---------|--------|---------|
| Inspection état système | ✅ PASS | Vue DevTools, Nuxt DevTools |
| Résultats déterministes | ✅ PASS | SSG = même build → même output |
| Validation NFRs | ✅ PASS | Lighthouse CLI, Core Web Vitals API |
| Logging/Metrics | ⚠️ CONCERNS | Plausible analytics (privacy-first), pas de logging structuré |

**Points forts:**
- **SSG reproductibilité** = même input Markdown → même HTML output
- **Core Web Vitals natifs** = mesurables via Lighthouse CI
- **Cloudflare Analytics** = métriques CDN intégrées

**Recommandations:**
1. Configurer Lighthouse CI dans GitHub Actions
2. Ajouter `Server-Timing` headers pour traces performance (optional)

### 1.3 Reliability

| Critère | Status | Détails |
|---------|--------|---------|
| Tests isolés | ✅ PASS | Vitest workspaces (unit/nuxt), parallel safe |
| Reproduction des échecs | ✅ PASS | Seed data via Markdown fixtures, snapshots |
| Composants découplés | ✅ PASS | Architecture composants/composables/pages séparée |
| Cleanup discipline | ⚠️ CONCERNS | Pas de base de données runtime à nettoyer, mais D1 queries |

**Points forts:**
- **No runtime state** = SSG élimine la plupart des problèmes de state pollution
- **Vitest workspace** = séparation tests unitaires vs Nuxt environment
- **happy-dom** = environnement DOM rapide et stable

**Recommandations:**
1. Configurer `@nuxt/test-utils` avec isolation par test
2. Utiliser `mockNuxtImport` pour composables avec dépendances

---

## 2. Architecturally Significant Requirements (ASRs)

### 2.1 NFRs Critiques (extrait PRD)

| NFR | Exigence | Score Risque | Approche Test |
|-----|----------|--------------|---------------|
| NFR1 | LCP < 2.5s | 4 (2×2) | Lighthouse CI sur build |
| NFR2 | FID < 100ms | 3 (1×3) | Lighthouse CI |
| NFR3 | CLS < 0.1 | 6 (2×3) | Visual regression + Lighthouse |
| NFR5 | Lighthouse Performance ≥ 95 | 4 (2×2) | CI assertion threshold |
| NFR6 | WCAG 2.1 AA | 6 (2×3) | axe-core/playwright CI |
| NFR7 | Lighthouse Accessibility = 100 | 4 (2×2) | CI assertion threshold |
| NFR10 | Lighthouse SEO = 100 | 3 (1×3) | CI assertion threshold |
| NFR21 | HTTPS 100% | 2 (1×2) | Cloudflare auto (check deployment) |
| NFR23 | Mozilla Observatory B+ | 4 (2×2) | Headers validation test |

### 2.2 ASRs Spécifiques Architecture

| ASR | Challenge Testabilité | Score | Mitigation |
|-----|----------------------|-------|------------|
| **Cloudflare D1 obligatoire** | Runtime Worker sans filesystem | 6 (2×3) | wrangler.toml vérifié au build, smoke test deployment |
| **MiniSearch index JSON** | Index généré post-build | 4 (2×2) | Script validation post-generate |
| **Bilingue i18n** | URLs prefix_except_default | 4 (2×2) | E2E navigation cross-locale |
| **SSG + Hydration** | Client-side hydration mismatch | 4 (2×2) | happy-dom tests + E2E hydration checks |
| **shadcn-vue tooltips** | WCAG SC 1.4.13 non couvert | 6 (2×3) | Tests manuels NVDA/VoiceOver |

---

## 3. Test Levels Strategy

### 3.1 Répartition Recommandée

```
┌─────────────────────────────────────────────────────────────┐
│                    Test Pyramid sebc.dev                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│                         E2E (15%)                            │
│                     ╱─────────────╲                          │
│                   ╱   Playwright    ╲                        │
│                 ╱  User Journeys P0   ╲                      │
│               ╱─────────────────────────╲                    │
│                                                              │
│                    Component (25%)                           │
│               ╱───────────────────────────╲                  │
│             ╱   @nuxt/test-utils + Vitest   ╲                │
│           ╱   shadcn-vue components testing   ╲              │
│         ╱───────────────────────────────────────╲            │
│                                                              │
│                      Unit (60%)                              │
│       ╱───────────────────────────────────────────────╲      │
│     ╱   Vitest (node environment)                       ╲    │
│   ╱   Composables, Utils, Business Logic                  ╲  │
│ ╱───────────────────────────────────────────────────────────╲│
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Rationale par Niveau

| Niveau | % | Rationale | Outils |
|--------|---|-----------|--------|
| **Unit** | 60% | Composables purs (useReadingTime, useSearch, etc.), utils formatage | Vitest (node) |
| **Component** | 25% | Composants shadcn-vue, interactions UI, a11y assertions | @nuxt/test-utils + happy-dom |
| **E2E** | 15% | User journeys critiques, navigation i18n, search flow | Playwright |

### 3.3 Éviter la Duplication

| Comportement | Unit | Component | E2E |
|--------------|------|-----------|-----|
| Calcul temps lecture | ✅ | ❌ | ❌ |
| Filtrage articles | ✅ | ❌ | ❌ |
| Badge rendering | ❌ | ✅ | ❌ |
| ToC navigation | ❌ | ✅ | Smoke only |
| Search full flow | ❌ | ❌ | ✅ |
| Language switch | ❌ | ❌ | ✅ |
| Article reading | ❌ | ❌ | ✅ |

---

## 4. NFR Testing Approach

### 4.1 Performance (NFR1-5)

**Outil:** Lighthouse CI + PageSpeed Insights API

```yaml
# .github/workflows/lighthouse.yml
- name: Lighthouse CI
  uses: treosh/lighthouse-ci-action@v11
  with:
    urls: |
      ${{ env.PREVIEW_URL }}/
      ${{ env.PREVIEW_URL }}/articles
      ${{ env.PREVIEW_URL }}/en
    budgetPath: ./lighthouse-budget.json
    uploadArtifacts: true
```

**lighthouse-budget.json:**
```json
[{
  "resourceSizes": [
    { "resourceType": "document", "budget": 50 },
    { "resourceType": "script", "budget": 150 },
    { "resourceType": "stylesheet", "budget": 50 }
  ],
  "resourceCounts": [
    { "resourceType": "third-party", "budget": 5 }
  ]
}]
```

**Thresholds CI:**
- Performance: ≥ 90 (warning), ≥ 85 (fail)
- LCP: < 2.5s assertion
- CLS: < 0.1 assertion

### 4.2 Accessibility (NFR6-9)

**Outils:** @axe-core/playwright + Lighthouse

```typescript
// tests/a11y/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility', () => {
  test('homepage has no WCAG AA violations', async ({ page }) => {
    await page.goto('/');
    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();
    expect(results.violations).toEqual([]);
  });

  test('article page keyboard navigable', async ({ page }) => {
    await page.goto('/articles/example-article');
    // Tab through interactive elements
    await page.keyboard.press('Tab');
    await expect(page.locator(':focus')).toBeVisible();
    // Continue navigation assertions...
  });
});
```

**Limitations connues (waivers):**
- shadcn-vue tooltips: SC 1.4.13 non automatisable → tests manuels NVDA

### 4.3 SEO (NFR10-13)

**Outils:** Lighthouse + Schema.org validator + sitemap check

```typescript
// tests/seo/technical-seo.spec.ts
test('Schema.org TechArticle is valid', async ({ request }) => {
  const response = await request.get('/articles/example-article');
  const html = await response.text();

  // Extract JSON-LD
  const jsonLdMatch = html.match(/<script type="application\/ld\+json">(.*?)<\/script>/s);
  expect(jsonLdMatch).toBeTruthy();

  const schema = JSON.parse(jsonLdMatch[1]);
  expect(schema['@type']).toBe('TechArticle');
  expect(schema.author).toBeDefined();
  expect(schema.datePublished).toBeDefined();
});

test('sitemap.xml is valid', async ({ request }) => {
  const response = await request.get('/sitemap.xml');
  expect(response.status()).toBe(200);
  const xml = await response.text();
  expect(xml).toContain('<urlset');
  expect(xml).toContain('/articles/');
});

test('llms.txt exists for GEO', async ({ request }) => {
  const response = await request.get('/llms.txt');
  expect(response.status()).toBe(200);
});
```

### 4.4 Security (NFR21-23)

**Outils:** Headers check + Mozilla Observatory API

```typescript
// tests/security/headers.spec.ts
test('security headers are set correctly', async ({ request }) => {
  const response = await request.get('/');
  const headers = response.headers();

  // CSP
  expect(headers['content-security-policy']).toBeDefined();

  // X-Frame-Options
  expect(headers['x-frame-options']).toBe('DENY');

  // X-Content-Type-Options
  expect(headers['x-content-type-options']).toBe('nosniff');

  // Strict-Transport-Security
  expect(headers['strict-transport-security']).toContain('max-age=');
});
```

**Note:** Headers CSP configurés via `_headers` file Cloudflare Pages

---

## 5. Test Environment Requirements

### 5.1 Environnements

| Environnement | Usage | Configuration |
|---------------|-------|---------------|
| **Local (dev)** | Unit + Component | `npm run test` / `npm run test:nuxt` |
| **CI (GitHub Actions)** | Unit + Component + E2E | Vitest + Playwright |
| **Preview (Cloudflare)** | E2E + Lighthouse | URL preview branch |
| **Production** | Smoke tests post-deploy | URL production |

### 5.2 Infrastructure CI

```yaml
# .github/workflows/test.yml
jobs:
  unit-component:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm test:run --coverage
      - uses: codecov/codecov-action@v4

  e2e:
    runs-on: ubuntu-latest
    needs: [unit-component]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
      - run: pnpm install
      - run: npx playwright install --with-deps chromium
      - run: pnpm generate
      - run: pnpm dlx serve .output/public &
      - run: pnpm test:e2e
```

### 5.3 Dépendances Test Stack

```json
{
  "devDependencies": {
    "@nuxt/test-utils": "^3.14.0",
    "@playwright/test": "^1.49.0",
    "@axe-core/playwright": "^4.10.0",
    "vitest": "^2.1.0",
    "@vitest/coverage-v8": "^2.1.0",
    "happy-dom": "^15.0.0"
  }
}
```

---

## 6. Testability Concerns

### 6.1 Concerns Identifiés

| ID | Concern | Sévérité | Mitigation | Owner |
|----|---------|----------|------------|-------|
| TC-001 | shadcn-vue tooltips WCAG SC 1.4.13 | MEDIUM | Tests manuels screen readers | QA |
| TC-002 | D1 sans accès filesystem | LOW | wrangler.toml validation script | Dev |
| TC-003 | MiniSearch index stale | LOW | Script validation post-build | Dev |
| TC-004 | Hydration mismatch SSG | LOW | E2E smoke tests, happy-dom component tests | Dev |

### 6.2 Décision Gate

**Statut: PASS avec recommandations**

Aucun concern ne bloque l'implémentation. Les risques identifiés ont des mitigations claires.

---

## 7. Recommendations for Sprint 0

### 7.1 Test Framework Setup (Priorité Haute)

1. **Configurer Vitest workspace**
   - `test/unit/` pour composables/utils (node environment)
   - `test/nuxt/` pour composants Vue (nuxt environment + happy-dom)

2. **Installer Playwright**
   - Configuration chromium uniquement (economie CI)
   - Projet E2E dans `tests/e2e/`

3. **Configurer @axe-core/playwright**
   - Tests a11y automatisés pages critiques

### 7.2 CI Pipeline (Priorité Haute)

1. **GitHub Actions workflow**
   - Job unit/component (Vitest)
   - Job E2E (Playwright)
   - Job Lighthouse CI (sur preview URL)

2. **Coverage thresholds**
   - Composables: 90%
   - Utils: 90%
   - Components: 70%
   - Pages: 60%

### 7.3 Test Data Fixtures (Priorité Moyenne)

1. **Fixtures Markdown**
   - `test/fixtures/content/fr/articles/test-article.md`
   - Contenu minimal mais complet (frontmatter + body)

2. **Factories composables**
   - `createTestArticle()` pour unit tests
   - `createTestUser()` si auth future

### 7.4 Documentation (Priorité Basse)

1. **TESTING.md** dans repo
   - Structure tests
   - Comment lancer les tests
   - Conventions

---

## 8. Coverage Targets

| Type | Metric | Target | Rationale |
|------|--------|--------|-----------|
| **Composables** | Lines | 90% | Logique métier pure |
| **Utils** | Lines | 90% | Fonctions critiques |
| **Stores** | Lines | 85% | State management |
| **Components** | Lines | 70% | UI behavior |
| **Pages** | Lines | 60% | E2E préférable |
| **Global** | Lines | 75% | Moyenne pondérée |

---

## 9. Quality Gate Criteria

### 9.1 Pre-merge (PR)

- [ ] Unit + Component tests pass (100%)
- [ ] Coverage thresholds met
- [ ] E2E smoke tests pass
- [ ] Lighthouse Performance ≥ 85
- [ ] axe-core 0 violations WCAG AA

### 9.2 Pre-release (Production)

- [ ] Full E2E suite pass
- [ ] Lighthouse all metrics ≥ 95
- [ ] Security headers validation pass
- [ ] Schema.org validation pass
- [ ] Manual a11y check (tooltips, screen reader)

---

## Appendix

### A. Related Documents

- PRD: `_bmad-output/planning-artifacts/prd/index.md`
- Architecture: `_bmad-output/planning-artifacts/architecture/index.md`
- Epics: `_bmad-output/planning-artifacts/epics/index.md`
- Testing Patterns: `architecture/implementation-patterns-consistency-rules/testing-patterns/`

### B. Knowledge Base References

- `risk-governance.md` - Scoring matrix utilisé pour ASRs
- `nfr-criteria.md` - Critères NFR validation
- `test-levels-framework.md` - Pyramide tests
- `test-quality.md` - Definition of Done tests

---

**Generated by:** BMad TEA Agent - Test Architect Module
**Workflow:** `_bmad/bmm/workflows/testarch/test-design`
**Version:** 4.0 (BMad v6)
