# Animations shadcn-vue et Reka UI

## Attributs data-state pour animations

Les composants shadcn-vue utilisent les attributs `data-state` de Reka UI pour déclencher les animations CSS. Combinés avec `tw-animate-css`, cela donne :

```html
<DialogContent class="
  data-[state=open]:animate-in
  data-[state=open]:fade-in-0
  data-[state=open]:zoom-in-95
  data-[state=closed]:animate-out
  data-[state=closed]:fade-out-0
  data-[state=closed]:zoom-out-95
  data-[state=closed]:duration-200
  data-[state=open]:duration-300
">
  Contenu du dialog
</DialogContent>
```

## Utilitaires tw-animate-css

| Utilitaire | Animation |
|------------|-----------|
| `animate-in` | Active l'animation d'entrée |
| `animate-out` | Active l'animation de sortie |
| `fade-in-0` / `fade-out-0` | Fondu depuis/vers opacité 0 |
| `zoom-in-95` / `zoom-out-95` | Scale depuis/vers 95% |
| `slide-in-from-right` | Glisse depuis la droite |
| `slide-out-to-left` | Glisse vers la gauche |

## Variables CSS Reka UI pour animations dynamiques

Reka UI 2.7.0 expose des **variables CSS pour les dimensions**, permettant des animations de hauteur fluides sans JavaScript :

```css
/* Accordion avec animation hauteur fluide */
.accordion-content {
  overflow: hidden;
  /* Variable exposée par Reka UI */
  height: var(--reka-accordion-content-height);
  transition: height 300ms ease-out;
}

/* Collapsible content */
.collapsible-content {
  height: var(--reka-collapsible-content-height);
  width: var(--reka-collapsible-content-width);
}
```

## Prop forceMount pour Vue Transition

La prop `forceMount` maintient l'élément dans le DOM pendant les animations de sortie, nécessaire pour Vue `<Transition>` ou Motion Vue :

```vue
<DialogContent force-mount>
  <Transition
    enter-active-class="transition-all duration-300"
    leave-active-class="transition-all duration-200"
  >
    <div v-if="open">
      Contenu avec transition Vue
    </div>
  </Transition>
</DialogContent>
```

---
