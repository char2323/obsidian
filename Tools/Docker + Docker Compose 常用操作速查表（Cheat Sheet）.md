# 🐳 Docker & Docker Compose Cheat Sheet

## 🔹 镜像 (Images)

|操作|命令|
|---|---|
|拉取镜像|`docker pull ubuntu:20.04`|
|查看本地镜像|`docker images`|
|删除镜像|`docker rmi <IMAGE_ID>`|

---

## 🔹 容器 (Containers)

|操作|命令|
|---|---|
|启动交互式容器|`docker run -it ubuntu /bin/bash`|
|后台运行并映射端口|`docker run -d --name mynginx -p 8080:80 nginx`|
|查看运行中的容器|`docker ps`|
|查看所有容器|`docker ps -a`|
|停止容器|`docker stop <CONTAINER_ID>`|
|启动容器|`docker start <CONTAINER_ID>`|
|删除容器|`docker rm <CONTAINER_ID>`|
|进入容器|`docker exec -it <CONTAINER_ID> /bin/bash`|

---

## 🔹 日志 & 监控

|操作|命令|
|---|---|
|查看容器日志|`docker logs -f <CONTAINER_ID>`|
|查看容器资源使用|`docker stats`|

---

## 🔹 数据卷 (Volumes)

|操作|命令|
|---|---|
|挂载目录|`docker run -v /host:/container ubuntu`|
|查看所有卷|`docker volume ls`|
|删除卷|`docker volume rm <VOLUME_NAME>`|

---

## 🔹 网络 (Networks)

|操作|命令|
|---|---|
|查看网络|`docker network ls`|
|创建网络|`docker network create mynet`|
|指定网络运行容器|`docker run -d --network=mynet nginx`|

---

# ⚙️ Docker Compose

## 基本命令

|操作|命令|
|---|---|
|启动服务|`docker-compose up -d`|
|停止并移除容器|`docker-compose down`|
|查看服务状态|`docker-compose ps`|
|查看日志|`docker-compose logs -f`|
|构建镜像|`docker-compose build`|
|重启服务|`docker-compose restart`|

---

## 单服务操作

|操作|命令|
|---|---|
|启动指定服务|`docker-compose up -d service_name`|
|停止指定服务|`docker-compose stop service_name`|
|重启指定服务|`docker-compose restart service_name`|
|进入容器|`docker-compose exec service_name /bin/bash`|

---

## 示例 `docker-compose.yml`

```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - mynet

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: testdb
    networks:
      - mynet

networks:
  mynet:
```

---

要不要我帮你再整理成 **Markdown 表格版 + PDF 版**？这样你可以直接在 Obsidian 里用 Markdown，看需要时也能导出 PDF 打印。