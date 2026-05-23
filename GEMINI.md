# GEMINI.md - AI Instructions & Project Constraints

This file provides critical context and operational rules for AI agents (like Gemini CLI) interacting with this repository.

## System Role & Persona
- Operate as a Senior Software Engineer specializing in Hugo.
- Adhere to the branch protection rules and development workflows defined below.

## Technical Stack & Constraints
- **SSG:** Hugo (Extended version is **MANDATORY** for SCSS compilation).
- **Theme:** `hugo-book` (Git Submodule located at `themes/book`).
  - **CRITICAL:** DO NOT modify files inside `themes/book/`. Use overrides in the root `assets/`, `layouts/`, or `static/` directories.
- **Locale:** `ja-JP`.
- **Search:** `BookSearch = true` in `hugo.toml`. Local indexing is enabled.

## Project Structure
- `content/docs/`: Primary documentation directory.
- `archetypes/`: Templates for new content.
- `.github/workflows/deploy.yml`: SFTP-based deployment logic.

## Operational Rules & Workflows

### Branching & PRs
- **Branch Naming:** `feat/`, `fix/`, or `entry/`.
- **Merge Strategy:** **Squash and merge** ONLY.
- **Review Policy:** 
  - At least 1 approval required for non-owners.
  - **Code Owners (@maepon)** have bypass authority for urgent/trivial tasks.
- **Automation:** Dependabot manages `github-actions` and `gitsubmodule` weekly.

### Deployment Process
1. Build via `hugo --minify`.
2. Deploy `public/` directory via SFTP (GitHub Action).
   - **Optimization:** Uses `--size-only` in `SFTP-Deploy-Action` to skip unchanged files and speed up transfer.
3. Triggered on merge to `main` or manual dispatch.

### Code Quality
- Ensure all new pages have correct Front Matter (title, weight, etc.).
- Maintain hierarchical structure in `content/docs/`.
