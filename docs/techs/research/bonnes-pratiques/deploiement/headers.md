# Cloudflare Pages _headers configuration for Nuxt 4 SSG

Configuring proper HTTP headers for a Nuxt 4 static site on Cloudflare Pages requires balancing aggressive caching for fingerprinted assets with immediate revalidation for HTML, while implementing robust security headers. The `_headers` file placed in your `public/` directory provides declarative control over these headers, though with important limitations compared to Workers.

## How Cloudflare Pages _headers syntax works

The `_headers` file uses a straightforward format: URL patterns followed by indented header declarations. Place this file in your Nuxt project's `public/` directory—it gets copied to `.output/public/_headers` during build and parsed by Cloudflare at deploy time.

```
# Basic syntax
/path
  Header-Name: value
  Another-Header: value

/another/path/*
  Header-Name: value
```

**Glob patterns support single asterisk wildcards** (`*`) that greedily match all characters, but only one splat is allowed per pattern. Double asterisk (`**`) recursive matching is not supported—use `/*` instead. Placeholders like `:name` match path segments and can be referenced in header values.

The critical gotcha is **header inheritance**: when a request matches multiple URL patterns, headers accumulate rather than override. If both `/*` and `/_nuxt/*` set `Cache-Control`, the values get comma-joined, breaking your caching strategy. Use the detach syntax (`! Header-Name`) to remove inherited headers when needed:

```
/*
  Cache-Control: no-cache

/_nuxt/*
  ! Cache-Control
  Cache-Control: public, max-age=31536000, immutable
```

**Key limitations** include a maximum of **100 header rules**, **2,000 characters per line** (problematic for complex CSP), and the fact that `_headers` only applies to static assets—SSR responses from Pages Functions require programmatic header setting.

## Caching strategy for hashed assets in _nuxt/

Nuxt generates fingerprinted assets in the `/_nuxt/` directory with content-based hashes in filenames (e.g., `D9xaLgge.js`). Since URL changes whenever content changes, these assets are perfect candidates for immutable caching:

```
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable
```

The `max-age=31536000` directive (exactly one year, the maximum recommended) tells browsers and Cloudflare's edge to serve cached versions without network requests. The **`immutable` directive** prevents unnecessary revalidation during page refreshes—browsers won't send conditional requests for assets they know won't change. This directive has broad support in Firefox, Safari, and Chrome.

Cloudflare's edge respects this header fully, caching assets globally and serving them with `cf-cache-status: HIT`. The `public` directive explicitly permits shared cache storage, important for CDN edge caching.

## HTML files need aggressive revalidation

HTML pages present the opposite caching challenge. As entry points that reference fingerprinted asset URLs, stale HTML would serve outdated references after deployments. Since HTML URLs can't be cache-busted (users navigate to `/about`, not `/about.a3f4.html`), they must always revalidate:

```
/*.html
  Cache-Control: no-cache

/
  Cache-Control: no-cache
```

The `no-cache` directive allows caching but requires origin revalidation before every use. This enables **conditional requests**: browsers send `If-None-Match` with the cached ETag, receiving `304 Not Modified` when content hasn't changed—saving bandwidth while ensuring freshness.

For slightly better user experience at the cost of potential staleness, add `stale-while-revalidate`:

```
/*.html
  Cache-Control: no-cache, stale-while-revalidate=86400
```

This serves cached HTML immediately while revalidating in the background, hiding latency from users. The **86400-second window** (one day) limits how stale content can get before requiring a blocking revalidation.

## Security headers following OWASP guidelines

Modern security headers protect against clickjacking, MIME sniffing attacks, and information leakage. Apply these to all responses via the `/*` pattern:

**Strict-Transport-Security** enforces HTTPS connections. OWASP recommends a **two-year max-age** (63072000 seconds) with subdomain coverage. Only add `preload` if all subdomains support HTTPS—removal from the preload list takes months:

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

**Content-Security-Policy** for static sites without inline scripts can be strict. The `frame-ancestors 'none'` directive supersedes X-Frame-Options for clickjacking protection:

```
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; form-action 'self'; base-uri 'self'; object-src 'none'; upgrade-insecure-requests
```

If Nuxt generates inline styles (common with CSS extraction disabled), you'll need `'unsafe-inline'` in `style-src` or calculate SHA-256 hashes of inline content. Hash-based allowlisting (`'sha256-...'`) is preferred over `'unsafe-inline'` for security.

**X-Frame-Options** remains useful for legacy browsers despite `frame-ancestors` being the modern replacement:

```
X-Frame-Options: DENY
```

**X-Content-Type-Options** prevents MIME sniffing attacks where browsers execute uploaded text files as scripts:

```
X-Content-Type-Options: nosniff
```

**Referrer-Policy** controls information leakage. The `strict-origin-when-cross-origin` value balances functionality with privacy—full URLs for same-origin, origin-only for cross-origin:

```
Referrer-Policy: strict-origin-when-cross-origin
```

**Permissions-Policy** disables browser features your static site doesn't need:

```
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()
```

**X-XSS-Protection should be disabled or omitted entirely**. Chrome removed its XSS Auditor in 2019; the header can actually create vulnerabilities through filter bypass techniques. OWASP recommends:

```
X-XSS-Protection: 0
```

## Blocking search engines on preview deployments

Cloudflare Pages automatically adds `X-Robots-Tag: noindex` to preview deployments, but not to your production `*.pages.dev` URL if you're using a custom domain. Use placeholder patterns to target these URLs:

```
# Block main pages.dev domain
https://:project.pages.dev/*
  X-Robots-Tag: noindex

# Block branch/commit preview URLs
https://:version.:project.pages.dev/*
  X-Robots-Tag: noindex
```

The `:project` and `:version` placeholders match any subdomain segments, ensuring all preview URLs get blocked from indexing while your custom domain remains indexable.

## Nuxt 4 directory structure and build output

Nuxt 4 introduces a new `app/` directory structure, but asset generation remains consistent with Nuxt 3. Running `nuxt generate` produces static output in `.output/public/`:

```
.output/public/
├── _nuxt/
│   ├── [hash].js           # JavaScript chunks
│   ├── entry.[hash].css    # Stylesheets
│   └── builds/meta/        # App manifest
├── index.html
├── _payload.json
└── _headers               # Copied from public/
```

**Place `_headers` in `public/`** (not the project root). Cloudflare Pages build settings should specify:

- **Build command**: `nuxt generate`
- **Output directory**: `.output/public`

For Cloudflare Pages deployment, disable **Rocket Loader**, **Mirage**, and **Email Address Obfuscation** in the Cloudflare dashboard—these features inject scripts that cause hydration errors in Vue applications.

## Complete production-ready _headers file

This configuration combines all best practices for a Nuxt 4 SSG site:

```
# ============================================
# Nuxt 4 SSG - Cloudflare Pages _headers
# ============================================

# Aggressive caching for fingerprinted assets
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

# Static assets in public/ (images, fonts without hashes)
/images/*
  Cache-Control: public, max-age=86400, stale-while-revalidate=604800

/fonts/*
  Cache-Control: public, max-age=31536000, immutable

# HTML pages - always revalidate
/*.html
  Cache-Control: no-cache

# Root index
/
  Cache-Control: no-cache

# Security headers for all responses
/*
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()
  X-XSS-Protection: 0
  Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; form-action 'self'; base-uri 'self'; object-src 'none'; upgrade-insecure-requests

# Prevent indexing of pages.dev URLs
https://:project.pages.dev/*
  X-Robots-Tag: noindex

https://:version.:project.pages.dev/*
  X-Robots-Tag: noindex
```

## Performance impact on Cloudflare's CDN

Proper header configuration significantly impacts edge caching behavior. With `max-age=31536000` on `/_nuxt/*` assets, Cloudflare caches at all **300+ edge locations** worldwide, serving subsequent requests without origin contact. The `cf-cache-status: HIT` response header confirms edge cache hits.

For HTML with `no-cache`, Cloudflare still stores responses but validates freshness via conditional requests. The `Age` header shows seconds since edge caching, resetting on revalidation. Using `s-maxage` allows CDN-specific TTLs:

```
Cache-Control: no-cache, s-maxage=3600
```

This keeps browsers always revalidating while letting Cloudflare edge cache HTML for one hour—useful for high-traffic sites where origin protection matters more than instant propagation.

## Conclusion

The optimal Nuxt 4 SSG headers strategy separates immutable caching for `/_nuxt/*` fingerprinted assets from strict revalidation for HTML entry points. Security headers should follow OWASP's current recommendations: HSTS with preload for committed HTTPS sites, strict CSP with `frame-ancestors` replacing X-Frame-Options, and explicit disabling of the deprecated X-XSS-Protection. The `_headers` file's inheritance behavior requires careful ordering—use the detach syntax to prevent unwanted header accumulation. For sites needing dynamic CSP nonces or complex conditional logic, Cloudflare Workers remains necessary, but the declarative `_headers` approach handles most static site requirements efficiently.