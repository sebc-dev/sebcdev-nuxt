# Checklist Performance INP

```markdown
# Hydratation lazy
- [ ] Identifier composants below-fold → ajouter `hydrate-on-visible`
- [ ] Identifier composants interactifs non-critiques → ajouter `hydrate-on-interaction`
- [ ] Convertir composants statiques (footer, sidebar) en `.server.vue` ou `hydrate-never`
- [ ] Vérifier absence de props spreading (`v-bind="obj"`) sur composants lazy
- [ ] Composants critiques above-fold : PAS de prefix `Lazy`

# Inputs et formulaires
- [ ] Debounce sur recherche/autocomplétion (300ms + maxWait 1000ms)
- [ ] Throttle sur validation temps réel (500ms)

# Événements continus
- [ ] Throttle sur scroll/resize (100ms)
- [ ] Passive listeners pour scroll/touch/wheel (`.passive` modifier)

# Optimisation des listes
- [ ] Listes > 50 items : implémenter event delegation
- [ ] Listes > 1000 items : ajouter `v-memo`
- [ ] Listes scrollables : `.passive` sur scroll handler

# Tâches longues
- [ ] Utiliser `scheduleIdle()` pour analytics/tracking
- [ ] Yield toutes les ~50ms dans boucles longues
- [ ] Séparer UI critique du travail non-critique dans handlers
- [ ] AbortController pour annulation si nécessaire
- [ ] Indicateur de progression pour tâches > 1s

# Code Splitting
- [ ] Configurer `manualChunks` Vite pour vendor splitting
- [ ] NuxtLink : `prefetch-on="interaction"` pour liens secondaires
- [ ] NuxtLink : `:prefetch="false"` pour pages lourdes

# Cleanup
- [ ] useEventListener pour cleanup automatique
- [ ] AbortController pour fetch dans watchers
- [ ] onUnmounted pour cancelIdleCallback

# Mesure et validation
- [ ] Vérifier INP < 200ms dans DevTools Performance
- [ ] Tester avec CPU 4x throttling (simulation mobile)
- [ ] Analyser chunks avec `npx nuxi analyze`
- [ ] Plugin web-vitals avec attribution en développement
```
