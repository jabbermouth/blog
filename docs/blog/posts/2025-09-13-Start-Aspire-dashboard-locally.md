---
title: Start Aspire dashboard locally
description: Run the Aspire dashboard in standalone mode using Docker
date: 2025-09-13
categories:
  - Aspire
  - Docker
  - Quick Tips
  - Observability
---
# Start Aspire dashboard locally

Running the Aspire dashboard locally and independent of Aspire can be useful.  Instructions are available in the [official documentation](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/overview?tabs=powershell).

The following command will run the Aspire dashboard in a Docker container, mapping ports 18888 and 4317 on the host to the container. The `--restart always` option ensures that the container will automatically restart if it stops or if the Docker daemon is restarted. `--env ASPIRE_DASHBOARD_UNSECURED_ALLOW_ANONYMOUS=true` sets the dashboard to run in open mode, removing the need to enter a token each time.

```powershell
docker run --restart always -it -p 18888:18888 -p 4317:18889 --env ASPIRE_DASHBOARD_UNSECURED_ALLOW_ANONYMOUS=true -d --name aspire-dashboard mcr.microsoft.com/dotnet/aspire-dashboard:9.4
```

Once running, you can access the dashboard at [http://localhost:18888](http://localhost:18888).