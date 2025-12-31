# Snapshot Testing Guidelines

## Quand utiliser les snapshots

| ✅ Utiliser pour | ❌ Éviter pour |
|------------------|----------------|
| Messages d'erreur | Interactions utilisateur |
| Transformations de code | Contenu dynamique (timestamps, IDs) |
| Petites structures (<10 lignes) | Arbres de composants géants |
| Output CSS-in-JS | Structures internes Reka UI |

## Inline vs File snapshots

```typescript
// ✅ Inline snapshot - output court
it('formate la devise correctement', () => {
  const result = formatCurrency(1234.56, 'EUR')
  expect(result).toMatchInlineSnapshot(`"1 234,56 €"`)
})

// ✅ File snapshot - structure plus grande
it('rend le card complet', () => {
  const wrapper = mount(Card, {
    props: { title: 'Hello', content: 'World' },
  })
  expect(wrapper.html()).toMatchSnapshot()
})
```

## Gérer les valeurs dynamiques

```typescript
// Mock du temps pour snapshots consistants
beforeEach(() => {
  vi.useFakeTimers()
  vi.setSystemTime(new Date('2025-01-01T12:00:00Z'))
})

afterEach(() => {
  vi.useRealTimers()
})

// Property matchers pour IDs générés
it('crée un user avec champs générés', () => {
  const user = createUser({ name: 'John' })

  expect(user).toMatchSnapshot({
    id: expect.any(String),
    createdAt: expect.any(Date),
    updatedAt: expect.any(Date),
  })
})
```

## Anti-patterns snapshots

- ❌ Snapshots >200 lignes - trop de bruit pour la review
- ❌ `expect(2 + 2).toMatchSnapshot()` - n'apporte rien
- ❌ `vitest -u` sans review - toujours vérifier les diffs
- ❌ Snapshoter les structures internes Reka UI - trop fragiles

---
