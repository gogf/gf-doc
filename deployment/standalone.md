[TOC]

# 独立部署

使用`GF`开发的应用程序可以独立地部署到服务器上，设置为后台守护进程运行即可。这种模式常用在简单的API服务项目中。

服务器我们推荐使用`*nix`服务器系列(包括:`Linux`, `MacOS`, `*BSD`)，以下使用`Ubuntu`系统为例，介绍如何部署使用`GF`框架开发的项目。

## 1. `nohup`
我们可以使用简单的`nohup`命令来运行应用程序，使其作为后台守护进程运行，即使远程连接的SSH断开也不会影响程序的执行。在流行的Linux发行版中往往都默认安装好了`nohup`命令工具。
命令如下：
```
nohup ./gf-app &
```

## 2. `tmux`
`tmux`是一款Linux下的终端复用工具，可以开启不同的终端窗口来将应用程序作为后台守护进程执行，即使远程连接的SSH断开也不会影响程序的执行。
在`ubuntu`系统下直接使用`sudo apt-get install tmux`安装即可。使用以下步骤将应用程序后台运行。
1. `tmux new -s gf-app`；
1. 在新终端窗口中执行`./gf-app`即可；
1. 使用`ctrl ` + `B & D`快捷键可以退出当前终端窗口；
1. 使用`tmux attach -t gf-app`可进入到之前的终端窗口；

## 3. `supervisor`
`supervisor`是用`Python`开发的一套通用的进程管理程序，能将一个普通的命令行进程变为后台`daemon`，并监控进程状态，异常退出时能自动重启。官方网站：http://supervisord.org/
常见配置如下：
```
[program:gf-app]
user=root
command=/var/www/main
stdout_logfile=/var/log/gf-app-stdout.log
stderr_logfile=/var/log/gf-app-stderr.log
autostart=true
autorestart=true
```
使用步骤如下：
1. 使用`sudo service supervisor start`启动`supervisor`服务；
1. 创建应用配置文件`/etc/supervisor/conf.d/gf-app.conf`, 内容如上;
1. 使用`sudo supervisorctl`进入`supervisor`管理终端；
1. 使用`reload`重新读取配置文件并重启当前`supoervisor`管理的所有进程；
1. 也可以使用`update`重新加载配置(默认不重启)，随后使用`start gf-app`启动指定的应用程序；
1. 随后可以使用`status`指令查看当前`supervisor`管理的进程状态；

## 4. `systemctl`
`Systemd` 是 `Linux` 系统工具，用来启动守护进程，已成为大多数发行版的标准配置。   
而 `systemctl` 是 `Systemd` 的主命令，用于管理系统。可以参考[阮一峰对于 `Systemd` 的解读](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)，文章的第四、五章节：Unit   
其实我们大部分服务都有使用 `systemctl` 管理，比如 `MySQL、Nginx` 等等。    
常见配置如下：   
```
[Unit]
# 单元描述
Description=GF APP
# 在什么服务启动之后再执行本程序
After=mysql.service

[Service]
Type=simple
# 程序执行的目录
WorkingDirectory=/data/server/gfapp/
# 启动的脚本命令
ExecStart=/data/server/gfapp/gfapp
# 重启条件
Restart=alway
# 几秒后重启
RestartSec=5

[Install]
WantedBy=multi-user.target
```
**使用方法**
1. 创建应用配置文件 `/etc/systemd/system/gfapp.service`, 内容如上;
1. 使用 `systemctl daemon-reload` 重新加载服务;
1. 执行 `systemctl start gfapp` 来启动服务;
1. 最后执行 `systemctl status gfapp` 来查看服务运行的状态信息;
1. 执行 `systemctl enable gfapp` 将服务添加到开机启动项;
- 注意：执行的 `gfapp` 是使用文件名作为服务名;
- 常见的命令有: `start(启动), stop(停止), restart(重启), status(查看运行状态), enable(添加到开机启动项), disable(将程序从开机启动中移除)`

## 5. `screen`
`Screen` 是一款由 `GNU` 计划开发的用于命令行终端切换的自由软件。用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。`GNU Screen`可以看作是窗口管理器的命令行界面版本。它提供了统一的管理多个会话的界面和相应的功能。   
安装方式: `sudo apt install -y screen`(debian 系列), `sudo yum install -y screen` (centos)   

**常用参数:**
- `screen -S yourname` -> 新建一个叫 yourname 的 session
- `screen -ls`         -> 列出当前所有的 session
- `screen -r yourname` -> 回到 yourname 这个 session
- `screen -d yourname` -> 远程detach某个 session
- `screen -d -r yourname` -> 结束当前 session 并回到 yourname 这个 session   

**使用方法:**
1. 使用命令 `screen -S gfapp` 创建一个 session;
1. 在新终端窗口中执行 `./gf-app` 即可；
1. 执行 `ctrl-a, ctrl-d` 暂时离开当前session;
1. 执行 `screen -r gfapp` 返回命令窗口; 若返回不成功, 可能是该窗口被占用(`Attached`)了, 可以尝试使用 `screen -Dr gfapp`;
1. 执行 `screen -X -S gfapp quit` 结束程序;
