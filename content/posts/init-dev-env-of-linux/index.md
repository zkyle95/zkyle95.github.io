---
title: 初始化linux开发环境
description: Initializing development host of linux. 
slug: init-dev-env-of-linux
date: 2021-12-20 00:00:00+0000
image: linux.jpg
categories:
    - DevOps
tags:
    - Linux
---

> ***Note***
>
> - This article is just a personal note for recording my common setting of every new developing host of linux. 
> - Remember to source `/etc/profile` after specify any environment variables.

> ***Principle***
>
> Set environment variables in `/etc/profile.d/{{app}}.sh`  if you want to use it of system wide, keep `.bashr_profile` or `.bashrc` simple. 

## Beginning

```bash
# specify default editor
echo "alias ll='ls -lah'" >> /etc/profile.d/common.sh
echo "export EDITOR=/usr/bin/vim" >> /etc/profile.d/common.sh

# config vim 
cat <<EOF >> /etc/vim/vimrc
set tabstop=4
set shiftwidth=4
set autoindent
set smartindent
EOF
```

## Update packages

- Set proxy for apt if needed. 

    ```bash
    vim /etc/apt/apt.conf
    # Acquire::http::Proxy "http://USERNAME:PASSWORD@SERVER:PORT";
    # Acquire::https::Proxy "https://USERNAME:PASSWORD@SERVER:PORT";
    ```
    
- Or change original repository to [mirror](https://www.debian.org/mirror/list) if needed

- Update

    ```bash
    apt update
    apt upgrade
    ```

## User

- Init user

  ```bash
  # create home dir \w -m
  useradd -m {user}
  passwd {user}
  ```

- Specify the login shell

  ```bash
  # check what kind of shells had been installed
  cat /etc/shells
  
  # make sure the default login shell of the user 
  grep {user} /etc/passwd
  usermod -s /bin/bash {user}
  ```
  
- Grant as sudoer

  ```bash
  # make sure the user is trusted
  visudo /etc/sudoers.d/{user}
  # insert config below into sudoers file
  # {user} ALL=(ALL:ALL) NOPASSWD: ALL
  
  # keep env \w -E for sudo commands
  sudo -E curl -I https://www.google.com
  ```
  
- Add to group

  > It's needed while running docker. 

  ```bash
  # add user to group
  usermod -a -G {group} {user}
  
  # or change the primary group of user 
  usermod -g {group} {user}
  ```

## Network

- Set proxy enviroments

  ```bash
  # /etc/profile.d/proxy.sh
  export http_proxy="http://{{username}}:{{passwd}}@{{host}}:{{port}}"
  export HTTP_PROXY=${http_proxy}
  # https_proxy, no_proxy etc.
  ```
  
- Set customed domain name for localhost or other host

  ```bash
  echo "127.0.0.1 xxx.dev" >> /etc/hosts
  ```

## SSH

- Generate key

  ```bash
  ssh-keygen
  ```

- Trust keys of other hosts

  ```bash
  # get the public key of other trusted host
  echo "{{public_key}}" >> ~/.ssh/authorized_keys
  ```

- Config agent

  For cloning git repository by ssh keys on host while developing in dev container with vscode. 

  See official [how to config ssh-agent on linux](https://code.visualstudio.com/docs/remote/containers#_using-ssh-keys). 
  
  - Add start script to `.bashrc` to execute after logining
  
      ```bash
      # start script, which should be added into .bash_profile
      if [ -z "$SSH_AUTH_SOCK" ]; then
         # Check for a currently running instance of the agent
         RUNNING_AGENT="`ps -ax | grep 'ssh-agent -s' | grep -v grep | wc -l | tr -d '[:space:]'`"
         if [ "$RUNNING_AGENT" = "0" ]; then
              # Launch a new instance of the agent
              ssh-agent -s &> $HOME/.ssh/ssh-agent
         fi
         eval `cat $HOME/.ssh/ssh-agent`
      fi
      ```
      
  - Add ssh key to agent
    
    
      ```bash
      ssh-add $HOME/.ssh/id_rsa
      ```
      
  - Test
    
      ```bash
      ssh-add -l
      echo $SSH_AUTH_SOCK
      
      # there should be only one process
      ps -fC ssh-agent
      ```


## Prompt Shell

- Fonts

  Use [Nerd Fonts](https://www.nerdfonts.com/) **in terminal**, which contains many icons. Recommoned [Meslo LGM NF](https://github.com/ryanoasis/nerd-fonts/releases). 

- [Oh My Posh](https://ohmyposh.dev/)

  ```bash
  # install (https://ohmyposh.dev/docs/installation/linux)
  sudo wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O /usr/local/bin/oh-my-posh
  sudo chmod +x /usr/local/bin/oh-my-posh
  
  # get themes
  mkdir ~/.poshthemes
  wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/themes.zip -O ~/.poshthemes/themes.zip
  unzip ~/.poshthemes/themes.zip -d ~/.poshthemes
  chmod u+rw ~/.poshthemes/*.omp.*
  rm ~/.poshthemes/themes.zip
  
  # setting 
  echo 'eval "$(oh-my-posh init bash)"' >> ~/.bash_profile
  ```
