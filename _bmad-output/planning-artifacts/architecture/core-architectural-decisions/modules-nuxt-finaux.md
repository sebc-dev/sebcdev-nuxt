# Modules Nuxt Finaux

```typescript
// ORDRE CRITIQUE: @nuxtjs/seo AVANT @nuxt/content
modules: [
  '@nuxt/image',
  '@nuxt/fonts',         // Fonts optimisées avec fallbacks métriques
  '@nuxtjs/i18n',
  '@nuxtjs/seo',         // Suite SEO complète (sitemap, robots, schema.org, og-image, link-checker)
  '@nuxt/content',       // ← APRÈS @nuxtjs/seo
  'nuxt-llms',           // Génération automatique /llms.txt
  'nuxt-security',       // Security headers + CSP hash generation SSG
  'nuxt-vitalizer',      // Optimisation LCP
  '@nuxtjs/critters',    // CSS critique inline (FCP -55%)
  'shadcn-nuxt',
  '@nuxtjs/color-mode',  // Dark mode avec classe .dark
]

// Configuration site (requise par @nuxtjs/seo)
site: {
  url: 'https://sebc.dev',
  name: 'sebc.dev',
  description: 'Blog technique sur le développement web',
  defaultLocale: 'en',  // Cohérent avec i18n.defaultLocale
}

// Configuration @nuxtjs/seo - Meta tags globaux
seo: {
  meta: {
    ogSiteName: 'sebc.dev',
    twitterCard: 'summary_large_image',
  }
}

// Configuration nuxt-og-image (inclus dans @nuxtjs/seo)
ogImage: {
  zeroRuntime: true,           // ESSENTIEL pour SSG pur (pas de server functions)
  runtimeCacheStorage: false,  // Pas de cache runtime en SSG
  defaults: {
    renderer: 'satori',        // Vue → SVG → PNG au build time
    width: 1200,
    height: 630,               // Ratio 1.91:1 standard OG
  }
}

// Configuration nuxt-schema-org (inclus dans @nuxtjs/seo)
schemaOrg: {
  // Optimisations SSG
  reactive: false,    // Désactive la réactivité client (inutile en SSG)
  minify: true,       // Minifie le JSON-LD en production
  defaults: true,     // Génère automatiquement WebSite et WebPage

  // Identité du propriétaire (Person pour blog personnel)
  identity: {
    type: 'Person',
    name: 'Sébastien C.',
    givenName: 'Sébastien',
    familyName: 'C.',
    url: 'https://sebc.dev',
    image: '/images/profile.jpg',
    description: 'Développeur full-stack spécialisé Vue.js et Nuxt',
    jobTitle: 'Lead Developer',

    // URLs équivalentes (social proof E-E-A-T)
    sameAs: [
      'https://github.com/sebcdev',
      'https://twitter.com/sebcdev',
      'https://linkedin.com/in/sebcdev'
    ],

    // Expertise technique (renforce E-E-A-T)
    knowsAbout: [
      { '@type': 'Thing', name: 'Vue.js', sameAs: 'https://en.wikipedia.org/wiki/Vue.js' },
      { '@type': 'Thing', name: 'Nuxt', sameAs: 'https://en.wikipedia.org/wiki/Nuxt.js' },
      'TypeScript',
      'Node.js',
      'Cloud Architecture'
    ]
  }
}

// Configuration @nuxtjs/critters - CSS critique inline
critters: {
  config: {
    preload: 'swap',      // Charge le reste du CSS en async
    inlineFonts: false,   // Ne pas inliner les @font-face (fonts self-hosted)
    pruneSource: false,   // Garder le CSS complet pour le chargement async
  }
}

// Configuration @nuxt/fonts - Fonts optimisées avec fallbacks métriques
// @nuxt/fonts utilise fontaine + capsize pour générer des fallbacks avec métriques ajustées
fonts: {
  // Poids par défaut (limiter pour réduire la taille du bundle)
  defaults: {
    weights: [400, 600, 700],
    styles: ['normal'],
    subsets: ['latin', 'latin-ext']
  },

  families: [
    // Self-hosting pour SSG (recommandé - zéro requête externe)
    {
      name: 'Satoshi',
      provider: 'local',
      src: '/fonts/Satoshi-Variable.woff2',
      weight: '400 700'
    },
    {
      name: 'JetBrains Mono',
      provider: 'local',
      src: '/fonts/JetBrainsMono-Variable.woff2',
      weight: '400 700'
    }
  ],

  // Fallbacks pour calcul automatique des métriques
  fallbacks: {
    'sans-serif': ['Arial', 'Helvetica Neue'],
    'monospace': ['Menlo', 'Monaco']
  },

  // Désactiver les providers externes pour full SSG
  providers: {
    google: false,
    bunny: false
  }
}

// Configuration @nuxtjs/color-mode - Complète
colorMode: {
  preference: 'system',           // Valeur par défaut
  fallback: 'light',              // Fallback si système non détectable
  classSuffix: '',                // CRITIQUE: vide pour Tailwind (.dark)
  storage: 'localStorage',        // Persistence
  storageKey: 'nuxt-color-mode'   // Clé localStorage
}

// Configuration nuxt-security - CSP et headers de sécurité
// Note: Les headers HTTP sont principalement définis dans public/_headers pour CF Pages
// nuxt-security ajoute le support SSG (hash scripts) et la validation en dev
//
// ⚠️ ALERTE SÉCURITÉ - CVE critiques @nuxtjs/mdc (utilisé par Nuxt Content 3)
// - CVE-2025-24981 : Contournement XSS via encodage HTML des URLs JavaScript
// - CVE-2025-54075 : Injection balise <base> détournant toutes URLs relatives
// → Version minimale requise : @nuxtjs/mdc >= 0.17.2
// → Vérifier : npm list @nuxtjs/mdc && npm update @nuxtjs/mdc
//
security: {
  nonce: true,  // Requis pour dev mode
  sri: true,    // Subresource Integrity - protection contre compromission CDN/scripts
  ssg: {
    meta: true,           // CSP via <meta> tag fallback
    hashScripts: true,    // Auto-calcul des hashes scripts au build
    hashStyles: false,    // false car 'unsafe-inline' requis pour shadcn-vue
    exportToPresets: true // Tente export vers headers plateforme
  },
  headers: {
    contentSecurityPolicy: {
      'default-src': ["'self'"],
      'script-src': ["'self'", "'strict-dynamic'"],
      'style-src': ["'self'", "'unsafe-inline'"],  // ⚠️ Requis pour Reka UI
      'img-src': ["'self'", "data:", "blob:", "https:"],
      'font-src': ["'self'", "data:"],
      'connect-src': ["'self'"],
      'frame-ancestors': ["'none'"],
      'base-uri': ["'none'"],
      'form-action': ["'self'"],
      'object-src': ["'none'"],
      'upgrade-insecure-requests': true
    },
    crossOriginOpenerPolicy: 'same-origin-allow-popups',  // OAuth compatible
    crossOriginResourcePolicy: 'same-origin',
    referrerPolicy: 'strict-origin-when-cross-origin',
    strictTransportSecurity: {
      maxAge: 63072000,      // 2 ans
      includeSubdomains: true,
      preload: true
    },
    xContentTypeOptions: 'nosniff',
    xFrameOptions: 'DENY',
    xXSSProtection: '0',    // Désactivé (filtre navigateur déprécié)
    permissionsPolicy: {
      camera: [],
      microphone: [],
      geolocation: [],
      payment: [],
      usb: [],
      bluetooth: [],
      accelerometer: [],
      gyroscope: []
    }
  }
}

// Script anti-FOUC + meta color-scheme (CRITIQUE pour SSG)
// Élimine le flash de thème en appliquant la classe AVANT le premier rendu
app: {
  head: {
    meta: [
      { name: 'color-scheme', content: 'light dark' }  // Évite flash scrollbars/inputs
    ],
    script: [{
      // Script bloquant ~300 octets : préférence user → système → défaut
      children: `(function(){try{var t=localStorage.getItem('nuxt-color-mode');var s=window.matchMedia&&window.matchMedia('(prefers-color-scheme: dark)').matches?'dark':'light';var m=t&&t!=='system'?t:s;document.documentElement.classList.add(m);window.__NUXT_COLOR_MODE__=m}catch(e){}})();`,
      tagPosition: 'head'
    }]
  }
}

// Performance (remplace experimental.inlineSSRStyles)
features: {
  inlineStyles: true,   // CLS 0.77 → 0.00
}

// Transpilation obligatoire pour SSR (évite erreurs ES modules)
build: {
  transpile: [
    'reka-ui',           // "Cannot split a chunk" sans transpilation
    'vee-validate',      // "Unexpected Token: export" sans transpilation
    '@vee-validate/rules'
  ]
}

// Cloudflare Pages SSG - Configuration complète
nitro: {
  preset: 'cloudflare_pages',

  // Configuration Cloudflare spécifique
  cloudflare: {
    deployConfig: true,    // Génère wrangler.json automatiquement
    nodeCompat: true,      // Compatibilité Node.js
    pages: {
      routes: {
        // Optimise _routes.json (limite 100 routes Cloudflare)
        exclude: ['/blog/*', '/categories/*', '/tags/*']
      }
    }
  },

  // Configuration prerendering optimisée
  prerender: {
    autoSubfolderIndex: false, // Évite les doubles redirects Cloudflare
    crawlLinks: true,          // Découvre automatiquement les liens internes
    routes: [                  // Routes additionnelles à prérender
      '/',
      '/sitemap.xml',
      '/robots.txt',
      '/rss.xml'
    ],
    ignore: [                  // Routes à exclure
      '/api/**',
      '/admin/**',
      '/_nuxt/**'
    ],
    failOnError: false,        // Continue malgré les erreurs
    concurrency: 4,            // Prerendering parallèle
    retry: 3,                  // Tentatives en cas d'échec
    retryDelay: 500
  },

  // Compression assets (~15-25% réduction vs gzip seul)
  compressPublicAssets: {
    gzip: true,
    brotli: true,
  },
}

// Lazy loading composants MDC lourds
components: [
  { path: '~/components/content/heavy', isAsync: true },
  '~/components',
],

// Route Rules - Cache Cloudflare optimisé
routeRules: {
  // Cache statique agressif pour assets buildés par Nuxt (1 an, immutable)
  '/_nuxt/**': { headers: { 'cache-control': 'public, max-age=31536000, immutable' } },
  // Cache court pour HTML SSG (permet rollbacks rapides)
  '/**': { headers: { 'cache-control': 'public, max-age=0, must-revalidate' } },
},

// Nuxt Content 3 - D1 Database REQUISE sur Cloudflare Pages
content: {
  database: {
    type: 'd1',
    bindingName: 'DB'  // Binding configuré dans Cloudflare Dashboard
  },

  // Syntax highlighting Shiki - Configuration v3
  build: {
    markdown: {
      highlight: {
        theme: {
          default: 'github-light',      // Thème light (WCAG AA)
          dark: 'github-dark'           // Thème dark (auto via html.dark)
        },
        // Limiter les langages pour réduire le bundle
        langs: ['json', 'js', 'ts', 'html', 'css', 'vue', 'shell', 'yaml', 'md', 'mdc', 'bash', 'toml']
      }
    }
  }
}
```

**Configuration Cloudflare D1 (wrangler.toml) :**

```toml
# wrangler.toml
name = "sebc-dev"
compatibility_date = "2024-09-19"

[[d1_databases]]
binding = "DB"
database_name = "content-db"
database_id = "votre-database-id"  # Généré via: wrangler d1 create content-db
```

**Création de la base D1 :**

```bash
# Créer la base D1
wrangler d1 create content-db

# Récupérer le database_id affiché et l'ajouter dans wrangler.toml
```
