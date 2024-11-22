# Getting a local Seq instance up and running

To get Seq up and running locally, the quickest way is using Docker. For our example, we'll create a directory called SeqData on our D drive and then start a container pointing at this folder. We'll expose the UI on port 8090 and the ingest API on it's default port of 5341.

If you require a different folder location or to use different ports, adjust as needed.

If you're using WSL2 (which I recommended), you shouldn't need to set up “Shared Drives” in advance.

Run the following commands:

```powershell
mkdir D:\SeqData
docker run -e ACCEPT_EULA=Y --name seq -d --restart always -p 8090:80 -p 5341:5341 -v D:/SeqData:/data datalust/seq:latest
```

You should now be able to access the UI on [http://localhost:8090/](http://localhost:8090/) and you should point your logging source (e.g. your Serilog sink) at [http://localhost:5341](http://localhost:5341).