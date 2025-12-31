# Hydratation Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Méthode** | Hydratation lazy native Nuxt 4 | Remplace nuxt-delay-hydration (obsolète) |
| **Composants lourds** | `hydrate-on-visible` | Hydrate quand visible dans viewport |
| **Composants interactifs** | `hydrate-on-interaction` | Hydrate sur hover/click/focus |
| **Composants différés** | `hydrate-on-idle` | Hydrate pendant idle time navigateur |
| **Composants statiques** | `hydrate-never` | Jamais hydraté - HTML statique uniquement |
| **Délai fixe** | `:hydrate-after="3000"` | Hydrate après délai en ms |
| **Conditionnel** | `:hydrate-when="condition"` | Hydrate selon condition booléenne |
| **Media query** | `hydrate-on-media-query="(max-width: 768px)"` | Hydrate selon media query |
