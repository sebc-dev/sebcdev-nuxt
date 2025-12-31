# HTTP Security Headers for Nuxt 4.2.x on Cloudflare Pages

For a Nuxt 4.2.x static site deployed on Cloudflare Pages with TailwindCSS 4.x and shadcn-vue, the most effective approach combines Cloudflare's `_headers` file for HTTP headers with the **nuxt-security module** for build-time CSP hash generation. The critical finding: while TailwindCSS 4.x is fully CSP-compliant in production builds, **shadcn-vue components require `style-src 'unsafe-inline'`** due to Radix Vue's extensive use of inline styles for positioning—a known limitation with no current workaround except forking components.

## Cloudflare Pages configuration delivers headers at zero cost

For SSG deployments, the `_headers` file placed in your `public/` directory is the **only method that works without Functions costs**. Headers defined in `nuxt.config.ts` routeRules only apply when Nitro serves requests—irrelevant for purely static hosting. The `_headers` file supports up to **100 rules with 2,000 characters per line**, uses wildcard path matching, and Cloudflare parses it during deployment rather than serving it as a static asset.

```
# public/_headers

/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: SAMEORIGIN
  Referrer-Policy: strict-origin-when-cross-origin
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()
  Cross-Origin-Opener-Policy: same-origin-allow-popups
  Cross-Origin-Resource-Policy: same-origin
  X-XSS-Protection: 0
  Content-Security-Policy: default-src 'self'; script-src 'self' 'strict-dynamic' 'sha256-REPLACE_WITH_HASH'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' https://fonts.gstatic.com data:; connect-src 'self'; frame-ancestors 'none'; base-uri 'none'; form-action 'self'; object-src 'none'; upgrade-insecure-requests

/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

https://:project.pages.dev/*
  X-Robots-Tag: noindex
```

Cloudflare's **Page Rules were deprecated in 2024**—use Transform Rules in the dashboard for header manipulation if you need dynamic values or path-specific overrides without modifying code. Note that Cloudflare **overwrites origin HSTS headers** when HSTS is enabled in the dashboard (SSL/TLS → Edge Certificates), so configure HSTS either in the dashboard or `_headers`, not both.

## The nuxt-security module handles SSG hash generation automatically

The nuxt-security module is **fully compatible with Nuxt 4.x** and provides OWASP-recommended security headers by default. For static site generation, it calculates SHA-256 hashes of inline scripts at build time and can inject CSP via `<meta http-equiv>` tags as a fallback—critical because nonces are impossible for SSG (they require per-request generation).

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  compatibilityDate: '2024-12-30',
  modules: ['nuxt-security'],
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      routes: ['/']
    }
  },
  
  security: {
    nonce: true, // Still needed for dev mode
    ssg: {
      meta: true,           // CSP via <meta> tag fallback
      hashScripts: true,    // Auto-calculate script hashes at build
      hashStyles: false,    // Keep false—use 'unsafe-inline'
      exportToPresets: true // Attempts export to platform headers
    },
    headers: {
      contentSecurityPolicy: {
        'default-src': ["'self'"],
        'script-src': ["'self'", "'strict-dynamic'"],
        'style-src': ["'self'", "'unsafe-inline'"],
        'img-src': ["'self'", "data:", "https:"],
        'font-src': ["'self'", "https://fonts.gstatic.com", "data:"],
        'connect-src': ["'self'"],
        'frame-ancestors': ["'none'"],
        'object-src': ["'none'"],
        'base-uri': ["'none'"],
        'form-action': ["'self'"],
        'upgrade-insecure-requests': true
      },
      crossOriginOpenerPolicy: 'same-origin-allow-popups',
      crossOriginResourcePolicy: 'same-origin',
      referrerPolicy: 'strict-origin-when-cross-origin',
      strictTransportSecurity: {
        maxAge: 63072000,
        includeSubdomains: true,
        preload: true
      },
      xContentTypeOptions: 'nosniff',
      xFrameOptions: 'SAMEORIGIN',
      xXSSProtection: '0',
      permissionsPolicy: {
        camera: [],
        microphone: [],
        geolocation: [],
        payment: [],
        usb: []
      }
    }
  }
})
```

The `ssg.hashScripts: true` option automatically computes hashes for inline scripts generated during static builds, injecting them into the CSP. Keep `hashStyles: false` because **Vue's hydration mechanism can be blocked** when style hashes don't match dynamically generated inline styles. The module also supports per-route configuration through `routeRules.security` for route-specific CSP policies.

## Individual header recommendations reflect 2025 best practices

**X-Content-Type-Options: nosniff** remains essential. This header prevents browsers from MIME-type sniffing, which could transform non-executable uploads into executable scripts. It's part of the Fetch Living Standard and expected by all security auditors. The only valid value is `nosniff`—no configuration needed.

**X-Frame-Options and CSP frame-ancestors should both be set** for defense in depth. Modern browsers prioritize `frame-ancestors` when both are present, but legacy browsers need X-Frame-Options. Use `DENY` / `'none'` for maximum security or `SAMEORIGIN` / `'self'` if your site needs to iframe itself. The deprecated `ALLOW-FROM` directive is unsupported in all modern browsers—use `frame-ancestors` for multiple allowed origins.

**Referrer-Policy: strict-origin-when-cross-origin** is now the browser default (since Chrome 85) and the OWASP recommendation. It sends the full URL for same-origin requests but only the origin for cross-origin HTTPS requests, providing a good balance between privacy and analytics functionality. Never use `unsafe-url` which leaks full URLs including query parameters across all requests.

**Permissions-Policy** uses updated syntax (not the deprecated Feature-Policy format). Empty parentheses `()` disable a feature entirely. For web apps not using device APIs:

```
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=(), bluetooth=(), accelerometer=(), gyroscope=()
```

Note that `interest-cohort=()` for FLoC blocking is **obsolete as of 2025**—Google abandoned FLoC in 2022 and Chrome v109+ ignores this directive entirely.

## CSP for SSG requires hash-based approaches since nonces are impossible

Static site generation fundamentally **cannot use nonces** because nonces require a unique random value generated per HTTP response. Since SSG produces static HTML files at build time, every user receives identical content. The solution is hash-based CSP with `'strict-dynamic'` for script loading chains.

The `'strict-dynamic'` directive allows scripts loaded by trusted (hashed) scripts to execute without their own hashes—essential for Nuxt's chunked module loading. For backwards compatibility with CSP Level 1 browsers, include `'unsafe-inline'` as a fallback (modern browsers ignore it when `'strict-dynamic'` is present):

```
script-src 'sha256-ENTRY_SCRIPT_HASH' 'strict-dynamic' https: 'unsafe-inline'
```

For styles, **`'unsafe-inline'` remains practically necessary** when using shadcn-vue. The `'strict-dynamic'` directive only works for `script-src`, not `style-src`. Additionally, inline `style` attributes (like Vue's `:style` bindings) cannot use nonces at all—CSP Level 2 only supports nonces for `<style>` tags, not attributes.

## TailwindCSS 4.x is CSP-safe but shadcn-vue requires unsafe-inline for styles

**TailwindCSS 4.x (released January 2025)** compiles all utility classes into external CSS files at build time, making it fully CSP-compliant. The new CSS-first configuration with `@theme` blocks generates native CSS custom properties that become part of the external stylesheet. The only exception is the CDN version (`cdn.tailwindcss.com`) which uses runtime style injection—never use this in production.

**shadcn-vue presents significant CSP challenges** documented in GitHub issues #4461 and #4901. Radix Vue (the underlying primitive library) extensively uses inline `style` attributes for positioning, visibility, and animations. Affected components include Toast/Sonner, Sheet, ScrollArea, Dialog, and Tabs. The Radix maintainers have acknowledged this as a known limitation with no current fix prioritized.

| Component | CSP Compatible | Issue |
|-----------|---------------|-------|
| Button, Input, Card | ✅ Yes | CSS classes only |
| Toast/Sonner | ❌ No | Inline positioning styles |
| Dialog, Sheet | ❌ No | Overlay positioning |
| ScrollArea | ❌ No | Radix scroll behavior |
| Tabs | ⚠️ Partial | Indicator positioning |

Vue's `v-show` directive also generates `style="display:none;"` during SSR, which violates strict CSP. Use `v-if` instead when possible, as it removes elements from the DOM rather than hiding them with inline styles.

## HSTS and cross-origin headers complete the security posture

**HSTS (Strict-Transport-Security)** should use **max-age=63072000** (2 years) with `includeSubDomains` and `preload` for production sites. The preload directive enables submission to browser preload lists at hstspreload.org, but **removal takes months**—only enable when fully committed to HTTPS on all subdomains. Cloudflare can manage HSTS via the dashboard (SSL/TLS → Edge Certificates), which provides simpler configuration but overwrites any origin HSTS headers.

For cross-origin headers, the configuration depends on your integration needs:

**Cross-Origin-Opener-Policy** should be `same-origin-allow-popups` rather than `same-origin` if your site uses OAuth flows, payment providers, or social login popups. The stricter `same-origin` policy breaks cross-origin window communication required by these integrations.

**Cross-Origin-Resource-Policy: same-origin** protects your resources from being embedded by other origins. For public CDN assets, use `cross-origin` instead.

**Cross-Origin-Embedder-Policy** should be `unsafe-none` for most static sites. The `require-corp` value enables SharedArrayBuffer access but requires all embedded resources to opt-in via CORP or CORS headers—impractical unless you specifically need high-resolution timers or shared memory.

## Complete production-ready configuration

For your Nuxt 4.2.x + TailwindCSS 4.x + shadcn-vue stack on Cloudflare Pages, use this dual-layer approach:

**public/_headers** (primary enforcement):
```
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: SAMEORIGIN
  Referrer-Policy: strict-origin-when-cross-origin
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=(), accelerometer=(), gyroscope=()
  Cross-Origin-Opener-Policy: same-origin-allow-popups
  Cross-Origin-Resource-Policy: same-origin
  X-XSS-Protection: 0
  Content-Security-Policy: default-src 'self'; script-src 'self' 'strict-dynamic' 'sha256-REPLACE'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src 'self' data: blob: https:; font-src 'self' https://fonts.gstatic.com data:; connect-src 'self'; frame-ancestors 'none'; base-uri 'none'; form-action 'self'; object-src 'none'; upgrade-insecure-requests

/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable
```

**nuxt.config.ts** (build-time hash generation and development):
```typescript
export default defineNuxtConfig({
  compatibilityDate: '2024-12-30',
  modules: ['nuxt-security', '@nuxtjs/tailwindcss'],
  
  nitro: {
    preset: 'cloudflare_pages'
  },
  
  security: {
    nonce: true,
    ssg: { meta: true, hashScripts: true, hashStyles: false },
    headers: {
      contentSecurityPolicy: {
        'default-src': ["'self'"],
        'script-src': ["'self'", "'strict-dynamic'"],
        'style-src': ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
        'img-src': ["'self'", "data:", "blob:", "https:"],
        'font-src': ["'self'", "https://fonts.gstatic.com", "data:"],
        'connect-src': ["'self'"],
        'frame-ancestors': ["'none'"],
        'base-uri': ["'none'"],
        'form-action': ["'self'"],
        'object-src': ["'none'"],
        'upgrade-insecure-requests': true
      }
    }
  }
})
```

After building, check the generated HTML for script hashes and update your `_headers` CSP accordingly. Test your configuration with securityheaders.com and Mozilla Observatory before production deployment.

## Conclusion

The key insight for this stack is the unavoidable security tradeoff with `style-src 'unsafe-inline'`—shadcn-vue/Radix Vue's architecture prevents strict style CSP without significant component modification. Prioritize strict `script-src` with hashes and `'strict-dynamic'` where real XSS protection lies. The nuxt-security module automates hash generation for SSG builds, while Cloudflare's `_headers` file provides zero-cost HTTP header delivery. Use CSP-Report-Only mode during development to identify violations before enforcing, and accept that perfect CSP scores may require abandoning components that rely on inline style attributes.