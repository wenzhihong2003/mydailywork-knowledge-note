ubuntu 16.04 service 基础要点
=========================

[来源](https://my.oschina.net/janpoem/blog/802708)  

16.04转用了systemd来对系统服务提供管理和控制（貌似15.04就已经转用了）。

#### 添加一个服务（service）

添加一个服务，需要创建一个服务的定义文件放在 /lib/systemd/system 目录下，这里以 nginx.service 为例：

```shell
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

创建了这个文件后，还无法直接使用。

```shell
service nginx start
systemctl start nginx
systemctl start nginx.service
```

执行上述的命令后，会提示以下的错误：

```
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
```

这时候你需要重载systemd的配置列表：

```shell
systemctl daemon-reload
```

你可以执行以下命令，来确认你的服务是否已经加入到配置列表：

```shell
# 这个是列举所有已经存在配置文件对应的服务状态列表
systemctl list-unit-files | grep nginx
# 列举出具有加载状态的服务列表（或者理解为最近被使用的服务）
systemctl --all | grep nginx
```

#### 服务操作

如果习惯了使用 service 指令，可以继续使用：

```shell
service nginx start
service nginx stop
service nginx reload
service nginx restart
service nginx status
```

和过去不同，现在不再需要在 `/etc/init.d` 目录下添加一个服务脚本了。

但实际上Ubuntu的Wiki上推荐改用：

```shell
systemctl start nginx
systemctl start nginx.service
systemctl stop nginx
systemctl reload nginx
systemctl restart nginx
systemctl status nginx
```

不过 systemctl (包括 service ) 执行以后，是没有任何特别提示，除非碰到了错误信息，否则都是静默的，也许这是为了配合 bash 脚本的设计需求。

#### 激活/禁用系统自启动服务

再次，要将一个服务激活为系统启动时的自启动服务，现在只要执行以下命令：

```shell
systemctl enable nginx.service
systemctl enable nginx
```

这样就正式激活了服务系统启动时的自启动。要禁用自启动，只要disable即可。

```shell
systemctl disable nginx.service
systemctl disable nginx
```

你可以执行以下的命令，来检查服务是否已经激活了自启动

```shell
systemctl is-enabled nginx
# enabled/disabled
```

检查一个服务是否启动：

```shell
systemctl is-active nginx
# active/inactive
```

#### 关于 /etc/init 目录

这个目录，其实未必真的需要添加进一个相关的控制进程启动的配置文件，这里添加的文件，和具体的服务启动没有具体的关联性，新版本的 service 配置文件，实际上已经明确了启动服务所需的必要服务和之后加载的服务。

`/etc/init` 目录下存放的，可以理解为一个综合性启动的脚本配置，他支持在配置文件中，使用bash代码块，比如：

```shell

# nginx

description "nginx http daemon"
author "George Shammas <georgyo@gmail.com>"

start on (filesystem and net-device-up IFACE!=lo)
stop on runlevel [!2345]

env DAEMON=/usr/sbin/nginx
env PID=/var/run/nginx.pid

expect fork
respawn
respawn limit 10 5
#oom never

pre-start script
        $DAEMON -t
        if [ $? -ne 0 ]
                then exit $?
        fi
end script

exec $DAEMON
```

可以理解为，对过去的启动脚本更简化版的一个启动配置文件，使用这个配置文件控制启动，还是使用 systemd 自行控制服务作为自启动，这个交给使用者去权衡吧。
