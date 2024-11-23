---
title: Build container from Visual Studio built Dockerfile
description: If you need to build a project using the Visual Studio generated Dockerfile, you need to run docker build from the solution folder and specify the file.
date: 2022-01-02
categories:
  - Docker
  - Quick Tips
  - Visual Studio
---
# Build container from Visual Studio built Dockerfile

If you need to build a project using the Visual Studio generated Dockerfile, you need to run docker build from the solution folder and specify the file.

Assuming the project you are working is called Your.Project.Api and this is the name of the project folder, run the following command from the root folder where the solution file and project folder(s) are located.

```powershell
docker build -f Your.Project.Api/Dockerfile .
```

For Windows, you can use a forward or back slash but Linux must be a forward slash.