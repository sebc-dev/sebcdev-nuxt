---
name: xml-handler
description: Manipuler TOUS les fichiers XML quelle que soit leur taille. TOUJOURS utiliser ce workflow pour tout fichier .xml. Mots-clés: XML, xpath, xmllint, xmlstarlet, extraction, structure, modification.
allowed-tools: Bash(xmllint:*), Bash(xmlstarlet:*), Bash(head:*), Bash(wc:*), Bash(grep:*), Bash(cat:*)
---

# Manipulation des Fichiers XML

## Prérequis

```bash
sudo apt install libxml2-utils xmlstarlet
```

## RÈGLE D'OR

**TOUJOURS** utiliser xmllint/xmlstarlet pour manipuler les fichiers XML :
- Économie de tokens (pas de lecture du contenu dans le contexte)
- Modification directe via XPath
- Ne JAMAIS utiliser Read/Edit sur un fichier XML

---

## Workflow en 3 Étapes

### Étape 1 : Reconnaissance (structure uniquement)

```bash
# Compter les lignes
wc -l fichier.xml

# Aperçu des 100 premières lignes
head -100 fichier.xml

# Arbre des balises (structure hiérarchique)
xmllint --format fichier.xml 2>/dev/null | grep -E "^[[:space:]]*<[^/!?]" | head -50

# Lister toutes les balises uniques
xmlstarlet el fichier.xml | sort -u

# Compter les occurrences de chaque balise
xmlstarlet el fichier.xml | sort | uniq -c | sort -rn
```

### Étape 2 : Extraction Ciblée (lecture minimale)

```bash
# Extraire une section spécifique
xmllint --xpath "//configuration/database" fichier.xml

# Extraire la valeur d'un élément
xmlstarlet sel -t -v "//element/child" fichier.xml

# Extraire un attribut
xmlstarlet sel -t -v "//element/@attribut" fichier.xml

# Extraire avec condition
xmlstarlet sel -t -c "//item[@price>100]" fichier.xml
```

### Étape 3 : Modification Directe (sans lire le fichier)

```bash
# Modifier la valeur d'un élément
xmlstarlet ed -u "//database/host" -v "nouveau-host" fichier.xml > fichier-modifie.xml

# Modifier en place (écrase le fichier)
xmlstarlet ed -L -u "//database/host" -v "nouveau-host" fichier.xml

# Modifier un attribut
xmlstarlet ed -L -u "//element/@attr" -v "nouvelle-valeur" fichier.xml

# Ajouter un élément
xmlstarlet ed -L -s "//parent" -t elem -n "nouveau" -v "contenu" fichier.xml

# Ajouter un attribut
xmlstarlet ed -L -i "//element" -t attr -n "nouvel-attr" -v "valeur" fichier.xml

# Supprimer un élément
xmlstarlet ed -L -d "//element-a-supprimer" fichier.xml

# Renommer une balise
xmlstarlet ed -L -r "//ancien-nom" -v "nouveau-nom" fichier.xml
```

---

## Référence Rapide xmlstarlet ed

| Action | Option | Exemple |
|--------|--------|---------|
| Update | `-u XPATH -v VALUE` | `-u "//host" -v "localhost"` |
| Delete | `-d XPATH` | `-d "//deprecated"` |
| Insert attr | `-i XPATH -t attr -n NAME -v VAL` | `-i "//el" -t attr -n "id" -v "1"` |
| Insert elem | `-s PARENT -t elem -n NAME -v VAL` | `-s "//root" -t elem -n "new" -v ""` |
| Rename | `-r XPATH -v NEWNAME` | `-r "//old" -v "new"` |
| In-place | `-L` | Écrase le fichier original |

## Référence XPath

| Besoin | Expression |
|--------|------------|
| Élément racine | `/*` |
| Tous les X | `//X` |
| Enfant direct | `/parent/enfant` |
| Attribut | `//element/@attr` |
| Valeur texte | `//element/text()` |
| Avec condition | `//item[@price>100]` |
| Premier | `(//item)[1]` |
| Position | `//item[position()<=5]` |

## Validation

```bash
# Vérifier syntaxe XML
xmllint --noout fichier.xml && echo "OK" || echo "INVALIDE"

# Valider contre XSD
xmllint --schema schema.xsd fichier.xml --noout
```

## Formatage

```bash
# Indenter
xmllint --format fichier.xml > formatted.xml

# Compacter
xmllint --noblanks fichier.xml > compact.xml
```
