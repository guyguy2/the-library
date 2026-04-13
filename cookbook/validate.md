# Validate Catalog Sources

## Context
Check every entry in the catalog and verify its source is reachable. Useful after moving local files, changing paths, or to catch broken entries before they cause a failed `use`.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before reading:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Read the Catalog
- Read `library.yaml`
- Parse all entries from `library.skills`, `library.agents`, and `library.prompts`

### 3. Check Each Source

For each entry, check the `source` field:

**If source is a local path** (starts with `/` or `~`):
- Resolve `~` to the home directory
- Check if the file exists:
  ```bash
  test -f <resolved_path> && echo OK || echo MISSING
  ```
- Status: `OK` if file exists, `MISSING` if not

**If source is a GitHub URL**:
- Parse the URL to extract: `org`, `repo`, `branch`, `file_path`
  - Browser URL pattern: `https://github.com/<org>/<repo>/blob/<branch>/<path>`
  - Raw URL pattern: `https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>`
- Check using the GitHub API:
  ```bash
  gh api "repos/<org>/<repo>/contents/<file_path>?ref=<branch>" --silent
  ```
- Status:
  - Exit code 0: `OK`
  - Exit code non-zero with "Not Found": `MISSING`
  - Exit code non-zero with auth error: `AUTH_ERROR` (may be private repo)

### 4. Display Results

Format output as a table:

```
## Catalog Validation

| Name | Type | Source | Status |
|------|------|--------|--------|
| gws | skill | github.com/... | OK |
| caveman | skill | /Users/.../SKILL.md | OK |
| fill-pdf | prompt | /Users/.../fill-pdf.md | MISSING |
```

### 5. Summary

At the bottom, show:
```
Sources checked: N
OK: X
MISSING: Y
AUTH_ERROR: Z
```

### 6. Suggest Fixes

If any entries are `MISSING`:
- For local paths: tell the user the file was not found at that path and suggest running `/library add` to update the source
- For GitHub URLs: tell the user to check if the repo/branch/path still exists

If any entries are `AUTH_ERROR`:
- Tell the user the source may be a private repo and to verify `gh auth status` is configured with access
