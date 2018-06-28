# phabricator-docker
（这是一个草稿，有时间再来完善）基于Docker安装phabricator的教程，Redpointgames phabricator 的安装教程。
此教程安装的是Redpointgames phabricator，
原库地址：
> https://hub.docker.com/r/redpointgames/phabricator/

## 1. 安装并运行用于phabricator的MySQL数据库
	docker run --name DBpha --restart=always -p 3307:3306 -v /data/docker/mysql/DBpha:/var/lib/mysql -v /data/docker/mysql/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=pha123456 -d mysql:5.7

## 2. 安装并运行一个查看phabricator 数据库的的PHPMyAdmin
	docker run --name myadmin -d --link DBpha:db -p 8090:80 phpmyadmin/phpmyadmin

## 3. 安装并运行phabricator
	docker run \
	    --name phabricator \
	    --restart=always -p 8080:80 -p 443:443 -p 2222:22 \
	    --link DBpha:mysql \
	    --env PHABRICATOR_HOST=your.domain.com \
	    --env MYSQL_HOST=8349bc05835b \
	    --env MYSQL_PORT=3306 \
	    --env MYSQL_USER=root \
	    --env MYSQL_PASS=pha123456 \
	    --env PHABRICATOR_REPOSITORY_PATH=/repos \
	    --env PHABRICATOR_HOST_KEYS_PATH=/hostkeys/persisted \
	    -v /data/docker/phabricator/repos:/repos \
	    -v /data/docker/phabricator/hostkeys:/hostkeys \
	    redpointgames/phabricator


##### 关键参数说明：
	--link {Your mysql name}
	--env PHABRICATOR_HOST={你的访问域名}
	--env MYSQL_HOST={你的容器ID}
	-v #容器中的目录文件映射到本地宿主机中
	#其他参数如端口号、具体目录可以使用默认，或换成自己的。
## 4. 设置宿主机中用于存放Repositories的文件夹权限
	chown 2000:2000 /data/docker/phabricator/repos

## 5. 设置phabricator的SSH访问端口（外网访问时所用）
	# 进入phabricator容器
	docker exec -it phabricator bash
	# 设置端口
	/srv/phabricator/phabricator/bin/config set diffusion.ssh-port 2222
## 6. 配置宿主机的Nginx，让外网可以通过非80端口访问 phabricator Container
	server {
		listen  80;
		server_name  your.domain.com;
		server_name_in_redirect off;
		proxy_set_header Host $host:$server_port;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header REMOTE-HOST $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		location / {
		    proxy_pass http://127.0.0.1:8080/;
		}
	}
## 7. 如果你的域名 your.domain.com 只是个测试域名，你可以配置本地Hosts 进行访问
	#根据自己的需要换成宿主机的IP地址
	192.168.000.000  your.domain.com
**至此，您在浏览器输入 your.domain.com 便可以正常使用 phabricator 了，GIT及SVN等都可正常使用了。**
----
# docker 常用命令
## 运行已经存在的容器
	docker container start
