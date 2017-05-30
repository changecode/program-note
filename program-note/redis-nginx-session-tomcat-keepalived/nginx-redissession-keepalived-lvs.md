1、准备环境：	
	五台服务器 centos6.5 jdk8
	一台nginx服务器、二台部署tomcat服务器。在部署二台tomcat服务器时，
	可能需要配置防火墙端口：
		vim /etc/sysconfig/iptables 开放8080端口、80；编辑好后
		service iptables restart重新加载防火墙配置。
		然后分别打开Tomcat并访问。分别在index.jsp中body加入
		sessionID= <%=session.getId()%>
2、导出war包，修改Tomcatserver.xml配置文件内容，修改engine标签，添加j	vmRoute,用于识别Nginx访问的是哪个服务器Tomcat。比如137服务器标识为1		37Server1,139服务器标识为139Server2		
	<Engine name="Catalina" defaultHost="localhost" jvmRoute="137Server1">
	二台Tomcat服务器添加server.xml
	<Context path="" docBase="testproject"/>
	重启Tomcat
3、133服务器上安装Nginx
	先使用yum安装gcc pcre zlib openssl
	yum -y gcc/pcre pcre-devel/zlib zlib-level/openssl openssl-develplain
	将Nginx安装在usr/local下，进入Nginx目录，一次执行./configure make make install 默认占用80端口。启动Nginx ./sbin/nginx,关闭
	Nginx ./sbin.nginx -s stop。 访问Nginx首页
4、通过访问133时，即可访问到137、139其中一台Tomcat服务器。在Nginx目录conf目录下的nginx.conf文件，修改配置内容：
	#Nginx所有用户和组
	#user niumd nuymd;
	#工作的子进程数量（通常等于cpu 或者2倍CPU）
	worker_processes 2;
	#错误日志存放路径
	#error_log logs/error.log;
	#error_log logs/error.log notice;
	#error_log logs/error.log info;
	#指定pid存放文件
	pid logs/nginx.pid

	events{
		#使用网络io模型Linux建议epoll fressbsd建议kqueue
		#use epoll
		#允许最大链接数
		worker_connections 1024
	}

	http{
		include mime.types;
		default_type application/octet-stream;
	}
	#定义日志格式
	#log_format main '$remote_addr' - $remote_user [$time_local] $request '"$status" $body_bytes_sent "$http_referer" ' '"$http_user_agent" "$http_x_forwrded_for"';
	#access_log off
	access_log logs.access.log;

	client_header_timeout 3m;
	client_body_timeout 3m;
	send_timeout 3m;

	client_header_buffer_size 1k;
	large_client_header_buffers 4 4k;

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;

	#fastcgi_intercept_errors on;
	error_page 404 /404.html;

	#keepalive_timeout 75 20;
	gzip on;
	gzip_min_length 1000;
	gzip_type text/plain text/css application/x-javascript;

	#配置倍代理的服务器
	upstream blank{
		#ip_hash;
		server 192.168.50.137:8080
		server 192.168.50.139:8080
	}

	server{
		#nginx监听80端口，请求改端口时转发到真实服务器上
		listen 80; (如133服务器有外网映射的话，此处端口号应该为映射对于的端口号)
		#配置访问域名
		server_name 192.168.11.133;

		localtion / {
			#配置代理是指上面定义的二个被代理目标，blank名字必须一致
			proxy_pass http://blank;

			#proxy_redirect off;
			#非80端口使用，将代理服务器收到的用户的信息传到真实服务器上
			proxy_set_header Host $host(133服务器有外网映射的话，此处端口号应该为映射对于的端口号);
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			client_max_body_size 10m;
			client_body_buffer_size 128k;
			proxy_connect_timeout 300;
			proxy_send_timeout 300;
			proxy_read_timeout 300;
			proxy_buffer_size 4k;
			proxy_buffers 4 32k;
			proxy_busy_buffers_size 64k;
			proxy_temp_file_write_size 64k;
			add_header Access-Control-Allow-Origin *;
		}

		#定义500 502 503 504错误页面
		error_page 500 502 503 504 /50x.html;
		#错误页面位置
		location = /50x.html {
			#root标识路径 html为Nginx安装目录中的HTML文件夹/usr/local/nginx/html下
			root html;
		}
	}
}
重启二台Tomcat Nginx服务器，访问133:80。这时session还不能共享

5、Nginx轮询策略：
	1）轮询（默认策略）
		每个请求按时间顺序逐一分配到不同的后台服务器，如果后台服务器down掉，能自动剔除
	2）weight
		指定轮询几率，weight和访问比率成正比，用于后台服务器性能不均的情况下，数字越大越容易命中
		upstream bakend{
			server xx weight=2;
			server xx weight=1;
		}
	3) ip hash
		每个请求按IP的hash结果分配，这样每个请求访问到都是固定服务器，可以解决session共享问题
		upstream bakend{
			ip_hash;
			server xx;
			server xx;
		}	

6、redis:
	140服务器上搭建redis环境：
	yum install -y gcc-c++
	进入redis目录，执行make命令，安装到usr/local/redis下
	make PREFIX=/usr/local/redis install port默认是6379
	命令完成后，将redis.conf拷贝到安装目录下
	后端启动redis: 修改redis.conf中的daemonize的值，修改为yes
	./bin/redis-server ./redis.conf
	./bin/rdis-cli shutdown

7、session共享：
	在所有需要共享session的服务器Tomcat lib添加三个jar包 注意保持版本的一致
	commons-pool-1.6  jedis-2.2.0 tomcat-redis-session-manager-1.2-tomcat-7.jar
	conf目录下的context.xml 配置redis服务：
	<Value className="com.radiadesign.catalina.session.RedisSessionHandlerValue"/>
	<Manager className="com.radiadesign.catalina.session.RedisSessionManager" host="192.168.50.140" port="6379" database="0" maxInacticeInterVal="60"（以秒统计作为过期时间
	） />

	环境为Tomcat7 jdk7/8
	commons-pool2-2.4.1 jedis-2.6.2 另外一个同上
	className=com.orangefunction.tomcat.redissessions.RedisSessionManager
启动redis tomcat Nginx  刷新二台Tomcat服务器页面，可以看到session值不变，说明session共享（可能此时访问redis无法访问，这时由于redis的安全机制默认只有127.0.0.1才能访问，在redis.conf可以找到bind 127.0.0.1 可以将IP改为访问者IP 如有多个访问者 可以将其注释掉 然后在配置文件中找到protected-mode 改为no关闭保护模式）
使用redis数据库，放入session中的对象必须实现序列化接口

同理再搭建一个Nginx服务器，修改监听IP

8、keepalived
	先在二台Nginx服务器上安装keepalived
	keepalived-1.2.7-3.el6.x86_64.rpm
	openssl-1.0.1e-30.el6_6.4.x86_64.rpm
	安装OpenSSL
	rpm -Uvh --nodeps ./openssl-1.0.1e-30.e16_6.4.x86_64.rpm
	安装keepalived
	rpm -Uvh --nodeps ./keepalived-1.2.7-3.e16.x86_64.rpm
	找到keepalived的配置文件，keepalived.conf
	1) 配置133
		global_defs {
			#指定keepalived在发生切换时需要发送的email对象,一行一个
			notification_email {
				acassen@xxx.com
			}
			#指定发件人
			notification_email_from Alexandre.xxx(邮箱)
			#指定smtp服务器地址
			#smtp_server 192.186.200.1
			#指定SMTP连接超时时间
			#smtp_connect_timeout 30
			#运行keepalived机器的一个标识
			router_id LVS_DEVEL	
		}

		#主备配置
		vrrp_instance VI_1 {
			#标识状态为master 备份机为backup
			state MASTER
			#设置keepalived实例绑定的服务器网卡
			interface eth0
			#同一个实例下（同一组主备机）virtual_router_id必须相同
			virtual_router_id 51
			#MASTER权重高于backup
			priority 100
			#master与backup负载均衡之间同步检查 的时间间隔单位为秒
			advert_int 1
			#设置认证
			authentication {
				auto_type PASS
				#密码
				auth_pass 1111
			}
			#设置虚拟IP
			virtual_ipaddress {
				192.168.50.88
			}
		}
backup的配置，修改state为backup priority比master低，virtual_router_id和master的值一致

启动keepalived命令：
	service keepalived start/stop  在主机和备机上使用 ip addr命令，可以看到133带有虚拟IP ，在浏览器中输入虚拟IP即可访问主机Nginx133，然后转发到Tomcat服务器上，备机上 没有绑定虚拟IP







