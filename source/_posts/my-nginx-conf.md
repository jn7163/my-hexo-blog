title: 我的nginx配置
date: 2016-10-05 20:38:55
tags: [nginx, html]
categories: 网络
---
自己网站的nginx配置，减少重复编译nginx查找资料耗费的时间。
<!-- more -->

## 需求

- 反向代理谷歌
- 较高的压缩率，加快网页传输
- https安全性提高

最终选用了以下nginx模块

|模块|功能|
|----|---|
|[ngx_http_substitutions_filter_module](https://github.com/yaoweibin/ngx_http_substitutions_filter_module)|正则表达式|
|[ngx_brotli](https://github.com/google/ngx_brotli)|谷歌开源压缩brotli库|
|[nginx-ct](https://github.com/grahamedgecombe/nginx-ct)|CT证书透明|

其中需要系统编译或者配合的其它非nginx模块有

|模块|功能|
|----|---|
|[ct-submit](https://github.com/grahamedgecombe/ct-submit)|提交CT，golang版|
|[libbrotli](https://github.com/bagder/libbrotli)|动态库，谷歌的字典压缩|

## 编译

### libbrotli

编译安装字典压缩库

	git clone https://github.com/bagder/libbrotli
	cd libbrotli
	./autogen.sh
	./configure
	make install
	
编译后添加默认的lib路径到系统变量中，避免启动Nginx找不到libbrotlienc.so.1

如果需要自启动nginx，还要将系统变量添加到/etc/rc.local中，建议也

	vi /etc/rc.local 
	# 增加
	export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH

### ct-submit

用于每次签证书后提交证书透明度报告

	apt install golang
	wget https://github.com/grahamedgecombe/ct-submit/archive/v1.1.2.zip
	unzip v1.1.2.zip
	cd ct-submit-1.1.2
	go build
	
可执行文件备用，需要签证书后提交 *cert + intermediates*的合成文件，称为chained.pem

### nginx

模块下载

	mkdir ~/nginx_my && cd nginx_my
	git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module
	git clone https://github.com/cuber/ngx_http_google_filter_module
	git clone https://github.com/google/ngx_brotli
	git clone https://github.com/grahamedgecombe/nginx-ct
	
openssl取最新版

	wget https://www.openssl.org/source/openssl-1.1.0b.tar.gz
	tar xf openssl-1.1.0b.tar.gz

贴出configure配置，这里选用了几个我用到的模块，还有add_module上文的模块

	./configure \
	--conf-path=/etc/nginx/nginx.conf  \
	--with-pcre-jit \
	--with-http_ssl_module  \
	--with-http_stub_status_module  \
	--with-http_realip_module  \
	--with-http_auth_request_module  \
	--with-http_addition_module  \
	--with-http_dav_module  \
	--with-http_geoip_module  \
	--with-http_gunzip_module  \
	--with-http_gzip_static_module  \
	--with-http_image_filter_module  \
	--with-http_v2_module  \
	--with-http_sub_module  \
	--with-http_xslt_module  \
	--with-stream  \
	--with-stream_ssl_module  \
	--with-threads  \
	--add-module=../ngx_http_google_filter_module  \
	--add-module=../ngx_http_substitutions_filter_module \
	--add-module=../ngx_brotli \
	--add-module=../nginx-ct \
	--with-openssl=../openssl
	
直接编译即可

	make && make install
	
## 配置

### 反向代理

参考[这篇文章](/2016/07/31/nginx-reverse-proxy-for-google/)，可以非常傻瓜化的部署反代。

### 压缩网页

修改/etc/nginx/nginx.conf达到基本的gzip效果

	gzip  on;
	gzip_vary          on;
	gzip_comp_level    6;
	gzip_buffers       16 8k;
	gzip_min_length    1k;
	gzip_proxied       any;
	gzip_disable       "msie6";
	gzip_http_version  1.0;
	gzip_types         text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;
	
再加上谷歌的黑科技brotli压缩

	brotli             on;
    brotli_comp_level  6;
    brotli_types       text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript image/svg+xml;
	
### 提升ssl安全等级

前提是网站有HTTPS部署，且对相关概念有基本了解。可以自行参考[imququ大神博客](https://imququ.com)，学习基本的https部署方法！

一、自定义生成密钥交换，取代nginx默认的weak exchange

这个diffie-hellman算法是用来干啥的：作用就是让访问的者浏览器和你的服务器能安全的交换密钥，具体原理就自行谷歌吧，你不能强迫数学渣渣来给你解释密码学的东西是不是？总之nginx默认采用1024位的diffie-hellman，强度太低太危险，容易被别有用心的黑客叔叔破解，自己生成一个4096位的去替换掉就好啦。

	# 如果你的机器cpu性能不够强大，耗时很长，可以用 2048 位
	# 我的笔记本（i5 3317u）跑4096位，跑了二十分钟。。。直接拖到VPS上面/root/
	openssl dhparam -out dhparam.pem 4096
	
nginx.conf中的server标签中添加

	ssl_dhparam /root/dhparam.pem;
	
二、开启OCSP

nginx.conf中的server标签中添加

	ssl_stapling               on;
	ssl_stapling_verify        on;
	
三、手动指定优先的加密算法

	ssl_ciphers                EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
	ssl_prefer_server_ciphers  on;
	ssl_protocols              TLSv1 TLSv1.1 TLSv1.2;
	
四、跨站攻击

	add_header X-Frame-Options SAMEORIGIN;
	add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection 1;
	
五、强制HSTS

新建一个http服务器，强制跳转到https:

	server_name xxx.com
	listen 80;
	location / {
		rewrite ^(.*)$  https://$host$1 permanent;
	}

在https服务器中添加

	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains;" always;
	
有兴趣可以加入google chrome的Preload List里面，将从浏览器自动强制使用https。需要申请，审批需要几天。但是加进去preload list后很难从其中移除，需要发邮件移除。。

六、证书透明度

CT证书透明这部份实测不好用。。貌似只对chrome有用，还没有大规模部署。下次补回来。
	