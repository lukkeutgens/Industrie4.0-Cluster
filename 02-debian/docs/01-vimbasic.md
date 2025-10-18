# VIM Basic Install
Debian ships with `vim.tiny` by default, which is too limited for practical use — especially when editing YAML files, scripts, or configuration files in a cluster environment.
This step installs the full version of VIM and applies a few usability tweaks.

## Installation
```bash
sudo apt update
sudo apt install vim
```

## Colorscheme setup
I'm using the desert colorscheme for better readability
Edit file:
```bash
sudo vi /etc/vim/vimrc
```
Make the following changes:
- Uncomment the line: syntax on
- Add the line directly below syntax on: colorscheme desert
```bash
syntax on
colorscheme desert
```
Save & Exit the file

