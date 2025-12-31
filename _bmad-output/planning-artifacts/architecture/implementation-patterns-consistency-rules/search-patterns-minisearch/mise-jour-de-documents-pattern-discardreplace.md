# Mise à jour de documents (pattern discard/replace)

MiniSearch n'a **pas de méthode `update()` native**. Pour modifier un document indexé :

```typescript
// Option 1 : discard() + add() (explicite)
miniSearch.discard(documentId)      // Marque comme supprimé
miniSearch.add(updatedDocument)     // Ré-indexe avec le même ID

// Option 2 : replace() (convenience method)
miniSearch.replace(updatedDocument) // Fait discard + add en interne
```

**⚠️ Pour SSG** : Ce pattern est rarement nécessaire car l'index est regénéré à chaque build. Il est utile uniquement pour des applications avec édition de contenu runtime.
