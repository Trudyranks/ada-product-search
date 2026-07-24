# AGENTS.md

## Cursor Cloud specific instructions

This repository (`ada-product-search`) currently contains only a Cloud Agent smoke test
(`CLOUD_AGENT_SMOKE_TEST.md`) and a `.gitignore`. There is no application code, no
dependencies, no package manager, and no services to run.

- There is nothing to install: the update script is intentionally a no-op. Do not add
  dependency installation, build, or service-startup steps until real project code
  (e.g. a `package.json`, `requirements.txt`, `composer.json`, etc.) lands in the repo.
- The only "app workflow" today is the smoke test: set `Status: PASSED` in
  `CLOUD_AGENT_SMOKE_TEST.md`, add today's date, then commit and push / open a PR.
- The `.gitignore` references a future car-parts / product-search workflow (e.g.
  `**/carparts-products.csv`, `_carparts_test/`, `_pptx_build/`), but none of that code
  exists yet. When it is added, revisit this file and the update script with real
  install/run instructions.
