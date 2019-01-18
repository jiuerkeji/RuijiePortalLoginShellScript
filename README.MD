# 简介

本脚本实现了锐捷 ePortal Web 认证的登录，需要 curl 指令的支持，通过 curl 发送认证所需的 POST 数据包来实现 Web 认证。

如果你所在的学校也是使用锐捷进行校园网认证，并且开启了 Web 的认证方式（如下图），那么大概率是可以使用本脚本进行认证的。

![ruijie_web](images/ruijie_web.png)

项目中的脚本文件以 **_jmu** 结尾命名的仅适用于集美大学，其他学校请使用 **_general** 结尾命名的脚本文件。

脚本依赖于 curl，所以请确保系统已经安装了 curl。

# 用法

## 一般用法

```shell
./ruijie_general.sh username password
```

- username参数为认证用户名。
- password参数为认证密码。

## 在路由器上使用

这里主要介绍一下如何借助 Openwrt / LEDE 里的 cron 定时任务实现路由器每天**定时**自动进行锐捷认证。

其实这也是我当初写这个脚本的主要目的，就是想着每天早上断网结束之后，路由器能自动帮我锐捷认证上，然后我睡醒了就能直接上网了。

以下均使用刷了 LEDE Reboot 17.01.4 的 Netgear WNDR3800 作为演示，我 Luci 的语言设置为了中文，使用英文的只要照着相对应的方法操作就行。

### 安装 curl

首先通过 ssh 连接到路由器。然后执行以下指令，安装 curl：

```bash
opkg update
opkg install curl
```

### 上传脚本到路由器

上传脚本，我这里直接：

```sh
vi /usr/bin/ruijie.sh
```

然后按 **I** 进入 **Insert** 模式，把代码粘贴进去。之后按 **ESC** 退出 Insert 模式，输入 **:wq** 并按下回车保存文件并退出 vi。
之后给脚本赋予执行权限：

```sh
chmod +x /usr/bin/ruijie.sh
```

这里也可以通过 SCP 把文件直接上传到相应位置，然后赋予执行权限。SCP 上传文件的方法这里不再详细说明，看哪种方法对你来说比较方便就用哪种，总之，只要能顺利把脚本文件上传到路由器中，并赋予正确的执行权限即可。

### 设置路由器时区

Openwrt / LEDE 默认的时区是 UTC，这里为了下面配置 crontab 中设置时间方便，以及让路由器的时间能跟北京时间同步。需要将时区改成 Asia/Shanghai。

打开并登陆到路由器后台管理页面，之后进入"系统" —— "系统"：
![Set timezone](images/set_timezone.png)
将其中的时区选项修改成 **Asia/Shanghai**，同时注意底下的 **启用 NTP 客户端** 是否开启以及 **候选 NTP 服务器** 是否设置正确。

### 配置 crontab

打开浏览器，登陆到路由器的后台管理页面（Luci）。之后选择 "系统" —— "计划任务"：
![Crontab](images/select_cron.png)

这里先说明一下 Openwrt / LEDE 的 crontab 格式：

```
 f1 f2 f3 f4 f5 program
 分 时 日 月 周 执行的命令
```

根据这个格式设置好 crontab 即可，以下是我自己所用的 crontab：

```
8 6 * * * /usr/bin/ruijie.sh 用户名 密码
```

我设定为每天早上 6:08 的时候进行锐捷认证，因为我所在的学校是每天晚上11点断网，隔天早上6点左右恢复，具体恢复时间似乎有一定的误差，所以延迟了8分钟。其实可以设得更晚一点的，毕竟我也不会那么早起床。你可以根据自己所在学校的情况设置执行脚本的时间。

![Crontab edit](images/crontab.png)

填写好了点击 "提交" 即可。
提交完了之后记得重启路由器，或者到 "系统" —— "启动项" 中重启 cron 服务：

![Restart cron](images/restart_cron.png)

# 局限性

在锐捷 ePortal Web 认证的 POST 数据包中发现了 validcode 参数，根据名字猜测这个参数应该是验证码。

我所在的学校并没有开启验证码，只有在登录尝试次数过多的时候才会出现验证码。如果你的学校在登录的时候需要验证码，那么本脚本将无法进行认证。

本脚本代码编写的很乱，工作效率不高。

# 吐槽

锐捷的 ePortal Web 认证上了之后，向 InterFace.do?method=getOnlineUserInfo 这个接口中 Post 过去 userIndex 参数会明文显示用户的密码，感觉一点安全性都没有。（集美大学在认证系统升级之后已修复该问题）

以及，我所在的学校在未认证的情况下，53端口(DNS)是可以正常通信的。如果你所在的学校也有类似的情况，也许可以尝试搭建个53端口的代理实现免认证上网。