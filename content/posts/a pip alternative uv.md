---
title: "UV - a   Pip Alternative"
date: 2024-04-10T19:19:02+08:00
tags:
 - python
 - tools
draft: false
---
Recently I discover a new package manager for Python, `uv`, which could be a pip alternative written in rust. Just like all the "Rust version" remakes, it has an incredible speed boost compared to pip or poetry. UV can even generate a virtual environment (like the python venv).  

![alt text](../../images/uv-official-benchmark.png)  
This is the official benchmark from the uv project description.  
Just by the glance of this graph I'm alredy amazed by the claim.

[Official Website](https://astral.sh/blog/uv)
## Intro
### Installation
```shell
pip install uv
# or 
$ curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Virtual Environment
#### Usage
```shell
# original
python -m venv $virtual_env_name
# uv
uv venv $virtual_env_name
```
#### Local Performance
```shell
$ time python3 -m venv env
python3 -m venv env  2.11s user 0.58s system 72% cpu 3.695 total

# vs 

$ time uv venv env
Using Python 3.10.13 interpreter at: /Users/username/.pyenv/versions/3.10.13/bin/python3
Creating virtualenv at: env
Activate with: source env/bin/activate
uv venv env  0.01s user 0.02s system 5% cpu 0.537 total
```
### Package Installation
#### Usage
```shell
# original
pip install $package_name
# uv
uv pip install $package_name
```
#### Local Performance
```shell
$ time pip3 install --no-cache-dir requests
Collecting requests
  Downloading requests-2.31.0-py3-none-any.whl (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.6/62.6 kB 1.1 MB/s eta 0:00:00
...

Successfully installed certifi-2024.2.2 charset-normalizer-3.3.2 idna-3.6 requests-2.31.0 urllib3-2.2.1
pip3 install --no-cache-dir requests  0.74s user 0.18s system 31% cpu 2.907 total

# vs

$ time uv pip install --no-cache-dir requests
Resolved 5 packages in 325ms
Downloaded 5 packages in 256ms
Installed 5 packages in 7ms
 + certifi==2024.2.2
 + charset-normalizer==3.3.2
 + idna==3.6
 + requests==2.31.0
 + urllib3==2.2.1
uv pip install --no-cache-dir requests  0.08s user 0.13s system 16% cpu 1.228 total
```

### Package Installation Using Same Requirements.txt
```shell
$ time pip3 install -r requirements.txt
Collecting anyio==3.7.1
  Using cached anyio-3.7.1-py3-none-any.whl (80 kB)

...

pip3 install -r requirements.txt  13.57s user 3.98s system 69% cpu 28.342 total

# vs 

$ time uv pip install -r requirements.txt --no-cache-dir
Resolved 77 packages in 22ms
Installed 77 packages in 171ms

...

uv pip install -r requirements.txt  0.05s user 0.29s system 40% cpu 0.849 total
```


> Note that `uv pip` & `pip` are seperate package managers  

> In the official uv benchmarks, uv is 8-10x faster than pip and pip-tools without caching, and 80-115x faster when running with a warm cache (e.g., recreating a virtual environment or updating a dependency)

### Poetry Compatibility
Poetry is eliminating all pip usage in installations, it seems they don't plan to support uv in near future: 
[issue link](https://github.com/python-poetry/poetry/issues/8978)