---
course: COMP47780 Cloud Computing
week: 1
topic: Practical 1 - Docker
---

# Practical 1 知识点

## 1. Docker 的目的

`Docker` 将 application（应用）、runtime（运行环境）和 dependencies（依赖）一起封装，使程序能在 laptop、server 和 cloud 上以一致方式运行。这种一致性称为 portability（可移植性）。

## 2. Image 和 Container

- `image`（镜像）：read-only（只读）的运行模板，例如 `nginx`。
- `container`（容器）：由 image 创建的 runnable instance（可运行实例）。

image 可以创建多个 container；删除 container 不会删除 image。

```text
image → docker run → container
```

container 的 main process（主进程）结束后，container 会进入 exited state（退出状态）。`docker ps` 只显示 running container；`docker ps -a` 显示全部 container。

## 3. docker run 的行为

```bash
docker run IMAGE
```

`docker run` 会：

1. 查找 local image（本地镜像）；
2. image 不存在时自动从 registry（镜像仓库）pull；
3. create（创建）并 start（启动）container。

## 4. Port mapping

```bash
docker run -p 8080:80 IMAGE
```

格式为：

```text
host-port:container-port
```

- `8080` 是 host（宿主机）的 port；
- `80` 是 container 内部的 port。

多个 container 可同时使用自己的 container port `80`，因为各自有 network namespace（网络命名空间）；但两个 container 不能同时绑定同一个 host port，例如 `8080`。

`-d` 表示 detached mode（后台运行）；`--name` 为 container 指定可读名称。

## 5. Container 的数据默认不持久

写入 container 内部的文件会进入 writable layer（可写层）。删除 container 后，这一层也被删除；重新从同一 image 启动的 container 不会保留该文件。

需要持久保存 data（数据）时，使用：

- `volume`（Docker 管理的持久化存储）；
- `bind mount`（挂载 host 文件夹）。

## 6. Container 和 VM 的区别

- `container` 共享 Linux host kernel（宿主内核）。
- `virtual machine / VM`（虚拟机）运行独立的 guest OS（客户操作系统）。

Linux 上，container 与 host 通常报告相同 kernel。macOS / Windows 上，Docker Desktop 会在后台运行 lightweight Linux VM（轻量 Linux 虚拟机），container 实际运行在其中。

## 7. Docker networking

container 默认具有自己的 private IP（私有 IP）和 network namespace。

```bash
docker network create labnet
docker run -d --network labnet --name web nginx
docker run -it --rm --network labnet busybox
wget -O - http://web:80
```

在 user-defined network（自定义网络）中，Docker embedded DNS（内置 DNS）会将 container name（容器名称）如 `web` 解析为对应 IP。因此 multi-container application（多容器应用）可以使用服务名称通信，而不是写死 IP。

`-p 8080:80` 只用于让 host/browser 访问 container；同一个 Docker network 内的 container 之间通信不需要 `-p`。

## 8. Registry、Repository 和 Tag

`registry`（镜像仓库）负责存储和分发 images；Docker Hub 是默认 registry。

image name 的结构：

```text
registry/namespace/repository:tag
docker.io/library/nginx:alpine
```

- `registry`：镜像仓库；
- `namespace`：命名空间，例如 Docker Hub username；
- `repository`：镜像项目名；
- `tag`：版本/变体标签。

`nginx` 默认相当于 `nginx:latest`，但 `latest` 不表示 newest version（最新版本）；它是 mutable tag（可变标签）。生产环境应优先使用明确 version tag 或 immutable digest（不可变内容摘要）。

## 9. docker tag、push 和 layer reuse

```bash
docker tag nginx:alpine <username>/comp-nginx:v1.0
docker push <username>/comp-nginx:v1.0
```

`docker tag` 不复制 image data；它只为相同 image 添加另一个 reference（引用），所以两个 tag 显示相同 `IMAGE ID`。

Docker image 由 layers（层）组成，使用 content-addressing（内容寻址）和 digest（摘要）识别。相同 layer 在 pull/push 时会被 reuse（复用），不会重新下载或上传。

## 10. Cleanup commands

```bash
docker ps -a
docker rm -f CONTAINER
docker rmi IMAGE
docker system df
docker container prune
```

- `docker rm -f`：强制 stop 并 remove container；
- `docker rmi`：删除 image；若仍被 container 引用会失败；
- `docker system df`：查看 Docker storage（存储）占用；
- `docker container prune`：删除所有 stopped container，执行前应确认无需要保留的 container。
