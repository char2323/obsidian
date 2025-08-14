### 🐳 一、镜像（Image）相关

```bash
# 查看本地已有镜像
docker images

# 搜索镜像
docker search 镜像名

# 拉取镜像
docker pull 镜像名[:标签]

# 删除镜像
docker rmi 镜像ID或镜像名

# 构建镜像（需要Dockerfile）
docker build -t 镜像名[:标签] .

# 给镜像打标签
docker tag 原镜像:标签 新镜像:标签
```

---

### 🧱 二、容器（Container）相关

```bash
# 运行一个新容器（后台）
docker run -d --name 容器名 镜像名

# 运行容器并进入交互模式
docker run -it --name 容器名 镜像名 /bin/bash

# 运行容器并映射端口
docker run -d -p 主机端口:容器端口 镜像名

# 运行容器并挂载卷
docker run -v 主机路径:容器路径 镜像名
#在ubuntu挂载卷的时候通常不需要--privileged参数

# 查看运行中的容器
docker ps

# 查看所有容器（包括已停止）
docker ps -a

# 停止容器
docker stop 容器名或ID

# 启动容器
docker start 容器名或ID

# 重启容器
docker restart 容器名或ID

# 删除容器
docker rm 容器名或ID

# 查看容器日志
docker logs -f 容器名

# 进入正在运行的容器
docker exec -it 容器名 /bin/bash

# 在容器中运行命令
docker exec 容器名 命令
```

---

### 📦 三、卷（Volume）相关

```bash
# 创建卷
docker volume create 卷名

# 查看所有卷
docker volume ls

# 删除卷
docker volume rm 卷名

# 查看卷详细信息
docker volume inspect 卷名
```

---

### 🌐 四、网络（Network）相关

```bash
# 查看网络
docker network ls

# 创建网络
docker network create 网络名

# 删除网络
docker network rm 网络名

# 让容器加入网络
docker network connect 网络名 容器名

# 让容器离开网络
docker network disconnect 网络名 容器名
```

---

### 🛠 五、系统管理

```bash
# 查看Docker系统资源使用情况
docker system df

# 清理无用数据（未使用的镜像、容器、卷等）
docker system prune

# 只清理未使用的镜像
docker image prune
```

---

### 📋 六、Docker Compose 简要

```bash
# 启动服务
docker-compose up -d

# 停止服务
docker-compose down

# 查看服务状态
docker-compose ps

# 重启服务
docker-compose restart

# 构建镜像
docker-compose build
```
