# Upgrade Changesets CLI v2 → v3

Go forward. Pinning `changesets/action` back to `@v1` is a fallback only if a floor below blocks v3, and it leaves the repo exposed to the npm 12 republish bug (see the pairing table in SKILL.md Step 5).

1. **Check the floors** — the only things that can block the upgrade:

   | Requirement | v3 floor |
   |---|---|
   | Node | `^22.11 \|\| ^24 \|\| >=26` |
   | npm | `>=10.9.0` |
   | pnpm | `>=10.0.0` |
   | yarn | `>=4.5.2` — Yarn Classic support dropped entirely |

   `"engines": { "node": ">=22" }` nominally allows 22.0, below the floor. Check what CI actually pins before widening it.

2. **Bump both pins** — `"@changesets/cli": "^3.0.0"` in `package.json`, and the `.changeset/config.json` `$schema` to `@changesets/config@4.0.0`.

3. **Bump the action to `@v2` and rename its inputs** (pairing table, SKILL.md Step 5). If the workflow is shared from `<org>/.github`, it is often already on v2 — that is what turned the repo red, and only the CLI side needs fixing.

4. **Walk the breaking changes against what the repo actually uses.** Most repos need no config migration beyond the schema URL; every common key (`changelog`, `commit`, `fixed`, `linked`, `access`, `baseBranch`, `updateInternalDependencies`, `ignore`) survives into config v4 unchanged.

   | Change | What to do |
   |---|---|
   | `prettier` option removed | Replaced by `format`: `"auto"` (default, detects the project's formatter), `"prettier"`, `"oxfmt"`, `"deno"`, `"dprint"`, or `false`. Repos on biome or another unsupported formatter leave it at `auto` — it finds nothing and skips. |
   | Private packages no longer versioned by default | Set `privatePackages: true` (or `{ version, tag }`) if you relied on bumps or tags for them. A private package already in `ignore` is unaffected. |
   | `changeset tag` → `changeset git-tag` | Grep scripts and CI for `changeset tag`. |
   | `--sinceMaster` removed from `changeset status` | Use `--since=main`. |
   | `changeset version` exits 1 when no changesets | Breaks `changeset version && <next>` chains run outside the action. The action only invokes the version script when changesets exist. |
   | `snapshot.useCalculatedVersion` | Replaces the removed `___experimentalUnsafeOptions_WILL_CHANGE_IN_PATCH.useCalculatedVersionForSnapshots`. |
   | Prerelease changesets move to `.changeset/pre/` | `pre.json`'s `initialVersions` is gone. Only matters mid-prerelease — exit the prerelease first if you can. |
   | Peer dependency bumps now `patch`, not `major` | Expect smaller bumps for packages with peer dependents. |
   | Published as ESM | Fine for CLI use; matters only if something `require()`s changesets packages programmatically. |

5. **Grep for other consumers.** Scripts wrapping `changeset version`, or reading the versions it decides, run inside the same `version` script and break silently rather than loudly.

6. **Prove the pending changesets survive.** `changeset status` only parses them — run the real thing and throw it away:

   ```bash
   <pm> exec changeset status   # parses every pending changeset, prints the bumps
   <pm> run version             # full version + changelog + any wrapper scripts
   git diff                     # confirm versions, changelog, derived files
   git checkout -- .changeset packages
   ```

Commit the CLI bump, the schema URL, and the lockfile. A devDependency-only change needs no changeset of its own.
