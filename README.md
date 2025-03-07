# Shared GitHub Action Workflows

Reusable workflows for standardizing CI/CD across projects.

## Project Structure

```bash
.github/
   ├── actions/
   │   └── project-analyzer/
   │       └── action.yml
   └── workflows/
       ├── smart-ci.yml
       ├── node-base.yml
       ├── python-base.yml
       ├── java-base.yml
       ├── go-base.yml
       ├── docker.yml
       └── deploy.yml
```

## Available Workflows

### Node.js Base (`node-base.yml`)
Basic CI workflow for Node.js projects.

```yaml
jobs:
  ci:
    uses: org-name/shared-workflows/.github/workflows/node-base.yml@v1
    with:
      node-version: '20.x'
      package-manager: 'pnpm'
```

### Docker (`docker.yml`)
Build and push Docker images.

```yaml
jobs:
  docker:
    uses: org-name/shared-workflows/.github/workflows/docker.yml@v1
    with:
      image-name: my-image
    secrets:
      REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
```

### Deploy (`deploy.yml`)
Deploy to various environments.

```yaml
jobs:
  deploy:
    uses: org-name/shared-workflows/.github/workflows/deploy.yml@v1
    with:
      environment: 'production'
      needs-aws: true
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## Versioning

This repository follows semantic versioning:
- Major version changes may include breaking changes
- Minor version changes add functionality in a backward compatible manner
- Patch version changes include backward compatible bug fixes

Always pin to a major version (e.g., `@v1`) at minimum.

## Contributing

1. Create a feature branch
2. Make your changes
3. Update CHANGELOG.md
4. Create a pull request
5. After merge, create a new release with appropriate version tag



## How to use

Now, here's how to use these reusable workflows in your projects. Create a workflow file in your project (e.g., `.github/workflows/ci.yml`) with minimal configuration:

```yaml
name: CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  ci:
    uses: org-name/shared-workflows/.github/workflows/node-base.yml
    with:
      node-version: '20.x'
      package-manager: 'pnpm'  # or 'npm' or 'yarn'
      
  docker:
    needs: ci
    if: github.ref == 'refs/heads/main'
    uses: org-name/shared-workflows/.github/workflows/docker.yml
    with:
      image-name: ${{ github.repository }}
    secrets:
      REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
      
  deploy:
    needs: docker
    if: github.ref == 'refs/heads/main'
    uses: org-name/shared-workflows/.github/workflows/deploy.yml
    with:
      environment: 'production'
      needs-aws: true
      aws-region: 'us-west-2'
      deploy-command: './deploy.sh'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

Key features of these reusable workflows:

1. **Node.js Base Workflow**:
   - Supports npm, yarn, and pnpm
   - Intelligent caching of dependencies
   - Configurable commands for install, build, test, and lint
   - Ability to skip certain steps
   - Works with monorepos via working-directory input

2. **Docker Workflow**:
   - Automated image tagging based on git refs
   - Build caching for faster builds
   - Supports custom registries
   - Flexible tag configuration
   - BuildKit enabled by default

3. **Deployment Workflow**:
   - Environment-based deployments
   - AWS integration (optional)
   - Custom deployment commands
   - Secure secrets handling

To use these templates:

1. Create a `.github/workflows` directory in your repository
2. Copy these workflow files into that directory
3. Customize the workflow files as needed for your project
4. Commit and push the changes to your repository
5. Monitor the Actions tab in your GitHub repository for workflow runs and results



## How to use Smart Worflows

Smart workflow can detect project structure and configurations automatically. We'll create a composite workflow that analyzes your repository and chooses the appropriate workflow based on the detected settings.

This smart workflow system provides:

1. **Automatic Detection**:
   - Project structure (monorepo vs single repo)
   - Programming languages
   - Frameworks
   - Package managers
   - Database types
   - Test frameworks
   - Docker configuration

2. **Smart Workflow Selection**:
   - Automatically chooses appropriate base workflows
   - Handles multiple languages in the same repo
   - Supports monorepo workspace detection
   - Configures framework-specific settings

3. **Extensible Design**:
   - Easy to add new language/framework detection
   - Modular workflow structure
   - Reusable components
   - Clear separation of concerns

To use this in your projects:

In your project repositories, you only need a minimal workflow file:
   ```yaml
   name: CI/CD
   
   on:
     push:
       branches: [ main ]
     pull_request:
       branches: [ main ]
   
   jobs:
     pipeline:
       uses: org-name/shared-workflows/.github/workflows/smart-ci.yml@v1
       secrets: inherit
   ```

The workflow will automatically:
1. Analyze your project structure
2. Detect languages and frameworks
3. Run appropriate build workflows
4. Handle Docker builds if needed
5. Manage deployments

## Java Example

### Key Features

1. **Flexibility**:
   - Supports both Maven and Gradle builds
   - Configurable Java version and distribution
   - Works with any subdirectory in monorepos
   - Optional test skipping and custom test commands

2. **Quality Gates**:
   - Automatic code coverage calculation and threshold validation
   - SonarQube integration (when token provided)
   - OWASP dependency vulnerability scanning
   - Code style checking via Checkstyle

3. **Performance Optimizations**:
   - Dependency caching for Maven/Gradle
   - SonarQube package caching
   - Configurable memory settings

4. **Spring Framework Support**:
   - Configurable Spring profiles
   - Environment variable passing

5. **Build Artifacts**:
   - Optional artifact publishing
   - Test results archiving
   - Code coverage report archiving

### Usage Example

In your project repository, you would reference this workflow like:

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  java-build:
    uses: your-org/shared-workflows/.github/workflows/java-base.yml@v1
    with:
      java-version: '21'
      build-tool: 'maven'
      coverage-threshold: 85
      publish-artifact: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Customization Options

You can adjust several parameters:
- Java version and distribution
- Build tool (Maven or Gradle)
- Test execution behavior
- Coverage requirements
- Spring profiles
- Artifact publication


## Node.js Example

This Node.js base workflow is designed to handle both backend and frontend Node.js projects with comprehensive quality gates and flexible configuration options.

### Key Features

1. **Universal Node.js Support**:
   - Works with any Node.js project (backend, frontend, or fullstack)
   - Supports npm, yarn, and pnpm
   - Compatible with monorepos via working-directory input

2. **Multi-Stage Pipeline**:
   - Setup: Dependency installation with caching
   - Lint: Code style and quality checks
   - Test: Unit and integration tests with coverage
   - Build: Production-ready build
   - E2E: End-to-end testing for UI components
   - Lighthouse: Performance auditing for frontend projects
   - SonarQube: Code quality scanning

3. **Frontend-Specific Features**:
   - Automatic detection of test frameworks (Jest, Vitest)
   - E2E testing with Playwright or Cypress
   - Lighthouse performance auditing
   - Build artifact publishing

4. **Backend-Specific Features**:
   - Coverage threshold enforcement
   - Environment variable injection
   - Security scanning via SonarQube

5. **Quality Gates**:
   - Code style validation
   - Test coverage thresholds
   - Performance benchmarks for frontend
   - Comprehensive reporting

### Usage Examples

#### For a Frontend React Project:

```yaml
name: Frontend CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  frontend-build:
    uses: your-org/shared-workflows/.github/workflows/node-base.yml@v1
    with:
      node-version: '20.x'
      package-manager: 'npm'
      project-type: 'frontend'
      run-lighthouse: true
      skip-e2e: false
      coverage-threshold: 75
      publish-artifact: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

#### For a Backend Node.js API:

```yaml
name: Backend CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  backend-build:
    uses: your-org/shared-workflows/.github/workflows/node-base.yml@v1
    with:
      node-version: '20.x'
      package-manager: 'pnpm'
      project-type: 'backend'
      coverage-threshold: 90
      env-vars: '{"NODE_ENV":"test","DB_HOST":"localhost"}'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Customization Options

This workflow is highly customizable through inputs:
- Project type (backend, frontend, fullstack)
- Package manager preference
- Custom build/test/lint commands
- Test coverage requirements
- E2E testing configuration
- Performance testing options
- Environment variable injection
- Artifact publishing

