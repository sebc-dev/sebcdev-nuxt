# Limitations SSG + Cloudflare Pages

La détection de langue présente des **limitations importantes en SSG** sur hébergement statique :

| Mécanisme | Disponibilité | Notes |
|-----------|---------------|-------|
| `Accept-Language` header | ❌ Non disponible | Serveur statique = pas d'accès aux headers HTTP |
| `navigator.language` | ✅ Client uniquement | Disponible après hydratation |
| Cookies | ✅ Client uniquement | Cloudflare ne lit pas les cookies pour fichiers statiques |

**Comportement en SSG :**

```
Premier accès sur /
       ↓
HTML statique servi (pas de détection serveur)
       ↓
Hydratation client
       ↓
navigator.language détecté → redirection si nécessaire
       ↓
Cookie posé pour visites futures
```

**Pourquoi `redirectOn: 'root'` est CRITIQUE pour SEO :**

| redirectOn | Comportement Google Crawler | Impact SEO |
|------------|----------------------------|------------|
| `'root'` | Seule `/` redirige | ✅ `/fr/article` indexé correctement |
| `'all'` | Toutes pages redirigent | ❌ Crawler (sans Accept-Language) redirigé partout |

**Recommandation SSG :** Pour les sites où le SEO prime sur l'UX de détection automatique, considérez `detectBrowserLanguage: false` et laissez les utilisateurs choisir via un sélecteur de langue explicite.
