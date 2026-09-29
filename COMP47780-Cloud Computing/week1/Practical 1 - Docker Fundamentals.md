---
course: COMP47780 Cloud Computing
week: 1
topic: Practical 1 - Docker
---

你说得对，刚才我把 `Docker image` 说成 image，跑偏了。下面直接按“你现在坐在 terminal 前，应该做什么、为什么做、绿色题怎么答”来讲。

这不是传统“交作业”的 practical。它要你学会：**Docker 如何下载程序、启动程序、让程序对外提供网页、让多个程序通信、再上传到 Docker Hub。** 不用提交文件，但绿色 `QUESTION` 是以后可能会问你的概念。

## 你先把 Docker 想成什么？

假设你写了一个网站。它可能需要 Python、Node.js、某个 package、特定 Linux setting。直接把代码给同学，他电脑上常常跑不起来。

`Docker image`（镜像）就是把“网站 + 运行环境 + dependencies”封装成一个可复用模板。

`Docker container`（容器）就是“根据这个模板启动出来的一次实际运行”。

类比：

- image = 游戏安装包；
- container = 你当前打开、正在玩的游戏进程；
- 删除 container = 关掉并卸载这次运行；
- image 还在 = 安装包还在，以后可再次启动。

这也是 Cloud Computing 里常见的 `deployment`（部署）方式：同一 image 可在 laptop、server、cloud 上稳定运行。

---

# Exercise Zero：先让 Docker 能运行

## 安装后必须验证

```bash
docker version
docker run hello-world
```

第一条检查 Docker 的两个部分：

- `Client`：你输入命令的程序；
- `Server` / `Docker daemon`：真正创建和管理 container 的后台服务。

如果没有 `Server`，说明 Docker Desktop 没启动。Mac 上打开 Docker Desktop，等图标显示 Docker 已运行后再试。

第二条：

```bash
docker run hello-world
```

虽然你没有先下载 `hello-world`，Docker 会自动：

1. 在本地找 image；
2. 找不到就去 Docker Hub 下载；
3. 创建并启动 container；
4. container 输出欢迎文字后退出。

这是后面所有 `docker run` 的基本逻辑。

---

# Exercise One：image 和 container 到底区别是什么？

## Task 1：为什么 `hello-world` 不见了？

运行：

```bash
docker ps
docker ps -a
```

- `docker ps`：只显示正在运行的 container。
- `docker ps -a`：显示所有 container，包括已经退出的。

你会看到 `hello-world` 出现在第二条中。

绿色题可以这样回答：

> `hello-world` 的 main process（主进程）打印完消息就结束，因此 container 也变成 exited。Container 不像 VM 一样会持续开机；它通常只在主进程运行期间存在于 running state（运行状态）。

关键点：container 不是“一个小电脑一直开着”，而是“一个隔离的进程运行环境”。

## Task 2：下载网页服务器并启动它

```bash
docker pull docker/welcome-to-docker
docker images
docker run -d -p 8080:80 --name welcome docker/welcome-to-docker
docker ps
```

打开：

```text
http://localhost:8080
```

逐个看：

- `docker run`：创建并启动 container；
- `-d`：detached（后台运行），否则 terminal 会一直被日志占住；
- `-p 8080:80`：把你电脑的 `8080` 转给 container 内的 `80`；
- `--name welcome`：把难记的 container ID 改成名字 `welcome`；
- 最后一段是 image 名称。

## Task 3：删除 container 后，为什么 image 还在？

```bash
docker stop welcome
docker rm welcome
docker images
```

这里要观察：网页没了、container 没了，但 `docker/welcome-to-docker` image 还在。

绿色题的一句标准答案：

> An image is a reusable read-only template, while a container is one runnable instance created from that template; deleting the container does not delete the image.

## Task 4：为什么没 `pull` 也能启动？

```bash
docker rmi docker/welcome-to-docker
docker run -d -p 8080:80 --name welcome docker/welcome-to-docker
```

你刚才删除了 image，却又直接 `run`。Docker 发现本地没有它，所以自动 pull。

因此 `docker run` 可以包含三步：

```text
find local image → pull if absent → create and start container
```

---

# Exercise Two：port、隔离和数据消失

## Task 1：为什么 `8080` 会冲突？

先启动第二个 container：

```bash
docker run -d -p 8081:80 --name welcome2 docker/welcome-to-docker
```

现在：

- `localhost:8080` → 第一个 container；
- `localhost:8081` → 第二个 container。

两个 container 内部都可以用 port `80`。然后故意执行：

```bash
docker run -d -p 8080:80 --name welcome3 docker/welcome-to-docker
```

这会失败，因为 host port `8080` 已被第一个 container 占用。

绿色题的重点：

```text
-p 8080:80
   ↑    ↑
 host  container
```

每个 container 都有自己的 `network namespace`（隔离网络空间），因此它们内部可以各自使用 `80`。但你电脑上的 `8080` 是共享入口，不能被两个 mapping 同时占用。

## Task 2：为什么 container 内的文件会消失？

进入运行中的 container：

```bash
docker exec -it welcome sh
```

这里不是创建新 container，而是进入现有 `welcome`。

- `-i`：保持输入；
- `-t`：提供互动式 terminal；
- `sh`：在里面启动 shell。

在里面运行：

```bash
echo "hello from inside" > /tmp/practical1.txt
cat /tmp/practical1.txt
exit
```

你已经确认文件存在。然后在 host terminal：

```bash
docker rm -f welcome
docker run -d -p 8080:80 --name welcome docker/welcome-to-docker
docker exec welcome cat /tmp/practical1.txt
```

最后会报找不到文件。

原因不是 Docker “忘了”文件：文件是写在旧 container 的 writable layer（可写层）中；`docker rm` 把整个 container 删除了。新 container 虽然来自同一 image，却是全新的实例。

要保存数据，需要：

- `volume`（Docker 管理的持久化存储），或
- `bind mount`（把 host 文件夹挂进 container）。

这就是为什么数据库 container 在 cloud 中不能只把数据写进 container 自己的文件系统。

## Task 3：为什么 Mac/Windows 里会出现 Linux？

```bash
uname -a
docker run --rm alpine uname -a
```

- 第一条：查看你的 host kernel；
- 第二条：查看 container 所使用的 kernel；
- `--rm`：命令结束自动删除 container。

在 Linux，两个结果基本相同：container 直接共享 Linux host kernel。

在 macOS / Windows，第二条会显示 Linux kernel。因为 Docker containers 需要 Linux kernel feature，所以 Docker Desktop 在后台运行一个轻量 Linux VM。你的 container 实际在这台隐藏 Linux VM 内运行。

---

# Exercise Three：让 containers 互相访问

先清理旧资源：

```bash
docker rm -f welcome welcome2 welcome3
docker rmi hello-world docker/welcome-to-docker
```

## Task 1：BusyBox 的 IP 为什么不同？

```bash
docker run -it --rm busybox
ifconfig
```

BusyBox 是极小 Linux image。它看到的通常是 Docker 分配的私有 IP，例如 `172.x.x.x`，不是你电脑 Wi-Fi 的 IP。

这证明 container 有独立 network namespace，不是直接等于 host。

## Task 2：为什么 `web` 可以当网址？

创建一个自定义网络：

```bash
docker network create labnet
docker run -d --network labnet --name web -p 8080:80 nginx
```

然后启动同网络的 BusyBox：

```bash
docker run -it --rm --network labnet --name webtest busybox
wget -O - http://web:80
```

你应看到 nginx HTML。

重点不是 `wget`，而是这里：

```text
http://web:80
       ↑
  container name
```

Docker 在 `labnet` 这个 user-defined network（自定义网络）里提供 embedded DNS（内置 DNS）。它把名字 `web` 自动解析为 nginx container 的 IP。

为什么要自定义 network？

- 在默认 `bridge` 网络中，container 通常不能可靠地按 name 自动发现彼此；
- 自定义 network 适合多 container app，例如：

```text
frontend → api → database
```

它们可用 `api`、`database` 作为 hostname，而不用写死 IP。

再把 nginx 重启成不含 `-p 8080:80` 的版本后：

- browser 的 `localhost:8080` 会失效；
- BusyBox 的 `http://web:80` 仍然可用。

因为：

- `-p` 是把服务发布给 host/browser；
- 同 Docker network 内部通信不需要 `-p`。

---

# Exercise Four：tag、Docker Hub、上传 image

## Task 1：`nginx` 和 `nginx:alpine` 有什么不同？

```bash
docker pull nginx
docker pull nginx:alpine
docker images nginx
```

- `nginx` 实际通常是 `nginx:latest`；
- `nginx:alpine` 基于 Alpine Linux，系统更小，因此 image 更小。

最容易错的是：`latest` 不等于“永远最新版本”。

`latest` 只是一个 tag（标签）。发布者可以随时把它指向不同版本，所以生产环境通常更适合写明确版本，例如：

```text
nginx:1.27.5
```

或使用 immutable digest（不可变内容摘要）。

## Task 2：为什么要重新 tag？

登录：

```bash
docker login
```

然后：

```bash
docker tag nginx:alpine <你的Docker用户名>/comp-nginx:v1.0
```

例如 username 是 `alex123`：

```bash
docker tag nginx:alpine alex123/comp-nginx:v1.0
```

这不是复制 image。两个名字会有相同 `IMAGE ID`。

`tag` 做的是给现有 image 增加一个新名字，类似给同一个文件加 reference（引用）。

上传必须使用你自己的 namespace（命名空间），即 Docker Hub username；你不能 push 到官方的 `library/nginx`。

## Task 3：上传并理解 layer reuse

```bash
docker push <你的Docker用户名>/comp-nginx:v1.0
docker pull <同学用户名>/comp-nginx:v1.0
```

如果同学上传的是同一个 nginx:alpine 内容，pull 时很多 layer 会显示 `Already exists`。

这是因为 Docker image 按 layer 内容的 digest 识别。内容完全一样就不重新传输，直接复用本地 layer。这是 container 在 cloud 中高效分发的重要原因。

## 最后清理

```bash
docker system df
docker container prune
docker rmi nginx nginx:alpine
```

`docker system df` 看 Docker 占了多少 storage。`docker container prune` 会删除所有 stopped containers；确认无重要内容再执行。

这份 practical 真正要你掌握的结论只有五个：

1. `image` 是模板，`container` 是运行实例。
2. `docker run` 会在必要时自动 pull。
3. `-p host:container` 决定 host 如何访问 container。
4. container 内数据默认不持久，需 volume 或 mount。
5. 自定义 Docker network 让 containers 可通过名字发现彼此。
