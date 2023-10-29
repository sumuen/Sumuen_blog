---
title: 曲线救国使用GPT4
author: Sumuen
date: '2023-04-26'
categories:
  - xlog
tags:
  - Markdown
---

> 完全解决something went wrong 这个恶心人的报错，我尊贵的GPT4用户怎么能受这委屈！

# 会用到

**docker**

**[pandora](https://github.com/pengzhile/pandora)**

**nginx proxy manager**

**[pake](https://github.com/tw93/Pake/blob/master/README_CN.md)**

# 需要

一台可以访问gpt的服务器，我用的日本，warp ipv6解锁的GPT（可选，也可使用在线部署平台vercel）

域名解析到服务器上

openai账号

了解linux命令行基础操作

# 开始操作

## 服务器warp ipv6解锁gpt

参考的[勇哥ykk的脚本](https://github.com/yonggekkk/warp-yg)

选择1安装warp-go

然后选择2. 安装/切换WARP单栈IPV6

![image-20230426011250748](https://easyimage.muzi.studio/i/2023/04/26/1uzbgy-1.png)

## 服务器安装docker与docker compose

> 是个服务器应该都有趴！我的linux系统是Ubuntu，以下为Ubuntu系统演示，不同的linux版本命令会有所不同自行谷歌

### 卸载旧版本

```
sudo apt-get remove docker \
               docker-engine \
               docker.io
```

### 使用 APT 安装

```
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### 安装docker

```
sudo apt-get install docker.io
```

### 安装docker compose

```
curl -L "https://github.com/docker/compose/releases/download/v2.16.0/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```



## 安装Nginx proxy manager

### 新建一个npm文件夹来存储数据

```
cd /root
mkdir npm
cd npm
```

### 编辑docker-compose.yml文件

```
vim docker-compose.yml
```

```
version: "3"
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP
    environment:
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "npm"
      DB_MYSQL_PASSWORD: "npm"
      DB_MYSQL_NAME: "npm"
      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db

  db:
    image: 'jc21/mariadb-aria:latest'
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'npm'
      MYSQL_DATABASE: 'npm'
      MYSQL_USER: 'npm'
      MYSQL_PASSWORD: 'npm'
    volumes:
      - ./data/mysql:/var/lib/mysql
```

### 运行容器

```
docker-compose up -d 
```

### 进入npm web面板

> 记得开启服务器硬件防火墙（服务器商界面的firewall）与服务器本身的软件防火墙，Ubuntu为ufw

npm网页地址即你的服务器ip:81

一开始会让你设置用户名和密码之类的

默认用户名为 admin@example.com

默认密码 changeme

更多npm细节可以看这篇文章，我的linux基础很多都是实战他的教程学会的

[教程](https://blog.laoda.de/archives/nginxproxymanager#nginx-proxy-manager)

## 安装pandora

> 这里我们演示的是，把Accesstoken写死在环境变量里面的方式，我很懒不想访问一次就输入一次，所以直接写死，后面会用npm的access list实现不被滥用

如何获取accesstoken

> 请在ChatGPT官网登陆完成后，打开F12查看https://chat.openai.com/api/auth/session请求返回的accessToken

![image-20230426010400758](https://easyimage.muzi.studio/i/2023/04/26/1pxa00-1.png)

```
docker run \
    -itd \
    --name pandora2 \
    --restart always \
		-p 7862:7861 \
    -e PANDORA_ACCESS_TOKEN=你的acesstoken  \
    -e PANDORA_SERVER='0.0.0.0:7861' \
    pengzhile/pandora
```

确认防火墙开启响应端口（7861）后，先用ip:端口看看是否正常运行

如果不正常的话！docker logs pandora 查看一下报错信息然后问chatgpt，一般来说不会有问题！

## 配置反向代理

现在npm和pandora已经正常运行了，现在要做的就是实现域名访问，首先确保你的域名已经解析到了你的ip

我是dnspod托管的，可以看到我这边已经解析好了

![image-20230426011842148](https://easyimage.muzi.studio/i/2023/04/26/1yi0yy-1.png)

然后设置npm

![image-20230426011642437](https://easyimage.muzi.studio/i/2023/04/26/1xb9hi-1.png)

domain names 就是你要设置的域名名称，我建议二级域名，如果你就一个项目，那就直接用一级域名就行了。

然后这个forward hostname/ip **不要无脑填127.0.0.1!**

因为npm是在docker容器里通过bridge桥接到服务器的，穿透了服务器本机的端口，所以这里要填你的docker的网关，来实现你的数据始终在你的服务器中传输，（我这里说的不明白，我尽力了，docker的六种网络模式我学过都忘了，还有我是文科生

### 获取docker网关

```
ip route
```

![image-20230426012340965](https://easyimage.muzi.studio/i/2023/04/26/21h0sc-1.png)

然后forward port端口 我用的7862 因为我布置了两个，一个gpt普通号给朋友一起用，占用了7861端口，我又开了7862端口给我的20美刀一个月的尊贵的gpt4订阅用户用。

![image-20230426012551362](https://easyimage.muzi.studio/i/2023/04/26/22pxnx-1.png)

然后ssl证书这里选获取新证书，或者一部到位获取泛域名证书，自行研究

![image-20230426012636336](https://easyimage.muzi.studio/i/2023/04/26/2388az-1.png)

### 配置访问限制

这是很重要的，关乎到你自己开小灶还是完全暴露到公网

![image-20230426012738264](https://easyimage.muzi.studio/i/2023/04/26/23u31t-1.png)

```
allow 这里改成你的ip地址或者你分流的访问这个gpt的特定ip;
deny all;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

```

因为我在宿舍，无论怎么拨号都会给一个确定的号段的ip，所以我就设置了仅限这个号段的ip区间，当然你也可以通过npm的accesss lists去限制访问的ip，设置密码等等，我试了，我只能说，我玩不来，我还是吃nginx的老本直接写在配置里。

当我在宿舍时，很好的就能访问。

当我在外面时，我通过连接内网穿透我宿舍软路由的vmess server端口进行利用宿舍的内网环境进行访问，我是个懒狗，这样穿透内网端口很危险，被爆破的话所有内网设备就都暴露了。

你也可以设置一个单独的代理服务器进行访问，或者设置密码。

所有都设置好之后，手机流量访问一下确认一下ip限制是否成功了，

## 通过pake打包镜像网页

### 环境配置Windows/Linux 注意点

- **十分重要** 查看 Tauri 提供的[依赖指南](https://tauri.app/v1/guides/getting-started/prerequisites)

- 把依赖指南的东西全部安装然后重启

  ### 安装

  ```
  npm install -g pake-cli
  ```

  ### 打包你的gpt网页

  ```
  pake url
  ```

  然后他会指引你起名字的，或者diy图标，pake的教程文档真的非常非常清楚，很nice！

  不多赘述。

## 大功告成！

![image-20230426013801378](https://easyimage.muzi.studio/i/2023/04/26/2a5wf6-1.png)

![image-20230426013830412](https://easyimage.muzi.studio/i/2023/04/26/2ac675-1.png)

两个字，优雅！

任何不懂的欢迎在[Twitter](https://twitter.com/muzi93040929)私信我，喂饭教程第一次写，肯定有很多漏洞请见谅，但这真的是我实践得出的成果，感谢你的阅读。