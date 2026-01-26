# Github actions library

![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)

## Overview
Welcome to the [Rebels'](https://www.rebels.software/) **Github actions library**! This repository provides a set of predefined and parametrizable Github Actions workflows and custom actions for automating CI/CD pipelines across multiple projects which follow LICENSE conditions.

## Available Workflows

### 1. `build-and-test.yaml`
Build and test workflow - the first gate for all operations.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `dotnet-version` | No | `9.0.x` | .NET SDK version |
| `csproj-path` | Yes | - | Path to the .csproj file |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `CODE_COV_TOKEN` | Yes | Codecov token for coverage upload |

**Steps:**
1. Checkout repository
2. Validate NOTICE file (Rebels Software copyright)
3. Setup .NET
4. Restore dependencies
5. Build (-c Release)
6. Run tests with coverage
7. Upload coverage to Codecov

---

### 2. `create-version-tag.yaml`
Creates a version tag based on the version in .csproj file.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `dotnet-version` | No | `9.0.x` | .NET SDK version |
| `csproj-path` | Yes | - | Path to the .csproj file |

**Steps:**
1. Checkout repository
2. Setup .NET
3. Read version from .csproj
4. Check if tag already exists (skip if exists)
5. Create and push tag `v{VERSION}`

---

### 3. `publish-nuget.yaml`
Publishes a NuGet package. Only works for library projects.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `dotnet-version` | No | `9.0.x` | .NET SDK version |
| `csproj-path` | Yes | - | Path to the .csproj file |
| `package-source` | No | `https://api.nuget.org/v3/index.json` | NuGet package source |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `NUGET_API_KEY` | Yes | NuGet API key for publishing |

**Steps:**
1. Checkout repository
2. Check if project is a Library (skip if not)
3. Setup .NET
4. Restore dependencies
5. Build (-c Release)
6. Read version from .csproj
7. Add build number for pre-release versions (rc/alpha/beta)
8. Pack project
9. Push to NuGet

---

## Example Usage

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: ['**']        # Build+test on all branches
    tags: ['v*']            # Publish on tags
  pull_request:
    branches: [main]

jobs:
  # 1. BUILD & TEST - always first, gate for everything
  build-and-test:
    uses: rebels/.github-actions/.github/workflows/build-and-test.yaml@main
    with:
      csproj-path: src/MyLib/MyLib.csproj
    secrets:
      CODE_COV_TOKEN: ${{ secrets.CODECOV_TOKEN }}

  # 2. TAG - only after successful merge to main
  create-tag:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    needs: [build-and-test]  # Waits for build, won't run if it fails
    uses: rebels/.github-actions/.github/workflows/create-version-tag.yaml@main
    with:
      csproj-path: src/MyLib/MyLib.csproj

  # 3. PUBLISH - only on tags
  publish:
    if: startsWith(github.ref, 'refs/tags/v')
    needs: [build-and-test]  # Waits for build
    uses: rebels/.github-actions/.github/workflows/publish-nuget.yaml@main
    with:
      csproj-path: src/MyLib/MyLib.csproj
    secrets:
      NUGET_API_KEY: ${{ secrets.NUGET_API_KEY }}
```

## Requirements

All projects using these workflows must include a `NOTICE` file with a valid copyright statement:
```
Copyright YYYY Rebels Software
```

## License
This project is licensed under the [Apache 2.0 License](LICENSE).

## Contact
For questions or support, open an issue or contact us at [hello@rebels.software](mailto:hello@rebels.software).
