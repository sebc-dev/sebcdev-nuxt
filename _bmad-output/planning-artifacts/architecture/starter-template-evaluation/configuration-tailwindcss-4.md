# Configuration TailwindCSS 4

```css
/* app/assets/css/main.css */
@import "tailwindcss";

/* Scanner les fichiers Markdown pour les classes Tailwind dans les attributs MDC */
@source "../../../content/**/*";

@theme {
  /* Typographie */
  --font-display: "Satoshi", "sans-serif";
  --font-mono: "JetBrains Mono", "monospace";
  
  /* Palette oklch dark-first */
  --color-primary: oklch(0.92 0.19 114.08);
  --color-secondary: oklch(0.85 0.15 242.32);
  --color-background: oklch(0.13 0.01 264.05);
  --color-foreground: oklch(0.98 0.01 264.05);
  
  /* Breakpoints personnalisés */
  --breakpoint-3xl: 1920px;
}
```
