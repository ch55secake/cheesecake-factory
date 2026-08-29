# cheesecake-factory

Reusable GitHub workflows for my repositories.

## Go Build and Test

Call the workflow from a normal `pull_request` or `push` workflow:

```yaml
jobs:
  go-build:
    uses: ch55secake/cheesecake-factory/.github/workflows/go-build.yml@main
    with:
      build-command: make compile
      test-command: make test
  go-lint:
    uses: ch55secake/cheesecake-factory/.github/workflows/go-lint.yml@main
```

The build, test, and lint jobs run independently. Each Go job caches modules and build
artifacts. The default commands are `go build ./...` and `go test ./...`; the test job
also runs `go test -cover ./...`.

## Go Lint

The workflow reads the consuming repository's `go.mod` and golangci-lint configuration.
The linter version and arguments can be overridden with `golangci-lint-version` and
`args`.

## UV Build and Test

```yaml
jobs:
  python-ci:
    uses: ch55secake/cheesecake-factory/.github/workflows/uv-build.yml@main
    with:
      python-version: '3.12'
      smoke-command: uv run hypertension --help
```

The workflow runs independent format, lint, build, and test jobs. Each job performs a
frozen UV install with dependency caching. The test job runs pytest and the optional
smoke command.

## Go Vulnerability Check

The reusable workflow runs Go's official `govulncheck` tool against the consuming
repository. It uses the Go version from `go.mod` and enables dependency and build
caching by default.

```yaml
jobs:
  go-vuln:
    uses: ch55secake/cheesecake-factory/.github/workflows/go-vuln.yml@main
    with:
      go-package: ./...
```

Use `go-version-input` or `go-version-file` to select the Go version, and use
`work-dir` and `cache-dependency-path` for Go modules outside the repository root. The
default text output fails the job when a vulnerability is found.

## SonarQube

The reusable workflow supports both SonarQube Cloud and SonarQube Server. The
consuming repository must contain a `sonar-project.properties` file and configure a
`SONAR_TOKEN` secret.

```yaml
jobs:
  sonarqube:
    uses: ch55secake/cheesecake-factory/.github/workflows/sonarqube.yml@main
    with:
      sonar-host-url: ${{ vars.SONAR_HOST_URL }}
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_ROOT_CERT: ${{ secrets.SONAR_ROOT_CERT }}
```

`sonar-host-url` is only needed for SonarQube Server. `project-base-dir` and scanner
`args` can be supplied for projects that do not use the repository root defaults.

## Pull Request Labeler

The labeler is reusable, but the consuming repository owns its label rules. Call it
from `pull_request_target` so it can write labels on pull requests from forks:

```yaml
name: Pull Request Labeler

on:
  pull_request_target:
    types: [opened, reopened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  label:
    uses: ch55secake/cheesecake-factory/.github/workflows/labeler.yml@main
    with:
      configuration-path: .github/labeler.yml
```

Keep the referenced configuration in the consuming repository at
`.github/labeler.yml`, or pass a different `configuration-path`.

For reproducible CI, replace `@main` with a release tag or commit SHA when consuming
these workflows.
