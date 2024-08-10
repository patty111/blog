---
title: "Setup My Mac"
date: 2024-08-10T17:04:06+08:00
tags:
 - setup
 - macbook
draft: false
---
### System Settings
#### Keyboard
- Key repeat rate and delay until repeat
![alt text](../../images/mac-keyboard.png)
- Modify shortcuts for screenshots
- Under `Desktop & Doc > Shortcuts` set `ShowDesktop` to right `option` key
- Enable Hot Corners under `Desktop & Doc > Hot Corners`
![alt text](../../images/hotcorners.png)
#### Menu Bar and Control Center
#### Cleanup Dock and move to right

### Configs
```bash
cd ~/Desktop/code
git clone https://github.com/patty111/configs.git
```
#### Homebrew
- Install homebrew
- Install packages
```bash
cd ~/Desktop/code/configs/macOS
brew bundle
```
#### raycast
- disable window navigation
- disable spotlight
#### iterm
- install nerd font CascadiaMono or CascadiaCove and import
#### rectangle
#### github
- ssh key setup
- git config
#### Install cargo
#### Solana cli
#### Zsh
- Install oh-my-zsh
- config file and plugins setup
```bash
cp ~/Desktop/code/configs/macOS/.zshrc ~/.zshrc
```
- replace username with new username in paths in config file
- install plugins and other tools
  - zsh-autosuggestions
  - zsh-autocomplete
  - colorls
#### ttyper install
#### pyenv
- setup, reference to [pyenv setup](https://patty111.github.io/blog/posts/pyenv-python%E7%89%88%E6%9C%AC%E7%AE%A1%E7%90%86/)
#### node
- fnm setup
#### vim and nvim

### Arc Browser
- Sync sidebar
- Import password csv
- extensions
### Bitwarden
### VSCode
### itsycal
### RunCat
### Postman
### Obsidian
- install
```bash
cd ~/Desktop
git clone git@github.com:patty111/obsidian-backup.git  
```
- replace .obsidian with the extracted .zip file, it includes the plugins and themes
- setup device name for git plugin