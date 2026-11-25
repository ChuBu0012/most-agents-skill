# 09: Bundling, Packaging, Publishing & CI/CD

To ensure fast startup times and minimal disk footprint, extensions should be bundled with **`esbuild`**, packaged with **`@vscode/vsce`**, and published to both the **Visual Studio Marketplace** and the **Open VSX Registry**.

---

## 1. Bundling with `esbuild`

Bundling combines all TypeScript files and third-party npm packages into a single JavaScript file. This reduces extension startup time from 400ms down to ~20ms by eliminating thousands of individual file system read operations.

### Production `esbuild.js` Script
```javascript
const esbuild = require('esbuild');

const production = process.argv.includes('--production');
const watch = process.argv.includes('--watch');

/** @type {import('esbuild').BuildOptions} */
const buildOptions = {
    entryPoints: ['./src/extension.ts'],
    bundle: true,
    format: 'cjs',
    minify: production,
    sourcemap: !production,
    sourcesContent: false,
    platform: 'node',
    outfile: './dist/extension.js',
    // CRITICAL: 'vscode' module is provided by the Extension Host at runtime!
    external: ['vscode'],
    logLevel: 'info'
};

async function main() {
    if (watch) {
        const ctx = await esbuild.context(buildOptions);
        await ctx.watch();
        console.log('Watching for changes...');
    } else {
        await esbuild.build(buildOptions);
    }
}

main().catch((e) => {
    console.error(e);
    process.exit(1);
});
```

---

## 2. `.vscodeignore` Best Practices

The `.vscodeignore` file works like `.gitignore` to prevent source files, unneeded configs, and heavy dependencies from entering the final `.vsix` package:

```gitignore
.vscode/**
.vscode-test/**
out/**
.vscode-test.mjs
src/**
test-workspace/**
.gitignore
.vscodeignore
tsconfig.json
esbuild.js
**/*.map
node_modules/**
!dist/**
```

*Verification Tip*: Run `npx @vscode/vsce ls` to list every file that will be included in the package before publishing.

---

## 3. Packaging with `@vscode/vsce`

Install the official Visual Studio Code Extension Manager CLI:

```bash
npm install --save-dev @vscode/vsce
```

### Packaging into `.vsix`
```bash
npx @vscode/vsce package
```
This generates `my-extension-1.0.0.vsix`. Users can manually install it via `code --install-extension my-extension-1.0.0.vsix`.

### Pre-Requisites for Packaging
1. `package.json` must include: `publisher`, `name`, `displayName`, `version`, `engines.vscode`.
2. A `README.md` must be present in the root folder.
3. A `LICENSE` file or `"license": "MIT"` in `package.json`.
4. If an icon is specified, it must be at least 128x128 PNG.

---

## 4. Publishing to Marketplaces

### 1. Visual Studio Marketplace
1. Sign in to [Azure DevOps](https://dev.azure.com).
2. Generate a Personal Access Token (PAT) with Scope: **Marketplace (Manage)**.
3. Create a publisher account at [Visual Studio Marketplace Management Portal](https://marketplace.visualstudio.com/manage).
4. Publish:
```bash
npx @vscode/vsce publish -p <YOUR_PERSONAL_ACCESS_TOKEN>
```
To publish a pre-release version:
```bash
npx @vscode/vsce publish --pre-release -p <YOUR_PERSONAL_ACCESS_TOKEN>
```

### 2. Open VSX Registry (VSCodium, Gitpod, Eclipse Theia)
1. Sign in to [Open VSX](https://open-vsx.org) and generate an access token.
2. Publish:
```bash
npx ovsx publish -p <YOUR_OPENVSX_TOKEN>
```

---

## 5. Complete GitHub Actions CI/CD Pipeline

Create `.github/workflows/ci.yml` for automated linting, testing, and continuous deployment:

```yaml
name: CI & Publish

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

jobs:
  test:
    name: Test (${{ matrix.os }})
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Compile & Typecheck
        run: npm run compile

      - name: Run Tests (Linux)
        if: runner.os == 'Linux'
        run: xvfb-run -a npm test

      - name: Run Tests (macOS / Windows)
        if: runner.os != 'Linux'
        run: npm test

  publish:
    name: Publish Release
    needs: test
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Package Extension
        run: npx @vscode/vsce package

      - name: Publish to VS Code Marketplace
        run: npx @vscode/vsce publish -p ${{ secrets.VSCE_PAT }}

      - name: Publish to Open VSX
        run: npx ovsx publish -p ${{ secrets.OVSX_PAT }}

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: '*.vsix'
          generate_release_notes: true
```
