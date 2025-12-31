# Visual Design Foundation

## Color System

**Mode : Dark-Only**

### Core Palette
| Token | Value | Usage |
|-------|-------|-------|
| `--background` | `#0A0A0B` | Fond principal |
| `--background-secondary` | `#141415` | Cartes, panneaux |
| `--background-tertiary` | `#1E1E20` | Hover states |
| `--foreground` | `#FAFAFA` | Texte principal |
| `--foreground-muted` | `#A1A1AA` | Texte secondaire |
| `--border` | `#27272A` | Bordures |

### Accent & Functional
| Token | Value | Usage |
|-------|-------|-------|
| `--accent` | `#14B8A6` | Liens, focus, CTA |
| `--success` | `#22C55E` | Confirmations |
| `--warning` | `#EAB308` | Avertissements |
| `--error` | `#EF4444` | Erreurs |

### Pillar Colors
| Pilier | Couleur |
|--------|---------|
| IA | `#8B5CF6` (Violet) |
| Ingénierie | `#3B82F6` (Bleu) |
| UX | `#EC4899` (Rose) |

## Typography System

### Font Families
- **Sans-serif** : Satoshi Variable (moderne, géométrique, variable font)
- **Monospace** : JetBrains Mono (code, ligatures)

### Type Scale
| Token | Size | Line Height | Usage |
|-------|------|-------------|-------|
| `--text-base` | 16px | 1.6 | Corps |
| `--text-lg` | 18px | 1.6 | Lead |
| `--text-2xl` | 24px | 1.4 | H3 |
| `--text-3xl` | 30px | 1.3 | H2 |
| `--text-4xl` | 36px | 1.2 | H1 |
| `--text-5xl` | 48px | 1.1 | Hero |

## Spacing & Layout Foundation

### Spacing Scale (Base 4px)
| Token | Value |
|-------|-------|
| `--space-1` | 4px |
| `--space-2` | 8px |
| `--space-4` | 16px |
| `--space-6` | 24px |
| `--space-8` | 32px |
| `--space-12` | 48px |

### Breakpoints
| Name | Width | Columns |
|------|-------|---------|
| Mobile | < 768px | 1 |
| Tablet | 768-1023px | 2 |
| Desktop | ≥ 1024px | 3 |

### Article Layout (Desktop)
- Content max-width : 720px
- ToC sidebar : 240px
- Gap : 48px
- Container max : 1200px

### Border Radius
| Token | Value |
|-------|-------|
| `--radius-sm` | 4px |
| `--radius` | 6px |
| `--radius-md` | 8px |
| `--radius-lg` | 12px |

## Accessibility Considerations

| Requirement | Standard | Implementation |
|-------------|----------|----------------|
| Text contrast | WCAG AA 4.5:1 | 7.2:1 minimum |
| Focus visible | WCAG 2.1 | 2px ring accent |
| Touch targets | 44px min | 48px mobile |
| Reduced motion | `prefers-reduced-motion` | Supported |
| Screen readers | ARIA labels | All interactive elements |
