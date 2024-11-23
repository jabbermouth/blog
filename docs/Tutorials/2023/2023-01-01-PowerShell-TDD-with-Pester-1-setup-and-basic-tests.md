# PowerShell TDD with Pester – Setup and Basic Tests (part 1 of 3)

Following on from my experiment with [TDD using C#](../2022/2022-12-29-Test-Driven-Development-with-csharp-part-1-introduction-and-setup.md), I wanted to look at TDD with PowerShell as that’s another language I currently use quite a bit. To do this, I will be using the [Pester](https://pester.dev/) framework. These articles are covering how I built a function to generate a new version number for a NuGet package, Helm chart, etc… based upon the version number in a file (e.g. chart.yaml or csproj) and the latest published version.

In these articles you may see more casting (e.g. `[SemanticVersion]`) than is necessary however I find it’s clearer to include it when dealing with non-basic types likes strings and ints.

As with my C# TDD articles, this will also have a GitHub repository made available. When it is, the link to it will appear here.

These articles are evolving so, particularly the third part, will change over time.

## Pester Installing or Updating

I’m using Windows 11, PowerShell Core 7 and VS Code for this project so any instructions will be for that platform.

To install the latest version of Pester and ensure any existing versions don’t cause issues, including the default v3 installed with Windows, run the following command:

```powershell
Install-Module -Name Pester -Force
Import-Module Pester
```

The second command is to force the current session to pick up the latest version of Pester.

## First Test – Return 0.0.1 If No Version Exists

The first test should return 0.0.1 if no existing version is found and the csproj file version is 0.0.1 or less, no pre-release tag is specified and no build number is specified.

### Failing Test

```powershell
using namespace System.Management.Automation

BeforeAll {
  . $PSScriptRoot/Get-NextVersion.ps1
}

Describe 'Get-NextVersion' {
  It 'Return 0.0.1 when no meaningful version exists' {
    # Arrange
    $fileVersion = [SemanticVersion]"0.0.0"
    $publishedVersion = [SemanticVersion]"0.0.0"
    $preReleaseLabel = ""
    $buildLabel = ""

    $expectedVersion = [SemanticVersion]"0.0.1"

    # Act
    $newVersion = [SemanticVersion](Get-NextVersion -FileVersion $fileVersion -PublishedVersion $publishedVersion -PreReleaseLabel $preReleaseLabel -BuildLabel $buildLabel)

    # Assert
    $newVersion | Should -Be $expectedVersion
  }
}
```

To make this test run, an empty function definition should be added to the main ps1 file:

```powershell
function Get-NextVersion {
  
}
```

The tests can then be ran using the following command:

```powershell
Invoke-Pester -Output Detailed .\Get-NextVersion.Tests.ps1
```

### Making the Test Green

To make the test pass, the easiest thing to do is simply return the expected version of 0.0.1 so that is what we’ll do.

```powershell
function Get-NextVersion {
  Write-Output "0.0.1"
}
```

## Second Test – Returning One Patch Higher Than The Latest Published Package

This test will ensure that any version number has a one higher patch number than the latest published patch. At this stage, the pre-release tag and build number are being ignored.

### Failing Test
  
```powershell
It 'Return one patch higher than existing published version' {
  # Arrange
  $fileVersion = [SemanticVersion]"0.0.0"
  $nugetVersion = [SemanticVersion]"0.3.1"
  $preReleaseLabel = ""
  $buildLabel = ""

  $expectedVersion = [SemanticVersion]"0.3.2"

  # Act
  $newVersion = [SemanticVersion](Get-NextVersion -FileVersion $fileVersion -PublishedVersion $publishedVersion -PreReleaseLabel $preReleaseLabel -BuildLabel $buildLabel)

  # Assert
  $newVersion | Should -Be $expectedVersion
}
```

### Making the Test Green

To make the test green, we need to add a parameter to the `Get-NextVersion` function and then use it to add suitable logic. We should only add the needed parameter from the ones currently being passed.

The resulting function code will look like this:

```powershell
using namespace System.Management.Automation

function Get-NextVersion {
  param (
    [SemanticVersion] $PublishedVersion
  )

  $newVersion = [SemanticVersion]"0.0.1"
  $notSet = [SemanticVersion]"0.0.0"
  
  if ($NuGetVersion -ne $notSet) {
    $newVersion = [SemanticVersion]::new($PublishedVersion.Major, $PublishedVersion.Minor, $PublishedVersion.Patch + 1, $PublishedVersion.PreReleaseLabel, $PublishedVersion.BuildLabel)
  }

  Write-Output $newVersion
}
```

This will work but could mean a lot of repeated and similar code each time one or more parts of the version change. Therefore, it would make sense to create a function for building up a new version, only updating those values which have changed.

## Third Test – Function To Build New SemanticVersion When All Parameters Are Set

This test will pass in all new values for a SemanticVersion object and return a new version. This is effectively replicating the constructor but test four will handle no parameters being passed making this new function useful as only the changed values will need to be passed.

### Failing Test

```powershell
Describe 'Get-UpdatedSemanticVersion' {
  It 'Returns completely new version when all parameters are set' {
    # Arrange
    $currentVersion = [SemanticVersion]"1.2.3-dev+build1"
    $expectedVersion = [SemanticVersion]"4.5.6-new+build2"

    # Act
    $newVersion = [SemanticVersion](Get-UpdatedSemanticVersion $currentVersion -MajorVersion 4 -MinorVersion 5 -PatchVersion 6 -PreReleaseLabel "new" -BuildLabel "build2")

    # Assert
    $newVersion | Should -Be $expectedVersion
  }
}
```

Also, so the test fails (rather than fails to execute), add the following method to Get-NextVersion.ps1:

```powershell
function Get-UpdatedSemanticVersion {
}
```

### Making the Test Green

Making the test pass when all the parameters are specified is relatively straight forward:

```powershell
function Get-UpdatedSemanticVersion {
  param (
    [Parameter(Mandatory, Position=0)] [SemanticVersion] $CurrentVersion,
    [int] $MajorVersion,
    [int] $MinorVersion,
    [int] $PatchVersion,
    [string] $PreReleaseLabel,
    [string] $BuildLabel
  )

  $newVersion = [SemanticVersion]::new($MajorVersion, $MinorVersion, $PatchVersion, $PreReleaseLabel, $BuildLabel)

  Write-Output $newVersion
}
```

But to make this test useful, only updating the new values passed is key so a new test is needed.

## Fourth Test – Return Current Version When No Parameters Passed

### Failing Test

```powershell
  It 'Returns current version when all parameters except CurrentVersion are not set' {
    # Arrange
    $currentVersion = [SemanticVersion]"1.2.3-dev+build1"
    $expectedVersion = [SemanticVersion]"1.2.3-dev+build1"

    # Act
    $newVersion = [SemanticVersion](Get-UpdatedSemanticVersion $currentVersion)

    # Assert
    $newVersion | Should -Be $expectedVersion
  }
```

### Making the Test Green

We need to check if all parameters were passed or not. We can use `$PSBoundParameters` to do this by checking each parameter name.

```powershell
function Get-UpdatedSemanticVersion {
  param (
    [Parameter(Mandatory, Position=0)] [SemanticVersion] $CurrentVersion,
    [int] $MajorVersion,
    [int] $MinorVersion,
    [int] $PatchVersion,
    [string] $PreReleaseLabel,
    [string] $BuildLabel
  )

  if (($PSBoundParameters.ContainsKey('MajorVersion') -eq $false) -and
      ($PSBoundParameters.ContainsKey('MinorVersion') -eq $false) -and 
      ($PSBoundParameters.ContainsKey('PatchVersion') -eq $false) -and 
      ($PSBoundParameters.ContainsKey('PreReleaseLabel') -eq $false) -and
      ($PSBoundParameters.ContainsKey('BuildLabel') -eq $false)) {
    $newVersion = $CurrentVersion
  } else {
    $newVersion = [SemanticVersion]::new($MajorVersion, $MinorVersion, $PatchVersion, $PreReleaseLabel, $BuildLabel)
  }

  Write-Output $newVersion
}
```

## Fifth Test – Removing Labels

This fifth test will be testing two things in a way – removing the labels when they’re already defined and that only some parameters can be specified for updating.

### Failing Test

```powershell
  It 'Returns current version with labels when labels are set to blanks strings' {
    # Arrange
    $currentVersion = [SemanticVersion]"1.2.3-dev+build1"
    $expectedVersion = [SemanticVersion]"1.2.3"

    # Act
    $newVersion = [SemanticVersion](Get-UpdatedSemanticVersion $currentVersion -PreReleaseLabel "" -BuildLabel "")

    # Assert
    $newVersion | Should -Be $expectedVersion
  }
```

### Making the Test Green

We’ll do a check per parameter and set the new value to use to the current version if a new one isn’t specified.

```powershell
function Get-UpdatedSemanticVersion {
  param (
    [Parameter(Mandatory, Position=0)] [SemanticVersion] $CurrentVersion,
    [int] $MajorVersion,
    [int] $MinorVersion,
    [int] $PatchVersion,
    [string] $PreReleaseLabel,
    [string] $BuildLabel
  )

  if ($PSBoundParameters.ContainsKey('MajorVersion') -eq $false) {
    $MajorVersion = $CurrentVersion.Major
  }

  if ($PSBoundParameters.ContainsKey('MinorVersion') -eq $false) {
    $MinorVersion = $CurrentVersion.Minor
  }

  if ($PSBoundParameters.ContainsKey('PatchVersion') -eq $false) {
    $PatchVersion = $CurrentVersion.Patch
  }

  if ($PSBoundParameters.ContainsKey('PreReleaseLabel') -eq $false) {
    $PreReleaseLabel = $CurrentVersion.PreReleaseLabel
  }

  if ($PSBoundParameters.ContainsKey('BuildLabel') -eq $false) {
    $BuildLabel = $CurrentVersion.BuildLabel
  }

  $newVersion = [SemanticVersion]::new($MajorVersion, $MinorVersion, $PatchVersion, $PreReleaseLabel, $BuildLabel)

  Write-Output $newVersion
}
```

### Refactoring `Get-NextVersion` to Use This New Method

The code change is pretty simple and then, once made, re-run the tests to make sure all responses are working as expected.

Change the following line:

```powershell
$newVersion = [SemanticVersion]::new($PublishedVersion.Major, $PublishedVersion.Minor, $PublishedVersion.Patch + 1, $PublishedVersion.PreReleaseLabel, $PublishedVersion.BuildLabel)
```

To the following:

```powershell
$newVersion = [SemanticVersion](Get-UpdatedSemanticVersion $PublishedVersion -PatchVersion ($PublishedVersion.Patch + 1))
```

## Sixth Test – New Version Is One Version Higher Than the Higher of File Version and NuGet Version

This test is about ensuring the next version is higher than any existing version. Three data sets will be tried; one where file version is higher than published version, one where it’s lower and one where it’s the same.

### Failing Test

```powershell
  It 'New version is one version higher (<expectedVersion>) than the higher of file version (<fileVersion>) and published version (<publishedVersion>)' -ForEach @(
    @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.3"; ExpectedVersion = "1.0.4" }
    @{ FileVersion = "1.1.0"; PublishedVersion = "1.0.3"; ExpectedVersion = "1.1.1" }
    @{ FileVersion = "1.2.0"; PublishedVersion = "1.2.0"; ExpectedVersion = "1.2.1" }
  ) {
    # Arrange
    $fileVersion = [SemanticVersion]$fileVersion
    $publishedVersion = [SemanticVersion]$publishedVersion
    $preReleaseLabel = ""
    $buildLabel = ""

    $expectedVersion = [SemanticVersion]$expectedVersion

    # Act
    $newVersion = [SemanticVersion](Get-NextVersion -FileVersion $fileVersion -PublishedVersion $publishedVersion -PreReleaseLabel $preReleaseLabel -BuildLabel $buildLabel)

    # Assert
    $newVersion | Should -Be $expectedVersion
  }
```

You’ll notice that there are now 8 tests passing/failing even though we’ve only written six. That is because the -ForEach is effectively creating multiple variants of one test.

### Making the Test Green

We’ll need to add a new parameter (FileVersion) and use this to determine what the highest new version is. By doing independent checks and updating newVersion as appropriate, the code is easier to read. newVersion is now defaulting to 0.0.0 so that the patch can also be incremented. We’ll see in later tests that we may not always want this.

```powershell
function Get-NextVersion {
  param (
    [SemanticVersion] $FileVersion,
    [SemanticVersion] $PublishedVersion
  )

  $newVersion = [SemanticVersion]"0.0.0"
  $notSet = [SemanticVersion]"0.0.0"
  
  if ($FileVersion -ne $notSet -and $FileVersion -gt $newVersion) {
    $newVersion = $FileVersion
  }
  if ($PublishedVersion -ne $notSet -and $PublishedVersion -gt $newVersion) {
    $newVersion = $PublishedVersion
  }

  $newVersion = [SemanticVersion](Get-UpdatedSemanticVersion $newVersion -PatchVersion ($newVersion.Patch + 1))

  Write-Output $newVersion
}
```

## Seventh Test

As this article is getting quite long, I’ll stop here and continue in [the next article](2023-01-03-PowerShell-TDD-with-Pester-2-adding-more-rules.md) which will see more rules added and some mocking done.