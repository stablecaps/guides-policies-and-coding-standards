# Standards & Coding Guidelines for Azure DevOps Pipelines - part 1

This article specifies coding guidelines that ease troubleshooting of Azure Pipeline definitions by making them:

1. Stable - by reducing hard-to-catch-bugs
2. Readable and less cluttered - reduce screen cruft that increases the mental effort required to parse the code
3. Standardised - reducing the number of various _ad hoc_ styles to help points 1 & 2

## 1. Logging operations and variables

A common helper method for logging output in PowerShell is this:
```
function Log($content) {
	Write-Host $content -ForegroundColor Green
}
```

`Log` is preferred over using plain `Write-Host` as it reduces the mental parsing effort required to read the code.

Variables should be logged in the following format because it is the most direct way to find the line of code that is throwing the error.

#### Good:
```
Log "varName = $varName"
```

Example:
```
Log "StorageAccountName = $StorageAccountName"
```
or:
```
Log "env:subscription = $env:tenantId"
```

#### Bad:

```
Log "Managed Application Definition Name: $(serviceCatalogDefinitionName)"
```

The reader has to mentally map `Managed Application Definition Name == serviceCatalogDefinitionName` and remember that forever. The good ways listed above are more direct and easier to debug.

#### Better:

Use `LogEx` function to output the variable value and exit if it's empty:

```
function LogEx($varName, $varValue) {
    # Functions logs the output of a variable Name string
    # Exits if variable is empty or null

    if (-not $varValue) {
        Write-Host "Error: variable <$varName> does not have a value`nExiting.."
        Exit 42
    }
    Write-Host "$varName = $varValue" -ForegroundColor Green
}
```

Usage
```
LogEx -varName "StorageAccountName" -varValue $StorageAccountName
```

## 2. Log, Log, Log, Log, Log, Log, Log, Log, Log, Log, Log!!!!!!

Log things so that errors can be easily tracked. What this practically means is:
1. Log to check that a **variable is not null/empty** (see point 1 for style).
2. Do not Log unnecessarily things that add visual cruft that makes it hard for debugging. Examples:
3. Log if an operation has failed, not if it completed successfully.
4. Do not log intermediate states in an if condition unless you need it for debugging or it somehow provides extra useful info.

#### Bad:
```
if ($INFRA_BUILD_OVERRIDE -eq "build")
{
  Log "INFRA_BUILD_OVERRIDE is in build mode" # <------!!!!!

  $TODAYS_DATE = $(Get-Date -Format yyyyMMdd)
  $INFRA_RESOURCE_GROUP_NAME = "TAM-PRO-WESTEUROPE-INTER-$($TODAYS_DATE)-$($UNIQUEID)"
  $SET_PULUMI = "True"
}
elseif ($INFRA_BUILD_OVERRIDE -ne "build" -and $INFRA_PULUMI_REBUILD -eq "True")
{
  Log "INFRA_BUILD_OVERRIDE is rebuilding $INFRA_RESOURCE_GROUP_NAME" # <------!!!!!

  $INFRA_RESOURCE_GROUP_NAME = "$INFRA_BUILD_OVERRIDE"
  $SET_PULUMI = "True"
}
### <--snipped--->

Log "INFRA_RESOURCE_GROUP_NAME = $INFRA_RESOURCE_GROUP_NAME"
Log "UNIQUEID = $UNIQUEID"
Log "SET_PULUMI = $SET_PULUMI"
```

#### Good:
```
if ($INFRA_BUILD_OVERRIDE -eq "build")
{
  $TODAYS_DATE = $(Get-Date -Format yyyyMMdd)
  $INFRA_RESOURCE_GROUP_NAME = "TAM-PRO-WESTEUROPE-INTER-$($TODAYS_DATE)-$($UNIQUEID)"
  $SET_PULUMI = "True"
}
elseif ($INFRA_BUILD_OVERRIDE -ne "build" -and $INFRA_PULUMI_REBUILD -eq "True")
{
  $INFRA_RESOURCE_GROUP_NAME = "$INFRA_BUILD_OVERRIDE"
  $SET_PULUMI = "True"
}
### <--snipped--->

Log "INFRA_RESOURCE_GROUP_NAME = $INFRA_RESOURCE_GROUP_NAME"
Log "UNIQUEID = $UNIQUEID"
Log "SET_PULUMI = $SET_PULUMI"
```


## 3. Use an external script file vs the inline script option

If a given BASH or PowerShell script is long. i.e More than ~20-25 lines, please use an external script file. This is easier to debug because editors like VSCode afford syntax highlighting and code-error-checking. This allows a lot of bugs to be solved quickly. When all the code is inline, everything appears as a string.

## 4. AVOID VARIABLE PROLIFERATION AT ALL COSTS!!!!!

Variable proliferation is a phenomenon in which values are reassigned from one variable name to another in different parts of the pipeline.

#### Bad:
```
### Input parameter
- name: preBuiltInfra # <------!!!!!
  displayName: "Attach to any arbitrary Infra RG that has been previously created"
  type: string
  default: TAM-PRO-WESTEUROPE-INTER-MAIN


### Another tasks converts this into a local variable with a different name which subtly changes it's meaning. We now have to look at x2 variable names to understand what $RESOURCE_GROUP_NAME is. Is it the MRG, ManagedAppsCoreResources, or core-dev resource grouup?
- task: AzureCLI@2
  name: GenerateCustomCoreVars
  inputs:
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    arguments: ${{ parameters.preBuiltInfra }} # <------!!!!!
    inlineScript: |
            #!/bin/bash

            RESOURCE_GROUP_NAME=$1 # <------!!!!!

### At the end of the above script block we export a variable that can be used by downstream tasks (and introduce another name change)
echo "##vso[task.setvariable variable=infraResourceGroupName]${RESOURCE_GROUP_NAME}"

### This is then used in another inline script
./another-repo/deploy-more-resources.ps1 `
  -ResourceGroupName $(infraResourceGroupName) `  # <------!!!!!
  -appPrefix ${{ parameters.appPrefix }}  `
  <--snipped--->

### Then the powershell script might do yet another reasignment
$rg = $ResourceGroupName # <------!!!!!
```

So in the above example, the prebuilt core underwent the following transmutations:

`$preBuiltInfra --> $parameters.preBuiltInfra --> $RESOURCE_GROUP_NAME --> $infraResourceGroupName --> $ResourceGroupName --> $rg`


- This is extremely confusing to keep track of and this means it is hard to debug.
- It is the result of not properly considering what the name of the object should be. [Names have great power because they enable us to know **what something is** unequivocally](https://www.litcharts.com/lit/a-wizard-of-earthsea/symbols/true-names#:~:text=Throughout%20A%20Wizard%20of%20Earthsea,shadow%20self%2C%20and%20cosmic%20balance.). If we don't label things correctly it can cause a lot of confusion and wasted time.
- It is unnecessarily complex and increases the amount of code the reader has to parse which will impact delivery and stability
- It is lazy


## 5. Convert parameters to global variables ASAP

The syntax for Azure DevOps can be complex especially when referring to names from other jobs, steps, tasks, etc. One method to reduce some of this complexity is to convert incoming parameters to global variables immediately. Then they can be referred to with the shorter `$(myVar)` syntax throughout the pipeline even in different files.

#### Good:
```
### This is the pipeline entrypoint
parameters:
  - name: buildAgent
    displayName: Build Agent
    type: string
    default: Spot Agents
    values:
      - Spot Agents
      - Static Build Agents

  - name: godMode
    displayName: "GodMode: Check this to enable godMode access. Not suitable for production"
    type: boolean
    default: false

variables:
  - template: variables/standard.yaml
  - name: buildAgent
    value: ${{ parameters.buildAgent }}
  - name: godMode
    value: ${{ parameters.godMode }}
  - name: yourPublicIp
```

#### A task using above variables
```
  ### <--snipped--->
  script: |
    $cleanedYourPublicIp = "$(yourPublicIp)".replace('/','')
    if (-Not $$(godMode) -and "$cleanedYourPublicIp" -ne "none") {
      Log "Error: You cannot use a public IP address unless you are using GodMode"
      ExitWithNotice
    }
  ### <--snipped--->
```
(Note that $$(godMode) has double dollar signs because ADO returns booleans as true/false strings and adding a `$` converts it correctly into $true/$false booleans.)


To refer to global variables from within script blocks, do not use parameters.

#### Bad:

```
steps:
  - task: AzureCLI@2
    name: PulumiLogin
    displayName: Pulumi Login
    inputs:
      scriptType: 'pscore'
      scriptLocation: 'inlineScript'
      inlineScript: |
        Write-Host "Logging in to pulumi state file on ${{ parameters.AZURE_STORAGE_ACCOUNT }}"
        pulumi login azblob://pulumistate?storage_account=${{ parameters.AZURE_STORAGE_ACCOUNT }}
```

You can instead, use environment variables via the following pattern:

### Good:
```
steps:
  - task: AzureCLI@2
    name: PulumiLogin
    env:
      AZURE_STORAGE_ACCOUNT: $(AZURE_STORAGE_ACCOUNT)
    inputs:
      scriptType: 'pscore'
      scriptLocation: 'inlineScript'
      inlineScript: |
        Log "env:AZURE_STORAGE_ACCOUNT = $env:AZURE_STORAGE_ACCOUNT"
        pulumi login azblob://pulumistate?storage_account=$env:AZURE_STORAGE_ACCOUNT
```

Note the addition of `env` block and referring to `$env:AZURE_STORAGE_ACCOUNT` instead of `${{ parameters.AZURE_STORAGE_ACCOUNT }}`

## 6. Prefer multiline over single line for long arguments list and comments

Reading long lines is troublesome and error-prone during both development and code review.

#### Bad:
```
- task: Bash@3
  displayName: "docker build"
  inputs:
    arguments: "$(System.AccessToken) $(tag) $(containerRepositoryPrefix)-${{ lower(function) }} $(containerRegistryName).azurecr.io $(StorageAccountKey)"
```

In the example above, the single line requires scrolling, thus making it easy to miss something.
Compare to good example below:

#### Good:
```
- task: Bash@3
  displayName: "docker build"
  inputs:
    arguments: |
      $(System.AccessToken)
      $(tag)
      $(containerRepositoryPrefix)-${{ lower(function) }}
      $(containerRegistryName).azurecr.io
      $(StorageAccountKey)
```

## 7. Avoid silent failures when running Azure CLI commands

We often need to run some arbritary commands (such as `az`) in our PowerShell scripts. However, these commands have a tendency to continue script execution after failure, and not throw an error to `stderr` as the wise might advise. The following wrapper function exits in a cleaner manner.

#### Good:
```
function RunCmdAndExitOnErr($cmdStr) {
    # This function runs arbitrary powershell commands (typically az ones)
    # If the command is unsuccessful, the erroring command is printed & the program exits
    $output = (Invoke-Expression -Command $cmdStr 2>&1)

    if (-not $output) {
        Log "Error running: $cmdStr "
        Log "`nExiting.."
        Exit 42
    }
    return $output
}
```

Usage:
```
$faNames = (RunCmdAndExitOnErr -cmdStr "az functionapp list --resource-group $InfraResourceGroupName --query '[].name' -o tsv")
```
