# Configuration CSS-first

TailwindCSS v4 remplace `tailwind.config.js` par une configuration entièrement en CSS :

```css
/* app/assets/css/main.css */
@import "tailwindcss";

/* Scanner les fichiers MDC pour les classes Tailwind */
@source "../../../content/**/*";
```
