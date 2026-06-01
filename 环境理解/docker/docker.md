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

