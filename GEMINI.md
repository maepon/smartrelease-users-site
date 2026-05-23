# GEMINI.md - SmartRelease U ユーザーズサイト

## Project Overview
This project is the users site for **SmartRelease U**, built using the **Hugo** static site generator. It utilizes the **hugo-book** theme to provide a clean, documentation-focused layout.

- **Main Technology:** [Hugo](https://gohugo.io/) (Extended version required for SCSS)
- **Theme:** [hugo-book](https://github.com/alex-shpak/hugo-book) (Managed as a git submodule)
- **Primary Language/Locale:** Japanese (`ja-JP`)
- **Base URL:** `https://smartrelease-users.net/`

## Building and Running
### Prerequisites
- [Hugo Extended](https://gohugo.io/installation/) version installed on your local machine.

### Local Development
To run the site locally with a live-reloading server:
```bash
hugo server -D
```
- `-D` includes content marked as `draft: true`.

### Build
To generate the static files for production:
```bash
hugo --minify
```
The output will be generated in the `public/` directory.

## Development Workflow
To maintain code quality and ensure stable deployments, we follow these rules for Pull Requests (PRs), reviews, and merging.

### PR and Review Rules
- **Branching:** Create a feature or fix branch (e.g., `feat/`, `fix/`) and open a PR against the `main` branch.
- **PR Description:** Clearly state "what was changed" and "how to verify it."
- **Review Requirement:** At least one approval is generally required before merging.
- **Bypass Authority:** **Code Owners** have the authority to bypass the standard review process and merge directly when necessary (e.g., urgent fixes or trivial updates).

### Merging and Post-Merge
- **Merge Strategy:** Use **Squash and merge** to keep the `main` branch history clean.
- **Cleanup:** Delete the feature/fix branch immediately after a successful merge.

## Deployment
Deployment is automated via **GitHub Actions** (`.github/workflows/deploy.yml`):
1. **Trigger:** When a Pull Request to the `main` branch is closed and merged (or via `workflow_dispatch`).
2. **Process:**
   - Checks out the repository with submodules.
   - Sets up Hugo Extended.
   - Builds the site using `hugo --minify`.
   - Deploys the contents of `public/` to a remote server via **SFTP**.

## Development Conventions
### Content Structure
- **Home Page:** `content/_index.md`
- **Documentation:** Located in `content/docs/`.
  - Adding a new `.md` file to `content/docs/` will automatically add it to the sidebar menu (if `BookMenuBundle = false` in `hugo.toml`).
- **Archetypes:** Use `archetypes/default.md` for new content.

### Style and Configuration
- **Configuration:** Main settings are in `hugo.toml`.
- **Search:** Local search indexing is enabled via `BookSearch = true`.
- **Theme Mode:** Set to `auto` (supports light/dark mode based on system settings).
- **Dependabot:** Configured to check for updates weekly for GitHub Actions and Git submodules (`.github/dependabot.yml`).
- **Code Owners:** Defined in `.github/CODEOWNERS` to manage review requirements and bypass permissions.

### Theme Customization
The theme is a submodule in `themes/book`. Avoid making direct changes inside this directory. If customization is needed, override the theme's files by mimicking its structure in the root directory (e.g., `assets/`, `layouts/`).
