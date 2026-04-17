# Contributing to tacacs-ng-ui

Thank you for your interest in contributing to **tacacs-ng-ui**! This document outlines the coding conventions, style guidelines, and development workflow for the project.

## Development Setup

Please refer to [development.md](./development.md) for instructions on setting up your local development environment.

---

## Code Style Guidelines

### Python (Backend)

#### Tooling

- **Formatter/Linter**: [Ruff](https://docs.astral.sh/ruff/) — configured in `pyproject.toml`
- **Type Checker**: [mypy](https://mypy-lang.org/) with `strict = true`
- **Pre-commit**: Automatically runs Ruff on staged files

#### Conventions

1. **Type Annotations**: Use modern Python 3.12+ union syntax. Do **not** use `typing.Optional` or `typing.List`.

   ```python
   # ✅ Correct
   name: str | None = None
   items: list[str] = []

   # ❌ Avoid
   from typing import Optional, List
   name: Optional[str] = None
   items: List[str] = []
   ```

2. **Datetime**: Always use timezone-aware datetimes.

   ```python
   # ✅ Correct
   from datetime import datetime, timezone
   now = datetime.now(timezone.utc)

   # ❌ Avoid (deprecated in Python 3.12)
   now = datetime.utcnow()
   ```

3. **String Formatting**: Use f-strings for all string interpolation.

   ```python
   # ✅ Correct
   message = f"Host {host.name} not found"

   # ❌ Avoid
   message = "Host {} not found".format(host.name)
   ```

4. **Logging**: Use the `logging` module. Never use `print()` in production code.

   ```python
   import logging
   logger = logging.getLogger(__name__)

   # ✅ Correct
   logger.info(f"Processing config for {filename}")

   # ❌ Avoid
   print(f"Processing config for {filename}")
   ```

5. **Error Messages**: Ensure error messages match the actual entity being operated on.

   ```python
   # ✅ Correct
   raise HTTPException(status_code=404, detail="Host not found")

   # ❌ Avoid (copy-paste error)
   raise HTTPException(status_code=404, detail="User not found")  # in a host endpoint!
   ```

6. **Docstrings**: Ensure docstrings accurately describe the function's purpose.

   ```python
   # ✅ Correct
   def delete_host(...):
       """Delete a host."""

   # ❌ Avoid
   def delete_host(...):
       """Delete an item."""  # wrong entity name
   ```

7. **Return Types**: All functions must have accurate return type annotations.

   ```python
   # ✅ Correct
   def get_stats(...) -> dict[str, Any]:
       return {"count": 42}

   # ❌ Avoid (lying return type)
   def get_stats(...) -> None:
       return {"count": 42}
   ```

#### Architecture Rules

- **CRUD layer** (`app/crud/`): Should **not** raise `HTTPException`. Raise domain-specific errors or return `None`; let the route layer translate to HTTP responses.
- **Route layer** (`app/api/routes/`): Handles HTTP concerns (status codes, error responses). Delegates business logic to CRUD.
- **Models** (`app/models.py`): Define SQLModel schemas. Keep database models and API schemas in the same module (per SQLModel convention).

---

### TypeScript / React (Frontend)

#### Tooling

- **Formatter/Linter**: [Biome](https://biomejs.dev/) — configured in `biome.json`
- **Indentation**: Spaces (2-space)
- **Quotes**: Double quotes
- **Semicolons**: As needed (ASI-safe)

#### Conventions

1. **React Hooks**: Only call hooks from React components or custom hooks. Never call hooks from plain functions.

   ```typescript
   // ✅ Correct — hook called inside a component
   function MyComponent() {
     const { showErrorToast } = useCustomToast()
   }

   // ❌ Avoid — hook called in a plain function
   export const handleError = (err: ApiError) => {
     const { showErrorToast } = useCustomToast()  // ILLEGAL
   }
   ```

2. **Type Safety**: Avoid `any`. Use proper types or `unknown` when the type is truly not known.

   ```typescript
   // ✅ Correct
   const rules: { required?: string; minLength?: { value: number; message: string } } = {}

   // ❌ Avoid
   const rules: any = {}
   ```

3. **Component Design**: Prefer generic/reusable components over duplicating similar components for each entity.

4. **API Client**: The API client is **auto-generated** from the backend's OpenAPI spec. Do **not** edit files in `src/client/` manually. Run `npm run generate-client` after backend API changes.

---

## Commit Messages

Use descriptive commit messages. When fixing specific issues, reference them:

```
fix: correct error messages in host and config delete endpoints (#42)
feat: add date range filter to AAA statistics dashboard
refactor: split models.py into separate modules
```

---

## Pre-commit Hooks

Pre-commit hooks run automatically on `git commit`. To run them manually:

```bash
pre-commit run --all-files
```

This will:
- Run Ruff (lint + format) on Python files
- Run Biome (check + format) on TypeScript/React files
- Check for trailing whitespace, large files, and YAML/TOML validity

---

## Testing

### Backend

```bash
cd backend
uv run pytest
```

### Frontend (E2E)

```bash
cd frontend
npx playwright test
```

---

## Security

- **Never commit secrets** (passwords, API keys, tokens) to the repository.
- Use `.env.example` with placeholder values for documentation. Keep the actual `.env` in `.gitignore`.
- Report security vulnerabilities per [SECURITY.md](./SECURITY.md).
