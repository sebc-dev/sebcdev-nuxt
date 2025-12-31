# Script Setup Structure Pattern

**Ordre canonique des éléments dans `<script setup lang="ts">`** — obligatoire pour cohérence :

```vue
<script setup lang="ts">
// 1. IMPORTS EXTERNES
import { ref, computed, watch, onMounted } from 'vue'

// 2. IMPORTS COMPOSANTS
import BaseButton from '@/components/ui/BaseButton.vue'

// 3. IMPORTS COMPOSABLES
import { useBlogPost } from '@/composables/useBlogPost'

// 4. IMPORTS TYPES
import type { BlogPost, ApiResponse } from '@/types'

// 5. TYPES LOCAUX
interface Props {
  postId: string
  mode?: 'view' | 'edit'
}

// 6. PROPS & EMITS
const props = withDefaults(defineProps<Props>(), {
  mode: 'view'
})

const emit = defineEmits<{
  save: [data: BlogPost]
  cancel: []
}>()

// 7. COMPOSABLES
const { post, isLoading, refresh } = useBlogPost(() => props.postId)
const route = useRoute()

// 8. STATE RÉACTIF
const isEditing = ref(false)
const formData = ref<BlogPost | null>(null)

// 9. COMPUTED
const canSave = computed(() => !isLoading.value && formData.value !== null)

// 10. WATCHERS
watch(() => props.postId, () => refresh(), { immediate: true })

// 11. MÉTHODES
function handleSave(): void {
  if (!canSave.value || !formData.value) return
  emit('save', formData.value)
}

// 12. LIFECYCLE HOOKS
onMounted(() => {
  console.log('Component mounted')
})

// 13. EXPOSE (optionnel)
defineExpose({ refresh })
</script>
```
