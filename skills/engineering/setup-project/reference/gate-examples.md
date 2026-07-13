# Gate config examples

Pin versions are placeholders — look up the latest tag for each tool before using.

## TypeScript / JavaScript

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/biomejs/pre-commit
    rev: vX.Y.Z
    hooks:
      - id: biome-check
        args: ["--write"]
  - repo: https://github.com/gitleaks/gitleaks
    rev: vX.Y.Z
    hooks:
      - id: gitleaks
  - repo: local
    hooks:
      - id: tsc
        name: type-check
        entry: pnpm tsc --noEmit
        language: system
        pass_filenames: false
```

## Python

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: vX.Y.Z
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: https://github.com/gitleaks/gitleaks
    rev: vX.Y.Z
    hooks:
      - id: gitleaks
  - repo: local
    hooks:
      - id: mypy
        name: type-check
        entry: mypy .
        language: system
        pass_filenames: false
```

## Go

```yaml
repos:
  - repo: https://github.com/golangci/golangci-lint
    rev: vX.Y.Z
    hooks:
      - id: golangci-lint
  - repo: https://github.com/gitleaks/gitleaks
    rev: vX.Y.Z
    hooks:
      - id: gitleaks
  - repo: local
    hooks:
      - id: gofmt
        name: format check
        entry: gofmt -l -w .
        language: system
        pass_filenames: false
```
