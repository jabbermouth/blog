# Bitbucket, SSH auth, Visual Studio and VS Code

To get SSH key authentication working with Bitbucket from with both Visual Studio 2019 (or earlier potentially) and VS Code can be challenging. Some commands I’ve used are below.

## VS Code

```powershell
git config --global core.sshCommand C:/Windows/System32/OpenSSH/ssh.exe
```

## Visual Studio

Visual Studio 2022 doesn’t require additional changes but if running 2019 (i.e. 32-bit) or earlier, I had to run the following to get it to work but this then caused VS Code to stop authing properly.

```powershell
git config --global core.sshCommand "\"C:\Program Files\Git\usr\bin\ssh.exe\""
```

This requires Git being installed from [https://git-scm.com/](https://git-scm.com/).