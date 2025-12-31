# Aspect-ratio CSS Moderne

En **décembre 2025**, `aspect-ratio` CSS bénéficie d'un support navigateur de **95.33%**. Aucun fallback padding-bottom n'est nécessaire.

## Classes natives et ratios custom

```css
/* app/assets/css/main.css */
@import "tailwindcss";

@theme {
  /* Ratios personnalisés pour le blog */
  --aspect-retro: 4 / 3;
  --aspect-cinema: 21 / 9;
  --aspect-instagram-feed: 4 / 5;
  --aspect-story: 9 / 16;
  --aspect-og-image: 1200 / 630;
}
```

## Usage dans les templates

```html
<!-- Classes natives TailwindCSS 4 -->
<img class="aspect-video w-full object-cover" src="/cover.jpg" />
<div class="aspect-square bg-muted">Carré</div>
<iframe class="aspect-[21/9] w-full" src="https://youtube.com/embed/..."></iframe>

<!-- Ratios custom depuis @theme -->
<img class="aspect-retro w-full" src="/photo.jpg" />
<div class="aspect-story max-w-sm mx-auto">Story format</div>
<div class="aspect-og-image bg-muted">Preview OG Image</div>
```

## Classes aspect-ratio disponibles

| Classe | Ratio | Cas d'usage |
|--------|-------|-------------|
| `aspect-auto` | auto | Ratio intrinsèque de l'image |
| `aspect-square` | 1 / 1 | Avatars, thumbnails carrés |
| `aspect-video` | 16 / 9 | YouTube, Vimeo, vidéos standard |
| `aspect-[4/3]` | 4 / 3 | Photos classiques |
| `aspect-[21/9]` | 21 / 9 | Films cinématographiques |
| `aspect-[9/16]` | 9 / 16 | Stories, Reels, TikTok |

## Pattern avec object-fit

```html
<!-- Image couvrant tout l'espace avec crop -->
<div class="aspect-video">
  <img src="/hero.jpg" class="w-full h-full object-cover" />
</div>

<!-- Image contenue sans crop -->
<div class="aspect-video bg-muted">
  <img src="/diagram.png" class="w-full h-full object-contain" />
</div>
```

---
