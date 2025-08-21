# CI/CD

Build and deploy docs and apps with GitHub Actions.

## Docs to GitHub Pages

```yaml
name: docs
on:
  push:
    branches: [ main ]
    paths: [ 'bond-docs/**', 'bond-docs/mkdocs.yml' ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

## App builds (matrix)

```yaml
name: build
on: [push]
jobs:
  android:
    runs-on: ubuntu-latest
    strategy:
      matrix: { flavor: [production, staging] }
    steps: []
```
