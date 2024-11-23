---
title: Setup local Redis container
description: A quick guide on how to set up a local Redis container.
date: 2022-08-09
categories:
  - Docker
  - Redis
  - Quick Tips
---
# Setup local Redis container

To set up a local Redis container that automatically starts up on reboot, run the following command:

```powershell
docker run --name redis -d --restart always -p 6379:6379 redis:latest
```

To include a UI, run the following command:

```powershell
docker run --name redis-ui -d --restart always -p 8001:8001 redislabs/redisinsight
```