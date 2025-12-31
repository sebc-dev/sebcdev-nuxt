# Tests Manuels avec Lecteurs d'Écran

Les tests automatisés détectent **~30% des problèmes** d'accessibilité. Les 70% restants nécessitent des tests manuels.

## NVDA (Windows, gratuit)

Lecteur d'écran le plus utilisé. Raccourcis essentiels :

| Raccourci | Action |
|-----------|--------|
| **H** | Naviguer entre les titres (headings) |
| **K** | Naviguer entre les liens |
| **B** | Naviguer entre les boutons |
| **F** | Naviguer entre les champs de formulaire |
| **D** | Naviguer entre les landmarks |
| **Insert + F7** | Liste de tous les éléments navigables |
| **Insert + F5** | Liste des liens |
| **Tab** | Élément interactif suivant |
| **Shift + Tab** | Élément interactif précédent |

## VoiceOver (macOS/iOS)

Utilise le modificateur **VO** (Ctrl + Option) pour toutes les commandes.

| Raccourci | Action |
|-----------|--------|
| **VO + U** | Rotor (navigation par type d'élément) |
| **VO + →** / **VO + ←** | Élément suivant/précédent |
| **VO + Cmd + H** | Titre suivant |
| **VO + Cmd + J** | Élément de formulaire suivant |
| **VO + Cmd + L** | Lien suivant |
| **Tab** | Élément interactif suivant |

**Différence clé** : VoiceOver n'a pas de modes browse/focus séparés comme NVDA.

## Checklist Tests Manuels

```markdown
# Tests obligatoires avant release
- [ ] Flux de navigation Tab logique et prévisible
- [ ] Focus trap fonctionnel dans les modales
- [ ] Zones live annoncent les mises à jour dynamiques
- [ ] Messages d'erreur formulaire associés aux champs
- [ ] Texte des liens compréhensible hors contexte visuel
- [ ] Hiérarchie des headings cohérente (H pour naviguer)
- [ ] Images informatives ont un alt descriptif
- [ ] Boutons icône ont un label accessible
- [ ] Skip link fonctionne et mène au contenu principal
```

---
