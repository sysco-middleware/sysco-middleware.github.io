---
layout: post
title: Simple tip for file transfers
categories: Quick tips
tags: [Python, Quick tips]
author: catoaune
---
Quick tip: Simple solution for file transfers

Sometimes you need to transfer files from a server or pc, without having the necessary tools available. Other times you just need a http server since some other system requires an url to get the files you need to import (like when you are importing a template into an Oracle VM repository)

This tips is not new, but it is still good, and I think more people should know about it.

If you have python installed, simply go the the directory containing the files you want to access

```bash
python -m SimpleHTTPServer
```
This command starts a (simple) HTTP server on port 8000
If you would like to use a different port, just add the port number (ports under 1024 requires special permission to use, you have to start the server with root/administrator privileges)

```bash
python -m SimpleHTTPServer 8888
```

To stop the server, just press CTRL and C

NB! As many other simple solutions, this could also do harm. Think (and check the security policies for your company) before you run a web server
