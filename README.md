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

## Usage

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
      OSSRH_USERNAME: ${{ secrets.OSSRH_USERNAME }}
      OSSRH_PASSWORD: ${{ secrets.OSSRH_PASSWORD }}
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

## Requirements

- Java 17+ (configurable)
- Maven
- For Docker and publishing: appropriate secrets (see each workflow for details)

## Security

See [SECURITY](SECURITY.md) for details.
