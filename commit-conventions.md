
# 🧩 Conventional Commits Cheat Sheet

## Commit Types

| **Type** | **Purpose / When to Use** | **Example Commit Message** |
|-----------|----------------------------|-----------------------------|
| **feat** | Introduces a **new feature or functionality** | `feat(form): add address autocomplete feature` |
| **fix** | Fixes a **bug or defect** | `fix(api): resolve null response issue on fetch` |
| **chore** | Routine maintenance, cleanup, or non-functional updates | `chore(deps): update npm dependencies` |
| **docs** | Changes to **documentation** only | `docs(readme): update setup instructions` |
| **style** | Changes that **don’t affect logic** (formatting, spacing, etc.) | `style(ui): fix indentation and remove unused imports` |
| **refactor** | **Code restructuring** without changing behavior | `refactor(step-4): simplify address validation logic` |
| **perf** | **Performance improvements** | `perf(db): optimize query execution time` |
| **test** | Adding or updating **tests** | `test(api): add unit tests for address endpoint` |
| **build** | Changes that affect **build system or external dependencies** | `build(ci): update webpack configuration` |
| **ci** | Changes to **CI/CD pipelines or configuration** | `ci(pipeline): add staging deployment job` |
| **revert** | **Reverts** a previous commit | `revert: revert feat(form) add autocomplete feature` |

---

## Syntax Structure

| **Component** | **Description** | **Example** |
|----------------|----------------|--------------|
| **Type** | The category of change (`feat`, `fix`, `chore`, etc.) | `feat` |
| **Scope (optional)** | Area/module affected | `(auth)`, `(ui)`, `(form)` |
| **Separator** | Colon and space `: ` | `feat(form):` |
| **Short Summary** | Concise description (≤ 72 chars) | `add address autocomplete field` |
| **Body (optional)** | Detailed explanation of what/why | `- Added validation logic`<br>`- Improved UX for input errors` |
| **Footer (optional)** | Metadata or links (e.g., breaking changes, issue refs) | `BREAKING CHANGE: updated API response format`<br>`Closes #123` |

---

## ✅ Best Practices

| **Rule** | **Description** |
|-----------|----------------|
| **Use imperative mood** | e.g., “add”, “fix”, not “added”, “fixed” |
| **Keep summary short** | Ideally under 72 characters |
| **Include body when needed** | Explain “why” and “how”, not just “what” |
| **Use lowercase for type/scope** | Example: `fix(api): ...` not `Fix(API): ...` |
| **Reference issues or tasks** | Example: `Closes #101`, `Refs JIRA-123` |
| **Mark breaking changes** | Use `BREAKING CHANGE:` in footer |
