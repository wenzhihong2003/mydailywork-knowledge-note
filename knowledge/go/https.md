安装 let's Encrypt提供的ssl证书
========================

https://ruby-china.org/topics/31983
https://github.com/Neilpang/acme.sh/wiki/Blogs-and-tutorials

### 安装acme.sh

```
curl https://get.acme.sh | sh
```

然后重新载入一下 .bashrc

```
source ~/.bashrc
```

现在你就有了 `acme.sh` 的命令.

### 申请签发 SSL 证书

acme.sh --issue -d www.your-app.com -w /home/gm3/acmesshtestdir  

```
/home/gm3/acmesshtestdir  这个是你的本地目录, 可以往里面写文件
```

> 上面这段过程将会往 /home/gm3/acmesshtestdir 创建一个 .well-known 的文件夹，同时 Let’ s Encrypt 将会通过你要注册的域名去访问那个文件来确定权限，它可能会去访问 http://www.your-app.com/.well-known/ 这个路径。 所以你需要确保 /home/ubuntu/www/your-app/current/public 是在 Nginx 上配置成 root 目录，里面任意文件可以直接域名访问的。

可通过在 你的nginx里配置加上 

```
location /.well-known {
  alias /home/gm3/acmesshtestdir/.well-known;
}
```

如果成功的话，你就会看到这样的结果:

```
[Fri Dec 23 11:20:15 CST 2016] Renew: 'www.your-app.com'
[Fri Dec 23 11:20:15 CST 2016] Single domain='www.your-app.com'
[Fri Dec 23 11:20:15 CST 2016] Getting domain auth token for each domain
[Fri Dec 23 11:20:15 CST 2016] Getting webroot for domain='www.your-app.com'
[Fri Dec 23 11:20:15 CST 2016] _w='/home/ubuntu/www/your-app/current/public/'
[Fri Dec 23 11:20:15 CST 2016] Getting new-authz for domain='www.your-app.com'
[Fri Dec 23 11:08:57 CST 2016] The new-authz request is ok.
[Fri Dec 23 11:08:57 CST 2016] Verifying:www.your-app.com
[Fri Dec 23 11:09:01 CST 2016] Success
[Fri Dec 23 11:09:01 CST 2016] Verify finished, start to sign.
[Fri Dec 23 11:09:02 CST 2016] Cert success.
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
[Fri Dec 23 11:09:02 CST 2016] Your cert is in  /home/ubuntu/.acme.sh/www.your-app.com/www.your-app.com.cer 
[Fri Dec 23 11:09:02 CST 2016] Your cert key is in  /home/ubuntu/.acme.sh/www.your-app.com/www.your-app.com.key 
[Fri Dec 23 11:09:04 CST 2016] The intermediate CA cert is in  /home/ubuntu/.acme.sh/www.your-app.com/ca.cer 
[Fri Dec 23 11:09:04 CST 2016] And the full chain certs is there:  /home/ubuntu/.acme.sh/www.your-app.com/fullchain.cer
```
 
你的证书文件已经申请成功了，并放到了 ~/.acme.sh/ 目录里面。

> TIP: 所有的 acme.sh 配置都记录在 ~/.acme.sh/ 目录里面，acme.sh 有自动的配置读取，并按域名划分，下次你再次执行的时候，它知道你之前是用的那个目录，只需要告诉它域名就好了。


###  将 SSL 证书安装到网站的路径，并配置好 restart Nginx 的动作

> NOTE: 这个比较重要，因为它会让 acme.sh 记住重启 Nginx 的命令，以后自动更新证书的动作需要重启 Nginx

```
acme.sh --installcert -d www.your-app.com \
               --keypath       /home/ubuntu/www/ssl/www.your-app.com.key  \
               --fullchainpath /home/ubuntu/www/ssl/www.your-app.com.key.pem \
               --reloadcmd     "sudo service nginx force-reload"
```              
然后你会看到结果

```
[Fri Dec 23 11:59:57 CST 2016] Installing key to:/home/ubuntu/www/ssl/www.your-app.com.key
[Fri Dec 23 11:59:57 CST 2016] Installing full chain to:/home/ubuntu/www/ssl/www.your-app.com.key.pem
[Fri Dec 23 11:59:57 CST 2016] Run Le_ReloadCmd: sudo service nginx force-reload
Restarting nginx: nginx.
[Fri Dec 23 11:59:58 CST 2016] Reload success
```

修改一下 sudoer 文件，让 sudo service nginx force-reload 不需要输入密码
```
sudo visudo
```
打开文件以后新增:

> NOTE: ubuntu 是 acme.sh 安装所用的账号

```
ubuntu  ALL=(ALL) NOPASSWD: /usr/sbin/service nginx force-reload
```

生成 dhparam.pem 文件
```
openssl dhparam -out /home/ubuntu/www/ssl/dhparam.pem 2048
```

修改 Nginx 启用 SSL
```
http {
  server {
    listen 80 default_server;
    # 新增
    listen 443 ssl;
    ssl_certificate         /home/ubuntu/www/ssl/www.your-app.com.key.pem;
    ssl_certificate_key     /home/ubuntu/www/ssl/www.your-app.com.key;
    # ssl_dhparam 
    ssl_dhparam             /home/ubuntu/www/ssl/dhparam.pem;

    # 其他省略
  }
}
```

检查 Nginx 配置是否正确后重启

```
sudo service nginx configtest
sudo service nginx restart
```

### 后续维护
Let's Encrypt 的证书有效期是 90 天的，你需要定期 renew 重新申请，这部分 acme.sh 以及帮你做了，在安装的时候往 crontab 增加了一行每天执行的命令 acme.sh --cron:

```
$ crontab -l
0 0 * * * "/home/ubuntu/.acme.sh"/acme.sh --cron --home "/home/ubuntu/.acme.sh" > /dev/null
```

可以把这个定时任务写成 每个月的12日执行

```
0 0 12 * * "/home/ubuntu/.acme.sh"/acme.sh --cron --home "/home/ubuntu/.acme.sh" > /dev/null
```

这样就是正常的:
```
[Fri Dec 23 11:50:30 CST 2016] Renew: 'www.your-app.com'
[Fri Dec 23 11:50:30 CST 2016] Skip, Next renewal time is: Tue Feb 21 03:20:54 UTC 2017
[Fri Dec 23 11:50:30 CST 2016] Add '--force' to force to renew.
[Fri Dec 23 11:50:30 CST 2016] Skipped www.your-app.com
```
`acme.sh --cron` 命令执行以后将会 申请新的证书 并放到相同的文件路径。由于前面执行 --installcert 的时候告知了重新 Nginx 的方法，acme.sh 也同时会在证书更新以后重启 Nginx。

最后走一下 acme.sh --cron 的流程看看能否正确执行
```
acme.sh --cron -f
```
这个过程应该会得到这样的结果，并在最后重启 Nginx (不需要输入密码)

```
[Tue Dec 27 14:28:09 CST 2016] Renew: 'www.your-app.com'
[Tue Dec 27 14:28:09 CST 2016] Single domain='www.your-app.com'
[Tue Dec 27 14:28:09 CST 2016] Getting domain auth token for each domain
[Tue Dec 27 14:28:09 CST 2016] Getting webroot for domain='www.your-app.com'
[Tue Dec 27 14:28:09 CST 2016] _w='/home/ubuntu/www/your-app/current/public/'
[Tue Dec 27 14:28:09 CST 2016] Getting new-authz for domain='www.your-app.com'
[Tue Dec 27 14:28:16 CST 2016] The new-authz request is ok.
[Tue Dec 27 14:28:16 CST 2016] www.your-app.com is already verified, skip.
[Tue Dec 27 14:28:16 CST 2016] www.your-app.com is already verified, skip http-01.
[Tue Dec 27 14:28:16 CST 2016] www.your-app.com is already verified, skip http-01.
[Tue Dec 27 14:28:16 CST 2016] Verify finished, start to sign.
[Tue Dec 27 14:28:19 CST 2016] Cert success.
... 省略
[Fri Dec 23 11:09:02 CST 2016] Your cert is in  /home/ubuntu/.acme.sh/www.your-app.com/www.your-app.com.cer 
[Fri Dec 23 11:09:02 CST 2016] Your cert key is in  /home/ubuntu/.acme.sh/www.your-app.com/www.your-app.com.key 
[Fri Dec 23 11:09:04 CST 2016] The intermediate CA cert is in  /home/ubuntu/.acme.sh/www.your-app.com/ca.cer 
[Fri Dec 23 11:09:04 CST 2016] And the full chain certs is there:  /home/ubuntu/.acme.sh/www.your-app.com/fullchain.cer 
[Tue Dec 27 14:28:22 CST 2016] Run Le_ReloadCmd: sudo service nginx force-reload
 * Reloading nginx nginx                                                                                                                                     [ OK ] 
[Tue Dec 27 14:28:22 CST 2016] Reload success
```
