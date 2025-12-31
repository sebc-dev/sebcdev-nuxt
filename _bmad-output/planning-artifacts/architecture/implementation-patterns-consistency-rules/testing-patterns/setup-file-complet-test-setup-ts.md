# Setup File Complet (test/setup.ts)

**Configuration essentielle pour happy-dom et shadcn-vue/Reka UI :**

```typescript
// test/setup.ts
import { vi, afterEach } from 'vitest'
import { enableAutoUnmount, config } from '@vue/test-utils'
import { PropertySymbol } from 'happy-dom'

// ═══════════════════════════════════════════════════════════════
// AUTO-CLEANUP - Démonte automatiquement les composants
// ═══════════════════════════════════════════════════════════════
enableAutoUnmount(afterEach)

afterEach(() => {
  vi.restoreAllMocks()
  vi.clearAllTimers()
  config.global.renderStubDefaultSlot = false
})

// ═══════════════════════════════════════════════════════════════
// TIMER WORKAROUND - Compatibilité happy-dom + Vitest
// ═══════════════════════════════════════════════════════════════
const browserWindow =
  global.document?.[PropertySymbol.ownerWindow] ||
  global.document?.[PropertySymbol.window]

if (browserWindow) {
  global.setTimeout = browserWindow.setTimeout
  global.clearTimeout = browserWindow.clearTimeout
  global.setInterval = browserWindow.setInterval
  global.clearInterval = browserWindow.clearInterval
  global.requestAnimationFrame = browserWindow.requestAnimationFrame
  global.cancelAnimationFrame = browserWindow.cancelAnimationFrame
}

// ═══════════════════════════════════════════════════════════════
// BROWSER API MOCKS - APIs manquantes dans happy-dom
// ═══════════════════════════════════════════════════════════════
Element.prototype.getBoundingClientRect = vi.fn(() => ({
  width: 120, height: 120,
  top: 0, left: 0, bottom: 120, right: 120,
  x: 0, y: 0,
  toJSON: () => {},
}))

global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}))

Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
})

// ═══════════════════════════════════════════════════════════════
// REKA UI SHIMS - Requis pour shadcn-vue composants
// ═══════════════════════════════════════════════════════════════
window.PointerEvent = MouseEvent as typeof PointerEvent

global.DOMRect = class DOMRect {
  x = 0; y = 0; width = 0; height = 0
  top = 0; right = 0; bottom = 0; left = 0
  toJSON() { return this }
}

Element.prototype.scrollIntoView = vi.fn()
Element.prototype.hasPointerCapture = vi.fn(() => false)
Element.prototype.releasePointerCapture = vi.fn()
Element.prototype.setPointerCapture = vi.fn()
```

**Référencer dans vitest.config.ts :**

```typescript
test: {
  setupFiles: ['./test/setup.ts'],
  // ...
}
```

---
