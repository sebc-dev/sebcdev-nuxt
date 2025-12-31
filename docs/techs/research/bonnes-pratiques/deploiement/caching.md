# Nuxt 4 SSG caching on Cloudflare Pages

Deploying a Nuxt 4 static site on Cloudflare Pages with optimal caching requires a layered strategy: **aggressive 1-year caching for fingerprinted assets**, **revalidation-based caching for HTML**, and **Cloudflare-specific headers to control edge behavior separately from browsers**. The free tier has zero caching limitations—unlimited bandwidth and requests for static assets—making it ideal for this zero-cost deployment.

The critical insight is that Cloudflare Pages already provides sensible defaults (ETags, automatic edge caching until next deployment), but custom `_headers` configuration unlocks significant performance gains. Nuxt 4's automatic content hashing via Vite enables safe "cache forever" strategies for the `/_nuxt/` directory, while HTML requires careful handling to ensure users always see fresh content after deployments.

## Nuxt 4 asset fingerprinting enables aggressive caching

Nuxt 4 automatically fingerprints all bundled assets using Vite/Rollup's content hashing. Output files in `/_nuxt/` include alphanumeric hashes that change when content changes—this is the foundation for safe long-term caching.

**Build output structure** after `nuxt generate`:
```
.output/public/
├── _nuxt/
│   ├── DC5HVSK5.js          # Content-hashed JS chunks
│   ├── entry.6cce99fb.css   # Hashed CSS
│   └── builds/meta/         # Build metadata
├── index.html
├── 404.html
└── _headers                 # Your cache config
```

**Nuxt 4.1+ import maps** prevent cascading hash invalidation—changing one component no longer forces all chunks to rehash. This dramatically improves cache hit rates across deployments.

The `nuxt.config.ts` for full SSG on Cloudflare Pages:

```typescript
export default defineNuxtConfig({
  compatibilityDate: "2024-09-19",
  ssr: true,
  
  nitro: {
    preset: "cloudflare_pages",
    prerender: {
      crawlLinks: true,
      routes: ['/sitemap.xml', '/robots.txt'],
    },
  },
  
  routeRules: {
    '/**': { prerender: true },
  },
})
```

The `cloudflare_pages` preset auto-generates `_routes.json` to route static assets correctly. For pure SSG with zero server functions, all requests serve directly from Cloudflare's edge—no Workers invocations, no request quotas consumed.

## Complete _headers file for production deployment

Place this file at `public/_headers` in your Nuxt project. It will be copied to `.output/public/_headers` during build and parsed by Cloudflare Pages (not served as a static file).

```
# ===========================================
# FINGERPRINTED ASSETS - Cache 1 year
# ===========================================
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable
  Cloudflare-CDN-Cache-Control: max-age=31536000

# ===========================================
# FONTS - Cache 1 year (rarely change)
# ===========================================
/fonts/*
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

/*.woff2
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

/*.woff
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

# ===========================================
# IMAGES - Cache 30 days (non-fingerprinted)
# ===========================================
/images/*
  Cache-Control: public, max-age=2592000

/*.webp
  Cache-Control: public, max-age=2592000

/*.avif
  Cache-Control: public, max-age=2592000

/*.svg
  Cache-Control: public, max-age=2592000

/*.png
  Cache-Control: public, max-age=2592000

/*.jpg
  Cache-Control: public, max-age=2592000

# ===========================================
# HTML PAGES - Always revalidate
# ===========================================
/*.html
  Cache-Control: public, max-age=0, must-revalidate

/
  Cache-Control: public, max-age=0, must-revalidate

# ===========================================
# SECURITY + DEFAULT HEADERS
# ===========================================
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()

# ===========================================
# PREVENT INDEXING PREVIEW DEPLOYMENTS
# ===========================================
https://:hash.:project.pages.dev/*
  X-Robots-Tag: noindex
```

**Key syntax rules**: Each path starts at column 0, headers are indented with 2 spaces. Maximum **100 rules** and **2,000 characters per line**. Wildcards (`*`) match greedily; only one splat allowed per URL.

## CDN-Cache-Control versus Cache-Control headers

Cloudflare provides three distinct headers for cache control, evaluated in this precedence order:

| Header | Controls | Passed downstream? |
|--------|----------|-------------------|
| `Cloudflare-CDN-Cache-Control` | Cloudflare edge only | No |
| `CDN-Cache-Control` | All CDN caches | Yes |
| `Cache-Control` | Browsers + shared caches | Yes |

**Practical application**: When you want browsers to revalidate but Cloudflare's edge to cache longer:

```
/api/data.json
  Cache-Control: max-age=60
  Cloudflare-CDN-Cache-Control: max-age=3600
```

Browsers see 60-second TTL; Cloudflare edge caches for 1 hour. This reduces origin load while ensuring browsers get reasonably fresh data.

The `s-maxage` directive in `Cache-Control` achieves similar CDN-vs-browser differentiation but applies to **all** shared caches (including downstream CDNs). Use `Cloudflare-CDN-Cache-Control` when you want Cloudflare-specific behavior without affecting other caches.

## Edge caching versus browser caching mechanics

**Cloudflare Pages default behavior**: Assets receive `Cache-Control: public, max-age=0, must-revalidate` with ETags. The edge cache stores assets for **1 week per data center**, automatically invalidated on deployment. This means:

- First visitor to a region triggers origin fetch
- Subsequent visitors get edge-cached responses (CF-Cache-Status: HIT)
- Browser still revalidates using ETag (304 responses if unchanged)

**CF-Cache-Status header values** to monitor:

| Status | Meaning |
|--------|---------|
| `HIT` | Served from Cloudflare edge cache |
| `MISS` | Fetched from origin, now cached |
| `BYPASS` | Response has `no-cache` or `private` |
| `DYNAMIC` | HTML (not cached by default) |
| `REVALIDATED` | Stale content revalidated via ETag |

**Critical: HTML is not edge-cached by default**. Cloudflare treats HTML as dynamic content. For pure static sites, this is fine—the origin is Cloudflare Pages itself, which is fast. Adding explicit HTML caching requires Cache Rules in the dashboard (not available via `_headers` alone).

## stale-while-revalidate pattern for HTML

For static sites that update infrequently, `stale-while-revalidate` offers an excellent balance—instant responses with background freshness checks:

```
/*.html
  Cache-Control: max-age=600, stale-while-revalidate=86400
```

This configuration means:
- **0-10 minutes**: Served from cache immediately (fresh)
- **10 minutes to 24 hours**: Served immediately (stale), browser revalidates in background
- **After 24 hours**: Full network fetch required

**Trade-off analysis**: Users may see content up to 10 minutes old on first load, but page loads are instantaneous. For blogs or documentation sites, this staleness is typically acceptable. For e-commerce or real-time content, prefer `no-cache` or very short `max-age`.

The alternative approach `Cache-Control: no-cache` forces revalidation on every request but still leverages ETags for **304 Not Modified** responses—the browser sends a conditional request, and if content hasn't changed, only headers are transferred (~200 bytes vs full HTML).

## Asset-specific cache duration recommendations

| Asset Type | Cache-Control Value | Rationale |
|------------|---------------------|-----------|
| **Fingerprinted JS/CSS** (`/_nuxt/*`) | `public, max-age=31536000, immutable` | URL changes on content change; safe forever |
| **HTML pages** | `max-age=0, must-revalidate` or `no-cache` | Must always serve latest version |
| **Fonts** (WOFF2/WOFF) | `public, max-age=31536000, immutable` | Rarely change; add CORS header |
| **Fingerprinted images** | `public, max-age=31536000, immutable` | URL includes hash |
| **Non-fingerprinted images** | `public, max-age=2592000` | 30 days; balance freshness/performance |
| **SVG icons** | `public, max-age=31536000, immutable` | Typically versioned with build |
| **favicon.ico** | `public, max-age=86400` | 1 day; may change during rebrand |

The `immutable` directive prevents browsers from revalidating even on Shift+Refresh—supported in Firefox 49+, Safari 11+, and modern Chrome/Edge. For fingerprinted assets, this eliminates unnecessary round-trips entirely.

## Common pitfalls and how to avoid them

**1. Caching HTML aggressively without understanding deployment behavior**
Cloudflare Pages invalidates edge cache on every deployment. However, if you set `max-age=86400` on HTML, browsers cache it locally for 24 hours regardless of server-side changes. Users won't see updates until browser cache expires. **Solution**: Use `max-age=0, must-revalidate` or short TTLs with `stale-while-revalidate`.

**2. Missing CORS headers for fonts**
Self-hosted fonts require `Access-Control-Allow-Origin: *` when loaded via CSS `@font-face`. Without this, browsers block the font request. Always include CORS headers in your font rules.

**3. Forgetting to disable conflicting Cloudflare features**
Disable these in Cloudflare dashboard to prevent Nuxt breakage:
- Speed → Optimization → Rocket Loader™
- Speed → Optimization → Mirage
- Scrape Shield → Email Address Obfuscation

**4. Over-relying on CDN purging**
Purging Cloudflare's cache doesn't clear browser caches. If you ship a bug in an HTML file cached for 1 hour, users may experience issues for the full hour. Fingerprinted assets avoid this entirely—new deployments reference new URLs.

**5. Setting `private` on assets that should be CDN-cached**
`Cache-Control: private` bypasses Cloudflare's edge cache entirely. Use `public` for all static assets; reserve `private` only for user-specific content (which shouldn't exist in a pure SSG site).

## Performance validation checklist

After deployment, verify caching behavior using browser DevTools (Network tab) or curl:

```bash
# Check response headers
curl -I https://yoursite.com/_nuxt/DC5HVSK5.js

# Expected response for fingerprinted assets:
# cache-control: public, max-age=31536000, immutable
# cf-cache-status: HIT (after first request)
```

**Core Web Vitals impact**: Proper caching dramatically improves **Largest Contentful Paint (LCP)** by eliminating network latency for cached resources. For repeat visitors, fingerprinted assets load from disk cache (~1ms) instead of network (~50-200ms per resource).

**Free tier confirmation**: Cloudflare Pages free tier has **no bandwidth limits**, **no request limits for static assets**, and **full access to `_headers` configuration**. The only limits are 500 builds/month and 100 header rules—neither affects runtime caching performance.

## Conclusion

Optimal Nuxt 4 SSG caching on Cloudflare Pages follows a simple hierarchy: **immutable 1-year caching for anything with a content hash** (`/_nuxt/*`, fonts, fingerprinted images), **revalidation-based caching for HTML** (`max-age=0` or `stale-while-revalidate`), and **30-day caching for non-versioned assets**. The `Cloudflare-CDN-Cache-Control` header enables independent edge TTLs when needed, though for pure SSG sites, standard `Cache-Control` typically suffices.

The Nuxt 4.1+ import map feature ensures minimal cache invalidation across deployments—only changed chunks get new hashes. Combined with Cloudflare Pages' automatic edge invalidation on deploy, this creates an architecture where both edge and browser caches work efficiently without manual intervention. The complete `_headers` file provided above is production-ready for most Nuxt 4 SSG deployments.