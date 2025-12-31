# Contrainte fondamentale : un document = une collection

**Limitation critique** (issue nuxt/content#2966) : Un document Markdown ne peut appartenir qu'à **UNE SEULE collection**. Cette contrainte explique pourquoi nous utilisons des collections séparées par locale plutôt qu'une collection unique avec filtre.

| Approche | Faisabilité | Raison |
|----------|-------------|--------|
| Collection unique + filtre locale | ❌ Impossible | Document ne peut pas être dans 2 collections |
| Collections séparées (`articles_fr`, `articles_en`) | ✅ Recommandée | Chaque fichier dans sa collection dédiée |

**Conséquence** : Les fichiers `content/fr/` et `content/en/` appartiennent à des collections distinctes, même si leur structure est identique.
