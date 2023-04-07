安装docker环境
(注：已具备docker及docker-compose环境可以忽略这一步，如果只缺docker-compose，只执行docker-compose那一步)

安装docker引擎 docker的安装大家可以看docker的官方文档，里面的步骤都非常的详细，这里只演示Centos

#### 如果是CentOS8，先清除自带的docker命令(CentOS8自带的docker是podman，不是原生docker，非CentOS8可以不操作)

`sudo yum remo`ve docker \`
              `docker-client \`
              `docker-client-latest \`
              `docker-common \`
              `docker-latest \`
              `docker-latest-logrotate \`
              `docker-logrotate \`
              docker-engine`

####添加 docker 源

`sudo curl -o  /etc/yum.repos.d/docker-ce.repo  https://download.docker.com/linux/centos/docker-ce.repo`

####安装 docker 引擎

`sudo yum -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin`

安装docker-compose

####下载docker-compose

sudo sh -c "curl -SL https://get.daocloud.io/docker/compose/releases/download/v2.12.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"

####给 docker-compose 执行权限

`sudo chmod +x /usr/local/bin/docker-compose`
`sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`


配置docker环境

####配置国内镜像加速，和log文件大小

`sudo sh -c 'cat > /etc/docker/daemon.json << EOF`
`{`
  `"log-driver":"json-file",`
  `"log-opts":{ "max-size" :"50m","max-file":"3"},`
  `"registry-mirrors": [`
    `"https://hub-mirror.c.163.com",`
    `"https://mirror.baidubce.com"`
  `]`
`}`
`EOF'`

####将当前用户加入docker组

`sudo usermod -aG docker $USER`

####启动docker服务并配置自启

sudo systemctl start docker && sudo systemctl enable docker

安装Redash
选定Redash安装目录这里假定是/opt/redash
`sudo mkdir /opt/redash`
`sudo chown -R ${USER} /opt/redash`
`cd /opt/redash`

创建env文件，写环境变量，根据自己环境调整环境变量的值

####/opt/redash/env/内容

`PYTHONUNBUFFERED=0`
`REDASH_LOG_LEVEL=INFO`
`REDASH_REDIS_URL=redis://redis:6379/0`
`POSTGRES_PASSWORD=aaa123456`
`REDASH_COOKIE_SECRET=wo3urion23i4un2l34jm2l34k`
`REDASH_SECRET_KEY=u2o34nlfksjelruirk`
`REDASH_DATABASE_URL="postgresql://postgres:aaa123456@postgres/postgres"`
`ORACLE_HOME="/usr/lib/oracle/12.2/client64"`
`LD_LIBRARY_PATH="/usr/lib/oracle/12.2/client64/lib"`
`REDASH_FEATURE_ALLOW_CUSTOM_JS_VISUALIZATIONS="true"`
`REDASH_ADDITIONAL_QUERY_RUNNERS="redash.query_runner.oracle,redash.query_runner.python"`

在安装目录创建docker-compose.yml(可以自己新建，也可以重其他地方拷贝),下面是内容,注意image的值需要修改为使用的redash镜像id。
version: "2"
x-redash-service: &redash-service

  #现在image的值为中文开源版的tag如果要使用官方的镜像，在docker hub上查看官方tag，然后替换。

  image: dazdata/redash:v10-21.11.19
  depends_on:

   - postgres
   - redis
     env_file: /opt/redash/env
       restart: always
     services:
       server:
         <<: *redash-service
         command: server
         environment:
     REDASH_WEB_WORKERS: 4
       scheduler:
         <<: *redash-service
         command: scheduler
       worker:
         <<: *redash-service
         command: worker
         environment:
     WORKERS_COUNT: 4
       redis:
         image: redis:5.0-alpine
         restart: always
       postgres:
         image: postgres:12-alpine
         env_file: /opt/redash/env
         volumes:
     - /opt/redash/postgres-data:/var/lib/postgresql/data
       restart: always
       nginx:
         image: dazdata/redash-nginx:latest
         ports:
     - "5000:80"
       depends_on:
     - server
       links:
     - server:redash
       restart: always

在安装目录，拉起Redash。
`sudo docker-compose up -d`

初始化数据库，然后通过浏览器访问服务器5000端口即可
`sudo docker-compose run --rm server create_db`
