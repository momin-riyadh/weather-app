Project Development Guidelines

Audience: Advanced contributors to this repository.

Overview
- Stack: Static frontend (HTML/CSS/JS) with Tailwind CSS v4 (standalone CLI).
- Entry point: index.html links to src/assets/css/style.css (prebuilt Tailwind output).
- Styles:
  - src/assets/css/input.css: Tailwind entry (uses @import "tailwindcss" and custom CSS variables).
  - src/assets/css/style.css: generated artifact committed to the repo (Tailwind build output used by index.html).
  - src/output.css: an alternative/lightweight Tailwind build output likely produced during experimentation. Not referenced by index.html.

Build and Configuration
- Dependencies are intentionally minimal to rely on Tailwind’s v4 standalone toolchain. See package.json:
  - dependencies: @tailwindcss/cli, tailwindcss (both ^4.1.13).
- No npm scripts are defined; use npx directly.

Tailwind build commands
- Recommended development command (watch + source maps if desired):
  - npx @tailwindcss/cli -i ./src/assets/css/input.css -o ./src/assets/css/style.css --watch
- One-off build:
  - npx @tailwindcss/cli -i ./src/assets/css/input.css -o ./src/assets/css/style.css
- Notes:
  - index.html expects ./src/assets/css/style.css to exist; ensure you build before opening the HTML if you change utility classes or input.css.
  - Tailwind v4 removes the need for a tailwind.config.js for many cases; custom configuration can still be added if needed, but keep the CLI invocation consistent with the above paths.
  - If you introduce arbitrary values or plugins requiring configuration, check them against v4’s conventions.

Local development flow
1. Install deps once (only if node_modules is missing or after a fresh clone):
   - npm install
2. Start Tailwind in watch mode while editing HTML/CSS:
   - npx @tailwindcss/cli -i ./src/assets/css/input.css -o ./src/assets/css/style.css --watch
3. Open index.html in a browser (no dev server is strictly required). If you prefer a server, any static file server works.

Testing
Given this repo is a static site without a test harness, keep testing lightweight. The current recommendation is a Node-based smoke test that validates core DOM/content expectations in index.html without external libs.

How to add a simple smoke test
- Create a temporary Node script (e.g., scripts/smoke-test.mjs) with the following content:

  import { readFileSync } from 'node:fs';
  import { JSDOM } from 'jsdom';

  // Minimal dependency: install jsdom if not present
  //   npm i -D jsdom

  const html = readFileSync('index.html', 'utf8');
  const dom = new JSDOM(html);
  const doc = dom.window.document;

  // Assert the page links to the built CSS
  const link = doc.querySelector('link[rel="stylesheet"][href="src/assets/css/style.css"]');
  if (!link) throw new Error('Missing stylesheet link to src/assets/css/style.css');

  // Assert the Units dropdown button exists and is wired with expected attributes
  const btn = doc.getElementById('unitsDropdownBtn');
  if (!btn) throw new Error('Units dropdown button not found');
  if (btn.getAttribute('aria-controls') !== 'unitsMenu') throw new Error('unitsDropdownBtn aria-controls must reference unitsMenu');

  // Assert the heading renders expected copy (keeps product language stable)
  const h1 = doc.querySelector('h1');
  if (!h1 || !/How's the sky looking today\?/i.test(h1.textContent)) throw new Error('Expected page headline not found');

  console.log('Smoke test passed.');

- Run the test:
  1) npm i -D jsdom
  2) node scripts/smoke-test.mjs
- Remove the test script afterwards to keep the repo clean, unless you decide to formalize a test suite.

Guidelines for adding more tests
- Keep tests fast and deterministic. Avoid network calls; stub or isolate any data fetching logic into separate modules that can be unit-tested with pure inputs and outputs.
- If you start accumulating tests, add a formal test runner (e.g., Vitest or Jest) and wire an npm script:
  - npm i -D vitest jsdom @vitest/browser @vitest/ui
  - Add to package.json scripts: { "test": "vitest run" , "test:watch": "vitest" }
- Structure:
  - tests/ for unit tests (e.g., DOM manipulation helpers, unit conversion logic).
  - e2e/ for high-level checks (consider Playwright later if interactions become complex).

Code Style and Conventions
- Tailwind usage:
  - Prefer utility-first classes directly in markup. Co-locate custom variables in input.css under @layer base and expand as needed.
  - Use meaningful grouping and ordering in class attributes (layout -> spacing -> color -> state) to maintain readability; do not over-optimize ordering as Tailwind v4 handles conflicts deterministically.
- CSS artifacts:
  - Commit style.css only if you need to allow GitHub Pages or a static host to serve without a build step. If you move to a CI build, consider ignoring generated CSS and let CI produce artifacts.
- Assets & paths:
  - Keep paths relative to the project root consistent with index.html (e.g., src/assets/images/...). Avoid absolute filesystem paths.

Debugging Tips
- Dropdown behavior is implemented inline in index.html. If extending, extract to a small JS module for testability and reuse. Add unit tests around keyboard and click interactions.
- If styles appear missing:
  - Verify that style.css is regenerated from input.css changes.
  - Ensure the class names used in HTML are not dynamically constructed in a way Tailwind cannot detect.
- When introducing new color tokens, extend :root variables in input.css and reference them via Tailwind if you add a config later.

Repo Hygiene
- Temporary test files should not live in the repo. Create them locally for validation and delete them when done (as demonstrated in the smoke test section). Only keep .junie/guidelines.md as a persistent contributor doc.

Last reviewed: 2025-09-09 15:20 local time.
