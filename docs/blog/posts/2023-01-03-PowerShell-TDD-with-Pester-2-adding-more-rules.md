---
title: PowerShell TDD with Pester – Adding More Rules (part 2 of 3)
description: In this article, we’ll add more rules to our version number generator to handle things like bug fixes to major versions, pre-release labels and build labels.
date: 2023-01-03
categories:
  - TDD
  - PowerShell
  - Pester
  - Tutorials
---
# PowerShell TDD with Pester – Adding More Rules (part 2 of 3)

In this article, we’ll add more rules to our version number generator to handle things like bug fixes to major versions, pre-release labels and build labels.

To help guide the rules being implemented, below is a table of the incrementation rules that will be applied.

| Element | Incrementation Notes |
| --- | --- |
| Major | Version in file used with major versions treated separately for incrementation purposes |
| Minor | Latest value used within major version. |
| Patch | One higher than previous version unless major or minor version has increased in which case should be 0 |
| Pre-Release Label | Removing it or leaving one in place won’t cause an increase (if no existing version exists on that level) but adding it will increase the patch version the label changing doesn’t matter |
| Build Label | Does not cause any incrementations. |

## Seventh Test – Minor Version Increase Resets Patch

This test is about minor version number changes causing the patch to reset to 0 when the minor version number changes but otherwise it should be increased.

### Failing Test

```powershell
It 'Minor version change in file version (<fileVersion>) compared to published version (<publishedVersion>) causes patch to reset to 0 (<expectedVersion>)' -ForEach @(
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0"; ExpectedVersion = "1.0.1" }
  @{ FileVersion = "1.1.0"; PublishedVersion = "1.0.0"; ExpectedVersion = "1.1.0" }
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.1.0"; ExpectedVersion = "1.1.1" }
  @{ FileVersion = "1.1.0"; PublishedVersion = "1.1.0"; ExpectedVersion = "1.1.1" }
) {
  # Arrange
  $fileVersion = [SemanticVersion]$fileVersion
  $publishedVersion = [SemanticVersion]$nugetVersion
  $preReleaseLabel = ""
  $buildLabel = ""

  $expectedVersion = [SemanticVersion]$expectedVersion

  # Act
  $newVersion = [SemanticVersion](Get-NextVersion -FileVersion $fileVersion -PublishedVersion $publishedVersion -PreReleaseLabel $preReleaseLabel -BuildLabel $buildLabel)

  # Assert
  $newVersion | Should -Be $expectedVersion
}
```

### Making the Test Green

To make this test work, a few small changes are needed. A resetPatch boolean variable is needed, defaulting to false and the published version check needs splitting and a new method adding. Finally, the new patch version calculator needs some extra logic to handle the resetPatch variable.

Once this is all put together, it looks like this:

```powershell
function Get-NextVersion {
  param (
    [SemanticVersion] $FileVersion,
    [SemanticVersion] $PublishedVersion
  )

  $newVersion = [SemanticVersion]"0.0.0"
  $notSet = [SemanticVersion]"0.0.0"
  $resetPatch = $false
  
  if ($FileVersion -ne $notSet -and $FileVersion -gt $newVersion) {
    $newVersion = $FileVersion
  }
  if ($PublishedVersion -ne $notSet) {
    if ($newVersion.Minor -gt $PublishedVersion.Minor) {
      $resetPatch = $true
    }
    if ($PublishedVersion -gt $newVersion) {
      $newVersion = $PublishedVersion
    }    
  }

  $patch = $newVersion.Patch + 1;
  if ($resetPatch) {
    $patch = 0
  }
  $newVersion = [SemanticVersion](Get-UpdatedSemanticVersion $newVersion -PatchVersion $patch)

  Write-Output $newVersion
}
```

This change will, however, cause our previous test to fail. To resolve this, update the second data set’s expected value from `1.1.1` to `1.1.0`.

## Eighth Test – Pre-release Label Increments Patch If Added But Not If Removed Or Already Present

This test builds the logic for handling the use of pre-release tags (e.g. “-dev” or “-test”). It should only increase the patch version if adding a label. The value of the label isn’t important.

### Failing Test

```powershell
It 'Pre-release tag increments patch if added (<preReleaseTag>) but not if removed or already present (<publishedVersion> --> <expectedVersion>)' -ForEach @(
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0"; PreReleaseLabel = "dev"; ExpectedVersion = "1.0.1-dev" }
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0-dev"; PreReleaseLabel = "dev"; ExpectedVersion = "1.0.0-dev" }
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0-dev"; PreReleaseLabel = ""; ExpectedVersion = "1.0.0" }
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0-dev"; PreReleaseLabel = "test"; ExpectedVersion = "1.0.0-test" }
  @{ FileVersion = "1.1.0"; PublishedVersion = "1.0.1-dev"; PreReleaseLabel = "dev"; ExpectedVersion = "1.1.0-dev" }
  @{ FileVersion = "1.1.0"; PublishedVersion = "1.0.1-dev"; PreReleaseLabel = ""; ExpectedVersion = "1.1.0" }
) {
  # Arrange
  $fileVersion = [SemanticVersion]$fileVersion
  $publishedVersion = [SemanticVersion]$publishedVersion
  $buildLabel = ""

  $expectedVersion = [SemanticVersion]$expectedVersion

  # Act
  $newVersion = [SemanticVersion](Get-NextVersion -FileVersion $fileVersion -PublishedVersion $publishedVersion -PreReleaseLabel $preReleaseLabel -BuildLabel $buildLabel)

  # Assert
  $newVersion | Should -Be $expectedVersion
}
```

### Making the Test Green

The main code changes required are to do with determining if the patch version should be incremented or not. This is largely centred around whether or not the pre-release label is on the existing published package or not.

```powershell
function Get-NextVersion {
  param (
    [SemanticVersion] $FileVersion,
    [SemanticVersion] $PublishedVersion,
    [string] $PreReleaseLabel
  )

  $newVersion = [SemanticVersion]"0.0.0"
  $notSet = [SemanticVersion]"0.0.0"
  $incrementPatch = $true
  $resetPatch = $false
  
  if ($FileVersion -ne $notSet -and $FileVersion -gt $newVersion) {
    $newVersion = $FileVersion
  }
  if ($PublishedVersion -ne $notSet) {
    if (![string]::IsNullOrWhiteSpace($PublishedVersion.PreReleaseLabel)) {
      $incrementPatch = $false
    }
    if ($newVersion.Minor -gt $PublishedVersion.Minor) {
      $resetPatch = $true
    }
    if ($PublishedVersion -gt $newVersion) {
      $newVersion = $PublishedVersion
    }    
  }

  $patch = $newVersion.Patch
  if ($resetPatch) {
    $patch = 0
  } elseif ($incrementPatch) {
    $patch++
  }
  $newVersion = [SemanticVersion](Get-UpdatedSemanticVersion $newVersion -PatchVersion $patch -PreReleaseLabel $PreReleaseLabel)

  Write-Output $newVersion
}
```

## Ninth Test – Any Build Label Stops Patch Incrementation

If a build label is specified, it will stop patch incrementation from happening. It won’t stop resets though.

### Failing Test

```powershell
It 'Any build label (<buildLabel>) stops patch incrementation (<publishedVersion> --> <expectedVersion>)' -ForEach @(
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0-dev"; PreReleaseLabel = "dev"; BuildLabel = "1234"; ExpectedVersion = "1.0.0-dev+1234" }
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0-dev+1234"; PreReleaseLabel = "dev"; BuildLabel = "1234"; ExpectedVersion = "1.0.0-dev+1234" }
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0-dev+1233"; PreReleaseLabel = "dev"; BuildLabel = "1234"; ExpectedVersion = "1.0.0-dev+1234" }
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0"; PreReleaseLabel = ""; BuildLabel = "1234"; ExpectedVersion = "1.0.0+1234" }
  @{ FileVersion = "1.0.0"; PublishedVersion = "1.0.0"; PreReleaseLabel = "dev"; BuildLabel = "1234"; ExpectedVersion = "1.0.1-dev+1234" }
) {
  # Arrange
  $fileVersion = [SemanticVersion]$fileVersion
  $publishedVersion = [SemanticVersion]$publishedVersion

  $expectedVersion = [SemanticVersion]$expectedVersion

  # Act
  $newVersion = [SemanticVersion](Get-NextVersion -FileVersion $fileVersion -PublishedVersion $publishedVersion -PreReleaseLabel $preReleaseLabel -BuildLabel $buildLabel)

  # Assert
  $newVersion | Should -Be $expectedVersion
}
```

### Making the Test Green

To make this test, the build number parameter needs adding and an “else if” condition adding after the pre-release label check to stop incrementing if a new tag isn’t present but a build number is.

```powershell
function Get-NextVersion {
  param (
    [SemanticVersion] $FileVersion,
    [SemanticVersion] $PublishedVersion,
    [string] $PreReleaseLabel,
    [string] $BuildLabel
  )

  $newVersion = [SemanticVersion]"0.0.0"
  $notSet = [SemanticVersion]"0.0.0"
  $incrementPatch = $true
  $resetPatch = $false
  
  if ($FileVersion -ne $notSet -and $FileVersion -gt $newVersion) {
    $newVersion = $FileVersion
  }

  if ($PublishedVersion -ne $notSet) {
    if (![string]::IsNullOrWhiteSpace($PublishedVersion.PreReleaseLabel)) {
      $incrementPatch = $false
    } elseif ([string]::IsNullOrWhiteSpace($PreReleaseLabel) -and ![string]::IsNullOrWhiteSpace($BuildLabel)) {
      $incrementPatch = $false
    }
    if ($newVersion.Minor -gt $PublishedVersion.Minor) {
      $resetPatch = $true
    }
    if ($PublishedVersion -gt $newVersion) {
      $newVersion = $PublishedVersion
    }    
  }

  $patch = $newVersion.Patch
  if ($resetPatch) {
    $patch = 0
  } elseif ($incrementPatch) {
    $patch++
  }
  $newVersion = [SemanticVersion](Get-UpdatedSemanticVersion $newVersion -PatchVersion $patch -PreReleaseLabel $PreReleaseLabel -BuildLabel $BuildLabel)

  Write-Output $newVersion
}
```

## Tenth Test – Reading Files

The next test, covered in [the next blog](2023-01-03-PowerShell-TDD-with-Pester-3-adding-more-rules.md), will cover testing the process of reading files.