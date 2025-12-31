# Sécurité Nuxt 4.2.x SSG sur Cloudflare Pages : guide complet

Le module **nuxt-security v2.5.0** est pleinement compatible avec Nuxt 4.2.x et offre une note A+ par défaut sur SecurityHeaders.com. Cependant, le mode SSG impose des contraintes majeures : les nonces dynamiques sont impossibles (remplacés par des hashes SHA-256), les middlewares serveur (rate limiting, CORS, CSRF) ne fonctionnent pas, et la CSP est livrée via balise `<meta>` plutôt qu'en-têtes HTTP. Pour une protection optimale, il faut combiner nuxt-security pour la génération des hashes CSP avec les fonctionnalités natives de Cloudflare (HSTS via dashboard, rate limiting WAF, Turnstile).

## Configuration CSP adaptée au mode statique

La différence fondamentale entre SSR et SSG réside dans la gestion des scripts inline. En SSR, nuxt-security génère un **nonce cryptographique unique** par requête. En SSG, les fichiers HTML étant pré-générés au build, cette approche est impossible. Le module calcule donc des **hashes SHA-256** de chaque script inline pendant `nuxi generate` et les injecte dans une balise `<meta http-equiv="Content-Security-Policy">`.

```typescript
// nuxt.config.ts - Configuration SSG optimale
export default defineNuxtConfig({
  modules: ['nuxt-security'],
  security: {
    ssg: {
      meta: true,           // CSP via <meta> tag
      hashScripts: true,    // Calcul automatique des hashes
      hashStyles: false,    // CRITIQUE: désactiver (casse l'hydratation Vue)
      exportToPresets: true // Export vers _headers Cloudflare
    },
    sri: true,              // Subresource Integrity
    headers: {
      contentSecurityPolicy: {
        'script-src': ["'self'", "https:", "'unsafe-inline'", "'strict-dynamic'"],
        'style-src': ["'self'", "https:", "'unsafe-inline'"],
        'base-uri': ["'none'"],  // Bloque les attaques via <base>
        'object-src': ["'none'"],
        'frame-ancestors': ["'self'"]
      }
    }
  }
})
```

L'option `hashStyles: false` est **impérative** car le mécanisme d'hydratation côté client de Nuxt injecte dynamiquement des styles qui seraient bloqués par une CSP stricte. La documentation officielle met en garde : activer le hash des styles peut provoquer des comportements erratiques lors de l'hydratation.

## Contraintes imposées par shadcn-vue et TailwindCSS 4

**TailwindCSS 4** compile vers des fichiers CSS externes, ce qui est parfaitement compatible avec une CSP stricte sans `'unsafe-inline'` pour les styles. Cependant, **shadcn-vue** (basé sur Reka UI/Radix Vue) pose un problème incontournable : ces librairies utilisent des **attributs style inline** pour le positionnement dynamique des composants (Dialog, Toast, Sheet, Command, Tabs).

L'issue GitHub #4461 de shadcn-ui documente ce comportement. Les primitives Radix utilisent des styles inline pour :
- Le positionnement (top, left, width, height)
- Les états de visibilité
- Les animations et mesures dynamiques

Une proposition de migration vers des classes CSS existe (discussion Radix #3130) mais n'est pas implémentée. En pratique, l'utilisation de `'unsafe-inline'` pour `style-src` reste **obligatoire** pour ces composants. Cette concession est largement acceptée dans l'industrie : les exploits connus via CSS inline sont rares et se limitent à l'extraction d'URLs d'images, mitigeable via `img-src`.

Pour les scripts externes sans attribut `integrity` (Google Analytics, etc.), utilisez `useScript` qui fonctionne avec `'strict-dynamic'` :

```typescript
// Chargement compatible strict-dynamic
useScript('https://example.com/analytics.js')

// Ou avec integrity explicite pour SSG
useHead({
  script: [{
    src: 'https://cdn.example.com/script.js',
    crossorigin: 'anonymous',
    integrity: 'sha384-xxxxx'  // Requis en SSG + strict-dynamic
  }]
}, { mode: 'client' })
```

## HSTS : privilégier le dashboard Cloudflare

Pour HSTS en mode SSG, la stratégie optimale consiste à **déléguer entièrement à Cloudflare** plutôt qu'à nuxt-security. Voici pourquoi : nuxt-security ne peut pas injecter d'en-têtes HTTP purs en SSG (seulement via `<meta>`, incompatible avec HSTS). L'option `exportToPresets` génère un fichier `_headers`, mais ce fichier est limité à **100 règles** et **2000 caractères par ligne** sur Cloudflare Pages.

**Configuration recommandée via Cloudflare Dashboard** (SSL/TLS → Edge Certificates → HSTS) :
- **Max-Age** : 12 mois minimum (31536000 secondes), idéalement 2 ans (63072000)
- **Include Subdomains** : Activer si tous les sous-domaines supportent HTTPS
- **Preload** : Activer uniquement après validation complète (irréversible pendant des mois)

```typescript
// nuxt.config.ts - Désactiver HSTS dans nuxt-security
export default defineNuxtConfig({
  security: {
    headers: {
      strictTransportSecurity: false  // Cloudflare gère HSTS
    }
  }
})
```

Pour les autres en-têtes de sécurité, utilisez le fichier `public/_headers` :

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## Rate limiting impossible en SSG pur

Le rate limiting applicatif est **techniquement impossible** en mode SSG sans serveur. Le middleware de nuxt-security nécessite un runtime serveur pour maintenir l'état des compteurs par IP. La documentation officielle prévient d'ailleurs que ce rate limiter n'est pas adapté à la production, même en SSR.

**Architecture recommandée pour Cloudflare Pages** :

| Couche | Solution | Coût |
|--------|----------|------|
| Protection globale | Bot Fight Mode | Gratuit |
| Rate limiting API | WAF Rate Limiting Rules | Gratuit (1 règle/plan free) |
| Formulaires | Cloudflare Turnstile | Gratuit |
| API endpoints | Pages Functions + Workers Rate Limit | Inclus |

Configuration WAF via dashboard (Security → WAF → Rate limiting rules) :
```
Expression: (http.request.uri.path contains "/api/")
Caractéristique: IP Address
Seuil: 100 requêtes / minute
Action: Managed Challenge
```

Pour les formulaires, **Turnstile** offre une alternative CAPTCHA gratuite et respectueuse de la vie privée :

```typescript
// functions/api/submit.ts - Validation Turnstile
export async function onRequestPost(context) {
  const formData = await context.request.formData();
  const token = formData.get('cf-turnstile-response');
  
  const verification = await fetch(
    'https://challenges.cloudflare.com/turnstile/v0/siteverify',
    {
      method: 'POST',
      body: JSON.stringify({
        secret: context.env.TURNSTILE_SECRET_KEY,
        response: token,
        remoteip: context.request.headers.get('cf-connecting-ip')
      })
    }
  ).then(r => r.json());
  
  if (!verification.success) {
    return new Response('Bot détecté', { status: 403 });
  }
  // Traitement du formulaire...
}
```

## Vulnérabilités XSS critiques dans Nuxt Content 3

**Mise à jour urgente requise** : le module `@nuxtjs/mdc` utilisé par Nuxt Content 3 a subi deux CVE critiques en 2025. **CVE-2025-24981** permettait de contourner la protection XSS via encodage HTML des URLs JavaScript, et **CVE-2025-54075** permettait l'injection de balises `<base>` pour détourner toutes les URLs relatives vers un domaine attaquant.

```bash
# Vérifier et mettre à jour immédiatement
npm list @nuxtjs/mdc
npm update @nuxtjs/mdc  # Minimum requis: 0.17.2
```

Le validateur XSS de nuxt-security fonctionne côté serveur et inspecte les requêtes GET/POST. En SSG pur, cette protection s'applique uniquement au build-time, pas aux interactions runtime. Pour les sites de contenu, la règle OWASP est claire : **sanitizer APRÈS la conversion Markdown→HTML**, jamais avant.

```typescript
// composables/useSanitize.ts
import DOMPurify from 'isomorphic-dompurify';

export function useSanitize() {
  return {
    sanitizeHTML: (dirty: string) => DOMPurify.sanitize(dirty, {
      ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong', 'a', 'ul', 'ol', 'li', 'h1', 'h2', 'h3'],
      ALLOWED_ATTR: ['href', 'title'],
      FORBID_TAGS: ['base', 'script', 'iframe', 'form', 'object']
    })
  };
}
```

La directive CSP `'base-uri': ["'none'"]` est **critique** pour bloquer les attaques par balise `<base>` même si le contenu n'est pas sanitisé.

## Configuration complète pour blog multilingue

```typescript
// nuxt.config.ts - Configuration production SSG + i18n
export default defineNuxtConfig({
  modules: ['nuxt-security', '@nuxt/content', '@nuxtjs/i18n'],
  
  security: {
    ssg: {
      meta: true,
      hashScripts: true,
      hashStyles: false,
      exportToPresets: true
    },
    sri: true,
    
    // Désactiver XSS validator pour les routes content (build-time)
    xssValidator: {
      methods: ['GET', 'POST'],
      throwError: true
    },
    
    headers: {
      strictTransportSecurity: false,  // Via Cloudflare
      contentSecurityPolicy: {
        'default-src': ["'self'"],
        'script-src': ["'self'", "https:", "'unsafe-inline'", "'strict-dynamic'"],
        'style-src': ["'self'", "https:", "'unsafe-inline'"],
        'img-src': ["'self'", "data:", "https:"],
        'font-src': ["'self'", "https:", "data:"],
        'base-uri': ["'none'"],
        'object-src': ["'none'"],
        'frame-ancestors': ["'self'"],
        'form-action': ["'self'"],
        'upgrade-insecure-requests': true
      }
    }
  },
  
  routeRules: {
    '/api/_content/**': {
      security: { xssValidator: false }
    }
  }
})
```

Pour l'i18n, aucune configuration spéciale n'est requise : les URLs localisées (`/fr/`, `/en/`) héritent automatiquement des règles de sécurité globales.

## Conclusion

L'intégration nuxt-security v2.5.0 avec Nuxt 4.2.x en mode SSG sur Cloudflare Pages nécessite une **approche hybride**. Le module excelle pour la génération automatique des hashes CSP et la configuration SRI, mais les protections runtime (rate limiting, HSTS, CORS) doivent être déléguées à l'infrastructure Cloudflare. 

Les points d'attention critiques sont : maintenir `@nuxtjs/mdc` à jour (CVE actifs), accepter `'unsafe-inline'` pour les styles (contrainte shadcn-vue), et toujours inclure `'base-uri': ["'none'"]` dans la CSP. Pour les formulaires, Turnstile combiné au rate limiting WAF offre une protection robuste sans code serveur.