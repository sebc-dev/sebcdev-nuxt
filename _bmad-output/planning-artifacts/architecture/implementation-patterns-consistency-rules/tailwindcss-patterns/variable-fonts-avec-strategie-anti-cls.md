# Variable Fonts avec Stratégie Anti-CLS

## Configuration @font-face optimisée

La stratégie `font-display: optional` **élimine 100% du CLS** (Cumulative Layout Shift) :

```css
/* app/assets/css/main.css - AVANT @import "tailwindcss" */
@font-face {
  font-family: "Satoshi Variable";
  font-style: normal;
  font-weight: 100 900;
  font-display: optional;  /* ← Clé pour zéro CLS */
  src: url("/fonts/Satoshi-Variable.woff2") format("woff2-variations");
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
                 U+02DA, U+02DC, U+2000-206F, U+20AC, U+2122, U+FEFF;
}

@font-face {
  font-family: "JetBrains Mono Variable";
  font-style: normal;
  font-weight: 100 800;
  font-display: optional;
  src: url("/fonts/JetBrainsMono-Variable.woff2") format("woff2-variations");
}

@import "tailwindcss";
```

## Comparaison font-display

| Valeur | Comportement | CLS | Recommandation |
|--------|--------------|-----|----------------|
| `swap` | Flash de font fallback puis swap | **Élevé** | ⚠️ Acceptable si fallbacks métriques configurés |
| `block` | Texte invisible puis apparition | Moyen | ❌ Mauvaise UX |
| `fallback` | Swap si chargé < 100ms, sinon fallback | Faible | ⚠️ Compromis |
| `optional` | Utilise cache ou fallback direct | **Zéro** | ✅ **Recommandé SSG** |

**Recommandations contextuelles :**

| Contexte | font-display | Raison |
|----------|--------------|--------|
| **SSG blog** (ce projet) | `optional` | Zéro CLS, performance maximale |
| **App avec branding fort** | `swap` + fallbacks métriques | Web font visible, CLS minimisé par fallbacks |
| **Texte critique (navigation)** | `optional` | Ne jamais bloquer la navigation |
| **Titres décoratifs** | `swap` | Le swap est acceptable si non-critique |

**Note** : `@nuxt/fonts` utilise `swap` par défaut mais génère automatiquement des fallbacks avec métriques ajustées via fontaine/capsize, réduisant significativement le CLS. Pour zéro CLS garanti, utiliser `font-display: optional` dans les @font-face manuels.

## Fallback métrique pour CLS zéro (Capsize/Fontaine)

Pour éliminer complètement le CLS typographique avec `font-display: swap`, utiliser des fallbacks métriquement ajustés :

```css
/* assets/css/fonts.css */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-var.woff2') format('woff2');
  font-weight: 100 900;
  font-display: swap;
}

/* Fallback métrique généré par Capsize/Fontaine */
@font-face {
  font-family: 'Inter Fallback';
  src: local('Arial');
  ascent-override: 90.49%;
  descent-override: 22.56%;
  line-gap-override: 0%;
  size-adjust: 107.64%;
}

@theme {
  --font-sans: 'Inter', 'Inter Fallback', ui-sans-serif, system-ui, sans-serif;
}
```

**Propriétés de fallback métrique :**

| Propriété | Description | Valeur typique |
|-----------|-------------|----------------|
| `ascent-override` | Hauteur au-dessus de la baseline | 85-95% |
| `descent-override` | Profondeur sous la baseline | 20-25% |
| `line-gap-override` | Espace entre lignes | 0% |
| `size-adjust` | Ajustement de taille globale | 100-110% |

**Génération automatique :**
- [Capsize](https://seek-oss.github.io/capsize/) - Calcul des métriques depuis un fichier font
- [@nuxt/fonts](https://nuxt.com/modules/fonts) - Génère automatiquement les fallbacks via fontaine

## Subsetting fonts avec Glyphhanger

Réduire **96% du poids** en conservant uniquement les caractères nécessaires :

```bash
# Installation
npm install -g glyphhanger
pip install fonttools brotli

# Subset Latin uniquement
glyphhanger --LATIN --subset=Inter.ttf --formats=woff2

# Résultat : Inter 765KB → ~15KB

# Subset avec caractères français (accents)
glyphhanger --LATIN --whitelist="àâäéèêëïîôùûüç" --subset=Inter.ttf --formats=woff2

# Subset depuis contenu réel du site
glyphhanger https://monsite.com --spider --subset=Inter.ttf --formats=woff2
```

**Presets de subsetting :**

| Preset | Caractères | Taille typique |
|--------|------------|----------------|
| `--LATIN` | A-Z, a-z, 0-9, ponctuation | ~15KB |
| `--LATIN` + accents FR | + àâäéèêëïîôùûüç | ~18KB |
| `--US_ASCII` | ASCII 7-bit uniquement | ~12KB |
| Subset custom | Crawl du site réel | Variable |

## Preload obligatoire

Le preload avec `crossorigin="anonymous"` est **obligatoire** même en self-hosting :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    head: {
      link: [
        {
          rel: 'preload',
          href: '/fonts/Satoshi-Variable.woff2',
          as: 'font',
          type: 'font/woff2',
          crossorigin: 'anonymous'  // ← Obligatoire même en self-hosting
        },
        {
          rel: 'preload',
          href: '/fonts/JetBrainsMono-Variable.woff2',
          as: 'font',
          type: 'font/woff2',
          crossorigin: 'anonymous'
        }
      ]
    }
  }
})
```

## Connexion à Tailwind

```css
@theme {
  --font-sans: "Satoshi Variable", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "JetBrains Mono Variable", ui-monospace, Menlo, monospace;

  /* OpenType features pour Inter/Satoshi */
  --font-sans--font-feature-settings: "cv01", "cv02", "ss01";
}
```

---
