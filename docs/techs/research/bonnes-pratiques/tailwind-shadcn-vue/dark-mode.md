# Implémenter le Dark Mode dans Nuxt 4 avec TailwindCSS v4 et shadcn-vue

L'implémentation du dark mode en décembre 2025 repose sur une approche **CSS-native** avec TailwindCSS v4.1, où la configuration se fait directement en CSS via `@custom-variant`, combinée au module `@nuxtjs/color-mode v4.0.0` compatible Nuxt 4.2.x. La clé pour éviter le FOUC en SSG est d'injecter un **script bloquant dans le `<head>`** qui applique le thème avant le premier rendu. Cette architecture s'intègre parfaitement avec les CSS variables de shadcn-vue utilisant le format **oklch** et se déploie efficacement sur Cloudflare Pages.

## Configuration TailwindCSS v4.1 : l'approche CSS-first

TailwindCSS v4 abandonne totalement le fichier `tailwind.config.js` au profit d'une configuration CSS-native. Le dark mode se configure désormais avec la directive `@custom-variant` directement dans votre fichier CSS principal :

```css
/* assets/css/main.css */
@import "tailwindcss";

/* Active le dark mode basé sur la classe .dark */
@custom-variant dark (&:where(.dark, .dark *));
```

Cette syntaxe utilise `:where()` pour maintenir une spécificité basse et cible à la fois l'élément portant `.dark` et tous ses descendants. Pour les cas avancés combinant préférence système et toggle manuel, la syntaxe multi-règles permet un contrôle fin :

```css
@custom-variant dark {
  &:where(.dark, .dark *) { @slot; }
  @media (prefers-color-scheme: dark) {
    &:where(:not(.light *)) { @slot; }
  }
}
```

L'intégration avec **@tailwindcss/vite** simplifie drastiquement la configuration — plus besoin de PostCSS ni d'autoprefixer. Le plugin Vite gère automatiquement la détection du contenu et le prefixing.

## Définir les tokens couleurs avec oklch et @theme

TailwindCSS v4 adopte l'espace colorimétrique **oklch** (Oklab Lightness Chroma Hue) offrant une manipulation perceptuellement uniforme des couleurs. La directive `@theme inline` enregistre vos couleurs personnalisées :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

/* Variables CSS light/dark */
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --card: oklch(1 0 0);
  --border: oklch(0.922 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --card: oklch(0.205 0 0);
  --border: oklch(1 0 0 / 10%);
}

/* Enregistrement Tailwind */
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
}
```

## Configuration @nuxtjs/color-mode pour Nuxt 4

Le module **@nuxtjs/color-mode v4.0.0** (novembre 2025) supporte nativement Nuxt 4.x et sa nouvelle structure `app/`. La configuration dans `nuxt.config.ts` doit impérativement définir `classSuffix: ''` pour la compatibilité Tailwind :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  compatibilityDate: '2024-12-01',
  future: { compatibilityVersion: 4 },
  
  modules: ['@nuxtjs/color-mode'],
  
  colorMode: {
    preference: 'system',       // Valeur par défaut
    fallback: 'light',          // Fallback si système non détectable
    classSuffix: '',            // CRITIQUE: vide pour Tailwind
    storage: 'localStorage',    // Persistence
    storageKey: 'nuxt-color-mode'
  }
})
```

Le composable `useColorMode()` expose quatre propriétés réactives essentielles : `preference` (modifiable, peut être 'system'), `value` (lecture seule, 'light' ou 'dark' résolu), `unknown` (true pendant SSR/SSG avant hydratation), et `forced` (true si la page force un mode).

## Éviter le FOUC avec un script bloquant

Le problème majeur en SSG est le **Flash of inAccurate coloR Theme** : la page se rend avec le thème par défaut avant que JavaScript ne détecte la préférence utilisateur. La solution consiste à injecter un **script inline bloquant** dans le `<head>` qui s'exécute avant tout rendu :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    head: {
      meta: [
        { name: 'color-scheme', content: 'light dark' }
      ],
      script: [{
        children: `(function(){try{var t=localStorage.getItem('nuxt-color-mode');var s=window.matchMedia&&window.matchMedia('(prefers-color-scheme: dark)').matches?'dark':'light';var m=t&&t!=='system'?t:s;document.documentElement.classList.add(m);window.__NUXT_COLOR_MODE__=m}catch(e){}})();`,
        tagPosition: 'head'
      }]
    }
  }
})
```

Ce script minifié (~300 octets) suit la priorité : **préférence utilisateur → préférence système → défaut**. Le meta tag `color-scheme` est crucial car il informe le navigateur des modes supportés, évitant le flash sur les éléments natifs (scrollbars, inputs).

Pour les composants de toggle, utilisez systématiquement `<ClientOnly>` avec un fallback dimensionné pour éviter le layout shift :

```vue
<template>
  <ClientOnly>
    <button @click="colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark'">
      {{ colorMode.value === 'dark' ? '🌙' : '☀️' }}
    </button>
    <template #fallback>
      <div class="w-8 h-8" />
    </template>
  </ClientOnly>
</template>
```

## Synchronisation temps réel avec les préférences système

Pour réagir aux changements de thème système en temps réel (passage mode clair/sombre macOS), implémentez un listener `matchMedia` :

```typescript
// composables/useSystemThemeSync.ts
export function useSystemThemeSync() {
  const colorMode = useColorMode()
  
  onMounted(() => {
    if (!window.matchMedia) return
    
    const query = window.matchMedia('(prefers-color-scheme: dark)')
    
    const handler = (e: MediaQueryListEvent) => {
      if (colorMode.preference === 'system') {
        // @nuxtjs/color-mode gère automatiquement cette synchro
        // mais vous pouvez ajouter une logique custom ici
      }
    }
    
    query.addEventListener('change', handler)
    onUnmounted(() => query.removeEventListener('change', handler))
  })
}
```

Pour les navigateurs sans support `prefers-color-scheme` (Edge Legacy, IE), le fallback vers la valeur configurée (`fallback: 'light'`) s'applique automatiquement.

## Intégration shadcn-vue avec les tokens oklch

shadcn-vue utilise une convention **background/foreground** où chaque couleur sémantique a sa paire. Le fichier `globals.css` complet pour Tailwind v4 :

```css
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);
  --card-foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.371 0 0);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(1 0 0 / 10%);
  --input: oklch(1 0 0 / 15%);
  --ring: oklch(0.556 0 0);
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
}
```

Les composants Reka UI (base de shadcn-vue) exposent des attributs `data-state` permettant le styling conditionnel : `data-[state=open]:border-primary`.

## Déploiement Cloudflare Pages optimisé

La configuration Nitro pour Cloudflare Pages SSG requiert le preset approprié et la désactivation de `autoSubfolderIndex` pour le routage :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      autoSubfolderIndex: false,
      crawlLinks: true
    }
  }
})
```

Créez un fichier `public/_headers` pour optimiser le cache des assets thémés :

```plaintext
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

/*.html
  Cache-Control: public, max-age=0, must-revalidate

/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
```

Les assets hashés (`/_nuxt/*`) peuvent être mis en cache **1 an** car leur nom change à chaque modification. Les fichiers HTML doivent toujours être revalidés pour que le script anti-flash inline reste à jour.

## Configuration complète de référence

```typescript
// nuxt.config.ts - Configuration production complète
export default defineNuxtConfig({
  compatibilityDate: '2024-12-01',
  future: { compatibilityVersion: 4 },
  ssr: true,
  
  modules: ['@nuxtjs/color-mode'],
  
  colorMode: {
    preference: 'system',
    fallback: 'light',
    classSuffix: '',
    storage: 'localStorage',
    storageKey: 'nuxt-color-mode'
  },
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      autoSubfolderIndex: false,
      crawlLinks: true
    }
  },
  
  app: {
    head: {
      htmlAttrs: { lang: 'fr' },
      meta: [
        { name: 'color-scheme', content: 'light dark' }
      ],
      script: [{
        children: `(function(){try{var t=localStorage.getItem('nuxt-color-mode');var s=window.matchMedia&&window.matchMedia('(prefers-color-scheme: dark)').matches?'dark':'light';var m=t&&t!=='system'?t:s;document.documentElement.classList.add(m);window.__NUXT_COLOR_MODE__=m}catch(e){}})();`,
        tagPosition: 'head'
      }]
    }
  },
  
  css: ['~/assets/css/globals.css']
})
```

## Conclusion

L'implémentation du dark mode en 2025 avec cette stack repose sur trois piliers techniques : la directive `@custom-variant dark (&:where(.dark, .dark *))` de TailwindCSS v4 pour une configuration CSS-native, le module `@nuxtjs/color-mode v4.0.0` avec `classSuffix: ''` pour la gestion d'état, et un **script bloquant inline de ~300 octets** dans le `<head>` pour éliminer le FOUC en SSG. Le format oklch de shadcn-vue offre une meilleure uniformité perceptuelle que HSL, tandis que le déploiement Cloudflare Pages bénéficie d'un cache agressif sur les assets hashés et d'une revalidation systématique du HTML. Cette architecture garantit une expérience utilisateur sans flash visuel tout en maintenant des performances optimales sur l'edge.