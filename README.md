# Nginx-反代-TLS-证书申请

感谢金主大佬手把手

Nginx反代/TLS配置教程总结脚本来源于https://cxjiang.top/2019/08/02/v2ray-basic/

基于CentOS 7系统
-----------------------------
服务器更新
-------------------------
第1步：
`yum update`

第2步:
`yum upgrade`

如遇以下选项所有选择
`y`

![image](https://user-images.githubusercontent.com/94978556/144738610-11eee4a1-ee10-41e5-beeb-cbd1cba0679d.png)


-------------------------------
第3步可以检查一下upgrade是否安装：
`yum upgrade -y`也可以忽略这步  如下安装ok

![image](https://user-images.githubusercontent.com/94978556/144737966-7863922c-1ba2-46ff-9eff-290b279294ae.png)


安装TLS证书申请工具以下：
------------------------------
第4步：
`curl  https://get.acme.sh | sh`
如果提示openssl未安装如下：

![image](https://user-images.githubusercontent.com/94978556/144737872-e89c3a86-af81-45f3-b377-2a557ccb4e4e.png)

安装openssl：
`yum install openssl`

再执行一次
`curl  https://get.acme.sh | sh`

------------------------------------------------------

第5步acme脚本临时指定命令方便操作：
`alias acme.sh=~/.acme.sh/acme.sh`

--------------------------------------------------------
安装完acme证书申请工具后即可进行域名证书的申请
----------------------------------------------------
第6步：
`acme.sh --register-account -m xxxxxx@gmail.com`

第7步安装socat：
`yum install socat`

第8步申请证书：
`acme.sh --issue -d 换成你的域名 --standalone -k ec-256 --debug`成功如下：

![image](https://user-images.githubusercontent.com/94978556/144738517-247399a0-55ae-4935-8c6e-27f30dccee6a.png)


centos的默认包里没有nginx所以安一个软件源（有的忽略）
---------------------------------------------------
第9步：`yum install epel-release`

----------------------------------------------------
安装nginx
----------------------------------
第10步：`yum install nginx`

第11步生成dhparam：
`mkdir -p /etc/nginx/ssl`

第12步：
`openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048`

第13步先删除nginx原本配置文件：
`rm /etc/nginx/nginx.conf`
出现以下

![image](https://user-images.githubusercontent.com/94978556/144738765-8ec5eba4-1763-46ad-aee7-61e2a2e24b25.png)

强制删除nginx原本配置文件：
`rm -f /etc/nginx/nginx.conf`

-------------------------------------------------------------------------------------------------
配置nginx文件
-----------------------------------
第14步配置nginx：
`vim /etc/nginx/nginx.conf`

可访问：https://cxjiang.top/2019/08/02/v2ray-basic/  复制以下全部代码全部粘贴进去

```
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
#   tcp_nopush          on;
#   tcp_nodelay         on;
    keepalive_timeout   65;
#   types_hash_max_size 2048;

   	include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
}
```

![image](https://user-images.githubusercontent.com/94978556/146638192-eaf9fab4-b4d6-48ca-9f01-16d6008b31a3.png)

--------------------------------------------------------------------------
进入nginx的站点配置文件夹，并新建我们对应域名的配置文件
------------------------------------------------------------
第15步：
`cd /etc/nginx/conf.d`

第16步：
`vim 换成你的域名.conf`进入编辑配置文件

第17步访问https://cxjiang.top/2019/08/02/v2ray-basic/   粘贴以下内容复制进去

```
server
{
    listen 80;
    server_name 你的域名;

    #将http重定向到https
    return 301 https://你的域名$request_uri;
}

server
{
    listen 443 ssl http2;
    server_name 你的域名;
    ssl on;
    ssl_certificate /root/.acme.sh/你的域名_ecc/fullchain.cer; 
    ssl_certificate_key /root/.acme.sh/你的域名_ecc/你的域名.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5";
    ssl_session_cache builtin:1000 shared:SSL:10m;
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
    access_log off;
    
    location / {
        
        #向后端传递访客IP
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        #设定需要反代的域名，可以加端口号
        proxy_pass https://www.iqiyi.com/;
        #替换网站内容
        sub_filter 'www.iqiyi.com' '你的域名';

        # websocket设定，V2ray使用，这里的设置要和v2ray的设置一致。
        location /自定义/ {
            proxy_redirect off;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $http_host;
            proxy_intercept_errors on;
            if ($http_upgrade = "websocket" ){
                    proxy_pass http://127.0.0.1:自定义v2ray端口;
            }
        }
    }
}
```

![image](https://user-images.githubusercontent.com/94978556/144739173-82862e9f-00e9-4c02-a97b-2850a1939f69.png)

中文不需要管它  只需要删除以下内容改成你的

画蓝色直线的换成你的域名 

画蓝色椭圆修改你要反代的网址

画蓝线三角换成v2ray  ws协议路径（自定义英语加数字要记住自己配置ws会用到）

画蓝色长方形修改成你的v2ray端口

![image](https://user-images.githubusercontent.com/94978556/144739290-2f2e2ef9-77a8-4d99-8e66-8dc0ec321da7.png)

--------------------------------------------------------------------------------------------------------------------------------------
所有配置好重启nginx：
`service nginx restart`

如果出错尝试更换运行环境：

![image](https://user-images.githubusercontent.com/94978556/144739848-f5db4d54-426e-412b-9df6-3d2f16ab8618.png)

![image](https://user-images.githubusercontent.com/94978556/144756547-0dc2fbcb-975a-429c-b77d-fb217311553b.png)


第18步更换运行环境：
`chcon system_u:object_r:httpd_config_t:s0 /root/.acme.sh/换成你的域名_ecc/fullchain.cer`

第19步更换运行环境：
`chcon system_u:object_r:httpd_config_t:s0 /root/.acme.sh/换成你的域名_ecc/换成你的域名.key`

在次重启nginx：
`service nginx restart`

![image](https://user-images.githubusercontent.com/94978556/144739855-1342bb72-1983-4ca5-b158-415d0654e680.png)

--------------------------------------------------------------------------------------------------------------------
**这样访问你的网址就会跳转到你反代的网址了**  

**如果你配置了v2ray就可以达到伪装效果**

![image](https://user-images.githubusercontent.com/94978556/144739860-e9d13711-448d-422f-85f1-c10a836b0250.png)

![image](https://user-images.githubusercontent.com/94978556/144739872-7545db66-c199-48bc-a468-3a75918cdcc5.png)

----------------------------------------------------------------------------------------------------------------------
最后如果过程中遇到错误可以输入以下指令列出错误点：

`systemctl status nginx.service`

`systemctl status nginx.service -l`

赋读写执行权限：

`chmod 777 -R /root/.acme.sh`

**如果用Ubuntu系统**

有开头前缀这样的
`yum`换成
`apt`

**nginx常用指令和你可能会用到的指令**

启动nginx：
`start nginx`

修改配置后重新加载生效：
`nginx -s reload`

快速停止nginx：
`nginx -s stop`

完整有序的停止nginx：
`nginx -s quit`

重启nginx：
`service nginx restart`

查询VPS运行端口：
`lsof -i:加你想查询的端口号`

清除VPS端口运行程序：
`kill 加你查询的端口PID编号`

---------------------------------------------------------------------
**至此整个nginx反代加TLS证书申请就结束了**

如果需要配置v2ray  访问https://cxjiang.top/2019/08/02/v2ray-basic/

**小白分享  如有不对可以邮箱联系我**
