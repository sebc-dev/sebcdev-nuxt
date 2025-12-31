# Core User Experience

## Defining Experience

L'expérience centrale de sebc.dev est la **consommation efficace d'un article technique selon l'objectif de chaque persona**.

Le même article sert 3 parcours distincts :
- **Lucas (Pragmatique)** : TL;DR → Code → Copy → Termine (< 60s)
- **Chloé (Apprenante)** : TL;DR → Pattern → Details → Comprend (< 3min)
- **Maxime (Stratégique)** : Hook → Comparatifs → Décide → Termine (< 90s)

L'architecture "Pattern Onion" permet ces 3 parcours sur le même contenu sans compromis ni dilution.

## Platform Strategy

| Aspect | Décision |
|--------|----------|
| **Plateforme** | Web responsive |
| **Device prioritaire** | Desktop-first (développeurs codent sur desktop) |
| **Input primaire** | Clavier + souris |
| **Touch** | Support secondaire (lecture mobile) |
| **Offline** | Non requis MVP |

**Breakpoints :**
- Desktop : ≥ 1024px (ToC sticky visible, grilles 3 colonnes)
- Tablette : 768px - 1023px (ToC drawer, grilles 2 colonnes)
- Mobile : < 768px (ToC modal, grilles 1 colonne, touch targets 44px)

## Effortless Interactions

| Interaction | Design |
|-------------|--------|
| **Copier du code** | Bouton "Copy" visible en permanence, feedback "Copié!" 2s |
| **Naviguer article** | ToC sticky desktop, highlight section active, scroll smooth |
| **Trouver la réponse** | TL;DR en haut, structure Answer-First, réponse < 100 mots |
| **Filtrer articles** | Sidebar filtres, application immédiate, URL partageable |
| **Changer de langue** | Switch FR/EN préservant position et filtres |
| **Voir progression** | Barre horizontale sticky top + highlight ToC |

## Critical Success Moments

1. **"Ça compile du premier coup"** — Lucas colle le code TypeScript et ça fonctionne. Edge-cases gérés. Confiance établie.

2. **"Je comprends enfin pourquoi"** — Chloé lit la section Pattern et a son moment eureka. Syndrome imposteur diminué.

3. **"Ce gars me fait gagner du temps/argent"** — Maxime trouve le tableau comparatif en 30 secondes. Efficacité respectée.

4. **"C'est instantané"** — Page charge < 2.5s (LCP). L'utilisateur ne perçoit pas d'attente.

5. **"Je peux naviguer au clavier"** — Accessibilité complète. Aucun utilisateur exclu.

## Experience Principles

1. **Answer-First, Always**
   - La valeur est accessible dans les 60 premières secondes
   - TL;DR en haut, code copable immédiatement
   - Pas de friction avant la réponse

2. **Progressive Depth, Not Gates**
   - La profondeur est disponible, jamais obligatoire
   - Sections `<details>` pour approfondir sans forcer
   - Chaque persona contrôle sa profondeur de lecture

3. **Navigation as Orientation**
   - ToC, badges, filtres = outils d'orientation
   - L'utilisateur sait toujours où il est
   - L'utilisateur sait toujours où aller ensuite

4. **Performance is UX**
   - Lighthouse 100/100/100/100 = promesse UX
   - Chaque milliseconde de latence érode la confiance
   - Performance perçue > performance mesurée

5. **Code is Content**
   - Blocs de code = éléments de premier ordre
   - Copy button, syntax highlighting, language badge obligatoires
   - TypeScript strict, commentaires contextuels
