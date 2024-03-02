---
title: "ASGI? WSGI?"
date: 2023-12-11T04:45:36+08:00
tags:
 - python
 - backend
draft: false
---

Apache and Nginx are web servers. chorme, edge, etc... are web Clients

Web Servers serve static page, load balancing, reverse proxy, good at caching

**ASGI/WSGI** -> application server/web server gateway interface ^561c30

`gunicorn -w 5 -k uvicorn.workers.UvicornWorker main:app --access-logfile access.log --error-logfile error.log`

When deploying FastAPI apps we use Gunicorn(WSGI), which is actually managing ASGI servers(uvicorn), the `-k` part, assigning the worker type that is using. The app is actually still running on ASGI server, not running using WSGI. ^b0cb70

`-w 5` means 5 workers, can handle up to 5 requests simultaneously (simplify explaination)
