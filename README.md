1、安装Docker和Docker Compose

#安装Docker
```
curl -sSL https://get.docker.com/ | sh
```
#安装Docker Compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

2、安装FileRun镜像

```
docker pull afian/filerun
```

3、配置yml文件

建立docker-compose.yml文件，并使用默认的配置即可,docker会运行在1234端口

```
#以下是一整个命令，一起复制运行即可。
echo "version: '2'

services:
  db:
    image: mariadb:10.1
    environment:
      MYSQL_ROOT_PASSWORD: filerun
      MYSQL_USER: filerun
      MYSQL_PASSWORD: filerun
      MYSQL_DATABASE: filerun
    volumes:
      - /filerun/db:/var/lib/mysql

  web:
    depends_on:
      - db
    links:
      - db
    image: afian/filerun
    ports:
      - "1234:80"
    volumes:
      - /filerun/html:/var/www/html
      - /filerun/user-files:/user-files" > /root/docker-compose.yml
```

4、启动FileRun

```
docker-compose up -d
```
这时候就可以通过http://IP访问了，用户名和密码都是superuser

5、使用caddy反代开启https访问

安装caddy
```
wget --no-check-certificate -O caddy_install.sh https://raw.githubusercontent.com/xyh101/doubi/master/caddy_install.sh
chmod +x caddy_install.sh
./caddy_install.sh
```
```
启动：/etc/init.d/caddy start
停止：/etc/init.d/caddy stop
重启：/etc/init.d/caddy restart
查看状态：/etc/init.d/caddy status
查看Caddy启动日志：tail -f /tmp/caddy.log
安装目录：/usr/local/caddy
Caddy配置文件位置：/usr/local/caddy/Caddyfile
Caddy自动申请SSL证书位置：/.caddy/acme/acme-v01.api.letsencrypt.org/sites/xxx.xxx(域名)/
```
      
卸载Caddy

卸载不会删除虚拟主机的内容，只会删除Caddy自身和配置文件。
```
wget -N --no-check-certificate https://raw.githubusercontent.com/xyh101/doubi/master/caddy_install.sh && bash caddy_install.sh uninstall
```
caddyfile的写法
```
#https访问，该配置会自动签发SSL，请提前解析域名到VPS服务器
echo "wdwp.tk {
 gzip
 tls gcphym1@gmail.com
 proxy / 127.0.0.1:1234 {
    header_upstream Host {host}
    header_upstream X-Real-IP {remote}
    header_upstream X-Forwarded-For {remote}
    header_upstream X-Forwarded-Port {server_port}
    header_upstream X-Forwarded-Proto {scheme}
  }
}" > /usr/local/caddy/Caddyfile
```


      
