# Prettier Setup — Code Formatting for JS, HTML, CSS, JSON, Twig

This prompt installs and configures Prettier for consistent frontend code formatting.

**Prerequisites:** Node.js must be installed on the system.

---

## Step 1: Initialize npm (if not already done)

```bash
npm init -y
```

---

## Step 2: Install Prettier

```bash
npm install --save-dev prettier
```

Add `node_modules/` to `.gitignore` if not already present.

---

## Step 3: Create .prettierrc

Create `.prettierrc` at the project root:

```json
{
    "printWidth": 120,
    "tabWidth": 4,
    "useTabs": false,
    "semi": true,
    "singleQuote": false,
    "htmlWhitespaceSensitivity": "ignore",
    "overrides": [
        {
            "files": "*.twig",
            "options": {
                "parser": "html"
            }
        }
    ]
}
```

**Configuration explanation:**
- `printWidth: 120` — lines wrap at 120 characters (wider than default 80, suitable for modern monitors)
- `tabWidth: 4` — 4-space indentation matching PHP/Symfony conventions
- `useTabs: false` — spaces, not tabs
- `semi: true` — always add semicolons in JS
- `singleQuote: false` — use double quotes (matches Twig/HTML conventions)
- `htmlWhitespaceSensitivity: "ignore"` — cleaner Twig/HTML formatting
- Twig files use the `html` parser (no dedicated Twig parser; HTML is the closest match)

---

## Step 4: Create .prettierignore

Create `.prettierignore` at the project root:

```
vendor/
node_modules/
var/
public/build/
migrations/
*.min.js
*.min.css
```

**What is excluded:**
- `vendor/` — Composer dependencies (not our code)
- `node_modules/` — npm packages
- `var/` — Symfony cache, logs (generated files)
- `public/build/` — compiled assets
- `migrations/` — auto-generated Doctrine migration files (formatting could break them)
- `*.min.js` / `*.min.css` — already minified files

---

## Step 5: Configure VSCode

Create `.vscode/settings.json` at the project root:

```json
{
    "[php]": {
        "editor.defaultFormatter": "junstyle.php-cs-fixer",
        "editor.formatOnSave": true
    },
    "php-cs-fixer.executablePath": "${workspaceFolder}/vendor/bin/php-cs-fixer",
    "php-cs-fixer.config": ".php-cs-fixer.dist.php",
    "php-cs-fixer.onsave": true,

    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "prettier.documentSelectors": ["**/*.{js,html,css,json,twig}"]
}
```

**Required VSCode extensions:**
- `esbenp.prettier-vscode` — Prettier formatter
- `junstyle.php-cs-fixer` — PHP CS Fixer formatter (for PHP files)

Install them from the VSCode extensions marketplace, or add to `.vscode/extensions.json`:

```json
{
    "recommendations": [
        "esbenp.prettier-vscode",
        "junstyle.php-cs-fixer",
        "whatwedo.twig"
    ]
}
```

---

## Step 6: Add npm Scripts

Edit `package.json` to add Prettier scripts:

```json
{
  "scripts": {
    "format": "prettier --write \"templates/**/*.{html,twig}\" \"public/css/**/*.css\" \"public/js/**/*.js\"",
    "format:check": "prettier --check \"templates/**/*.{html,twig}\" \"public/css/**/*.css\" \"public/js/**/*.js\""
  }
}
```

Test the formatter:

```bash
npm run format:check   # check without writing
npm run format         # apply formatting
```

---

## Step 7: Verify Setup

Run Prettier on the project to verify it formats correctly:

```bash
npx prettier --check .
```

Fix any formatting issues:

```bash
npx prettier --write .
```

---

## Notes

- Prettier does NOT format PHP files — that is handled by PHP CS Fixer (configured in [3-GIT.md](3-GIT.md)).
- Twig files are formatted using the `html` parser, which handles Twig syntax reasonably well. Dedicated Twig plugins exist (e.g. `prettier-plugin-twig-melody`) but may conflict with newer Prettier versions — test before adding.
- If using a CI pipeline, add a Prettier check step (see [6-CI.md](6-CI.md)):
  ```yaml
  - name: Check Prettier formatting
    run: npm ci && npm run format:check
  ```
