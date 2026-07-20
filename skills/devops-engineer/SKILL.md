---
name: devops-engineer
description: Frontal SDK deployment specialist with expertise in TypeScript package publishing, monorepo CI/CD, npm registry management, and automated SDK distribution. Use when deploying Frontal SDK packages, setting up publishing workflows, or managing SDK infrastructure.
license: MIT
metadata:
  author: frontal-labs
  version: "1.0"
  category: devops-infrastructure
---

# Frontal SDK DevOps Engineer

**ROLE**: Infrastructure and deployment specialist for Frontal SDK packages, ensuring reliable publishing, CI/CD automation, and distribution of the @frontal/* package ecosystem.

**ACTIVATION**: This agent is triggered by: "Frontal SDK deployment", "@frontal package publishing", "SDK CI/CD", "npm registry management", "package distribution", "TypeScript monorepo deployment", "SDK infrastructure"

## Frontal SDK Deployment Architecture

###  Package Publishing Pipeline

**Multi-Package Publishing Strategy**:

```yaml
# .github/workflows/publish.yml
name: Publish Frontal SDK Packages

on:
  push:
    tags:
      - 'v*'
  workflow_dispatch:

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Setup Bun
      uses: oven-sh/setup-bun@v1
      with:
        bun-version: latest
    
    - name: Install dependencies
      run: bun install --frozen-lockfile
    
    - name: Build packages
      run: bun run build
    
    - name: Run tests
      run: bun run test
    
    - name: Publish packages
      run: |
        echo "//registry.npmjs.org/:_authToken=${{ secrets.NPM_TOKEN }}" > ~/.npmrc
        bun changeset publish
      env:
        NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Changeset-Based Publishing**:

```json
// package.json publishing scripts
{
  "scripts": {
    "build": "turbo run build",
    "test": "turbo run test", 
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "turbo run build --filter=!@frontal/docs && changeset publish",
    "publish": "changeset publish"
  }
}
```

```yaml
# .changeset/config.yml
$schema: https://unpkg.com/@changesets/config@2.3.0/schema.json

changelog: '@changesets/cli/changelog'
commit: false
fixed: []
linked: [
  ['@frontal/ai', '@frontal/agents'],
  ['@frontal/core', '@frontal/ai', '@frontal/functions']
]
access: public
baseBranch: 'main'
updateInternalDependencies: 'patch'
ignore: []
```

###  CI/CD Pipeline Architecture

**Turbo Monorepo CI/CD**:

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"],
      "cache": false
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"],
      "cache": false
    },
    "lint": {
      "outputs": [],
      "cache": false
    },
    "type-check": {
      "dependsOn": ["^build"],
      "outputs": [],
      "cache": false
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

**Quality Gates and Validation**:

```yaml
# .github/workflows/quality.yml
name: Quality Assurance

on:
  pull_request:
    branches: [main, develop]

jobs:
  quality-check:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Bun
      uses: oven-sh/setup-bun@v1
    
    - name: Install dependencies
      run: bun install --frozen-lockfile
    
    - name: Type checking
      run: bun run type-check
    
    - name: Linting
      run: bun run lint
    
    - name: Unit tests
      run: bun run test
    
    - name: Integration tests
      run: bun run test:integration
    
    - name: Build validation
      run: bun run build
    
    - name: Package size analysis
      run: |
        bunx bundlesize
```

###  Package Registry Management

**npm Registry Configuration**:

```json
// .npmrc
@frontal:registry=https://registry.npmjs.org/
//registry.npmjs.org/:_authToken=${NPM_TOKEN}
always-auth=true
```

**Multi-Registry Support**:

```json
// package.json per package
{
  "name": "@frontal/ai",
  "publishConfig": {
    "registry": "https://registry.npmjs.org/",
    "access": "public"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/frontal-labs/sdk-ts.git",
    "directory": "packages/ai"
  }
}
```

**Automated Version Management**:

```typescript
// scripts/version-manager.ts
import { readFileSync, writeFileSync } from 'fs';
import { execSync } from 'child_process';

class VersionManager {
  updatePackageVersions(version: string): void {
    const packages = this.getAllPackages();
    
    packages.forEach(pkg => {
      const packageJsonPath = `${pkg.path}/package.json`;
      const packageJson = JSON.parse(readFileSync(packageJsonPath, 'utf8'));
      
      // Update version
      packageJson.version = version;
      
      // Update internal dependencies
      this.updateInternalDependencies(packageJson, packages, version);
      
      writeFileSync(packageJsonPath, JSON.stringify(packageJson, null, 2));
    });
  }
  
  private updateInternalDependencies(
    packageJson: any, 
    packages: Package[], 
    version: string
  ): void {
    const dependencies = packageJson.dependencies || {};
    const devDependencies = packageJson.devDependencies || {};
    const peerDependencies = packageJson.peerDependencies || {};
    
    [dependencies, devDependencies, peerDependencies].forEach(dep => {
      Object.keys(dep).forEach(depName => {
        if (depName.startsWith('@frontal/')) {
          dep[depName] = `^${version}`;
        }
      });
    });
  }
}
```

###  Infrastructure as Code

**Docker Multi-Stage Builds**:

```dockerfile
# Dockerfile.sdk
FROM node:18-alpine AS base
WORKDIR /app
COPY package.json bun.lockb ./
RUN npm install -g bun

FROM base AS builder
COPY . .
RUN bun install --frozen-lockfile
RUN bun run build

FROM base AS publisher
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./
COPY --from=builder /app/.npmrc ./.npmrc
RUN npm publish --access public
```

**Kubernetes Deployment for Build Agents**:

```yaml
# k8s/build-agent.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: frontal-build
spec:
  template:
    spec:
      containers:
      - name: builder
        image: frontal/sdk-builder:latest
        env:
        - name: NPM_TOKEN
          valueFrom:
            secretKeyRef:
              name: npm-secrets
              key: token
        - name: GITHUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: github-secrets
              key: token
        command: ["/bin/sh"]
        args:
          - -c
          - |
            bun install --frozen-lockfile
            bun run build
            bun run test
            bun changeset publish
      restartPolicy: Never
```

###  Monitoring and Observability

**Package Download Analytics**:

```typescript
// scripts/analytics.ts
interface PackageStats {
  name: string;
  version: string;
  downloads: number;
  date: string;
}

class PackageAnalytics {
  async getDownloadStats(packageName: string): Promise<PackageStats[]> {
    const response = await fetch(
      `https://api.npmjs.org/downloads/point/last-month/${packageName}`
    );
    const data = await response.json();
    
    return Object.entries(data.downloads).map(([date, downloads]) => ({
      name: packageName,
      version: 'latest',
      downloads: downloads as number,
      date
    }));
  }
  
  generateReport(packages: string[]): string {
    // Generate comprehensive download report
    return `# Frontal SDK Download Report\n\n${packages.join(', ')}`;
  }
}
```

**Build Performance Monitoring**:

```yaml
# .github/workflows/monitoring.yml
name: Build Performance

on:
  schedule:
    - cron: '0 0 * * *'  # Daily

jobs:
  performance-monitor:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Bun
      uses: oven-sh/setup-bun@v1
    
    - name: Install dependencies
      run: bun install --frozen-lockfile
    
    - name: Measure build times
      run: |
        echo "Build started at $(date)"
        time bun run build
        echo "Build completed at $(date)"
    
    - name: Measure test times  
      run: |
        echo "Tests started at $(date)"
        time bun run test
        echo "Tests completed at $(date)"
    
    - name: Report metrics
      run: |
        # Send metrics to monitoring service
        curl -X POST "${{ secrets.METRICS_ENDPOINT }}" \
          -H "Content-Type: application/json" \
          -d '{"service": "frontal", "timestamp": "'$(date -Iseconds)'"}'
```

###  Security and Compliance

**Package Security Scanning**:

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Run npm audit
      run: npm audit --audit-level=moderate
    
    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
    
    - name: Check for vulnerabilities
      run: |
        bunx audit-ci --moderate
```

**Dependency Management**:

```json
// .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
    open-pull-requests-limit: 5
    commit-message:
      prefix: "deps"
      include: "scope"
```

###  Deployment Automation

**Automated Release Process**:

```typescript
// scripts/release.ts
import { execSync } from 'child_process';
import { readFileSync, writeFileSync } from 'fs';

class ReleaseManager {
  async performRelease(): Promise<void> {
    console.log(' Starting Frontal SDK release process...');
    
    // 1. Validate environment
    this.validateEnvironment();
    
    // 2. Run quality checks
    await this.runQualityChecks();
    
    // 3. Update versions
    await this.updateVersions();
    
    // 4. Build packages
    await this.buildPackages();
    
    // 5. Run tests
    await this.runTests();
    
    // 6. Publish packages
    await this.publishPackages();
    
    // 7. Create GitHub release
    await this.createGitHubRelease();
    
    console.log(' Release completed successfully!');
  }
  
  private validateEnvironment(): void {
    const requiredEnvVars = ['NPM_TOKEN', 'GITHUB_TOKEN'];
    const missing = requiredEnvVars.filter(env => !process.env[env]);
    
    if (missing.length > 0) {
      throw new Error(`Missing environment variables: ${missing.join(', ')}`);
    }
  }
  
  private async runQualityChecks(): Promise<void> {
    console.log(' Running quality checks...');
    execSync('bun run type-check', { stdio: 'inherit' });
    execSync('bun run lint', { stdio: 'inherit' });
    execSync('bun run test', { stdio: 'inherit' });
  }
  
  private async updateVersions(): Promise<void> {
    console.log(' Updating package versions...');
    execSync('bun changeset version', { stdio: 'inherit' });
  }
  
  private async buildPackages(): Promise<void> {
    console.log(' Building packages...');
    execSync('bun run build', { stdio: 'inherit' });
  }
  
  private async runTests(): Promise<void> {
    console.log(' Running tests...');
    execSync('bun run test', { stdio: 'inherit' });
  }
  
  private async publishPackages(): Promise<void> {
    console.log(' Publishing packages...');
    execSync('bun changeset publish', { stdio: 'inherit' });
  }
  
  private async createGitHubRelease(): Promise<void> {
    const version = this.getCurrentVersion();
    
    // Create GitHub release using GitHub API
    const response = await fetch('https://api.github.com/repos/frontal-labs/sdk-ts/releases', {
      method: 'POST',
      headers: {
        'Authorization': `token ${process.env.GITHUB_TOKEN}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        tag_name: `v${version}`,
        name: `Frontal SDK v${version}`,
        body: this.generateReleaseNotes(version),
        draft: false,
        prerelease: false
      })
    });
    
    if (!response.ok) {
      throw new Error(`Failed to create GitHub release: ${response.statusText}`);
    }
  }
  
  private getCurrentVersion(): string {
    const packageJson = JSON.parse(readFileSync('package.json', 'utf8'));
    return packageJson.version;
  }
  
  private generateReleaseNotes(version: string): string {
    return `# Frontal SDK v${version}\n\n## Changes\n\n- Updated package versions\n- Bug fixes and improvements\n- Documentation updates`;
  }
}
```

## Agent Output Requirements

**Provide comprehensive Frontal SDK deployment solutions** including:

- **Package Publishing**: Automated npm publishing workflows for @frontal/* packages
- **CI/CD Architecture**: Turbo monorepo pipelines with quality gates
- **Registry Management**: Multi-registry support and version management
- **Infrastructure**: Docker builds, Kubernetes deployments, and IaC
- **Monitoring**: Package analytics, build performance, and security scanning

**Always include**:
- Specific Frontal SDK deployment configurations
- CI/CD pipeline examples
- Package publishing workflows
- Infrastructure as code templates
- Security and compliance measures
