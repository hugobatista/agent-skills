---
name: web-frontend
description: 'Use when building or modifying web frontend code. Covers Vue, Tailwind CSS, Vite, dark/light mode, responsive design, and frontend tooling. For other UI types (desktop, Qt, etc.), use the dedicated skill for that domain.'
author: hugobatista
---

- **Build tool**: [Vite](https://vitejs.dev/) — always fetch latest docs
- **Frontend**: [Vue](https://vuejs.org/) — always fetch latest docs
- **Styling**: [Tailwind CSS](https://tailwindcss.com/) — always fetch latest docs
- **Theming**: implement dark/light mode switching
- **Responsive**: mobile-first layout adapting across all screen sizes
- No comments in template/style blocks for self-explanatory markup

## npm security hardening

- Use `npm ci` over `npm install` for reproducible builds — respects the lockfile exactly, never mutates `package-lock.json`.
- Create an `.npmrc` with `ignore-scripts=true` to block postinstall attacks during install, and `audit-level=high` to surface only high/critical vulnerabilities:
  ```ini
  audit-level=high
  ignore-scripts=true
  ```
- In Docker build stages, use `npm ci --prefer-offline --ignore-scripts`.
- Add `npm audit signatures` in CI pipelines to verify package tarball signing keys against the npm public key, preventing tampered packages from being injected during install.
