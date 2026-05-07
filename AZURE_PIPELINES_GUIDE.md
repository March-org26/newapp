# Azure Pipelines Syntax & Structure Guide

## Table of Contents
1. [Overview](#overview)
2. [Basic Structure](#basic-structure)
3. [Key Components](#key-components)
4. [Triggers](#triggers)
5. [Jobs & Stages](#jobs--stages)
6. [Tasks](#tasks)
7. [Variables](#variables)
8. [Examples](#examples)

---

## Overview

Azure Pipelines is a CI/CD service that works with your code repository to build, test, and deploy your application. It supports multiple languages, platforms, and clouds.

### Key Benefits
- **Multi-platform**: Linux, Windows, macOS
- **Multi-language**: .NET, Java, Node, Python, PHP, Ruby, C++, Go, etc.
- **Cloud agnostic**: Deploy to Azure, AWS, GCP, on-premises, etc.
- **Integrations**: GitHub, Azure Repos, Bitbucket, etc.




### Pipeline Component Hierarchy

```
Pipeline
 ├── Trigger
 │    ├── Branch (push)
 │    ├── Pull Request
 │    ├── Tag
 │    └── Schedule
 │
 ├── Variables
 │    ├── Scalar Variables
 │    ├── Variable Groups
 │    └── Secure Variables
 │
 ├── Pool (Agent)
 │    ├── Hosted Agents
 │    │    ├── Ubuntu
 │    │    ├── Windows
 │    │    └── macOS
 │    └── Self-Hosted Agents
 │
 └── Stages
      ├── Build Stage
      │    ├── Job 1
      │    │    ├── Step 1
      │    │    │    ├── Task (e.g., Checkout)
      │    │    │    └── Script
      │    │    ├── Step 2
      │    │    │    ├── Task (e.g., Build)
      │    │    │    └── Script
      │    │    └── Step 3
      │    │         ├── Task (e.g., Test)
      │    │         └── Script
      │    └── Job 2
      │         └── Steps...
      │
      ├── Test Stage
      │    ├── Job 1
      │    │    └── Steps...
      │    └── Job 2
      │         └── Steps...
      │
      └── Deploy Stage
           ├── Job 1
           │    └── Steps...
           └── Job 2
                └── Steps...
```

---

## Basic Structure

Azure Pipelines configuration is defined in a YAML file (typically `azure-pipelines.yml`).

```yaml
trigger:
  - main
  - develop

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - script: echo Hello World
```

---

## Key Components

### 1. **Trigger**
Specifies what events initiate the pipeline.

```yaml
# Trigger on specific branches
trigger:
  - main
  - develop

# Trigger on pull requests
pr:
  - main

# Trigger on tags
trigger:
  tags:
    include:
      - v*
```

### 2. **Pool**
Defines the agent/machine where jobs run.

```yaml
pool:
  vmImage: 'ubuntu-latest'  # Hosted agent
  # OR
  name: 'MyAgentPool'        # Self-hosted agent

# Specify OS versions
pool:
  vmImage: 'windows-latest'  # Windows
  vmImage: 'ubuntu-20.04'    # Ubuntu
  vmImage: 'macos-latest'    # macOS
```

### 3. **Variables**
Define variables for pipeline values.

```yaml
variables:
  buildConfiguration: 'Release'
  dotnetVersion: '6.0.x'

# Or define at group level
variables:
  - group: MyVariableGroup

# Or conditionally
variables:
  ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
    environment: 'production'
  ${{ else }}:
    environment: 'staging'
```

---

## Triggers

### Continuous Integration (CI)
```yaml
trigger:
  - main
  - develop
```

### Pull Request Trigger
```yaml
pr:
  - main
  branches:
    include:
      - main
      - develop
    exclude:
      - feature/*
```

### Scheduled Trigger
```yaml
schedules:
  - cron: "0 0 * * *"  # Daily at midnight
    displayName: Daily Build
    branches:
      include:
        - main
```

### No Trigger
```yaml
trigger: none  # Manual only
```

---

## Jobs & Stages

### Jobs
A job is a series of steps that run sequentially on an agent.

```yaml
jobs:
  - job: BuildJob
    displayName: 'Build Application'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - script: npm install
      - script: npm run build
```

### Stages
Stages allow you to organize jobs into logical sequences.

```yaml
stages:
  - stage: Build
    displayName: 'Build Stage'
    jobs:
      - job: BuildJob
        steps:
          - script: echo Building...

  - stage: Test
    displayName: 'Test Stage'
    dependsOn: Build  # Depends on Build stage
    jobs:
      - job: TestJob
        steps:
          - script: echo Running tests...

  - stage: Deploy
    displayName: 'Deploy Stage'
    dependsOn: Test
    condition: succeeded()  # Only run if Test passed
    jobs:
      - job: DeployJob
        steps:
          - script: echo Deploying...
```

### Conditions
Control when jobs/stages execute.

```yaml
condition: succeeded()              # Previous stage succeeded
condition: failed()                 # Previous stage failed
condition: always()                 # Always run
condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
```

---

## Tasks

Tasks are pre-built actions that perform work in your pipeline.

### Script Task
```yaml
steps:
  - script: echo Hello World
    displayName: 'Echo message'

  - script: |
      npm install
      npm run build
    displayName: 'Build app'
```

### Common Built-in Tasks

#### Install Dependencies
```yaml
steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '18.x'
    displayName: 'Install Node.js'

  - task: UseDotNet@2
    inputs:
      version: '6.0.x'
    displayName: 'Install .NET'
```

#### Publishing Artifacts
```yaml
steps:
  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: '$(Build.ArtifactStagingDirectory)'
      artifactName: 'drop'
    displayName: 'Publish artifacts'
```

#### Download Artifacts
```yaml
steps:
  - task: DownloadBuildArtifacts@0
    inputs:
      buildType: 'current'
      downloadType: 'single'
      artifactName: 'drop'
```

#### Copy Files
```yaml
steps:
  - task: CopyFiles@2
    inputs:
      sourceFolder: '$(Build.SourcesDirectory)'
      contents: '**'
      targetFolder: '$(Build.ArtifactStagingDirectory)'
```

#### Deploy to Azure App Service
```yaml
steps:
  - task: AzureWebApp@1
    inputs:
      azureSubscription: 'MySubscription'
      appType: 'webApp'
      appName: 'MyAppService'
      package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

---

## Variables

### Built-in Variables
```yaml
$(Build.SourcesDirectory)     # Source code directory
$(Build.ArtifactStagingDirectory)  # Staging directory
$(Build.BuildId)              # Build ID
$(Build.BuildNumber)          # Build number
$(Build.SourceBranch)         # Branch name
$(System.TeamProject)         # Project name
```

### Custom Variables
```yaml
variables:
  buildConfig: 'Release'
  buildPlatform: 'Any CPU'

steps:
  - script: echo $(buildConfig)
```

### Variable Groups (Secrets)
```yaml
variables:
  - group: MySecrets  # Contains connection strings, API keys

steps:
  - script: echo $(secretValue)  # Use secret
```

### Conditionally Set Variables
```yaml
steps:
  - script: echo "##vso[task.setvariable variable=myVar]myValue"
    displayName: 'Set variable'
  
  - script: echo $(myVar)  # Use in later step
```

---

## Examples

### Node.js CI Pipeline
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '18.x'
    displayName: 'Install Node.js'

  - script: npm install
    displayName: 'Install dependencies'

  - script: npm run build
    displayName: 'Build'

  - script: npm test
    displayName: 'Run tests'

  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: 'dist'
      artifactName: 'build'
```

### .NET Multi-Stage Pipeline
```yaml
trigger:
  - main

stages:
  - stage: Build
    jobs:
      - job: Build
        pool:
          vmImage: 'windows-latest'
        steps:
          - task: UseDotNet@2
            inputs:
              version: '6.0.x'
          - script: dotnet build
          - script: dotnet test
          - script: dotnet publish -c Release -o $(Build.ArtifactStagingDirectory)
          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: $(Build.ArtifactStagingDirectory)

  - stage: Deploy
    dependsOn: Build
    jobs:
      - job: Deploy
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: DownloadBuildArtifacts@0
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'MyServiceConnection'
              appName: 'MyAppService'
              package: '$(Pipeline.Workspace)/**/*.zip'
```

### Docker Build & Push
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  dockerRegistryServiceConnection: 'MyRegistry'
  imageRepository: 'myapp'
  containerRegistry: 'myregistry.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'

steps:
  - task: Docker@2
    displayName: 'Build image'
    inputs:
      command: build
      repository: $(imageRepository)
      dockerfile: $(dockerfilePath)
      containerRegistry: $(dockerRegistryServiceConnection)
      tags: |
        $(tag)
        latest

  - task: Docker@2
    displayName: 'Push image'
    inputs:
      command: push
      repository: $(imageRepository)
      containerRegistry: $(dockerRegistryServiceConnection)
      tags: |
        $(tag)
        latest
```

---

## Best Practices

1. **Keep pipelines DRY** - Use templates and variable groups
2. **Use stages for clarity** - Organize workflows logically
3. **Add display names** - Make pipeline output readable
4. **Handle secrets safely** - Use variable groups, not hardcoded values
5. **Use conditions** - Control execution flow
6. **Artifact management** - Publish and download artifacts between stages
7. **Fail fast** - Add validation early in pipeline
8. **Parallel jobs** - Use multiple agents for faster builds
9. **Cache dependencies** - Speed up builds with caching
10. **Version your pipeline** - Track changes in source control

---

## Resources

- [Official Azure Pipelines Documentation](https://docs.microsoft.com/azure/devops/pipelines)
- [YAML Schema Reference](https://docs.microsoft.com/azure/devops/pipelines/yaml-schema)
- [Task Reference](https://docs.microsoft.com/azure/devops/pipelines/tasks/reference)
- [Marketplace Tasks](https://marketplace.visualstudio.com/azuredevops)
