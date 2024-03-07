---
title: "Pyenv - Manage Python Environments Easily"
date: 2024-03-03T23:28:57+08:00
tags:
 - python
 - tools
draft: false
---
### What is Pyenv?
When you got multiple python projects at hand that use different python versions, uninstall and reinstall between them makes your python environment a mess.  
With pyenv, a Python version management tool, can let developers easily manage, install and switch between python versions.  
### Installation
pyenv only work on UNIX/MacOS systems, the alternative on Windows is `pyenv-win`  
  
You can install using homebrew  
```shell
brew update
brew install pyenv
```
#### Set Up Shell Environment
You should be able to check your shell enviornment with this command: `echo $SHELL`, then execute the following command accordingly    
##### ZSH
```shell
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```
> Refer to [this](https://arc.net/l/quote/gtbqrdas) for detail setup on fish shell or bash    

Restart your shell to let the settings take effect  
```shell
exec $SHELL
```
You can execute `pyenv init -` to check whether it works, this should output some command-like lines.  
### Install Python with Pyenv
```shell
# list all available versions
pyenv install -l
# install desire version
pyenv install 3.12.2
# check all versions pyenv has installed
pyenv versions
# check current activate pyenv version
pyenv version
```  
### Setting Global and Local Version
```shell
pyenv versions
# output
# * system (set by /Users/user/.pyenv/version)
#   3.9.18
#   3.10.13
pyenv global 3.10.13
# or
pyenv local 3.10.13
```