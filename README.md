# phabricator-docker
（这是一个草稿，有时间再来完善）基于Docker安装phabricator的教程，Redpointgames phabricator 的安装教程。
此教程安装的是Redpointgames phabricator，
原库地址：
> https://hub.docker.com/r/redpointgames/phabricator/

## 1. 安装并运行用于phabricator的MySQL数据库
	docker run --name DBpha -p 3307:3306 -v /data/docker/mysql/DBpha:/var/lib/mysql -v /data/docker/mysql/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

## 2. 安装并运行一个查看phabricator 数据库的的PHPMyAdmin
	docker run --name myadmin -d --link DBpha:db -p 8090:80 phpmyadmin/phpmyadmin

## 3. 安装并运行phabricator
	docker run \
	    --name phabricator \
	    --rm -p 8080:80 -p 443:443 -p 2222:22 \
	    --link DBpha:mysql \
	    --env PHABRICATOR_HOST=my.work.com \
	    --env MYSQL_HOST=f2c7a41233a9 \
	    --env MYSQL_PORT=3306 \
	    --env MYSQL_USER=root \
	    --env MYSQL_PASS=123456 \
	    --env PHABRICATOR_REPOSITORY_PATH=/repos \
	    --env PHABRICATOR_HOST_KEYS_PATH=/hostkeys/persisted \
	    -v /data/docker/phabricator/repos:/repos \
	    -v /data/docker/phabricator/hostkeys:/hostkeys \
	    redpointgames/phabricator

## 4. 设置宿主机中用于存放Repositories的文件夹权限
	chown 2000:2000 /data/docker/phabricator/repos

## 5. 设置phabricator的SSH访问端口（外网访问时所用）
	# 进入phabricator容器
	docker exec -it phabricator bash
	# 设置端口
	/srv/phabricator/phabricator/bin/config set diffusion.ssh-port 2222
  
----
# docker 常用命令
## 运行已经存在的容器
	docker container start
