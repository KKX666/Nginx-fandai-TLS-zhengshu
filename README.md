# Nginx-fandai-TLS

![GitHub Light](https://github.com/github-light.png#gh-dark-mode-only)

Nginx反代/TLS配置教程总结

基于CentOS 7系统
-----------------------------
服务器更新

第1步：`yum update`

第2步:`yum upgrade`

-------------------------------
第3步可以检查一下upgrade是否安装：`yum upgrade -y`也可以忽略这步  如下安装ok

![image](https://user-images.githubusercontent.com/94978556/144737966-7863922c-1ba2-46ff-9eff-290b279294ae.png)


安装TLS证书申请工具以下：

第4步：`curl  https://get.acme.sh | sh`
如果提示openssl未安装如下：

![image](https://user-images.githubusercontent.com/94978556/144737872-e89c3a86-af81-45f3-b377-2a557ccb4e4e.png)

安装openssl：`yum install openssl`

再执行一次`curl  https://get.acme.sh | sh`

``
