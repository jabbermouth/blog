---
title: Run an NGINX server for a folder
description: If you're testing a simple frontend-only web site and want an NGINX web server to point at it to let you access the page in a browser via a web server, run the following command from PowerShell within the folder that contains the HTML, etc… to set up a NGINX Docker container pointing at your HTML folder.
date: 2021-05-05
categories:
  - Docker
  - NGINX
  - Tutorials
---
# Run an NGINX server for a folder

If you're testing a simple frontend-only web site and want an NGINX web server to point at it to let you access the page in a browser via a web server, run the following command from PowerShell within the folder that contains the HTML, etc… to set up a NGINX Docker container pointing at your HTML folder:

```powershell
docker run -it -d -p 8100:80 -v ${PWD}:/usr/share/nginx/html nginx
```

If you're using command prompt instead of PowerShell, use:

```powershell
docker run -it -d -p 8100:80 -v %cd%:/usr/share/nginx/html nginx
```

You'll then be able to access your site on [http://localhost:8100/](http://localhost:8100/). You can set the port to any value that's not in use between 1 and 65535 by updating the first part of the `-p` attribute.