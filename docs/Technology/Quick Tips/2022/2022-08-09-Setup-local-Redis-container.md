# Setup local Redis container

To set up a local Redis container that automatically starts up on reboot, run the following command:

```powershell
docker run --name redis -d --restart always -p 6379:6379 redis:latest
```

To include a UI, run the following command:

```powershell
docker run --name redis-ui -d --restart always -p 8001:8001 redislabs/redisinsight
```