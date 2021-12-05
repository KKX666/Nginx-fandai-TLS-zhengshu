# Nginx-反代-TLS-证书申请

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

![image](https://user-images.githubusercontent.com/94978556/144738957-a45cde00-d454-4ea8-bd96-dac02e5d7f6c.png)

--------------------------------------------------------------------------
进入nginx的站点配置文件夹，并新建我们对应域名的配置文件
------------------------------------------------------------
第15步：
`cd /etc/nginx/conf.d`

第16步：
`vim 换成你的域名.conf`进入编辑配置文件

第17步访问https://cxjiang.top/2019/08/02/v2ray-basic/   粘贴以下内容复制进去

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

第18步：
`chcon system_u:object_r:httpd_config_t:s0 /root/.acme.sh/换成你的域名_ecc/fullchain.cer`

第19步：
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
最后如果过程中遇到错误可以输入以下指令查询错误点：

`systemctl status nginx.service`

`systemctl status nginx.service -l`

赋读写执行权限：

`chmod 777 -R /root/.acme.sh`

**如果用Ubuntu系统**

有开头前缀这样的
`yum`换成
`apt`

---------------------------------------------------------------------
**至此整个nginx反代加TLS证书申请就结束了**

感谢金主大佬手把手

如果需要配置v2ray  访问https://cxjiang.top/2019/08/02/v2ray-basic/

**小白分享  如有不对可以邮箱联系我**
