---
title: Flask项目部署：Nginx + Gunicorn
categories: Python
tags:
  - Python
  - Flask
  - Backend
date: 2019-01-27 16:45:05
---

最近在做自己的个人项目诗词书斋，用到了服务器，使用的是自己熟悉的 Flask 框架编写后端接口，写完之后需要部署到阿里云服务器上，本文是我对整个部署流程的一个总结。

### 总体方案

总体方案是 Nginx + Gunicorn，Nginx 负责代理转发，Gunicorn 是一个 WSGI 容器，负责启动应用。我使用的是阿里云 Ubuntu 系统。

### 具体步骤

1. 安装 Nginx

```shell
# 更新源
sudo apt-get update
# 安装nginx
sudo apt-get install nginx
# 启动nginx
sudo /etc/init.d/nginx start
```

启动之后，在浏览器上输入服务器的外网 IP 就可以看到 Nginx 的界面

2. 配置 Nginx

```shell
# 删除Nginx默认配置文件
sudo rm /etc/nginx/sites-enabled/default
# 创建自己的配置文件
sudo touch /etc/nginx/sites-available/flask_setting
# 创建连接
sudo ln /etc/nginx/sites-available/flask_setting /etc/nginx/sites-enabled/flask_setting
# 编辑配置文件
vim /etc/nginx/sites-enabled/flask_setting
```

配置文件内容：

```
server {
        location / {
                proxy_pass http://127.0.0.1:8000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
        }
}
```

解释：127.0.0.1:8000 是 flask 应用运行的本地地址，gunicorn 运行应用默认使用 8000 端口。

3. 安装 pipenv

pipenv 是 Python 的虚拟运行环境。

```shell
curl https://raw.githubusercontent.com/kennethreitz/pipenv/master/get-pipenv.py | python
```

4. 安装 Gunicorn

进入 flask 项目目录，运行虚拟环境

```shell
pipenv shell
pipenv install # 安装flask等项目需要用到的包
pipenv install gunicorn # 安装gunicorn
```

5. 配置 Gunicorn

在项目目录下创建`config.py`，填入内容

```python
accesslog = "/root/log/access.log"  # 访问日志文件
errorlog = "/root/log/error.log"  # 错误日志文件
loglevel = 'info'
daemon = True # 作为守护进程运行
```

6. 运行

```shell
gunicorn -c config.py release:app
```

release 是项目入口文件包名，app 为入口文件中的应用名，即 Flask 对象的名称。

### 遇到的问题

上面就是整个部署流程，这过程中遇到的问题有如下几个：

1. Ubuntu 默认的 Python 版本比较低，如果需要高版本的 Python，需要先安装 Python，再创建虚拟环境。

```python
# 使用Python3.7创建虚拟环境
pipenv --python 3.7
```

2. 代码使用 Git 部署，之前使用的 GitHub，后来发现阿里云竟然屏蔽了 GitHub😒，代码拉不下啦，没办法，给项目添加了一个 Coding.net 的远程仓库，这样才可以把代码拉下来。

### 总结

以上就是我部署 Flask 项目的整个流程，其实 Python 项目的部署方案有很多种，这只是其中的一种方案。
