# Optimisation des Listes

## Event Delegation pour listes

Au lieu d'attacher un handler à chaque élément de liste, utiliser un seul handler délégué :

```vue
<script setup lang="ts">
// Un seul handler délégué au lieu de N handlers
const handleListClick = (event: MouseEvent) => {
  const listItem = (event.target as HTMLElement).closest('[data-item-id]')
  if (!listItem) return

  const itemId = listItem.dataset.itemId
  if (itemId) {
    selectItem(parseInt(itemId))
  }
}
</script>

<template>
  <!-- ✅ BIEN : 1 seul listener pour toute la liste -->
  <ul @click="handleListClick">
    <li
      v-for="item in items"
      :key="item.id"
      :data-item-id="item.id"
      class="cursor-pointer hover:bg-muted"
    >
      {{ item.name }}
    </li>
  </ul>
</template>
```

**Avantages :**
- Réduit la mémoire (1 listener vs N listeners)
- Meilleur INP (moins de handlers à attacher/détacher)
- Fonctionne automatiquement avec les éléments ajoutés dynamiquement

| Nombre d'items | Handlers directs | Event delegation |
|----------------|------------------|------------------|
| 50 | 50 listeners | 1 listener |
| 500 | 500 listeners | 1 listener |
| 5000 | 5000 listeners | 1 listener |

## v-memo pour listes larges (1000+ items)

`v-memo` évite le re-render des items dont les dépendances n'ont pas changé :

```vue
<template>
  <div
    v-for="item in items"
    :key="item.id"
    v-memo="[item.id === selectedId]"
    :class="{ 'bg-primary': item.id === selectedId }"
    @click="selectedId = item.id"
  >
    <h3>{{ item.title }}</h3>
    <p>{{ item.description }}</p>
  </div>
</template>
```

**Fonctionnement :** Le template de l'item n'est re-rendu que si une valeur dans le tableau `v-memo` change.

| Scénario | Sans v-memo | Avec v-memo |
|----------|-------------|-------------|
| Sélection item dans liste 1000 | Re-render 1000 items | Re-render 2 items (ancien + nouveau sélectionné) |

**Quand utiliser :**
- Listes de 1000+ items avec sélection
- Composants item complexes (plusieurs enfants)
- Listes avec filtrage/tri côté client

**Quand éviter :**
- Listes < 100 items (overhead de v-memo > bénéfice)
- Items simples (texte uniquement)
- Virtualisation déjà en place (vue-virtual-scroller)

## Passive Listeners pour listes scrollables

Pour les listes avec scroll interne, toujours utiliser `.passive` :

```vue
<template>
  <!-- ✅ Passive scroll (ne bloque jamais le scroll) -->
  <div
    @scroll.passive="handleScroll"
    class="overflow-auto max-h-96"
  >
    <div v-for="item in items" :key="item.id">
      {{ item.name }}
    </div>
  </div>

  <!-- ✅ Passive touch pour mobile -->
  <div
    @touchstart.passive="handleTouchStart"
    @touchmove.passive="handleTouchMove"
  >
    Contenu swipeable
  </div>
</template>
```

---
