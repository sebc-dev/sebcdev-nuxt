# Anti-patterns à éviter

## ❌ Debounce sans maxWait

```typescript
// ❌ Risque d'attente infinie si utilisateur tape constamment
const search = useDebounceFn(async (q) => {
  await fetchResults(q)
}, 300)

// ✅ maxWait garantit une exécution périodique
const search = useDebounceFn(async (q) => {
  await fetchResults(q)
}, 300, { maxWait: 1000 })
```

## ❌ Throttle trop agressif

```typescript
// ❌ 10ms = trop fréquent, inutile
const throttled = useThrottleFn(handler, 10)

// ✅ 100ms = bon équilibre performance/réactivité
const throttled = useThrottleFn(handler, 100)
```

## ❌ Tâches longues sans yield

```typescript
// ❌ Bloque le main thread pendant le traitement
function processAllItems(items: Item[]) {
  items.forEach(item => {
    heavyProcessing(item)
  })
}

// ✅ Yield périodique pour préserver l'INP
async function processAllItems(items: Item[]) {
  for (let i = 0; i < items.length; i++) {
    heavyProcessing(items[i])
    if (i % 5 === 0) await yieldToMain()
  }
}
```

## ❌ Event listeners sans passive

```typescript
// ❌ Bloque potentiellement le scroll
window.addEventListener('scroll', onScroll)

// ✅ Passive = ne bloque jamais le scroll
window.addEventListener('scroll', onScroll, { passive: true })
```

---
