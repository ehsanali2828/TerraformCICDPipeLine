# Terraform tool installer
# Pre-requisites for the task

The task can run on the following build agent operating systems:
- Windows
- MacOS
### Parameters of the task

* **Display name\*:** Provide a name to identify the task among others in your pipeline.

* **Version\*:** Specify the keyword 'latest' to get the latest released version or specify exact version of Terraform to install.  
Example: 
    To install latest Terraform version use keyword: latest.  To install specific version Ex. 1.0.8, use 1.0.8.
For getting more details about exact version, refer [this link](https://releases.hashicorp.com/terraform/)


### Output Variables

* **Terraform location:** This variable can be used to refer to the location of the terraform binary that was installed on the agent in subsequent tasks.

### Example Task Usage
Below is a basic example usage of a few commands within the TerraformInstaller task.

```yaml
- task: TerraformInstaller@0
  displayName: Install Terraform 1.5.7
  inputs:
    terraformVersion: 1.5.7
```
