# Repository maintenance

Sol Advisor is maintained for upstream version compatibility. Routine work should
respond to a released model or a Codex/plugin version that requires compatibility
changes. Do not add features, roles, routing modes, dependencies, abstractions, or
unrelated refactors unless the user explicitly requests them. Keep each upgrade to
the smallest complete change, including safe migration and verification.

## Before changing a version

1. Start a separate branch from the latest `main` and inspect existing work and open
   PRs so an upgrade is not duplicated or bundled with unrelated changes.
2. Confirm the released version and exact model ID or compatibility requirement in
   official OpenAI documentation and release notes. Record the source URLs in the
   PR. Do not infer a model ID from a display name or assume every model family has
   the same version. A release with no relevant pin or compatibility change may
   require no repository change.
3. Inspect `plugins/sol-advisor/agents/*.toml` for the actual auxiliary pins, then
   search the entire repository, including hidden files, for the old ID, display
   name, and release number. Distinguish active settings from historical migration
   fingerprints and fixtures; do not replace historical values globally.
4. Preserve each role's purpose, reasoning effort, sandbox, and instructions unless
   the release requires a specific compatibility adjustment. If a larger change is
   necessary, explain that requirement in the PR and keep unrelated work out.

## Model upgrade scope

- Auxiliary model IDs are deliberately pinned in the three agent TOMLs. Update only
  the affected family; a new Sol release does not imply a Luna or Terra upgrade.
- The primary-session policy uses the latest available high-capability Sol or Astra
  without a fixed generation. For Astra releases, check for a concrete pin before
  editing. If none exists and compatibility is unchanged, report that no update is
  needed; do not introduce an Astra pin or a new role.
- Keep active model descriptions consistent in `README.md`,
  `plugins/sol-advisor/.codex-plugin/plugin.json`, and the orchestration skill's
  `SKILL.md`, `references/role-contracts.md`, `references/operations.md`, and
  `agents/openai.yaml` where applicable. Update existing surfaces only when they
  actually mention the affected version or behavior.
- Update the exact model expectations in `plugins/sol-advisor/scripts/verify.sh`.
  Keep unrelated model pins and historical fixtures unchanged.

## Preserve installed-agent migration

Changing a shipped agent TOML also changes the installer's byte-exact contract.
A model-only replacement is incomplete until an existing unmodified installation
can upgrade safely.

1. Before editing, capture the old TOML's SHA-256 from the committed, shipped file.
   For example, from the repository root, substituting the actual shipped commit
   and affected filename:

   ```sh
   git show <shipped-commit>:plugins/sol-advisor/agents/<role-file>.toml | shasum -a 256
   ```

2. Add that immutable fingerprint to
   `plugins/sol-advisor/scripts/install-agents.sh` with its source release/model.
   Retain all previously supported fingerprints. Wire the new fingerprint through
   initial classification, the post-preflight recheck, legacy replacement, and the
   post-install check. If the helper's digest slots are exhausted, extend only the
   necessary plumbing rather than dropping an older supported migration.
3. Permit replacement only for a byte-exact shipped historical template. Preserve
   refusal of modified, unknown, unreadable, nonregular, and symlinked destinations,
   and preserve preflight refusal before any destination file is changed. Never
   bypass these checks to make an upgrade pass.
4. Extend `verify.sh` using the existing fixture and snapshot patterns. Verify the
   new historical fixture's immutable digest, non-mutating rejection by `--check`
   (including the affected `--check-role`), refusal of a modified old profile with
   no partial mutation, and successful migration to the exact current template.
   Keep older migration, clean-install, idempotence, and unsafe-path tests passing.
5. Describe the added historical migration in `references/operations.md`. Keep
   user update instructions aligned with the existing README flow: update plugin,
   reinstall companion roles, then start a fresh task. Do not install into the
   maintainer's own Codex directory as a verification side effect.

## Release and verification

For a shipped model or compatibility change, increment the plugin manifest's patch
version unless a different release version is explicitly required. Update matching
release assertions and labels in `verify.sh` and `references/operations.md` together.
Documentation-only changes, including this guide, do not require a version bump.

Run from the repository root with Python 3.11+ (for `tomllib`), jq, rg, and shasum
available:

```sh
sh plugins/sol-advisor/scripts/verify.sh
git diff --check
git status --short
git diff --stat
```

The verifier uses disposable directories for installation and migration tests.
Review the full diff and remaining old-version matches to ensure they are intentional
historical references. Do not claim live model availability or runtime verification
from static checks; report actual model calls separately if performed.

Open a PR against `main` stating the old/new versions, affected roles, official
sources, migration behavior, verification results, and any untested runtime behavior.
Mention when Astra or another family needs no change. Keep the PR scoped to the
release response, and leave merging to the user unless explicitly requested.
