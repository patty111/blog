---
title: "Github SSH Setup"
date: 2024-08-10T16:44:57+08:00
tags:
 - github
 - ssh
draft: false
---
Setup Authentication with Github using SSH, github username/password authentication is deprecated.

### Generate SSH Key
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
This should generate a public and private key pair in `~/.ssh/` directory. Should be something like `id_rsa` and `id_rsa.pub`.
### Github Setup
Go to `github > settings > SSH and GPG keys > New SSH key` and paste the contents of `id_rsa.pub` file. Key type used is authentication key.
### Test Connection
```bash
ssh -T git@github.com

# output
Hi XXXXX! You've successfully authenticated, but GitHub does not provide shell access.
```

### Permission Denied Error
If you get a permission denied error, you might need to add the key to the ssh-agent.
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
```

All set, you can now clone repositories using SSH.

