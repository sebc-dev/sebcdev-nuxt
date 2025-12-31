# Testing Teleport/Portal (shadcn-vue)

Les composants Dialog, Sheet, Dropdown de shadcn-vue utilisent `Teleport`. Deux stratégies :

## Stratégie 1 : Créer le portal target

```typescript
beforeEach(() => {
  const el = document.createElement('div')
  el.id = 'radix-portal'
  document.body.appendChild(el)
})

afterEach(() => {
  document.body.innerHTML = ''
})

it('ouvre le dialog', async () => {
  const wrapper = await mountSuspended(MyDialog, {
    props: { open: true },
  })
  expect(document.body.textContent).toContain('Contenu dialog')
})
```

## Stratégie 2 : Stub Teleport globalement

```typescript
const wrapper = await mountSuspended(MyDialog, {
  props: { open: true },
  global: {
    stubs: {
      teleport: true,
    },
  },
})

// Le contenu reste dans le wrapper au lieu d'être téléporté
expect(wrapper.text()).toContain('Contenu dialog')
```

| Stratégie | Avantage | Inconvénient |
|-----------|----------|--------------|
| Portal target | Teste le vrai comportement | Setup/teardown requis |
| Stub Teleport | Plus simple | Ne teste pas le positionnement réel |

---
