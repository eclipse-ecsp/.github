# GitHub Workflows for ECSP

This repository contains reusable GitHub Actions workflows for Java projects, especially those using Maven and Docker. These workflows are designed to be called from other repositories via `workflow_call` for consistent CI/CD, code quality, and release processes.

## Workflows

- **workflow-maven-run.yml**  
  Runs Maven builds with configurable arguments and Java version. Caches dependencies for faster builds.

- **workflow-docker-push.yml**  
  Builds and optionally pushes a Docker image for a Java Spring Boot project. Supports custom image names, tags, registry, and Maven build steps.

- **workflow-sonar-analysis.yml**  
  Runs SonarCloud analysis on the codebase if a Sonar token is provided. Computes project key and organization automatically if not specified.

- **workflow-licences-analysis.yml**  
  Performs license analysis using Maven and Eclipse Dash. Optionally creates a review and checks that the `DEPENDENCIES` file is up to date.

- **workflow-publish-artifacts.yml**  
  Publishes Maven artifacts to OSSRH (Sonatype/Maven Central) if required secrets are present. Handles GPG signing, versioning, and license file inclusion.

- **workflow-dependencies-update.yml**  
  Automatically updates the `DEPENDENCIES` file using Eclipse Dash and creates a pull request if changes are detected.

- **workflow-eol-analysis.yml**
  It helps identify dependencies that are either EOL or approaching their end-of-life date.
  * creates a pull request if any eol dependencies are found(only schedule run).
  * create workflow summary with eol details
  * comment on the pr with eol report if the workflow event is pull_request

- **workflow-nodejs-build.yml**
  Builds Node.js project and runs tests. Optionally runs SonarCloud analysis.

- **workflow-nodejs-docker-push.yml**
  Builds and pushes Docker image for Node.js project.

- **workflow-nodejs-licence-analysis.yml**
  Checks license compliance for Node.js project.

## Actions

- **import-gpg-key**  
  Composite action to import a GPG private key for signing artifacts in CI/CD workflows.

### Action Usage

```yaml
jobs:
  import-gpg:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: eclipse-ecsp/.github/.github/actions/import-gpg-key@main
        with:
          gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
```

## Workflow Usage

To use these workflows in your repository, call them from your own workflow YAML files using the `workflow_call` event.

## Usage Examples

### Maven Run

```yaml
jobs:
  build:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-maven-run.yml@main
    with:
      java_version: '17'
      maven_args: 'clean verify'
      test-report: false # create unit-test report 
      summaries-test-report: false # add summary to the workflow run
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

### Docker Build and Push

```yaml
jobs:
  docker:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-docker-push.yml@main
    with:
      image_name: 'my-app'
      image_tag: 'latest'
      registry: 'docker.io'
      registry_namespace: 'eclipseecsp'
      maven_args: 'clean package'
      context_path: '.'
      push: true
      java_version: '17'
    secrets:
      DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      DOCKER_API_TOKEN: ${{ secrets.DOCKER_API_TOKEN }}
```

### SonarCloud Analysis

```yaml
jobs:
  sonar:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-sonar-analysis.yml@main
    with:
      java_version: '17'
      sonar_project_key: 'eclipse-ecsp_repo-name'
      sonar_organization: 'eclipse-ecsp'
    secrets:
      token: ${{ secrets.SONAR_TOKEN }}
```

### License Analysis

```yaml
jobs:
  check-licences:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-licences-analysis.yml@main
    with:
      java_version: '17'
      dash_project_id: 'eclipse-ecsp_repo-name'
      dash_repo: 'http://github.com/repo-name'
      create-review: false
    secrets:
      token: ${{ secrets.DASH_TOKEN }}
```

### Publish Artifacts

```yaml
jobs:
  publish:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-publish-artifacts.yml@main
    with:
      java_version: '17'
      release_version: '0.1.0'
    secrets:
      GPG_PASSPHRASE: ${{ secrets.GPG_PASSPHRASE }}
      GPG_PRIVATE_KEY: ${{ secrets.GPG_PRIVATE_KEY }}
      CENTRAL_SONATYPE_TOKEN_USERNAME: ${{ secrets.CENTRAL_SONATYPE_TOKEN_USERNAME }}
      CENTRAL_SONATYPE_TOKEN_PASSWORD: ${{ secrets.CENTRAL_SONATYPE_TOKEN_PASSWORD }}
```

### Dependency Update

```yaml
jobs:
  update-dependencies:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-dependencies-update.yml@main
    with:
      java_version: '17'
    secrets:
      ECSP_BOT_PAT: ${{ secrets.ECSP_BOT_PAT }}
    permissions:
      pull-requests: write
      contents: read
```
---
### EOL Analysis

```yaml
name: Check Dependencies EOL Status
on:
  schedule:
    - cron: '0 11 * * 1'  # Weekly on Mondays at 11:00 UTC
  pull_request:
    types: [opened, synchronize, reopened, closed]
  workflow_dispatch:

jobs:
  check-dependencies:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-eol-analysis.yml@main
    with:
      # Optional parameters - shown with their default values
      fail_on_eol: false # # Will fail the workflow if EOL dependencies are found
      days_threshold: '90' # artifacts retention time
      java_version: '17'
      java_distribution: 'zulu'
      internal_prefixes: 'org.eclipse.ecsp' # to exclude the internal dependency checking with eol db
      maven_args: ''
      custom_mappings: 'org.liquibase=liquibase'
```

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `fail_on_eol` | Fail the workflow if EOL dependencies are found | No | `false` |
| `days_threshold` | Number of days before EOL to start alerting | No | `90` |
| `java_version` | Java version to use | No | `17` |
| `java_distribution` | Java distribution to use | No | `zulu` |
| `internal_prefixes` | Comma-separated list of internal package prefixes to exclude | No | `org.eclipse.ecsp` |
| `maven_args` | Additional Maven arguments | No | `''` |
| `custom_mappings` | Comma-separated list of custom EOL mappings (e.g., "com.my.group=my-product") | No | `''` |


#### Outputs

| Output | Description |
|--------|-------------|
| `eol-count` | Number of EOL dependencies found |
| `approaching-count` | Number of dependencies approaching EOL |
| `total-issues` | Total number of EOL-related issues |
#### Generated EOL Reports

The workflow generates the following artifacts:

- `eol_report.md`: Detailed Markdown report of all dependencies and their EOL status
- `eol_results.json`: Raw JSON data of the EOL analysis results
- `eol_dependencies.json`: Mapped dependencies with their EOL products
- `dependency_analysis.json`: Metadata about the dependency analysis, including untracked dependencies.

#### Using Workflow Outputs
```yaml
jobs:
  check-dependencies:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-eol-analysis.yml@main
    
  process-results:
    needs: check-dependencies
    runs-on: ubuntu-latest
    steps:
      - name: Check EOL counts
        run: |
          echo "Found ${{ needs.check-dependencies.outputs.eol-count }} EOL dependencies"
          echo "Found ${{ needs.check-dependencies.outputs.approaching-count }} dependencies approaching EOL"
```
---

### Node.js Build

```yaml
jobs:
  build:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-nodejs-build.yml@main
    with:
      node-version: '20'
      sonar-analysis: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Node.js Docker Build and Push

```yaml
jobs:
  docker:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-nodejs-docker-push.yml@main
    with:
      registry: 'docker.io'
      image-name: 'my-node-app'
    secrets:
      DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      DOCKER_API_TOKEN: ${{ secrets.DOCKER_API_TOKEN }}
```

### Node.js License Analysis

```yaml
jobs:
  license:
    uses: eclipse-ecsp/.github/.github/workflows/workflow-nodejs-licence-analysis.yml@main
    with:
      node-version: '20'
      java-version: '17'
      create-review: true
```

## Requirements

- Java 17+ (configurable)
- Maven
- For Docker and publishing: appropriate secrets (see each workflow for details)

## Security

See [SECURITY](SECURITY.md) for details.
