---
title: Output variables in PowerShell and GitHub Actions
description: How to output variables in PowerShell within GitHub Actions and access them in other steps
date: 2025-01-05
categories:
  - GitHub Actions
  - PowerShell
  - Quick Tips
---
# Output variables in PowerShell and GitHub Actions

When running a PowerShell script within a GitHub Actions workflow, you may wish to output variables from one step and access them in another.

Below is a simple example of outputting a variable in one step and accessing it in another:

```yaml
name: Pass value from one step to another
on: 
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  PassTheParcel:
    runs-on: ubuntu-latest
    steps:
    - name: Determine some values
      id: calculate_values
      shell: pwsh
      run: |
        $ErrorActionPreference = "Stop" # not required but I always include it so errors don't get missed

        # Normally these would be calculated rather than just hard-coded strings
        $someValue = "Hello, World!"
        $anotherValue = "Goodbye, World!"

        # Outputting for diagnostic purposes - don't do this with sensitive data!
        Write-Host "Some value: $someValue"
        Write-Host "Another value: $anotherValue"

        "SOME_VALUE=$someValue" | Out-File -FilePath $env:GITHUB_OUTPUT -Append
        "ANOTHER_VALUE=$anotherValue" | Out-File -FilePath $env:GITHUB_OUTPUT -Append

    - name: Use the values
      shell: pwsh
      env: 
        VALUE_TO_USE_1: ${{ steps.calculate_values.outputs.SOME_VALUE }}
        VALUE_TO_USE_2: ${{ steps.calculate_values.outputs.ANOTHER_VALUE }}
      run: |
        $ErrorActionPreference = "Stop" # not required but I always include it so errors don't get missed

        Write-Host "Values received were `"$($env:VALUE_TO_USE_1)`" and `"$($env:VALUE_TO_USE_2)`""

```

Whilst both examples use PowerShell, there is no requirement that both steps use the same shell.