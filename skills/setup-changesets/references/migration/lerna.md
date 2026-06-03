# Migrate from lerna

Lerna is a monorepo task runner (`lerna run`, `lerna exec`) as well as a release tool — keep it installed and disable only the versioning/publishing parts.

1. Keep lerna installed — `lerna run` and `lerna exec` may still be in use
2. In `lerna.json`, remove the `command.version` and `command.publish` sections if present; keep `packages`, `npmClient`, and any task-runner config
3. Remove any `"lerna publish"` or `"lerna version"` calls from scripts in `package.json`
4. In CI config, find and remove the step running `lerna publish`
5. Secrets: `GH_TOKEN` / `GITHUB_TOKEN` and `NPM_TOKEN` reusable
