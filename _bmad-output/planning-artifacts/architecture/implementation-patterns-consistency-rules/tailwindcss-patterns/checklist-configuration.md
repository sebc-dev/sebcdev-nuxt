# Checklist Configuration

```markdown
# Fichier main.css
- [ ] `@import "tailwindcss";`
- [ ] `@source` pour fichiers MDC si applicable
- [ ] `@custom-variant dark (&:is(.dark *));` pour shadcn-vue
- [ ] Variables `:root` et `.dark` pour theming
- [ ] `@theme inline` pour connecter variables à Tailwind

# nuxt.config.ts
- [ ] `@tailwindcss/vite` dans vite.plugins
- [ ] NE PAS utiliser @nuxtjs/tailwindcss
- [ ] `css: ['~/assets/css/main.css']`

# Migrations v3 → v4
- [ ] Exécuter `npx @tailwindcss/upgrade@next`
- [ ] Vérifier renommages (shadow-sm → shadow-xs, etc.)
- [ ] Mettre à jour syntaxe variables arbitraires
```
