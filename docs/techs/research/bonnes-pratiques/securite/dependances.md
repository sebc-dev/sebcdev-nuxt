# Dependency security best practices for pnpm, Nuxt 4, and Cloudflare Pages

Modern JavaScript supply chain security requires a defense-in-depth approach combining automated auditing, intelligent dependency updates, and runtime protections. For the pnpm 10.26+/Nuxt 4/Cloudflare Pages stack in December 2025, the most effective strategy combines **pnpm audit with audit-ci for CI gating**, **Renovate Bot for automated updates** (preferred over Dependabot for pnpm), and **Socket.dev for supply chain attack detection**. This comprehensive guide provides actionable configurations for each layer of your security pipeline.

## pnpm audit fundamentals and configuration

The pnpm 10.x release series introduced significant security improvements including built-in supply chain protections. Run `pnpm audit --audit-level=high` as your baseline command, which returns exit code 1 when vulnerabilities at or above the specified severity are found—critical for CI integration.

**Key pnpm 10.x audit flags** include `--ignore-unfixable` (added in 10.11.0) to suppress noise from CVEs without available patches, and `--ignore CVE-2022-XXXXX` for explicitly accepting specific vulnerabilities. The `--json` flag enables machine-readable output for SARIF conversion. Unlike npm audit, **pnpm audit --fix doesn't modify the lockfile directly**—it adds entries to `pnpm.overrides` in your configuration, requiring a subsequent `pnpm install` to apply changes.

Configure false positive handling in `pnpm-workspace.yaml` (the new configuration home in pnpm 10.x):

```yaml
# pnpm-workspace.yaml
auditConfig:
  ignoreCves:
    - CVE-2022-36313
  ignoreGhsas:
    - GHSA-42xw-2xvc-qx8m

# Force non-vulnerable versions of transitive dependencies
overrides:
  "lodash@<4.17.21": "^4.17.21"
  "axios@<1.6.0": "^1.6.0"
```

New supply chain protections in pnpm 10.16+ deserve attention: `minimumReleaseAge: 1440` delays installation of packages published within the last 24 hours, while `blockExoticSubdeps: true` (10.26.0+) restricts transitive dependencies to registry-only sources, blocking git URLs and local paths that could introduce malicious code.

## npm-check-updates enables safe dependency upgrades

The npm-check-updates (ncu) tool version 19.x provides granular control over which updates to apply. The **safest workflow progresses through severity levels**: start with `ncu -t patch -u && pnpm install && pnpm test`, then `ncu -t minor -u`, and finally review major updates interactively via `ncu -i --format group --peer`.

Configure `.ncurc.json` for team consistency:

```json
{
  "$schema": "https://raw.githubusercontent.com/raineorshine/npm-check-updates/main/src/types/RunOptions.json",
  "target": "minor",
  "packageManager": "pnpm",
  "reject": ["@types/node", "typescript"],
  "peer": true,
  "cooldown": 3
}
```

The `cooldown` option (set to 3 days here) provides supply chain protection by refusing to suggest packages published within that window—critical for avoiding malicious packages that often get removed within 24-48 hours. Use `ncu --doctor -u --doctorInstall "pnpm install" --doctorTest "pnpm test"` for automated safe updates that test each dependency individually and only keep working upgrades.

## Renovate Bot outperforms Dependabot for pnpm projects

For automated dependency management, **Renovate significantly outperforms Dependabot** in the pnpm ecosystem. While Dependabot added pnpm support in 2023, it still has limitations with pnpm 10.x and workspace catalogs. Renovate offers full pnpm 10 support, native catalog handling, a dependency dashboard, and sophisticated grouping and scheduling.

A production-ready `renovate.json` configuration:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended", ":dependencyDashboard", ":semanticCommitTypeAll(chore)"],
  "timezone": "America/New_York",
  "schedule": ["after 9pm on sunday"],
  "postUpdateOptions": ["pnpmDedupe"],
  "lockFileMaintenance": {
    "enabled": true,
    "automerge": true,
    "schedule": ["before 6am on monday"]
  },
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security", "priority"],
    "automerge": true,
    "schedule": [],
    "minimumReleaseAge": null,
    "prCreation": "immediate"
  },
  "osvVulnerabilityAlerts": true,
  "packageRules": [
    {
      "description": "Automerge non-major dev dependencies",
      "matchDepTypes": ["devDependencies"],
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true,
      "automergeType": "branch",
      "minimumReleaseAge": "1 day"
    },
    {
      "description": "Group Nuxt ecosystem",
      "matchPackageNames": ["nuxt", "@nuxt/**", "nuxi", "@nuxtjs/**"],
      "groupName": "Nuxt packages"
    },
    {
      "description": "Major updates require dashboard approval",
      "matchUpdateTypes": ["major"],
      "dependencyDashboardApproval": true,
      "labels": ["dependencies", "breaking"]
    }
  ]
}
```

The key pnpm-specific settings are `postUpdateOptions: ["pnpmDedupe"]` which runs deduplication after lockfile updates, and enabling `lockFileMaintenance` which regenerates the entire lockfile including transitive dependencies. Set `minimumReleaseAge` to **3-14 days** for third-party production dependencies but disable it entirely for security patches via the `vulnerabilityAlerts` block.

## GitHub Actions workflow for comprehensive security scanning

A complete CI security pipeline should run pnpm audit, generate SARIF reports for GitHub's Security tab, and fail builds appropriately based on vulnerability severity. Use `pnpm/action-setup@v4` with built-in caching:

```yaml
name: Security Pipeline
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 6 * * 1'

permissions:
  contents: read
  security-events: write

env:
  PNPM_VERSION: 10
  NODE_VERSION: 22

jobs:
  dependency-audit:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}
          cache: true
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      
      - name: Security audit with audit-ci
        run: pnpm dlx audit-ci@^7 --config ./audit-ci.jsonc
      
      - name: Generate SARIF report
        if: always()
        run: |
          pnpm audit --json > audit.json || true
          npx npm-audit-sarif -o audit.sarif audit.json
      
      - name: Upload to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: audit.sarif
          category: 'pnpm-audit'
```

The `audit-ci` package from IBM provides superior CI integration with expiring allowlists and path-specific exceptions. Configure `audit-ci.jsonc`:

```json
{
  "$schema": "https://github.com/IBM/audit-ci/raw/main/docs/schema.json",
  "moderate": true,
  "allowlist": [
    "GHSA-xxxx-yyyy-zzzz",
    "*|@testing-library/*>*"
  ],
  "report-type": "important",
  "retry-count": 5
}
```

**Always use `--frozen-lockfile`** in CI to prevent lockfile modifications and ensure deterministic builds. pnpm auto-detects CI environments and enables this by default when `CI=true` is set.

## Socket.dev and OSV-Scanner provide defense in depth

Traditional vulnerability scanners like pnpm audit only detect known CVEs. **Socket.dev detects supply chain attacks** including typosquatting, malicious install scripts, obfuscated code, and credential theft attempts—threats that CVE databases miss entirely. Install the Socket GitHub App at `github.com/apps/socket-security` for automatic PR scanning.

For CLI usage with pnpm: `socket pnpm install` wraps the install command and alerts on suspicious packages. Socket's free tier covers open source projects; paid tiers add function-level reachability analysis that eliminates **60% of CVE false positives** by determining if vulnerable code paths are actually executed.

Google's **OSV-Scanner** provides comprehensive free vulnerability scanning against the OSV.dev database (40,000+ npm vulnerabilities). It natively supports `pnpm-lock.yaml`:

```bash
osv-scanner --lockfile=pnpm-lock.yaml --format=json > results.json
```

For CI integration, use the official reusable workflow:

```yaml
jobs:
  osv-scan:
    uses: google/osv-scanner-action/.github/workflows/osv-scanner-reusable.yml@v2.3.0
    with:
      scan-args: |
        --recursive
        --lockfile=pnpm-lock.yaml
        ./
```

**Snyk** offers pnpm CLI support in Early Access (enable via Settings > Snyk Preview). The key commands are `snyk test` for one-time scanning and `snyk monitor` for continuous dashboard monitoring. Configure a `.snyk` policy file for ignore rules with expiration dates.

## License compliance prevents legal exposure

Use `pnpm licenses list --prod --json` for built-in license scanning, or `license-checker` for CI gating:

```bash
npx license-checker --production --onlyAllow "MIT;ISC;Apache-2.0;BSD-3-Clause;BSD-2-Clause;0BSD;CC0-1.0"
```

For pnpm-specific tooling, `@quantco/pnpm-licenses` generates comprehensive disclaimer files suitable for distribution:

```bash
npx @quantco/pnpm-licenses generate-disclaimer --prod --output-file=THIRD_PARTY_LICENSES.txt
```

Copyleft licenses (GPL, AGPL, LGPL) require careful evaluation as they may impose source disclosure requirements on your project.

## Cloudflare Pages requires build-time security integration

Cloudflare Pages determines build success by exit code, making security audit integration straightforward. Set your build command to `pnpm audit --audit-level=high && nuxi generate` to fail deployments when vulnerabilities are detected. Cloudflare's build infrastructure uses **gVisor container sandboxing**, providing strong isolation during the build process.

Configure Nuxt 4 for Cloudflare Pages in `nuxt.config.ts`:

```typescript
export default defineNuxtConfig({
  ssr: true,
  nitro: {
    preset: 'cloudflare_pages'
  },
  runtimeConfig: {
    apiSecret: process.env.API_SECRET,  // Server-only
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_BASE  // Client-exposed
    }
  }
})
```

**Critical security consideration**: For SSG, environment variables used during `nuxt generate` are baked into static output. Never expose secrets to `runtimeConfig.public`—use server routes (which become Pages Functions) for sensitive operations.

Always use **encrypted secrets** in Cloudflare Pages (select "Encrypt" when adding variables). Secrets are never visible after creation, unlike plaintext environment variables. Enable Cloudflare Access for preview deployments to prevent unauthorized access to `*.project.pages.dev` URLs.

Create `public/_headers` for security headers:

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'
```

## Critical anti-patterns to avoid

The most common security mistakes in this stack undermine your entire security posture:

- **Running `pnpm audit --fix` without `pnpm install`** leaves vulnerabilities unfixed since pnpm only modifies configuration, not the lockfile
- **Using `continue-on-error: true` on all audit steps** silently ignores critical vulnerabilities; use tiered thresholds instead
- **Disabling `minimumReleaseAge` for production dependencies** exposes you to malicious package takeovers—maintain at least 3 days
- **Caching `node_modules` directly** creates large, platform-specific, fragile caches; cache the pnpm store instead
- **Storing secrets in `wrangler.toml` vars** exposes them in version control; use encrypted dashboard secrets or CLI `wrangler secret put`
- **Not disabling Cloudflare Rocket Loader™** causes Nuxt hydration failures; disable it along with Mirage and Email Obfuscation

## Conclusion

Effective dependency security for the pnpm/Nuxt 4/Cloudflare Pages stack in 2025 requires layered defenses. Use **pnpm audit with audit-ci** for CVE detection, **Renovate Bot** for intelligent automated updates with supply chain protections like `minimumReleaseAge`, and **Socket.dev** for detecting malicious packages that evade CVE databases. The GitHub Actions workflow provided generates SARIF reports for Security tab visibility and fails builds appropriately based on severity thresholds.

For Cloudflare Pages specifically, integrate security audits directly into the build command, use encrypted secrets exclusively, and configure proper CSP headers via `_headers`. The combination of build-time auditing, automated dependency updates with safety delays, and continuous monitoring through multiple scanning tools provides comprehensive protection against the evolving JavaScript supply chain threat landscape.