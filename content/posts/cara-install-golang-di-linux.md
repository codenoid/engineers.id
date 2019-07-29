---
title: "Cara Install Golang di Linux"
description: "Cara paling mudah install golang di linux"
date: 2019-07-30T00:13:29+07:00
author: codenoid
categories:
  - "setup"
tags:
  - "tutorial"
  - "install"
  - "golang"
  - "tutorial"
---

![Gopher](/posts/cara-install-golang-di-linux/main.jpg)

### My Device

1. Linux Ubuntu 14.04

### Customize this command and run !

```
cd ~
wget https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz
tar -xf go1.12.7.linux-amd64.tar.gz

export GOROOT="/home/$USER/go"
export GOPATH="/home/$USER/go/packages"
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
# puts export line to your ~/.bashrc or any bashprofile file
# please change $USER with your static username value
```

## THAT'S IT

By the way, checkout one of best Brian song

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/yeW1cCw3-Hc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>