# Skill: replace-project-name

## Responsibility
Replace all occurrences of a placeholder project name with the real project name across the codebase, preserving case variants and skipping non-applicable files.

---

## Input

- `placeholder` (required): The placeholder string currently in the codebase (e.g., `"my-app"`, `"PROJECT_NAME"`, `"MyApp"`)
- `project_name` (required): The real project name in its canonical form (e.g., `"onboard-iq"`)
- `display_name` (required): Human-readable display form of the name (e.g., `"OnboardIQ"`)
- `env_prefix` (required): SCREAMING_SNAKE_CASE prefix for environment variables (e.g., `"ONBOARD_IQ"`)
- `go_package` (required): Lowercase package name for Go imports (e.g., `"onboardiq"`)
- `dry_run` (optional, default: `false`): If `true`, list files that would be changed without modifying them

---

## Steps

1. **Identify placeholder variants.**
   - From `placeholder`, derive the case variants to search for:
     - kebab-case: `<placeholder>` as-is (e.g., `"my-app"`)
     - PascalCase: capitalize each segment (e.g., `"MyApp"`)
     - snake_case: replace hyphens with underscores, lowercase (e.g., `"my_app"`)
     - SCREAMING_SNAKE_CASE: uppercase snake_case (e.g., `"MY_APP"`)
   - Build the replacement mapping:
     - kebab-case placeholder → `project_name` (slug)
     - PascalCase placeholder → `display_name`
     - snake_case placeholder → `go_package`
     - SCREAMING_SNAKE_CASE placeholder → `env_prefix`

2. **Identify files to scan.**
   - Search for any file containing at least one placeholder variant.
   - Include: `.go`, `.md`, `.yaml`, `.yml`, `.json`, `.toml`, `.env.example`, `Makefile`, `Dockerfile`.
   - Exclude the following paths and patterns:
     - `.git/`
     - `vendor/`
     - `node_modules/`
     - Any binary file (images, compiled artifacts)

3. **Check for existing replacements (idempotency guard).**
   - Before replacing, search for `project_name` (the target value) in the identified files.
   - If a file already contains `project_name` but no placeholder remains, skip that file and log: `SKIP <file> — already updated`.
   - If a file contains both the placeholder and `project_name`, it is partially updated — process it and log: `PARTIAL <file> — completing update`.

4. **Apply replacements.**
   - If `dry_run` is `true`: print the list of files that would be modified and the number of replacements per file. Do not modify any file.
   - If `dry_run` is `false`: apply each replacement mapping in order across all identified files.
   - Use whole-word matching where possible to avoid partial replacements (e.g., replacing `"app"` in `"happy"`).

5. **Report results.**
   - List every file modified with the count of replacements made.
   - List every file skipped with the reason.
   - Report the total: files modified, files skipped, total replacements.

---

## Output

```
Replace Project Name Report
───────────────────────────
Placeholder:    <placeholder>
Project Name:   <project_name>
Display Name:   <display_name>
Env Prefix:     <env_prefix>
Go Package:     <go_package>
Dry Run:        <true|false>

Files Modified:
  MODIFIED  <file_path> — <N> replacements
  PARTIAL   <file_path> — <N> replacements (was partially updated)

Files Skipped:
  SKIP      <file_path> — already updated
  SKIP      <file_path> — no placeholder found

Summary:
  Files modified:   <N>
  Files skipped:    <N>
  Total replacements: <N>
```

---

## Validation

- [ ] All 4 placeholder variants (kebab, PascalCase, snake_case, SCREAMING_SNAKE_CASE) were searched for.
- [ ] `.git/`, `vendor/`, and `node_modules/` directories were excluded.
- [ ] No file was modified that already contained the target values without any remaining placeholders (idempotency guard passed).
- [ ] If `dry_run` was `true`, no files were modified.
- [ ] Every modified file is listed in the report with a replacement count.
- [ ] Every skipped file is listed with a reason.
- [ ] Summary totals match the sum of individual file entries.
- [ ] After replacement, run a final scan to confirm zero remaining occurrences of any placeholder variant.

---

## Notes

- Idempotent: running this skill twice with the same inputs must produce the same final file state. The idempotency guard in Step 3 ensures already-updated files are not double-processed.
- This skill modifies files in place. Ensure the working directory is clean (committed or stashed) before running in `dry_run: false` mode.
- Go import paths may embed the module name — check `go.mod` first and update it before replacing other Go files, as `go.mod` sets the canonical module path.
- After replacing, verify the project builds and tests pass. This skill does not run build or test validation — that is the responsibility of the calling playbook.
