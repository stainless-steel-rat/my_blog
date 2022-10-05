---
title: My first post
date: 2022-10-05 23:25:18
tags:
---


### INFO：  
        Hexo + nginx 部署到vps服务器，并利用git进行项目管理
        环境：client-windows  server-ubuntu18.04
        1.利用git hooks实现自动部署
        在用hexo d 命令进行部署的时候，配置git部署，在server上的git repo接受到最新的push时出发hooks中的脚本，自动实现git repo到VPS上网站目录的copy。

### 1.Client和Server都需要安装git

### 2.在server上创建git用户，并给予sudo权限
    执行命令：
    chmod 740 /etc/sudoers
    vim /etc/sudoers
    添加： git ALL=(ALL:ALL) ALL
    退出后，改回文件权限： chmod 440 /etc/sudoers

### 3.（可选） 关闭git用户的shell权限
	vim /etc/passwd
    最后一行 git:x:1001:1001:,,,:/home/git:/bin/bash 改为 git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
    这样就无法 ssh git@vps_ip 这样进行登录

### 4.初始化server上的git仓库
	cd /home/git
    mkdir blog.git
    cd blog.git
    git init --bare  (--bare 空仓库）

### 5.创建网站目录
	可以在默认的/var/www目录中添加blog目录

### 6.配置server上的ssh
    cd /home/git
    mkdir .ssh
    cd .ssh
    vim authorized_keys
    将client上的公钥（id_rsa.pub文件中的内容）复制到authorized_keys中

### 7.用户组修改
    上述的/home/git/blog.git  and .ssh  /var/www/blog 目录的用户组权限都需要改为git用户，否则git push时无法访问
    chown -R git.git /目录

### 8. nginx的安装和配置
    server上apt-get直接安装
    修改配置文件： vim /etc/nginx/sites-avaible/default
    设置网站目录： root /var/www/blog

    设置完成后重启nginx
    nginx -s reload
    or
    service nginx restart

### 9.设置git hooks
    cd /home/git/blog.git/hooks
    vim post-receive
    内容：
        #!/bin/bash
        echo "post-receive hook is running..."

        GIT_REPO=/home/git/blog.git
        TMP_GIT_CLONE=/tmp/blog
        PUBLIC_WWW=/var/www/blog

        rm -rf ${TMP_GIT_CLONE}
        git clone $GIT_REPO $TMP_GIT_CLONE
        rm -rf ${PUBLIC_WWW}/*
        cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}

    主要的作用就是复制repo中的内容到网站目录
    赋予post-receive可执行权限
    chmod +x post-receive

### 10.配置本地hexo
    打开博客根目录下的_config.yml文件
    修改：
        ## Deployment
        ## Docs: https://hexo.io/docs/deployment.html
        deploy:
            type: git
            repo: git@VPS IP:blog.git  # 默认端口
        branch: master

    安装hexo-deployer-git实现推送
    npm install hexo-deployer-git --save

    如果安装中出现报错：
        npm audit fix --force

### 11.执行hexo d 尝试部署
    如果报错： ! [remote rejected] HEAD -> master (unpacker error) error: failed to push some refs to '192.227.192.47:/~/blog.git'
    执行：
    git config user.name “name”
    git config user.email “zhoucw100@163.com”


