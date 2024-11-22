# .NET 5/6, Docker and custom NuGet server

When using a custom NuGet server and you've added a nuget.config file to the solution, you'll need to add the following line to the default Dockerfile build by Visual Studio to allow the container to be built.

```dockerfile
COPY ["nuget.config", "/src/"]
```

This should be placed before the `RUN dotnet restore …` line.

The filename is case sensitive within the container so using all lowercase is recommended for the file name. If you need to change the case, you may need to do two commits of the file (e.g. rename it `NuGet.config` –commit–> `nuget1.config` –commit–> `nuget.config`).

If running in a CI/CD pipeline and you have fixed custom NuGet servers, you can inject a nuget.config file into the CI pipeline however the file will still need referencing in the Dockerfile as above to be correctly used by the container build process.