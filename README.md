# poetry_pep621-poetry-core-2

Repro probe for **TKA-10596** — *Dynamic tool installer has no Poetry 2.4.1 in
supported map; falls back to Poetry 1.6.1.*

## What this reproduces

This is a **PEP 621** Poetry project:

- Dependencies are declared under the `[project]` table (PEP 621), **not** the
  legacy `[tool.poetry]` table.
- `[build-system]` pins `requires = ["poetry-core>=2.4.1"]`.

When Mend's classic repo-integration scanner processes this project:

1. The dynamic tool installer reads the `poetry-core>=2.4.1` build-system
   constraint.
2. The installed Poetry (`1.6.1`) does not satisfy `>=2.4.1`, and no compatible
   version is found in the *installed* or *supported* maps.
3. Versioning is skipped and the scan falls back to Poetry `1.6.1`.
4. Poetry `1.6.1` cannot parse the PEP 621 `[project]` table and
   `poetry install --no-root --without dev` / `poetry show --tree` fail with
   `'name'` (KeyError on `tool.poetry.name`).
5. Resolution returns **0 dependencies** (`totalFail` on
   `PRE_STEP` / `RESOLUTION` / `VERSIONING`).

## Correct expectation

A scanner that supports Poetry 2.4.x should resolve the tree from `poetry.lock`:

- **Direct (2):** `click`, `requests`
- **Transitive:** `certifi`, `charset-normalizer`, `idna`, `urllib3`
  (`colorama` is present in the lock under a `platform_system == "Windows"`
  marker).

The `autotest_config.json` encodes this correct expectation, so the test fails
while the product bug is present.

## Reference

- Jira: https://mend-io.atlassian.net/browse/TKA-10596
- Affected agent version: 26.5.1-hf1
- Original scan tokens: `8c849ab53a584e6282c3223ec5119280`,
  `ad58214cacad4866bafd3f7026659c05`
