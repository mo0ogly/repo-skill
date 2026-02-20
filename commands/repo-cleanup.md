# Repo Cleanup & Professionalization

Analyse and professionalize the repository at `$ARGUMENTS` (URL or local path).

## Phase 1 — Detect & Audit

1. Clone the repo if it's a URL, or cd into the local path
2. **Detect the primary language** by checking for:
   - `go.mod` → **Go**
   - `pyproject.toml` / `setup.py` / `requirements.txt` / `*.py` → **Python**
   - `package.json` → **JavaScript/TypeScript**
   - `Cargo.toml` → **Rust**
   - `pom.xml` / `build.gradle` → **Java/Kotlin**
   - Multiple languages → treat as polyglot, apply each relevant section
3. Audit the current state:
   - `.gitignore` completeness for detected language(s)
   - Tracked files that shouldn't be (logs, databases, secrets, IDE configs, build artifacts, result files, cache files)
   - Hardcoded developer paths (grep for `/home/`, `/Users/`, `C:\Users\`)
   - Debug/fix/temp scripts (e.g., `fix_*.py`, `quick_fix.sh`, `test_debug.*`)
   - Dead compatibility code (Python 2 shims, old polyfills)
   - Packaging state (modern vs legacy)
   - Existing tests, CI, linter config, community files
   - Lint error count (language-appropriate linter)
   - Stale references in README (URLs pointing to upstream/original repo if it's a fork, wrong badges, outdated descriptions)
   - Version mismatches (e.g., go.mod version vs CI version, engines field vs CI matrix)
4. **Present a summary of findings to the user and wait for approval before proceeding**

## Phase 2 — Git Cleanup

1. Write a comprehensive `.gitignore` adapted to the detected language:

   **Common (always include):**
   ```
   *.log
   logs/
   .env
   .env.*
   .vscode/
   .idea/
   *.code-workspace
   .DS_Store
   Thumbs.db
   *.swp
   *.swo
   *~
   ```

   **Python additions:**
   ```
   __pycache__/
   *.py[cod]
   *$py.class
   dist/
   build/
   *.egg-info/
   venv/
   .venv/
   .ruff_cache/
   .mypy_cache/
   .pytest_cache/
   .coverage
   .coverage.*
   htmlcov/
   coverage.xml
   *.db
   ```

   **Go additions:**
   ```
   build/
   bin/
   *.exe
   *.dll
   *.so
   *.dylib
   vendor/
   results/
   *.csv
   data/
   ```

   **JS/TS additions:**
   ```
   node_modules/
   dist/
   build/
   .next/
   .nuxt/
   coverage/
   .turbo/
   ```

   **Rust additions:**
   ```
   target/
   Cargo.lock  (for libraries only)
   ```

2. `git rm --cached` all files that should not be tracked
3. Delete debug/fix/temp scripts that don't belong in the repo
4. Delete dev-only status docs (e.g., `MIGRATION_SUMMARY.md`, `PROJECT_STATUS.md`)
5. Create `.example` template files for any removed config files (e.g., `config.json.example`)
6. Fix hardcoded paths → use relative paths or config-driven values
7. Fix version mismatches across files

## Phase 3 — Packaging & Linter Config

### Python
1. Create `pyproject.toml`:
   - `[build-system]` setuptools>=69.0
   - `[project]` metadata (name, version, license, requires-python>=3.9, dependencies)
   - `[project.optional-dependencies]` dev = ["pytest", "pytest-cov", "ruff"]
   - `[tool.pytest.ini_options]` testpaths, verbosity
   - `[tool.ruff]` line-length=120, target-version="py39", select=["E","F","W","I"]
   - `[tool.mypy]` python_version="3.9", warn_return_any, ignore_missing_imports
2. Remove `setup.py` if present
3. Clean `requirements.txt` (runtime deps only, remove stdlib packages)

### Go
1. Ensure `go.mod` has correct module path and Go version
2. Create `.golangci.yml` with linters: errcheck, gosimple, govet, ineffassign, staticcheck, unused, gosec, misspell, gofmt
3. Run `go mod tidy`

### JS/TS
1. Ensure `package.json` has correct metadata, scripts (test, lint, build)
2. Add `eslint.config.js` / `.eslintrc` if missing
3. Add `prettier` config if missing

### Rust
1. Ensure `Cargo.toml` metadata is complete
2. Add `clippy.toml` if needed

## Phase 4 — Lint & Code Quality

### Python
1. `ruff check --fix .` for auto-fixable errors
2. Manual fixes: remove Python 2 shims, bare `except:` → `except Exception:`, unused imports/vars, import sorting
3. Target: **0 ruff errors**

### Go
1. `go vet ./...`
2. Add GoDoc comments to all exported identifiers
3. Replace bare `return err` with `fmt.Errorf("context: %w", err)` for error wrapping
4. Config validation function with tests
5. Target: **go vet clean**, **golangci-lint clean** (if installed)

### JS/TS
1. `npx eslint --fix .`
2. `npx prettier --write .`
3. Target: **0 eslint errors**

### Rust
1. `cargo clippy --fix`
2. Target: **0 clippy warnings**

## Phase 5 — Tests

### Python
1. Create `tests/` with `__init__.py`, `conftest.py` (shared fixtures: mock_dns, prevent_network, tmp_output_dir)
2. One test file per source module, 100% mocked (no network, no DNS, no disk side effects)
3. Use `__new__()` for classes with heavy `__init__` (multiprocessing, signals)
4. Target: all tests passing, >=50% coverage

### Go
1. Create `*_test.go` files alongside source files (standard Go convention)
2. Use `t.TempDir()` for filesystem tests, mock HTTP with `httptest.NewServer`
3. Add integration tests (`//go:build integration` or separate `_integration_test.go`)
4. Add benchmarks (`Benchmark*` functions) for performance-critical code
5. Target: all tests passing with `go test ./... -count=1`

### JS/TS
1. Create `__tests__/` or `*.test.ts` files using jest/vitest
2. Mock external dependencies
3. Target: all tests passing

### Rust
1. Add `#[cfg(test)] mod tests` in each module
2. Add integration tests in `tests/` directory
3. Target: `cargo test` passing

## Phase 6 — Refactoring

1. **Identify large files** (>500 lines for Python, >600 lines for Go, >400 lines for JS/TS)
2. Split into focused sub-modules/files by responsibility:
   - Python: create sub-packages (`module/` with `__init__.py` re-exporting for backward compat)
   - Go: split into multiple files in the same package (same `package` declaration)
   - JS/TS: extract into separate modules with barrel exports
3. **Do not change any logic** — only move code and update imports
4. Verify build + tests after refactoring
5. If splitting a Python module into a package, update test patches (e.g., `module.attr` → `module.submodule.attr`)

## Phase 7 — CI & DevOps

1. **Check for existing CI** — adapt rather than overwrite. If no CI exists:

   Create `.github/workflows/ci.yml`:
   - Trigger: push + PR on main/master
   - SHA-pinned actions (e.g., `actions/checkout@<sha> # v4`)
   - `permissions: contents: read`

   **Python CI:**
   - Jobs: lint (ruff) + test (pytest matrix Python 3.9, 3.10, 3.12)
   - Coverage with `--cov-fail-under=50`

   **Go CI:**
   - Jobs: lint (golangci-lint) + test (go test matrix Go 1.21, 1.22)
   - Install platform deps if needed (e.g., `libgl1-mesa-dev` for Fyne)

   **JS/TS CI:**
   - Jobs: lint (eslint) + test (jest/vitest matrix Node 18, 20, 22)

2. Create `.github/workflows/release.yml`:
   - **Python:** build sdist+wheel, create GitHub Release
   - **Go:** multi-platform build matrix (linux/windows/darwin x amd64/arm64), GitHub Release
   - **JS/TS:** npm publish or GitHub Release
   - **Rust:** cargo publish or cross-compiled binaries

3. Create `Dockerfile` + `.dockerignore` (multi-stage build appropriate for the language)

4. Create `.github/dependabot.yml` for the relevant package ecosystem

5. Create `.pre-commit-config.yaml`:
   - Common: trailing-whitespace, end-of-file-fixer, check-yaml, check-json, check-added-large-files, check-merge-conflict
   - **Python:** ruff hook
   - **Go:** golangci-lint hook
   - **JS/TS:** eslint + prettier hooks

## Phase 8 — Community Files

1. `CONTRIBUTING.md` — contribution guidelines adapted to the project's language/tooling
2. `SECURITY.md` — vulnerability disclosure policy
3. `CHANGELOG.md` — version history (or adapt existing)
4. `.github/ISSUE_TEMPLATE/bug_report.md`
5. `.github/ISSUE_TEMPLATE/feature_request.md`
6. `.github/PULL_REQUEST_TEMPLATE.md`

## Phase 9 — README & Documentation

1. **Update `README.md`:**
   - Fix stale references (if fork: update URLs, author, description)
   - Add badges: CI status, language version, license, linter, coverage
   - Accurate description, installation, usage, configuration
   - Sections: contributing, security, credits/attribution
2. **Create `Makefile`** with language-appropriate targets:
   - Common: `install`, `test`, `lint`, `lint-fix`, `clean`, `docs`, `docs-build`
   - **Go:** `build`, `run`, `bench`, `docker-build`, `docker-run`
   - **Python:** `install` (pip install -e ".[dev]"), `test` (pytest --cov)
3. **MkDocs documentation** (`mkdocs.yml` + `docs_src/` or `docs/`):
   - Material theme with dark/light toggle
   - Pages: index, installation, usage, configuration, architecture, API reference, contributing, changelog
4. **GitHub Pages** — `docs/index.html` landing page + `.github/workflows/pages.yml`

## Phase 10 — Enhancements

### Python
- Type hints on all public functions
- Google-style docstrings on all public functions
- Config validation (JSON Schema or Pydantic)
- Unified logging module (structured, no print statements)
- Async migration for I/O-bound code (aiohttp for HTTP, asyncio.gather)

### Go
- GoDoc comments on all exported identifiers (start with identifier name, end with period)
- Error wrapping with `fmt.Errorf("context: %w", err)` throughout
- Config validation function (`Validate() error`)
- Rate limiting for API calls (configurable via config)
- Cache TTL for external API results (configurable)
- CLI mode (headless, flag-based) alongside any existing GUI

### JS/TS
- JSDoc or TypeScript strict mode
- ESM module format
- Environment validation (zod or joi)

## Phase 11 — Security Audit

1. Verify no secrets/logs/databases/results in `git ls-files`
2. Verify `.gitignore` catches all sensitive patterns: `git check-ignore -v config.json *.db *.log`
3. Grep for hardcoded API keys, tokens, passwords in code
4. Verify no bare exception handlers that could swallow errors silently
5. Verify linter reports 0 errors
6. Verify all tests pass
7. Verify `git status` is clean (no untracked sensitive files)

## Rules

- **Detect language first** — never assume Python
- **Launch parallel agents** when tasks are independent (e.g., lint + tests + docs in parallel)
- **Verify build + tests after every phase** that modifies code
- **Never commit unless explicitly asked**
- **Keep backward compatibility** when refactoring (re-export in `__init__.py`, same package name in Go)
- **Adapt existing CI** — don't overwrite if CI already exists, merge improvements
- **Fix stale references** — URLs, badges, author names that point to upstream/original repos
- **Present findings and wait for approval** before starting destructive operations (git rm, file deletions)
- **Respond in the user's language** (match the language they use in conversation)
