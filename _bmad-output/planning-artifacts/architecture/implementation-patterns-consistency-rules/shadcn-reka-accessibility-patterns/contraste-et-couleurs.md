# Contraste et Couleurs

Les thèmes shadcn-vue doivent respecter WCAG AA :

| Élément | Ratio minimum | Vérification |
|---------|---------------|--------------|
| Texte normal | **4.5:1** | `--foreground` vs `--background` |
| Texte large (18px+ bold) | **3:1** | Titres |
| Composants UI | **3:1** | Bordures, icônes |
| Focus ring | **3:1** | `--ring` vs background |

**Outils de vérification :**
- Chrome DevTools → Accessibility panel
- Extension axe DevTools
- WebAIM Contrast Checker
