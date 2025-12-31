# Defining User Experience

## The Defining Experience

**"Trouver du code qui marche, le copier, avancer."**

L'expérience que les utilisateurs décriront à leurs collègues :
> "J'ai trouvé ce blog, le code TypeScript marchait du premier coup. J'ai gagné 3 heures."

Cette expérience définissante guide toutes les décisions UX — chaque élément de design doit réduire le temps entre "j'ai un problème" et "j'ai une solution qui fonctionne".

## User Mental Model

**Parcours Type :**
```
[Problème code] → [Recherche] → [Trouve sebc.dev] → [Scanne TL;DR] → [Copie code] → [Retourne coder]
```

**Attentes par Persona :**

| Persona | Attente Principale | Frustration à Éviter |
|---------|-------------------|---------------------|
| **Lucas** | "Solution qui marche en 2 min" | Code qui ne compile pas |
| **Chloé** | "Comprendre, pas juste copier" | Se sentir dépassée |
| **Maxime** | "ROI clair, pas de bullshit" | Perte de temps |

**Modèle Mental Commun :**
- Le code doit être testable immédiatement
- La structure doit permettre de scanner rapidement
- La profondeur doit être accessible mais optionnelle

## Success Criteria

| Critère | Mesure | Seuil |
|---------|--------|-------|
| **Time-to-value** | Temps jusqu'au copy | < 60s |
| **Code fonctionnel** | Compile du premier coup | 100% |
| **Orientation** | Utilisateur sait où il est | ToC + progression |
| **Compréhension** | Peut expliquer le code | Commentaires inline |
| **Retour** | Visite récurrente | > 15% |

**Signaux de Succès :**
1. Lucas copie, compile, soulagement
2. Chloé comprend, eureka, empowerment
3. Maxime décide, efficacité, respect

## Pattern Analysis

**Patterns Établis Adoptés :**
- Copy button visible (Stripe, GitHub)
- ToC sticky avec highlight (Josh Comeau)
- Barre progression (Medium)
- Answer-First structure (Stack Overflow)
- Badges cliquables (Dev.to)

**Pattern Distinctif : "Pattern Onion"**

Architecture de contenu à couches permettant 3 parcours sur 1 article :

| Couche | Cible | Contenu |
|--------|-------|---------|
| **Hook** | Tous | Titre + TL;DR |
| **Quick Win** | Lucas, Maxime | Code fonctionnel |
| **Pattern** | Chloé, Lucas | Explication structurée |
| **Deep Dive** | Chloé | Sections `<details>` |
| **Comparatifs** | Maxime | Tableaux ROI |

## Experience Mechanics

**1. Initiation (0-5s)**
| Trigger | Système |
|---------|---------|
| Arrive depuis recherche | Page charge < 2.5s (LCP) |
| Première impression | TL;DR visible above-the-fold |
| Orientation | ToC visible, structure claire |

**2. Interaction (5s-3min)**
| Action | Feedback |
|--------|----------|
| Scroll vers code | Code block visible, copy button apparent |
| Clique Copy | "Copié!" 2s, icône checkmark |
| Clique ToC | Smooth scroll, section highlight |
| Ouvre details | Animation slide, chevron rotation |

**3. Completion**
| Persona | Signal Fin | Next Action |
|---------|------------|-------------|
| Lucas | Code copié | Retourne IDE |
| Chloé | Pattern compris | Bookmark/Related |
| Maxime | Décision prise | Applique |
