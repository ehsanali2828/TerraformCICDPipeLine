# Terraform CI CD Pipeline for Azure DevOps

## Lets walkaround:

In this repo you will find the Azure DevOps Pipeline tasks for installing Terraform and running Terraform commands in a build or release pipeline. Just to guide someone the process of using Terraform to deploy infrastructure within Azure, Amazon Web Services(AWS), Google Cloud Platform.

Here are the following contributions:
- Terraform tool installer
- Terraform for executing the core Terraform commands.
- Amazon Web Services(AWS) service connection for creating a service connection for AWS to provide AWS credentials
- Google Cloud Platform(GCP) service connection for creating a service connection for GCP to provide GCP credentials

- [Terraform installer](https://aka.ms/AAf1a0p)
- [Terraform] (https://github.com/microsoft/azure-pipelines-terraform/tree/main/Tasks/TerraformTask/TerraformTaskV4)


This extension is intended to run on **Windows**, **Linux** and **MacOS** agents.

# Example: Install the latest version of Terraform

```yaml
- task: TerraformInstaller@1
  displayName: 'Install Terraform'
  inputs:
    terraformVersion: 'latest'
```

# Example: Install a specific version of Terraform

```yaml
- task: TerraformInstaller@1
  displayName: 'Install Terraform'
  inputs:
    terraformVersion: '1.11.3'
```
# Examples: Run Terraform init, plan and apply for Microsoft Azure
# Example: Using default settings with the same service connection for all tasks.

```yaml
- task: TerraformTask@5
  displayName: Run Terraform Init
  inputs:
    provider: 'azurerm'
    command: 'init'
    backendServiceArm: 'your-service-connection'
    backendAzureRmStorageAccountName: 'your-stg-name'
    backendAzureRmContainerName: 'your-container-name'
    backendAzureRmKey: 'state.tfstate'

- task: TerraformTask@5
  name: terraformPlan
  displayName: Run Terraform Plan
  inputs:
    provider: 'azurerm'
    command: 'plan'
    commandOptions: '-out tfplan'
    environmentServiceNameAzureRM: 'your-service-connection'


- task: TerraformTask@5
  displayName: Run Terraform Apply
  condition: and(succeeded(), eq(variables['terraformPlan.changesPresent'], 'true'))
  inputs:
    provider: 'azurerm'
    command: 'apply'
    commandOptions: 'tfplan'
    environmentServiceNameAzureRM: 'your-service-connection'
```
The following example shows an example of using a 3 service connection setup with the Terraform task:

```yaml
trigger:
- main

stages:
- stage: plan
  displayName: 'Terraform Plan'
  jobs:
  - job: plan
    displayName: 'Terraform Init and Plan'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: TerraformInstaller@1
      displayName: 'Install Terraform'
      inputs:
        terraformVersion: 'latest'

    - task: TerraformTask@5
      displayName: Run Terraform Init
      inputs:
        provider: 'azurerm'
        command: 'init'
        backendServiceArm: 'your-backend-service-connection'
        backendAzureRmStorageAccountName: 'your-storage-account-name'
        backendAzureRmContainerName: 'your-container-name'
        backendAzureRmKey: 'state.tfstate'
        backendAzureRmUseCliFlagsForAuthentication: true

    - task: TerraformTask@5
      name: terraformPlan
      displayName: Run Terraform Plan
      inputs:
        provider: 'azurerm'
        command: 'plan'
        commandOptions: '-out tfplan'
        environmentServiceNameAzureRM: 'your-plan-service-connection'

    - task: CopyFiles@2
      displayName: Create Module Artifact
      inputs:
        SourceFolder: '$(Build.SourcesDirectory)'
        Contents: |
          **/*
          !.terraform/**/*
          !.git/**/*
          !**/.terraform/**/*
          !**/.git/**/*
        TargetFolder: '$(Build.ArtifactsStagingDirectory)'
        CleanTargetFolder: true
        OverWrite: true

    - task: PublishPipelineArtifact@1
      displayName: Publish Module Artifact
      inputs:
        targetPath: '$(Build.ArtifactsStagingDirectory)'
        artifact: 'terraformModule'
        publishLocation: 'pipeline'

- stage: apply
  displayName: 'Terraform Apply'
  jobs:
  - job: apply
    displayName: 'Terraform Init and Apply'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DownloadPipelineArtifact@2
      displayName: Download Module Artifact
      inputs:
        source: 'current'
        artifactName: 'terraformModule'
        targetPath: '$(Build.SourcesDirectory)'

    - task: TerraformInstaller@1
      displayName: 'Install Terraform'
      inputs:
        terraformVersion: 'latest'

    - task: TerraformTask@5
      displayName: Run Terraform Init
      inputs:
        provider: 'azurerm'
        command: 'init'
        backendServiceArm: 'your-backend-service-connection'
        backendAzureRmStorageAccountName: 'your-storage-account-name'
        backendAzureRmContainerName: 'your-container-name'
        backendAzureRmKey: 'state.tfstate'
        backendAzureRmUseCliFlagsForAuthentication: true

    - task: TerraformTask@5
      displayName: Run Terraform Apply
      inputs:
        provider: 'azurerm'
        command: 'apply'
        commandOptions: 'tfplan'
        environmentServiceNameAzureRM: 'your-apply-service-connection'
```
# Example: Run Terraform init, plan and apply for AWS

```yaml
- task: TerraformTask@5
  displayName: Run Terraform Init
  inputs:
    provider: 'aws'
    command: 'init'
    backendServiceAWS: 'your-service-connection'
    backendAWSBucketName: 'your-bucket-name'
    backendAWSKey: 'state.tfstate'

- task: TerraformTask@5
  name: terraformPlan
  displayName: Run Terraform Plan
  inputs:
    provider: 'aws'
    command: 'plan'
    commandOptions: '-out tfplan'
    environmentServiceNameAWS: 'your-service-connection'

# Only runs if the 'terraformPlan' task has detected changes the in state.
- task: TerraformTask@5
  displayName: Run Terraform Apply
  condition: and(succeeded(), eq(variables['terraformPlan.changesPresent'], 'true'))
  inputs:
    provider: 'aws'
    command: 'apply'
    commandOptions: 'tfplan'
    environmentServiceNameAWS: 'your-service-connection'
```

# Example: Run Terraform init, plan and apply for GCP

```yaml
- task: TerraformTask@5
  displayName: Run Terraform Init
  inputs:
    provider: 'gcp'
    command: 'init'
    backendServiceGCP: 'your-service-connection'
    backendGCPBucketName: 'your-bucket-name'
    backendGCPPrefix: 'state.tfstate'

- task: TerraformTask@5
  name: terraformPlan
  displayName: Run Terraform Plan
  inputs:
    provider: 'gcp'
    command: 'plan'
    commandOptions: '-out tfplan'
    environmentServiceNameGCP: 'your-service-connection'

# Only runs if the 'terraformPlan' task has detected changes the in state.
- task: TerraformTask@5
  displayName: Run Terraform Apply
  condition: and(succeeded(), eq(variables['terraformPlan.changesPresent'], 'true'))
  inputs:
    provider: 'gcp'
    command: 'apply'
    commandOptions: 'tfplan'
    environmentServiceNameGCP: 'your-service-connection'
```

# Example: Run Terraform init, plan and apply for OCI

```yaml
- task: TerraformTask@5
  displayName: Run Terraform Init
  inputs:
    provider: 'oci'
    command: 'init'
    backendServiceOCI: 'your-service-connection'
    backendOCIPar: 'state.tfstate'

- task: TerraformTask@5
  name: terraformPlan
  displayName: Run Terraform Plan
  inputs:
    provider: 'oci'
    command: 'plan'
    commandOptions: '-out tfplan'
    environmentServiceNameOCI: 'your-service-connection'

# Only runs if the 'terraformPlan' task has detected changes the in state.
- task: TerraformTask@5
  displayName: Run Terraform Apply
  condition: and(succeeded(), eq(variables['terraformPlan.changesPresent'], 'true'))
  inputs:
    provider: 'oci'
    command: 'apply'
    commandOptions: 'tfplan'
    environmentServiceNameOCI: 'your-service-connection'
```