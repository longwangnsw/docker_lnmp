# Introduction

Deploy lnmp(Linux, Nginx, MySQL 5.7, PHP 7.1) plus Redis using docker.

### Architecture

![architecture][1]

The whole app is divided into four Containers:

1. Nginx is running in `Nginx` Container, which handles requests and makes responses.
2. PHP or PHP-FPM (aroud 630M) is put in `PHP-FPM` Container, it retrieves php scripts from host, interprets, executes then responses to Nginx. If necessary, it will connect to `MySQL` and `Redis` as well.
3. MySQL in `MySQL` Container,
4. Redis in `Redis` Container, 
5. 宿主机上nginx/conf.d/*.conf是vhost文件，映射进入了nginx容器。如果有修改，需要docker-compose restart，或者进入nginx这个容器重启nginx。
6. 宿主机上app目录是给项目用的，这个目录映射进了fpm、nginx容器，注意git clone操作建议在fpm容器内完成
7. 宿主机上mysql目录是为了数据库持久化，这个目录映射进了mysql容器

Your project scripts are located on host, you can edit files directly without rebuilding/restarting whole images/containers. But normally host has no php envirenment, so you should do php composer on fpm container.

### Build and Run

At first, you should have had [Docker](https://docs.docker.com) and [Docker Compose](https://docs.docker.com/compose) installed.

Without building images one by one, you can make use of `docker-compose` and simply issue:

    $ sudo docker-compose up

For more operations to containers, please refer to:

    $ sudo docker-compose --help

Check out http://docker-localhost

### Deploy any project

1. docker exec enter fpm container; go into app folder, git clone your project ; vi .env
2. docker exec enter fpm container; go into project foler, composer install, etc. (reference composer.json)
3. docker exec enter fpm container; mysql -h mysql's container, create database. php artisan migrate(only on fpm container)
3. go into nginx/con.f folder, modify or new a yourvhost.conf
4. sudo docker-compose restart
5. if add project more, just go again 1.2.3.4

ubuntu install node
1. apt install gnupg
2. curl -sL https://deb.nodesource.com/setup_10.x | bash -
3. apt-get install -y nodejs
4. which gulp || (npm install gulp -g; )
5. npm install
6. gulp release

  [1]: docker_lnmp_architecture.png

### 进一步说明

#### 1、较为复杂的是fpm这个容器。基础镜像是php-fpm（版本7.1.21）。在此基础上安装软件。

  gd：因为很多项目需要这个php图形处理工具，虽然我们没有直接用它。同时注意gd需要libfreetype、libjpeg
  
  mcrypt：这个不用多说
  
  git、vim、mysql-client、redis-tools：常用工具，使用中你会发现，我们主要是登陆fpm这个容器操作其他容器
  
  composer：这个是php composer
  
  nodejs：对，你没有看错，这个容器预装了nodejs（版本见php-fpm/Dockerfile）、npm。同时注意nodejs需要gnupg
  

#### 2、window 10 pro可以安装docker和docker-compose

  compose安装指导在：https://docs.docker.com/compose/install/#master-builds
  
  只需要两条命令（在powershell中执行）：
```
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
  Invoke-WebRequest "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-Windows-x86_64.exe" -UseBasicParsing -OutFile $Env:ProgramFiles\docker\docker-compose.exe
```

#### 3、window 10在运行docker-compose时有一个share权限的问题。

  方法1、右下角找docker，右键点setting，找到shared Driver（需要用户名/密码，如果域用户不行，试创建本地用户并加入Admin）
  
  方法2、也可以使用powershell命令：
```
  Set-NetConnectionProfile -interfacealias "vEthernet (DockerNAT)" -NetworkCategory Private
```
  同时也留意防火墙对445端口的设置
  
  https://stackoverflow.com/questions/42203488/settings-to-windows-firewall-to-allow-docker-for-windows-to-share-drive/43904051

#### 4、进入容器的命令是：docker exec -it docker_lnmp_redis_fpm_1 /bin/bash

  fpm、nginx、mysql、redis，四个都可以用上述命令进入其中，就是换掉上述命令中容器名字，比如docker_lnmp_redis_nginx_1

  windows上命令变成：winpty docker exec -it docker_lnmp_redis_fpm_1 bash

  不过windows上如果用powershell，不需要winpty

#### 5、四个容器之间自动完成了hosts设置。

  nginx访问fpm使用DOCKER_PHP_FPM，fpm访问mysql使用DOCKER_MYSQL，fpm访问redis使用DOCKER_REDIS。

  请看各个容器/etc/hosts，vhost中，我们用DOCKER_PHP_FPM:9000调用fpm

  注意，项目的.env中，请使用DOCKER_MYSQL/DOCKER_REDIS，这样就不用关心ip地址了

#### 6、因为调试代码需要，端口暴露了80/3306/6379，就是说，直接访问宿主机这些端口即可。

  注意：同时宿主机不能有相应进程使用这3个端口。

  如果不想暴露，修改docker-compose中ports域即可。想变成其他端口道理是一样的。
