---
title: 'SSH免密码部署:git-bash配置'
cover: '5.jpg'
date: 2016-03-07 19:10:29
category: "git"
tags:
    - git
    - SSH
---

在使用`hexo d`部署这个博客的时候，经常会遇到拒绝public key的问题，要求确认访问repository的权限或者确定repository存在。
而实际上按照github的说明我已经生成并添加了ssh key，并且使用
```
ssh -T git@github.com
```
测试成功。
但是由于ssh-agent是一个临时进程，它存储公钥的方式也是写入一个临时的session，因此每次重启git-bash都会重置连接，于是每次都需要
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
ssh-add -l
```
走一遍才能正确部署。  
<!--more-->
github提供了一个将开启ssh-agent、添加公钥与启动bash捆绑的脚本。这样在每次开启git-bash的时候，公钥就已经添加，因此可以直接部署，而不必重复添加操作。
```shell
# Note: ~/.ssh/environment should not be used, as it
#       already has a different purpose in SSH.

env=~/.ssh/agent.env

# Note: Don't bother checking SSH_AGENT_PID. It's not used
#       by SSH itself, and it might even be incorrect
#       (for example, when using agent-forwarding over SSH).

agent_is_running() {
    if [ "$SSH_AUTH_SOCK" ]; then
        # ssh-add returns:
        #   0 = agent running, has keys
        #   1 = agent running, no keys
        #   2 = agent not running
        ssh-add -l >/dev/null 2>&1 || [ $? -eq 1 ]
    else
        false
    fi
}

agent_has_keys() {
    ssh-add -l >/dev/null 2>&1
}

agent_load_env() {
    . "$env" >/dev/null
}

agent_start() {
    (umask 077; ssh-agent >"$env")
    . "$env" >/dev/null
}

if ! agent_is_running; then
    agent_load_env
fi

# if your keys are not stored in ~/.ssh/id_rsa or ~/.ssh/id_dsa, you'll need
# to paste the proper path after ssh-add
if ! agent_is_running; then
    ssh-add
    agent_start
    #指定你的多个keys，每个一行。如：ssh-add ~/.ssh/id_rsa_git
elif ! agent_has_keys; then
    ssh-add
    #指定你的多个keys，每个一行。如：ssh-add ~/.ssh/id_rsa_git
fi

unset env
```
将该脚本添加到`.profile`中即可。
参考：[Working with SSH key passphrases](https://help.github.com/articles/working-with-ssh-key-passphrases/)
