# Security Patterns

## Sanitisation HTML (DOMPurify)

### Composable useSanitize

Pour le contenu utilisateur ou Markdown converti en HTML, toujours sanitiser **APRÈS** la conversion Markdown→HTML (règle OWASP).

```typescript
// app/composables/useSanitize.ts
import DOMPurify from 'isomorphic-dompurify'

export function useSanitize() {
  return {
    /**
     * Sanitise le HTML en supprimant les balises et attributs dangereux
     * @param dirty - HTML potentiellement malveillant
     * @returns HTML nettoyé
     */
    sanitizeHTML: (dirty: string) => DOMPurify.sanitize(dirty, {
      ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong', 'a', 'ul', 'ol', 'li',
                     'h1', 'h2', 'h3', 'h4', 'code', 'pre', 'blockquote'],
      ALLOWED_ATTR: ['href', 'title', 'class', 'id'],
      FORBID_TAGS: ['base', 'script', 'iframe', 'form', 'object', 'embed', 'style'],
      FORBID_ATTR: ['onerror', 'onload', 'onclick', 'onmouseover']
    }),

    /**
     * Sanitise et extrait uniquement le texte (sans balises HTML)
     * @param dirty - HTML potentiellement malveillant
     * @returns Texte brut
     */
    sanitizeToText: (dirty: string) => {
      const clean = DOMPurify.sanitize(dirty, { ALLOWED_TAGS: [] })
      return clean.trim()
    }
  }
}
```

**Installation :**

```bash
pnpm add isomorphic-dompurify
pnpm add -D @types/dompurify
```

### Usage dans un composant

```vue
<script setup lang="ts">
const { sanitizeHTML } = useSanitize()

const props = defineProps<{
  htmlContent: string
}>()

const safeHTML = computed(() => sanitizeHTML(props.htmlContent))
</script>

<template>
  <div v-html="safeHTML" />
</template>
```

### Quand NE PAS sanitiser

| Source | Sanitisation | Raison |
|--------|--------------|--------|
| Nuxt Content MDC | ❌ Non requise | MDC compile le Markdown de manière sécurisée |
| Contenu CMS externe | ✅ Requise | Source non contrôlée |
| Input utilisateur | ✅ Requise | Jamais faire confiance |
| Fichiers Markdown locaux | ❌ Non requise | Contrôle total du contenu |

## Scripts Externes avec strict-dynamic

### useScript pour analytics et widgets

Le composable `useScript` de Nuxt (via `@unhead/vue`) charge les scripts de manière compatible avec CSP `strict-dynamic`.

```typescript
// app/composables/useAnalytics.ts
export function useAnalytics() {
  const { $script } = useScript('https://analytics.example.com/script.js', {
    trigger: 'idle',  // Charge pendant idle time navigateur
    defer: true
  })

  return {
    trackEvent: (name: string, data?: Record<string, unknown>) => {
      if ($script.value?.loaded) {
        window.analytics?.track(name, data)
      }
    }
  }
}
```

### Scripts avec integrity explicite (SSG)

Pour les scripts externes sans support strict-dynamic, utiliser `useHead` avec `integrity` :

```typescript
// app/plugins/external-widget.client.ts
export default defineNuxtPlugin(() => {
  useHead({
    script: [{
      src: 'https://cdn.example.com/widget.js',
      crossorigin: 'anonymous',
      integrity: 'sha384-xxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
      defer: true
    }]
  })
})
```

### Pattern conditionnel (consentement cookies)

```typescript
// app/composables/useConsentScript.ts
export function useConsentScript(src: string, category: 'analytics' | 'marketing') {
  const consent = useCookieConsent()

  const { load } = useScript(src, {
    trigger: 'manual'  // Ne charge pas automatiquement
  })

  // Charge uniquement si consentement accordé
  watch(() => consent.value[category], (hasConsent) => {
    if (hasConsent) load()
  }, { immediate: true })
}
```

## Protection XSS Nuxt Content

### Directive base-uri critique

La directive CSP `base-uri: ['none']` est **obligatoire** pour bloquer les attaques par injection de balise `<base>` (CVE-2025-54075).

```typescript
// nuxt.config.ts - Déjà configuré dans core-architectural-decisions.md
security: {
  headers: {
    contentSecurityPolicy: {
      'base-uri': ["'none'"],  // CRITIQUE - bloque <base> injection
      // ... autres directives
    }
  }
}
```

### Validation version @nuxtjs/mdc

Script de vérification à exécuter régulièrement :

```bash
# Vérifier la version installée
npm list @nuxtjs/mdc

# Mettre à jour si < 0.17.2
npm update @nuxtjs/mdc
```

**Versions vulnérables :**

| CVE | Version affectée | Version corrigée |
|-----|------------------|------------------|
| CVE-2025-24981 | < 0.17.2 | >= 0.17.2 |
| CVE-2025-54075 | < 0.17.2 | >= 0.17.2 |

## Anti-patterns Sécurité

| Anti-pattern | Risque | Solution |
|--------------|--------|----------|
| `v-html` sans sanitisation | XSS | Utiliser `useSanitize()` |
| `eval()` ou `new Function()` | Injection code | Éviter complètement |
| `innerHTML` direct | XSS | Utiliser DOMPurify |
| Scripts inline sans hash | Violation CSP | Utiliser `useScript` |
| `v-show` en mode strict CSP | Génère `style="display:none;"` | Utiliser `v-if` (retire du DOM) |
| `target="_blank"` sans `rel` | Tabnapping | Ajouter `rel="noopener noreferrer"` |
| Secrets en env publiques | Fuite données | Utiliser `runtimeConfig` (pas `public`) |

## Liens Externes Sécurisés

### Composant ProseA avec protection

```vue
<!-- app/components/prose/ProseA.vue -->
<script setup lang="ts">
const props = defineProps<{
  href: string
  target?: string
}>()

const isExternal = computed(() => {
  if (!props.href) return false
  return props.href.startsWith('http') && !props.href.includes('sebc.dev')
})

const secureTarget = computed(() => isExternal.value ? '_blank' : props.target)
const secureRel = computed(() => isExternal.value ? 'noopener noreferrer' : undefined)
</script>

<template>
  <a
    :href="href"
    :target="secureTarget"
    :rel="secureRel"
    :class="{ 'external-link': isExternal }"
  >
    <slot />
    <span v-if="isExternal" class="sr-only">(ouvre dans un nouvel onglet)</span>
  </a>
</template>
```

## CSP-Report-Only (Développement)

Avant d'enforcer une CSP stricte en production, utiliser le mode **Report-Only** pour identifier les violations sans bloquer :

```
# public/_headers (développement/staging)
/*
  Content-Security-Policy-Report-Only: default-src 'self'; script-src 'self' 'strict-dynamic'; ...
```

**Workflow recommandé :**

1. **Staging** : Déployer avec `Content-Security-Policy-Report-Only`
2. **Analyse** : Consulter les violations dans la console navigateur (F12)
3. **Ajustement** : Corriger les sources bloquées ou ajuster la CSP
4. **Production** : Remplacer par `Content-Security-Policy` (enforcement)

**Astuce** : Les violations apparaissent dans la console avec le préfixe `[Report Only]` et n'affectent pas le fonctionnement du site.

## Anti-patterns Sécurité Dépendances

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| `pnpm audit --fix` sans `pnpm install` | Vulnérabilités non corrigées | Toujours enchaîner les deux commandes |
| `continue-on-error: true` sur audit CI | Vulnérabilités ignorées silencieusement | Utiliser seuils (`--audit-level=high`) |
| `minimumReleaseAge` désactivé (prod) | Exposition packages malveillants | Minimum 3 jours pour prod deps |
| Cache `node_modules` directement | Cache fragile, platform-specific | Cacher le pnpm store à la place |
| Secrets dans `wrangler.toml` vars | Exposition dans version control | Dashboard secrets ou `wrangler secret put` |
| Ignorer les CVEs sans documentation | Perte de contexte | Documenter raison + date expiration |

**Workflow correct pnpm audit --fix :**

```bash
# ❌ INCORRECT - ne modifie que pnpm.overrides, pas le lockfile
pnpm audit --fix

# ✅ CORRECT - applique les overrides au lockfile
pnpm audit --fix && pnpm install
```

**Secrets Cloudflare - ordre de priorité :**

1. **Dashboard Secrets (Encrypted)** - Recommandé, jamais visibles après création
2. **`wrangler secret put NAME`** - CLI sécurisé
3. ❌ **`wrangler.toml` vars** - Exposé en version control

## Références

- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [nuxt-security Documentation](https://nuxt-security.vercel.app/)
- [DOMPurify](https://github.com/cure53/DOMPurify)
- [Content Security Policy (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
