# Anti-patterns Testing

## Ne pas tester les détails d'implémentation

```typescript
// ❌ BAD: Accès à l'état interne
expect(wrapper.vm.internalCounter).toBe(5)

// ✅ GOOD: Test du comportement observable
expect(wrapper.find('[data-testid="count"]').text()).toBe('5')
```

## Ne pas utiliser setData avec Composition API

```typescript
// ❌ BAD: setData ne fonctionne pas avec script setup
await wrapper.setData({ count: 5 })

// ✅ GOOD: Déclencher l'action qui change l'état
await wrapper.find('button').trigger('click')
```

## Toujours await les opérations async

```typescript
// ❌ BAD: await manquant
wrapper.find('button').trigger('click')
expect(wrapper.text()).toContain('Updated')  // Peut échouer!

// ✅ GOOD: Await le trigger
await wrapper.find('button').trigger('click')
expect(wrapper.text()).toContain('Updated')
```

## Ne pas over-stubber

```typescript
// ❌ BAD: shallowMount stubbe tout
const wrapper = shallowMount(Component)

// ✅ GOOD: mount + stub sélectif si nécessaire
const wrapper = mount(Component, {
  global: {
    stubs: {
      HeavyExternalComponent: true,  // Uniquement les dépendances lourdes
    },
  },
})
```

## Éviter les assertions sur wrapper.vm

```typescript
// ❌ BAD: Couplé à l'implémentation
expect(wrapper.vm.isLoading).toBe(true)

// ✅ GOOD: Test ce que l'utilisateur voit
expect(wrapper.find('.loading-spinner').exists()).toBe(true)
```
