# Conventional Commits and Git tooling for Nuxt 4 in 2025

The modern Git commit workflow for Nuxt 4.2.x projects in December 2025 centers on **Husky v9.1.7**, **lint-staged v16.2.7**, and **Commitlint v20.2.0**—all fully compatible with Node 22 LTS and pnpm 10.26+. The Conventional Commits specification remains at stable **v1.0.0**, with no major changes since its stabilization. For your SSG Nuxt 4 project deploying to Cloudflare Pages, the optimal setup requires ESM-compatible configurations using `.mjs` or `.cjs` file extensions, pnpm-specific initialization commands, and environment variables to skip hooks in CI environments.

---

## The Conventional Commits specification remains stable at v1.0.0

The Conventional Commits specification has been stable at version **1.0.0** since its formal release, with no breaking changes in 2024-2025. The specification formally requires only two commit types—`feat` and `fix`—but the community-standard `@commitlint/config-conventional` enforces **11 types**: feat, fix, docs, style, refactor, perf, test, build, ci, chore, and revert.

Each type serves a specific purpose in semantic versioning automation. The `feat` type triggers a **MINOR** version bump (x.Y.z), while `fix` triggers a **PATCH** bump (x.y.Z). Breaking changes can be indicated through two mechanisms: the `!` suffix immediately before the colon (`feat!:` or `feat(scope)!:`) for simple breaking changes, or the `BREAKING CHANGE:` footer for detailed migration instructions. Both can be combined for maximum clarity. Breaking changes trigger a **MAJOR** version bump regardless of the commit type.

Commit message structure follows a strict format: the header line (type, optional scope, description) must stay under **100 characters**, use lowercase type and scope, employ imperative present tense ("add" not "added"), and omit trailing periods. Multi-line commits require blank lines between the header, body, and footer sections. The body explains the "why" rather than the "what," while footers use token-value pairs like `Fixes: #123` or `Reviewed-by: Name`.

---

## Commitlint v20.2.0 brings ESM-first configuration for Node 22

The latest Commitlint release (**v20.2.0**, December 2025) runs natively on Node 22 LTS with pure ESM support. A significant v20 improvement: the `body-max-line-length` rule now **ignores lines containing URLs**, preventing false failures from long documentation links.

For Nuxt 4 projects with `"type": "module"` in package.json, use explicit file extensions to avoid resolution conflicts. The recommended configuration file is `commitlint.config.mjs` for ESM or `commitlint.config.cjs` if you prefer CommonJS syntax in an ESM project. TypeScript configurations work via `commitlint.config.ts` with the `@commitlint/types` package providing full type definitions.

```typescript
// commitlint.config.ts - Production-ready for Nuxt 4
import type { UserConfig } from '@commitlint/types';
import { RuleConfigSeverity } from '@commitlint/types';

const Configuration: UserConfig = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      RuleConfigSeverity.Error,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'build', 'ci', 'chore', 'revert'],
    ],
    'scope-enum': [
      RuleConfigSeverity.Error,
      'always',
      [
        // Nuxt 4 app/ directory
        'app', 'components', 'composables', 'layouts', 'pages', 
        'plugins', 'middleware', 'utils', 'assets',
        // Nuxt 4 server/ directory
        'server', 'api', 'server-routes',
        // Other directories
        'shared', 'content', 'public', 'config', 'types', 'deps',
      ],
    ],
    'scope-case': [RuleConfigSeverity.Error, 'always', 'kebab-case'],
    'header-max-length': [RuleConfigSeverity.Error, 'always', 100],
    'body-max-line-length': [RuleConfigSeverity.Error, 'always', 100],
  },
  ignores: [
    (commit) => commit.startsWith('Merge'),
    (commit) => /^v?\d+\.\d+\.\d+/.test(commit),
  ],
};

export default Configuration;
```

The scope configuration maps directly to Nuxt 4's directory structure, which separates client-side code in `app/` from server-side code in `server/`. Scopes should use **kebab-case** and remain minimal—overly granular scope lists create friction without adding value.

---

## Husky v9 eliminates boilerplate with simplified hook files

Husky **v9.1.7** dramatically simplifies Git hook management compared to v8. The migration removes several patterns: the `husky install` command becomes simply `husky` or `husky init`, the shebang lines and `. husky.sh` sourcing are no longer needed in hook files, and the `HUSKY_SKIP_HOOKS` variable is replaced by `HUSKY=0`.

Installation with pnpm requires the `exec` command:

```bash
# Install and initialize
pnpm add -D husky lint-staged @commitlint/cli @commitlint/config-conventional
pnpm exec husky init

# Create hooks (v9 format - no shebang needed)
echo "pnpm lint-staged" > .husky/pre-commit
echo 'pnpm exec commitlint --edit "$1"' > .husky/commit-msg
```

The `.husky/` directory structure is minimal: just the hook files (`pre-commit`, `commit-msg`) and an internal `_/` directory that's auto-generated. Hook files in v9 are plain shell scripts that execute commands directly—no wrapper functions or sourcing required.

For package.json, add the prepare script that initializes Husky after `pnpm install`:

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

---

## lint-staged v16.2.7 requires careful TypeScript handling

lint-staged **v16.2.7** provides staged-file-only linting, critical for SSG projects where full-project linting would be prohibitively slow. The tool runs as a pure ESM module and supports multiple configuration formats: package.json inline, `.lintstagedrc.mjs`, or `lint-staged.config.mjs`.

For Nuxt 4's `app/` directory structure, configure glob patterns that target your specific directories:

```javascript
// lint-staged.config.mjs
/** @type {import('lint-staged').Configuration} */
export default {
  'app/**/*.{ts,vue}': ['eslint --fix', 'prettier --write'],
  'server/**/*.ts': ['eslint --fix', 'prettier --write'],
  'components/**/*.vue': ['eslint --fix', 'prettier --write'],
  '*.{json,md,yaml,css}': 'prettier --write',
  
  // TypeScript type-checking requires function syntax
  '**/*.{ts,vue}': () => 'nuxt typecheck',
};
```

The function syntax for TypeScript type-checking is essential. When lint-staged passes file arguments to `tsc` or `nuxt typecheck`, TypeScript ignores `tsconfig.json` and uses default settings, causing cryptic errors like `TS17004: Cannot use JSX unless the '--jsx' flag is provided`. Using `() => 'command'` prevents lint-staged from appending file arguments, forcing a full project type check that respects your configuration.

Performance optimization matters for pre-commit hooks. Avoid overlapping glob patterns that cause race conditions, and use the `--concurrent` flag to control parallelism. For large codebases, consider moving type-checking to a pre-push hook instead of pre-commit.

---

## Cloudflare Pages deployment requires HUSKY=0

The CI/CD integration for Cloudflare Pages requires disabling Husky during builds. Set `HUSKY=0` in your Cloudflare Pages environment variables—this prevents the prepare script from attempting to install Git hooks in the build container, which would fail and break your deployment.

For more robust handling, create a conditional install script:

```javascript
// .husky/install.mjs
if (process.env.NODE_ENV === 'production' || process.env.CI === 'true') {
  process.exit(0);
}
const husky = (await import('husky')).default;
console.log(husky());
```

Then update package.json:

```json
{
  "scripts": {
    "prepare": "node .husky/install.mjs"
  }
}
```

Cloudflare Pages build settings should use `nuxt generate` for SSG mode with the output directory set to `.output/public` or `dist`. Set `NODE_VERSION=22` to ensure consistency with your local environment.

Commitlint should **not** run in CI—commit messages are validated locally before pushing, making CI validation redundant. The hooks are purely a developer experience tool.

---

## Common pitfalls destroy developer experience

Several anti-patterns consistently cause problems in Conventional Commits setups. Running full-project linting in pre-commit hooks instead of using lint-staged creates unacceptable delays—a **30-second pre-commit hook** will have developers bypassing it with `--no-verify` within days.

ESM/CJS configuration conflicts manifest as `ERR_REQUIRE_ESM` or `require is not defined` errors. The solution is consistent file extensions: `.mjs` for ESM configs with `export default`, `.cjs` for CommonJS configs with `module.exports`. Never use bare `.js` extensions in projects with `"type": "module"` unless you're certain of the resolution behavior.

Over-complicated scope lists create more friction than value. A scope list of 20+ options makes developers guess which category their change belongs to. Keep scopes to **5-10 top-level categories** maximum, or make the `scope-empty` rule a warning rather than an error to allow scopeless commits.

pnpm-specific issues include phantom dependency problems and hook scripts failing to find commands. Always use `pnpm exec` rather than `npx` in hook files, and ensure your shell initializes nvm/fnm properly by creating `~/.config/husky/init.sh` with your version manager's initialization commands.

The permission denied error on hook files requires making them executable: `chmod +x .husky/pre-commit .husky/commit-msg`. This is especially common after cloning a repository on a fresh machine.

---

## Complete installation sequence for Nuxt 4.2.x

Execute this installation sequence for a working setup:

```bash
# 1. Install all dependencies
pnpm add -D husky lint-staged @commitlint/cli @commitlint/config-conventional @commitlint/types

# 2. Initialize Husky
pnpm exec husky init

# 3. Configure hooks
echo "pnpm lint-staged" > .husky/pre-commit
echo 'pnpm exec commitlint --edit "$1"' > .husky/commit-msg

# 4. Make hooks executable
chmod +x .husky/pre-commit .husky/commit-msg

# 5. Create configuration files (commitlint.config.ts and lint-staged.config.mjs)

# 6. Test the setup
echo "feat: test conventional commits" | pnpm exec commitlint
git add . && git commit -m "chore: configure conventional commits tooling"
```

The complete package.json configuration:

```json
{
  "name": "nuxt4-project",
  "type": "module",
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt generate",
    "prepare": "husky",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "typecheck": "nuxt typecheck"
  },
  "devDependencies": {
    "@commitlint/cli": "^20.2.0",
    "@commitlint/config-conventional": "^20.2.0",
    "@commitlint/types": "^20.2.0",
    "husky": "^9.1.7",
    "lint-staged": "^16.2.7"
  },
  "engines": {
    "node": ">=22"
  }
}
```

---

## Conclusion

The Git commit tooling ecosystem for 2025 has matured into a stable, well-integrated stack. Husky v9's simplified architecture removes the ceremony of previous versions, lint-staged v16 handles ESM projects natively, and Commitlint v20's URL-aware line length rules reduce false positives. For Nuxt 4 specifically, the key insight is matching your configuration file extensions to your project's module system—use `.mjs` or `.cjs` explicitly rather than relying on resolution inference.

The most impactful practice isn't any single tool configuration but maintaining **sub-second pre-commit hooks**. Fast hooks get used; slow hooks get bypassed. Configure lint-staged to run only on staged files, move type-checking to pre-push, and keep scope lists minimal. A lean configuration that developers actually use provides more value than a comprehensive one they circumvent.