---
title: Getting Mongo DB running locally in a container
description: A quick guide to getting Mongo DB up and running in a container for you to connect to. I've included authentication even though it's not essential for working locally just for completeness.
date: 2021-02-07
categories:
  - MongoDB
  - Tutorials
---
# Getting Mongo DB running locally in a container

The following is a quick guide to getting Mongo DB up and running in a container for you to connect to. I've included authentication even though it's not essential for working locally just for completeness.

We'll assume you have a D drive for this example and that you want to persist your database in a folder on this drive.

```powershell
d:
cd \
mkdir Mongo
docker run --name mongo -v d:/Mongo:/data/db -d -e MONGO_INITDB_ROOT_USERNAME=mongoadmin -e MONGO_INITDB_ROOT_PASSWORD=OnlyForLocal123 -p 27017:27017 --restart always mongo
```

There you go – Mongo is now running on your local instance of Docker with a simple superuser username and password.

To connect to this database from C#, use the following connection string:

```
mongodb://mongoadmin:OnlyForLocal123@host.docker.internal:27017/?authSource=admin
```

To get a UI up and running, you can also instantiate the following container:

```powershell
docker run --name mongo-ui -d -e ME_CONFIG_MONGODB_ADMINUSERNAME=mongoadmin -e ME_CONFIG_MONGODB_ADMINPASSWORD=OnlyForLocal123 -e ME_CONFIG_MONGODB_SERVER=host.docker.internal -p 8081:8081 --restart always mongo-express
```

This will then be accessible via [http://localhost:8081](http://localhost:8081) once it's started.