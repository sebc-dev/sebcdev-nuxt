# Composants async imbriqués

Pour les composants avec données async imbriqués dans MDC :

```vue
<script setup lang="ts">
const { data } = await useAsyncData('key', () => fetchData(), {
  immediate: true  // Requis pour composants imbriqués
})
</script>

<template>
  <Suspense suspensible>
    <div v-if="data">{{ data }}</div>
  </Suspense>
</template>
```
