PowerShell TDD with Pester – Adding More Rules (part 3 of 3)

This final article will cover reading mocked files and retrieving the version number from the CSProj or Chart.yaml file.

## Tenth Test – Reading From Chart.yaml File
The first test is to read a Helm’s Chart.yaml file, extract the current current chart version and call the Get-NextVersion function.

### Failing Test

```powershell
Describe 'Read-CurrentVersionNumber' {
  It 'Extract current chart version from Chart.yaml and call Get-NextVersion' {
    # Arrange
    $expected = [SemanticVersion]"1.0.0"

    # Act
    $actual = [SemanticVersion](Read-CurrentVersionNumber -File "Chart.yaml)

    # Assert
    $actual | Should -Be $expected
  }
}
```

To give a fail (rather than non-running) test, add the following function:

```powershell
function Read-CurrentVersionNumber {
  
}
```

### Making the Test Green

As we’re reading a YAML file, we’ll use the powershell-yaml module.

To make the test pass, simple return “1.0.0” from the function:

```powershell
function Read-CurrentVersionNumber {
  Write-Output "1.0.0"
}
```

### Refactor to Read (Faked) File

To make the code (simulate) reading a file, we need to add a parameter to pass in the file name, process the YAML and return the version.

```powershell
function Read-CurrentVersionNumber {
  param (
    [Parameter(Mandatory=$true)] [string] $file
  )

  Import-Module powershell-yaml

  $chartYaml = (Get-Content -Path $file | ConvertFrom-Yaml)

  Write-Output $chartYaml.version
}
```

We’ll also add some code to BeforeAll to mock certain functions to prevent module installs happening as part of unit tests.

```powershell
  function Get-Content {
    Write-Output "
    apiVersion: v2
    appVersion: 1.0.0
    description: A Helm chart to deploy applications to a Kubernetes cluster using standard settings
    name: Generalised
    type: application
    version: 1.0.0    
    "
```

## Eleventh Test – Reading From CSProj File

This will cover reading from the XML-formatted CSProj file.

```powershell
  It 'Extract current chart version from a CSProj file' {
    # Arrange
    $expected = [SemanticVersion]"1.2.0"

    # Act
    $actual = [SemanticVersion](Read-CurrentVersionNumber -File "My.Project.csproj")

    # Assert
    $actual | Should -Be $expected
  }
```

### Making the Test Green

We’ll put some logic around the code reading the version so that it handles YAML files vs XML files. We’ll also need to add another mock to handle the reading of the CSProj/XML file.

The code changes involve a new if statement that analyses the file name being passed:

```powershell
function Read-CurrentVersionNumber {
  param (
    [Parameter(Mandatory=$true)] [string] $file
  )

  $fileName = Split-Path $file -Leaf
  
  if ($fileName -eq "Chart.yaml") {
    Import-Module powershell-yaml

    $chartYaml = (Get-Content -Path $file | ConvertFrom-Yaml)

    Write-Output $chartYaml.version
  } elseif ($fileName.EndsWith(".csproj")) {
    $projectVersion = $(Select-Xml -Path "$file" -XPath '/Project/PropertyGroup/Version').Node.InnerText

    Write-Output $projectVersion
  }
}
```

To work, this needs to following stub adding:

```powershell
  function Select-Xml {
    Write-Output @{
      Node = @{
        InnerText = "1.2.0"
      }
    }
  }
```