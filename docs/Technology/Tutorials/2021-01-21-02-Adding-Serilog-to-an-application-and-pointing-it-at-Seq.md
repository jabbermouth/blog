# Adding Serilog to an application and pointing it at Seq

Firstly, let's add all the NuGet packages we'll need:

- Serilog.AspNetCore
- Serilog.Enrichers.Environment
- Serilog.Enrichers.Process
- Serilog.Enrichers.Thread
- Serilog.Settings.Configuration
- Serilog.Sinks.Seq

Create a new file called serilog.json and populate it with the following, replacing the serverUrl and apiKey (or completely removing that element if not required).

```json
{
  "Serilog": {
    "Using": [],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "Enrich": [ "FromLogContext", "WithMachineName", "WithProcessId", "WithThreadId" ],
    "WriteTo": [
      {
        "Name": "Seq",
        "Args": {
          "serverUrl": "http://seq.yourdomain.com:5341",
          "apiKey": "YourApiKey"
        }
      },
      { "Name": "Console" }
    ]
  }
}
```

Next we'll modify Main in Program.cs. If all your Main methods contains is CreateHostBuilder(args).Build().Run(); then you can simply replace the contents with the following:

```csharp
var loggingConfiguration = new ConfigurationBuilder()
    .AddJsonFile("serilog.json")
    .AddEnvironmentVariables()
    .Build();

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(loggingConfiguration)
    .CreateLogger();

try
{
    Log.Information("Application starting up");
    CreateHostBuilder(args).Build().Run();
}
catch(Exception ex)
{
    Log.Fatal(ex, "Application start failed");
}
finally
{
    Log.CloseAndFlush();
}
```

Then, in the CreateHostBuilder method, after `Host.CreateDefaultBuilder(args)` add the following:

```csharp
.UseSerilog()
```

Finally, if you want to log requests, in the Configure method of Startup.cs, before `app.UseRouting()`, add the following line:

```csharp
app.UseSerilogRequestLogging();
```

If you encounter any missing usings, use control+. and add them as needed.

Don't forget to inject and use an ILogger instance of your controllers to log errors, warnings, etc…

The last thing to do is remove the Logging section from appsettings.json.

If you want to override the API key with an environment variable, for example as part of a deployment process, you would use the following variable name:

```csharp
Serilog__WriteTo__0__Args__apiKey
```

If you want to exclude health checks from request logging, [Andrew Lock's blog](https://andrewlock.net/using-serilog-aspnetcore-in-asp-net-core-3-excluding-health-check-endpoints-from-serilog-request-logging/) has a great solution. I've packaged this in a NuGet package, `Serilog.Extras`, along with a couple of enrichers that I required. Note this package is targeting .NET 6.

Once you've referenced the above NuGet package, replace:

```csharp
app.UseSerilogRequestLogging();
```

With the following:

```csharp
app.UseSerilogRequestLoggingWithoutHealthChecks();
```

You'll also need to add a using for Serilog.Extras to enable this extension.

The two enrichers are `WithVersionNumber` and `WithDeploymentEnvironment` which add the values of the environment variables `VERSION_NUMBER` and `ASPNETCORE_ENVIRONMENT` respectively to your output logs.