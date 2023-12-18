---
title: "SSH on Mac OS"
author: "Haobo Gu"
tags: [linux]
date: 2019-08-19T14:26:42.689638+08:00
---

<!--more-->

## Create SSH key

First, create ssh key:

```shell
ssh-keygen -t rsa -C "yourmail@mail.com"
```

Then add the created private key to ssh-agent, and store it in keychain:

```shell
ssh-add -K ~/.ssh/id_rsa
```

`-K` can make sure that the ssh key will always be valid after restart. You can use `ssh-add -l` to check added ssh keys

## Config

We can use different ssh keys to connect different servers.

The configuration file is `~/.ssh/config`

Here is an example config file:

```python
# Use id_rsa_github to connect to git@github.com
Host myhost
	HostName github.com
	User git
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_rsa_github
  
# Use default ssh key to connect to git@gitlab.com
Host myhost2
	HostName gitlab.com
  User git
  PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_rsa
  
# Use id_rsa_myserver to connect root@12.34.56.78
Host myhost3
	HostName 12.34.56.78
  User root
  PreferedAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_myserver
```

If the server is not in config file, the default ssh key is `~/.ssh/id_rsa`

## Github setting

Please refer to [Github Help](https://help.github.com/en/articles/adding-a-new-ssh-key-to-your-github-account)