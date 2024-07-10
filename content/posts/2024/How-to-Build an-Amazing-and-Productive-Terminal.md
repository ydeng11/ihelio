---
title: "How to Build an Amazing and Productive Terminal"
date: 2024-07-09T21:37:47-04:00
draft: false
categories: 
    - Programming
tags: 
    - Terminal
    - CLI Tools
    - WezTerm
    - DevEnv
---

Everyone wants a cool terminal and WezTerm is the coolest one in my opinion. It is easy to manage and we can setup a new environment with minutes in association with GNS stow - a symlink manager.

WezTerm is easy to configure since it is using lua and very friendly to read and learn. I am dumb at iTerm2 and Tmux, but it only took me 30 mins to start tweaking the config. 
Everyone wants a cool terminal, and WezTerm is the coolest one in my opinion. It's easy to manage and we can set up a new environment within minutes using GNU Stow - a symlink manager.

WezTerm is easy to configure since it uses Lua, making it very friendly to read and learn. I was baffled by iTerm2 and Tmux, but it only took me 30 minutes to start tweaking the config for WezTerm.

# WezTerm

The installation can be found at the [download page](https://wezfurlong.org/wezterm/installation.html). You can create the config file under `~/.config/wezterm/wezterm.lua`. The [website](https://wezfurlong.org/wezterm/config/files.html) has a detailed description of how WezTerm resolves the config path, which is a nice decision tree. Since it's Lua, it's very easy to read and understand. I started from others' configs and you could also start with [mine](https://github.com/ydeng11/dotfiles/blob/main/config/.config/wezterm/wezterm.lua).

# Look Cool

First, we need to make it look cool. `zsh`, `oh-my-zsh`, and `powerlevel10k` should be your go-to tools since they take little time to set up and look amazing. With `stow`, we can reuse the config, so it's a one-time cost but for lifelong use. Their GitHub repos have very detailed instructions about installation, and I just want to highlight that you should always use a package manager to install them.

# Act Cool

## Interactive Search & Git Diff

Looking cool is one thing, but your terminal also needs to act cool. Here's how to improve the visualization of your terminal using `fzf-preview`. You need to install:

- fzf - a fuzzy search tool
- fzf-git - integrate fzf with git
- bat - a better `cat`
- git-delta - a better tool to show `git diff`
- fd - an opinionated tool to replace `find`
- eza - an alternative to `ls`

To install these tools:

`brew install fzf bat git-delta fd eza`

`git clone git@github.com:junegunn/fzf-git.sh.git ~/.fzf-git.sh`

After installing them, add the following snippet to your `.zshrc`:

```shell
# Setup fzf previews
show_file_or_dir_preview="if [ -d {} ]; then eza --tree --color=always {} | head -200; else bat -n --color=always --line-range :500 {}; fi"

export FZF_CTRL_T_OPTS="--preview '$show_file_or_dir_preview'"
export FZF_ALT_C_OPTS="--preview 'eza --tree --color=always {} | head -200'"

_fzf_comprun() {
  local command=$1
  shift

  case "$command" in
    cd)           fzf --preview 'eza --tree --color=always {} | head -200' "$@" ;;
    export|unset) fzf --preview "eval 'echo ${}'"         "$@" ;;
    ssh)          fzf --preview 'dig {}'                   "$@" ;;
    *)            fzf --preview "$show_file_or_dir_preview" "$@" ;;
  esac
}

source ~/fzf-git.sh/fzf-git.sh
```

There are more useful configs which can be found in my [dotfiles](https://github.com/ydeng11/dotfiles/blob/main/zsh/.zshrc).

Then you can type `fzf **` and press `tab` to get an interactive console like this: !

![Interactive console](https://i.imgur.com/tyuxKAL.png)
## zoxide

Zoxide is a smarter `cd` command for navigating your filesystem. It keeps track of the directories you use most frequently, allowing you to jump to them quickly with short commands. Zoxide learns your habits and improves over time, making filesystem navigation more efficient. It saves me a lot of time jumping among different folders.

To install and configure Zoxide:

`brew install zoxide`
## atuin

Atuin is a history management tool. Though `fzf` can also make history search easy and powerful, `atuin` has its place since you only need to press the up-arrow to get into the history interface, and it comes with fuzzy search. Additionally, `atuin` enables global history and allows you to share history across multiple platforms.

To install Atuin:

`brew install atuin`

Search the history:

![Atuin interface](https://i.imgur.com/zj75bBA.png)
## sdkman

SDKMAN! is a tool for managing parallel versions of multiple Software Development Kits (SDKs), including Java, Groovy, Scala, Kotlin, and more. It simplifies the installation, switching, and management of SDKs, ensuring that you always have the right version for your projects. For me, setting up the Java SDK was a nightmare before SDKMAN! It’s a game changer. `sdk list`, `sdk install <sdk version>`, and `sdk use <sdk version>` are all you need.

To install SDKMAN!:

`brew install sdkman`

To install a specific version of Java using SDKMAN!:

`sdk install java 11.0.11-open`

## tldr

Nobody wants to read the full man pages (no offense, `man`). Install `tldr` to get simplified, community-driven help pages for command-line tools.

To install tldr:

`brew install tlrc`

![tldr example](https://i.imgur.com/0ejnQYy.png)

## stow

Lastly, GNU Stow is very powerful and easy to use. I used "yet another dotfiles manager" before, but the experience was unpleasant. Stow, on the other hand, is very easy to use. It is a symlink manager that creates symlinks for dotfiles like below:
![stow example](https://i.imgur.com/Kf0HKue.png)

What you need to do is create a folder like `dotfiles` to hold the dot files. For example, I group Zsh-related files under `~/dotfiles/zsh`, and I run `stow zsh` in `~/dotfiles` to create these symlinks. For `.config`, I copy them into `~/dotfiles/config`, and then run `stow config` to create symlinks for the `.config` folder.

To install Stow:

`brew install stow`

By using Stow, you can have version control of your dotfiles by creating a Git repo. Note that the dotfiles should not exist in the home directory directly; otherwise, there will be conflicts for Stow to create symlinks. All my dotfiles can be found [here](https://github.com/ydeng11/dotfiles/tree/main).

In conclusion, by combining these tools and packages, you can create a terminal environment that is not only visually appealing but also highly functional and productive. Each tool adds a layer of efficiency, helping you work faster and smarter, and let's be honest, look cooler while doing it.
