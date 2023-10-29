---

title: PagerMaid-Telegram 人形自走 Bot 

date: '2022-07-23'

categories:

  - docker

tags:

  - Markdown

---

项目地址[GitHub](https://github.com/TeamPGM/PagerMaid-Pyro)，最开始的初衷是，TG的一些频道发布的信息非常具有时效性，我又非常需要这些信息，最开始的时候，

1. 我没有透明代理环境，导致我接收消息的时间有延迟

2. 点开一般然后进行信息处理太麻烦了，我需要自动化的处理信息进行基础的转发操作等

   

其他：使用TG经常有一个问题就是，烦人的机器人很多，手动拉黑很麻烦，删除消息很麻烦，还有出于安全性考虑自动删除群聊内消息等



# 安装前

**申请app_id与app_hash，节点推荐自建，干净ip通过率很高**

1. [登录](https://my.telegram.org/auth)，输入手机号之后，会在tg收到一个官方发送的验证码，不是SMS
2. 点击[API development tools](https://my.telegram.org/apps)
3. 按提示填写信息，生成 API，环境正常，账号正常稳过

**提前创建一个文件夹存储PagerMaid，并记住绝对路径**

**切忌不要设置两个机器人**

# 安装

> docker一键安装脚本，也可以手动进入容器安装，一键脚本非常完善且方便。

```bash
wget https://raw.githubusercontent.com/TeamPGM/PagerMaid-Pyro/development/utils/docker.sh -O docker.sh && chmod +x docker.sh && bash docker.sh
```

## 安装多个PagerMaid

> 务必修改docker.sh中对于端口的默认设置
>
> data_persistence () 语句中，-p 3333:3333改为-p 3334:3333
>
> ```
> docker run -dit -v "$data_path"/workdir:/pagermaid/workdir --restart=always --name="$container_name" --hostname="$container_name" -e WEB_ENABLE="$PGM_WEB" -e WEB_SECRET_KEY="$admin_password" -e WEB_HOST=0.0.0.0 -e WEB_PORT=3333 -p 3333:3333 teampgm/pagermaid_pyro <&1
> ```

没什么好说的，跟着sh脚本走就行，记得开通web端，持久化数据，方便迁移。

# 迁移

安装时路径下会生成一个workdir文件夹，里面包含所有数据，在旧机器拷贝之后，新机器先安装好pm，然后复制旧workdir进入新workdir之中

**重点**：务必进入容器重新进行登录

```
docker exec -it pagermaid /bin/bash  #进入容器内bash，记得改容器名
apt update -y && apt upgrade -y #国际惯例先更新，类似孙吧二楼打个🦶先
cd /pagermaid/workdir && pip3 install -r requirements.txt #进入pm目录，检查一下依赖
rm pagermaid.session #这是pm的登录信息，必需删除然后启动重新登录生成
python3 -m pagermaid #启动，然后按输出提示操作
```

![image-20231029162604886](http://easyimage.muzi.studio/i/2023/10/29/qw4ni4-0.png)

出现**设置用户标识成功**，代表登录成功，此时可以去 Telegram 任意聊天发送 `,status` 进行测试。



![image-20231029160109252](http://easyimage.muzi.studio/i/2023/10/29/qh9odj-0.png)

现在你就可以输入命令了，记住前面加上一个逗号`,`中文英文都可

发送 ",help <命令>" 以查看特定命令的帮助

# 常用命令

```
基础命令
，apt install 插件名 #安装插件
#插件安装也可以一次性多个，用空格隔开。
，apt install dme eat autochangename bc
，apt disabled 插件名 #禁用插件
，apt remove 插件名 #卸载插件
，update 插件名 #更新插件
，alias set 旧命令 新命令 
#重定向原插件命令，例如，alias set dme d，以后就只需要-d执行，而不是-dme。
，speedtest #测试机器速度
，sysinfo #查看机器信息
，status #查看机器运行状态
，restart #重启人形bot
```

# 必装插件

### 安装插件

**安装官方插件**

```
，apt install 插件名
```

如果不知道插件用法发送`，help 插件名`

### 安装自己的插件

```
在聊天框中对.py文件回复，apt install 即可自动安装
```

## shift

转发消息神中神，监听的必需法宝

```
使用方法: ,shift set [from channel] [to channel] 自动转发频道新消息（可以使用频道用户名或者 id）
del [from channel] 删除转发
backup [from channel] [to channel] 备份频道（可以使用频道用户名或者 id）
所需权限: plugins.shift
开启转发频道新消息功能
```

## pmcaptcha

此插件可以对私聊用户进行简易的算数验证，杜绝广告骚扰

## dme

```
删除xx条信息,已重定向为d
，d xx
```

## sendat 

```
定时发送消息。
,sendat 时间 | 消息内容
,sendat 16:00:00 date | 投票截止！
,sendat every 23:59:59 date | 又是无所事事的一天呢。
 #用于spy签到
,sendat every 1 minutes | 又过去了一分钟。
,sendat 3 times 1 minutes | 此消息将出现三次，间隔均为一分钟。
,sendat rm 2 - 删除某个任务
,sendat pause 1 - 暂停某个任务
,sendat resume 1 - 恢复某个任务
,sendat list  - 获取任务列表
```

## speed_test 

```
测速
，speedtest
```

## backup

备份

## recovery

还原

