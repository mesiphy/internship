# 基本概念与运行逻辑

Dockerfile  --build-->  Image 镜像  --run/up-->  Container 容器 `build` 建立的是镜像 Image。
compose.yml 负责统一管理多个容器：
backend 容器 + frontend 容器 + db 容器 + redis 容器 ... `compose up` 启动的是容器 Container。

Dockerfile：镜像的制作说明书
Image：制作好的模板
Container：模板运行出来的实例
Volume：容器外部保存数据的地方
Compose：统一管理多个容器的配置文件
docker build -t quant-backend ./backend
docker run quant-backend
docker compose build backend
docker compose up -d

# 运行流程
第一种：代码被打包进 image 里。
比如你的 `Dockerfile` 里有：

```
COPY . .
```

这表示在 build 的时候，把本地代码复制进 image。
本地代码  --docker build-->  image  --docker run/up-->  container
这种情况下，如果你修改了本地代码：

```
本地代码变了image 不会自动变container 也不会自动变
```
第二种：代码通过 volume 挂载进 container



cd FlowTrace-v2/V1
docker compose --env-file config/.env -f docker-compose.yml -f docker-compose.dev.yml up -d --build


# 项目移植
你要把当前 Docker 中的项目移植到其他平台，本质上要迁移三类东西：

```
1. 项目代码：backend、frontend、compose.yml、Dockerfile 等2. 配置文件：.env、config/.env、Nginx 配置等3. 数据文件：PostgreSQL 数据、data/、logs/、上传文件等
```

不一定需要迁移：

```
container 容器image 镜像Docker 临时网络build 缓存
```

必须迁移：

```
项目源码Dockerfilecompose.yml.env / config/.env数据库数据业务数据目录，例如 ./data必要的配置文件
```


# 数据库
|名称|当前值|含义|
|---|---|---|
|Compose 服务名|db|Docker 网络里的主机名，后端容器用它连接数据库|
|容器名|flowtrace-db|Docker Desktop 里看到的容器名字|
|PostgreSQL 数据库名|flowtrace|DBeaver 里要填的 Database|
|PostgreSQL 用户名|flowtrace|DBeaver 里要填的 Username|
|容器内端口|5432|Postgres 在容器里监听的端口|
|宿主机端口|5433|你电脑/DBeaver 连接用的端口|

关键在这里：

docker-compose.yml (line 8)

yaml

`services: db: image: postgres:16 container_name: flowtrace-db`

db 是服务名，flowtrace-db 是容器名。

然后这里：

yaml

`POSTGRES_DB: ${POSTGRES_DB:-flowtrace} POSTGRES_USER: ${POSTGRES_USER:-flowtrace} POSTGRES_PASSWORD: ${PG_PASSWORD:-flowtrace_dev}`

这才是 PostgreSQL 里的数据库名、用户名和密码。

所以 DBeaver 从你电脑连接时应该填：

text

`Host: localhost Port: 5433 Database: flowtrace Username: flowtrace Password: flowtrace_dev`

而后端容器内部连接时用的是：

text

`Host: db Port: 5432 Database: flowtrace Username: flowtrace Password: flowtrace_dev`

这里 db:5432 只能在 Docker Compose 网络内部使用；DBeaver 在你电脑上，不认识 db 这个主机名，所以要用 localhost:5433