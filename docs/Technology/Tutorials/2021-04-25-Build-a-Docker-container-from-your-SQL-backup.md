# Build a Docker container from your SQL backup

The following Dockerfile and restore-database.sql file combine with a SQL backup file (my-database.bak in this example) to produce a Docker container. This will allow you to spin up a SQL Server container with your backup available via the SA account. Needless to say, this should not be used in a production environment without adding users, etc… as part of the process. Also, by default, the Developer edition is used and this is not licenced for production use.

The first step is to create a folder and in it put your my-database.bak file and two new files as below:

## Dockerfile

```dockerfile
FROM mcr.microsoft.com/mssql/server:2019-latest AS build
ENV ACCEPT_EULA=Y
ENV SA_PASSWORD="ARandomPassword123!"

WORKDIR /tmp
COPY *.bak .
COPY restore-backup.sql .

RUN ( /opt/mssql/bin/sqlservr & ) | grep -q "Service Broker manager has started" \
    && sleep 5 \
    && /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "ARandomPassword123!" -i /tmp/restore-backup.sql \
    && pkill sqlservr

FROM mcr.microsoft.com/mssql/server:2019-latest AS release

ENV ACCEPT_EULA=Y

COPY --from=build /var/opt/mssql/data /var/opt/mssql/data
```

## restore-backup.sql
  
```sql
RESTORE DATABASE [MyDatabase] 
FROM DISK = '/tmp/my-database.bak'
WITH FILE = 1,
MOVE 'MyDatabase_Data' TO '/var/opt/mssql/data/MyDatabase.mdf',
MOVE 'MyDatabase_Log' TO '/var/opt/mssql/data/MyDatabase.ldf',
NOUNLOAD, REPLACE, STATS = 5
GO

USE MyDatabase
GO

DBCC SHRINKFILE (MyDatabase_Data, 1)
GO

ALTER DATABASE MyDatabase
SET RECOVERY SIMPLE;  
GO  

DBCC SHRINKFILE (MyDatabase_Log,1)
GO

ALTER DATABASE MyDatabase
SET RECOVERY FULL;  
GO  

DBCC SHRINKDATABASE ([MyDatabase])
GO
```

Note that your database will not be MyDatabase_Data and MyDatabase_Log so you will need to replace these values with your actual database file names. Note that sometimes the equivalent of MyDatabase_Data doesn’t have _Data on the end.

To build the container, you then run the following command from within your folder:

```powershell
docker build -t database-backup:latest .
```

You can then run this container using the following command:

```powershell
docker run -d -p 11433:1433 --memory=6g --cpus=2 database-backup:latest
```

This will also limit your container to 6GB of RAM use and 2 CPU cores.

After 5 or 10 seconds, you can connect to your database from SQL Server Management Studio. The server name is “MachineName,11433” where MachineName is the name of your computer. For Authentication, choose “SQL Server Authentication” and for username enter “sa” and for password enter “ARandomPassword123!”.s