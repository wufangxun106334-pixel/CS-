---
course: COMP47780 Cloud Computing
week: 3
type: practical-guide
topic: Building Docker Images
source: comp47780_practical2.pdf
semester: Autumn 2026
---

# Practical Two：Building Docker Images 作业过程

## 0. 作业性质与学习目标

这份 Practical（实践）不计分，PDF 明确说明 **there is nothing to submit（无需提交）**。但 Practical Three 会默认你已经掌握这里的内容。

完成后应能解释：

- `RUN` 与 `CMD` 的区别，以及它们分别何时执行。
- 如何在不 Rebuild（重新构建）Image（镜像）的情况下 Override（覆盖）默认命令。
- Dockerfile 指令顺序为何会影响 Build cache（构建缓存）。
- Dependency installation（依赖安装）与 Source code（源代码）复制的合理顺序。
- Containerized service（容器化服务）为什么必须在 Foreground（前台）运行。
- Build context（构建上下文）是什么，以及 `.dockerignore` 为什么重要。
- 如何 Build、Tag（标记）和 Push（推送）自己的 Image。

Prerequisite（前置要求）：已完成 Practical One，熟悉 `docker run`、`docker ps`、`docker exec`、Port publishing（端口发布），以及 Image name 与 Tag 的结构。

> 本指南整理题目流程和问题答案，不代替实际操作。命令应在 Docker Desktop / Docker Engine 正常运行后逐条执行。

---

# Exercise One：Your first Dockerfile

## 目标

第一次自行编写 Dockerfile，理解 `FROM`、`RUN`、`CMD`，完成 `build → inspect → run → override`。

## Task 1：创建目录和 Dockerfile

```bash
mkdir practical2-ex1
cd practical2-ex1
```

创建一个文件名恰好为 `Dockerfile` 的文件，不要加 `.txt`：

```dockerfile
FROM alpine:3.24
RUN apk add --no-cache git
CMD ["git", "--version"]
```

### 概念解释

- `FROM alpine:3.24`：选择 Base image（基础镜像）。Alpine 是 Minimal Linux distribution（精简 Linux 发行版）。
- `RUN apk add --no-cache git`：在 Build time（构建时）执行，安装 Git，并形成新的 Filesystem layer（文件系统层）。`apk` 是 Alpine 的 Package manager（包管理器）。
- `--no-cache`：不把 Package index（软件包索引）遗留在 Layer 中，减小 Image。
- `CMD ["git", "--version"]`：设置 Container start time（容器启动时）的默认命令；它不会在 Build 时运行。

## Task 2：Build 并 Tag

确保 Terminal（终端）的 Current working directory（当前目录）包含 Dockerfile：

```bash
docker build -t ex1:v1.0 .
```

参数含义：

- `-t ex1:v1.0`：把 Image 命名为 `ex1`，Tag 为 `v1.0`。
- 最后的 `.`：把当前目录作为 Build context。它不能省略。

确认 Image：

```bash
docker images ex1
```

预期看到 Repository 为 `ex1`、Tag 为 `v1.0`。

### 问题：Dockerfile 有三条指令，Image 有多少 Layers？哪条产生 Layer？

检查：

```bash
docker history ex1:v1.0
```

标准解释：

- Image 继承 Alpine 自身已有的 Layers。
- 你写的 `RUN apk add --no-cache git` 会改变 Filesystem，因此产生新的 Filesystem layer。
- `CMD` 主要写入 Image metadata（镜像元数据），通常在 `docker history` 中显示为 `0B` 的历史项，不新增有文件内容的 Layer。
- `FROM` 选择并继承 Base image，不是执行一次普通文件修改。
- “总共有几层”可能随 `alpine:3.24` 的具体 Image manifest（镜像清单）和 Docker 展示方式变化，应以本机 `docker history` 为准。重点不是死记数字，而是识别 Base layers 与 `RUN` 产生的 Layer。

## Task 3：运行和 Override CMD

```bash
docker run --rm ex1:v1.0
```

预期输出类似：

```text
git version 2.x.x
```

随后 Container 自动退出；`--rm` 会在退出后自动删除 Container。

覆盖默认命令：

```bash
docker run --rm ex1:v1.0 cat /etc/os-release
```

预期看到 Alpine 的发行版信息。

### 问题：为什么不用 Rebuild 就能执行不同命令？

Image name 后写出的命令 `cat /etc/os-release` 会 Override Dockerfile 的 `CMD`。`CMD` 是默认值，不是强制命令；用户可在 `docker run IMAGE COMMAND...` 中替换它。

## Task 4：进入 Interactive shell

```bash
docker run -it --rm ex1:v1.0 sh
```

进入 Container 后运行：

```sh
git --version
which git
exit
```

预期：`git --version` 输出版本；`which git` 输出 Git executable（可执行文件）路径，如 `/usr/bin/git`。

### `attach`、`exec` 与覆盖命令的区别

- `docker attach`：连接到已经作为 PID 1 运行的 Process（进程）的输入输出，不会新建 Shell。
- `docker exec`：在一个正在运行的 Container 中启动新命令。
- `docker run ... sh`：创建新 Container，并用 `sh` 替换默认 `CMD`。

### 常见错误

- Dockerfile 被保存为 `Dockerfile.txt`。
- 忘记 `docker build` 末尾的 `.`。
- 在不含 Dockerfile 的目录运行 `docker build`。
- 把 `RUN` 当成每次 Container 启动都会执行的命令。
- Container 已退出后尝试 `docker attach`。

---

# Exercise Two：Layers and the build cache

## 目标

理解 Layer cache（层缓存）、Cache invalidation（缓存失效），以及 Dockerfile 指令顺序为什么会显著影响 Rebuild time（重建时间）。这是本 Practical 最重要的 Exercise。

## Task 1：添加 message.txt 和 COPY

在同一目录创建 `message.txt`，写入任意文字。将 Dockerfile 改为：

```dockerfile
FROM alpine:3.24
RUN apk add --no-cache git
COPY message.txt /message.txt
CMD ["git", "--version"]
```

`COPY` 的两个路径含义不同：

```dockerfile
COPY message.txt /message.txt
```

- 第一个路径是 Source（源），相对于 Build context。
- 第二个路径是 Destination（目标），位于构建中的 Image filesystem。
- Source 不能跳出 Build context，因此 `COPY ../secrets.txt /` 会被拒绝。
- 目标路径以 `/` 结尾时，表示把文件放入该 Directory（目录）。

连续构建两次：

```bash
docker build -t ex1:v2.0 .
docker build -t ex1:v2.0 .
```

预期：第二次 Build 的步骤显示 `CACHED`，完成得非常快。

## Task 2：修改 message.txt 后 Rebuild

修改 `message.txt`，然后：

```bash
docker build -t ex1:v2.0 .
```

预期：

- `RUN apk add...` 仍命中 Cache，因为它位于变化的 `COPY` 之前。
- `COPY message.txt...` 重新执行。
- 该变化步骤之后的步骤会重新评估；`CMD` 是 Metadata instruction，通常很快。

### 问题答案：Docker 遵循什么规则？

一旦某条指令的输入发生变化导致 Cache miss（缓存未命中），从该条指令开始的后续构建链不能继续复用原来的 Cache chain；Docker 会重新处理该指令及其后续步骤。

## Task 3：把 COPY 放在 RUN 前面

改成：

```dockerfile
FROM alpine:3.24
COPY message.txt /message.txt
RUN apk add --no-cache git
CMD ["git", "--version"]
```

先 Build 一次，修改 `message.txt`，再 Build：

```bash
docker build -t ex1:v2.0 .
```

预期：由于 `COPY` 改变，后面的 `RUN apk add...` 也失去原 Cache，Git 被重新下载和安装。

### 问题：功能相同，Reordering（重新排序）损失了什么？

最终 Image 可以功能相同，但第二种顺序造成：

- 重复下载 Dependencies（依赖）。
- 重复执行耗时的安装步骤。
- 增加 Build time、网络流量和 CI 资源消耗。

### Dockerfile 排序的一般规则

> 把 Stable（稳定、很少变化）且 Expensive（昂贵、耗时）的步骤放前面；把 Volatile（频繁变化）的文件复制和步骤放后面。

真实 Application 常见顺序：

1. 先复制 Dependency manifest（依赖清单），如 `requirements.txt`。
2. 安装 Dependencies。
3. 再复制频繁变化的 Source code。

例如：

```dockerfile
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY . /app/
```

这样普通源代码变化不会强迫 Docker 重新安装全部 Dependencies。

### 常见错误

- 认为 Cache 只检查命令文字，不检查 `COPY` 的文件内容。
- 把 `COPY .` 放在依赖安装之前，导致每次改代码都重装依赖。
- 为追求少量 Dockerfile 行数，把无关操作随意合并，反而降低 Cache reuse（缓存复用）。

---

# Exercise Three：Building a real nginx image

## 目标

把 Static web page（静态网页）和 nginx 打包进自建 Image，理解 `EXPOSE`、Port publishing、PID 1 和 Foreground process。

在新目录工作：

```bash
mkdir ../practical2-nginx
cd ../practical2-nginx
```

## Task 1：创建 index.html

```html
<!DOCTYPE html>
<html>
  <head><title>Cloud Computing</title></head>
  <body>
    <h1>Hello from inside a container</h1>
    <p>Served by nginx, built from my own Dockerfile.</p>
  </body>
</html>
```

## Task 2：创建 nginx 配置 default

创建文件 `default`：

```nginx
server {
    listen 80 default_server;
    root /usr/share/nginx/html;
    index index.html;
}
```

它表示 nginx 在 Container 内监听 Port 80，并从 `/usr/share/nginx/html` 提供网页。

## Task 3：编写 Dockerfile

第一版按题意故意使用会 Daemonize（守护进程化）的 `CMD ["nginx"]`：

```dockerfile
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y nginx

COPY default /etc/nginx/sites-available/default
COPY index.html /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx"]
```

各项作用：

- `ENV DEBIAN_FRONTEND=noninteractive`：避免 `apt-get` 等待 Interactive input（交互输入）。
- `apt-get install -y`：自动回答 yes。
- `EXPOSE 80`：记录 Image 预期使用 Port 80；它不会自动把 Host port 发布出去。
- 真正的 Port publishing 仍由 `docker run -p HOST:CONTAINER` 完成。

### 问题：为什么 `apt-get update` 和 `apt-get install` 应在同一个 RUN？

如果分开：

```dockerfile
RUN apt-get update
RUN apt-get install -y nginx
```

Docker 可能复用旧的 `apt-get update` Cache，导致 `apt-get install` 使用 Stale package index（过期软件包索引）。合在同一 `RUN`：

```dockerfile
RUN apt-get update && apt-get install -y nginx
```

只要安装命令发生变化，该整步就会重新取得匹配的软件包索引再安装，避免索引与 Repository state（仓库状态）不一致。

进一步的 Best practice（最佳实践）通常还会在同一 Layer 删除 apt lists 以减小 Image：

```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends nginx \
 && rm -rf /var/lib/apt/lists/*
```

本行是解释性改进，不是 PDF 强制要求。

## Task 4：Build 和 Run

```bash
docker build -t mynginx:v1.0 .
docker run -d -p 8080:80 --name webserver mynginx:v1.0
```

- `-d`：Detached mode（后台模式）。
- `-p 8080:80`：Host 的 8080 映射到 Container 的 80。
- `--name webserver`：指定 Container name。

## Task 5：观察意外退出

```bash
docker ps
docker ps -a
docker logs webserver
```

预期现象：

- `docker ps` 中没有 `webserver`。
- `docker ps -a` 能看到它处于 Exited 状态。
- Logs 表明 nginx 启动过，但 Container 很快停止。

### 问题：nginx 明明启动成功，Container 为什么停止？

传统 Unix service（Unix 服务）会在启动后 Fork（派生）到 Background（后台），让原始 Foreground process 退出。Container 的 Lifetime（生命周期）与 PID 1 绑定：PID 1 结束，Docker 就认为 Container 工作结束，于是停止 Container。

`nginx -g 'daemon off;'` 禁止 nginx Daemonize，让它留在 Foreground 并作为 PID 1 持续运行。

## Task 6：修正 CMD

将最后一行改为：

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

注意：

- Exec form（执行形式）是 JSON array。
- `"daemon off;"` 是一个完整 Argument（参数），分号也在字符串内。
- 不能错误地拆成 `"daemon", "off;"`。

删除占用同名 Container 的旧实例，再重建运行：

```bash
docker rm -f webserver
docker build -t mynginx:v1.0 .
docker run -d -p 8080:80 --name webserver mynginx:v1.0
docker ps
```

验证：

```bash
curl http://localhost:8080
```

也可浏览器打开 `http://localhost:8080`。预期看到：

```text
Hello from inside a container
Served by nginx, built from my own Dockerfile.
```

### 常见错误

- 以为 `EXPOSE 80` 等同于 `-p 8080:80`。
- 把 Port mapping 写反；格式是 `HOST:CONTAINER`。
- 忘记删除已经占用 `webserver` 名称的停止 Container。
- 使用 `CMD ["nginx"]` 后只检查 `docker ps`，没有用 `docker ps -a` 和 `docker logs` Diagnose（诊断）。
- 把 `daemon off;` 拆成两个 Arguments。

---

# Exercise Four：Publish an image you built

## 目标

把自己的 Image 发布到 Docker Hub Registry（镜像注册中心），理解 Namespace（命名空间）、Tag 和 Content-addressed layers（内容寻址层）。

> `docker push` 会更改远端 Docker Hub 状态。PDF 要求实际执行；确认使用自己的 Docker Hub account 和 Repository namespace。

## Task 1：Login

```bash
docker login
```

按提示登录。使用 Personal access token（个人访问令牌）通常比直接输入密码更合适。

## Task 2：Build 并 Push v1.0

把 `<your-docker-username>` 替换为真实 Docker Hub username：

```bash
docker build -t <your-docker-username>/mynginx:v1.0 .
docker push <your-docker-username>/mynginx:v1.0
```

命名结构：

```text
namespace/repository:tag
```

## Task 3：修改页面并 Push v1.1

修改 `index.html` 中一行可见文字，然后：

```bash
docker build -t <your-docker-username>/mynginx:v1.1 .
docker push <your-docker-username>/mynginx:v1.1
```

### 问题：为什么第二次 Push 大多显示 `Layer already exists`？

Docker Layer 由内容 Digest（摘要）识别。v1.0 与 v1.1 共用相同的 Ubuntu Base layers、nginx 安装 Layer 和未变化的配置 Layer，因此 Registry 已有这些 Digests，不需重复上传。

修改 `index.html` 后，包含该文件的 `COPY index.html ...` Layer 内容发生变化；对应的新 Layer 需要上传。实际显示的上传项可能受 Dockerfile 顺序和 BuildKit 表现影响，但核心原则是：**只上传 Registry 尚未拥有的内容 Layer。**

### 常见错误

- `denied: requested access to the resource is denied`：Tag 中的 Namespace 不等于登录的 Docker Hub username，或尚未登录。
- 忘记把 `<your-docker-username>` 替换掉。
- 修改页面后仍使用旧 Tag，无法清晰表达 Version（版本）。
- 误以为 Push 新 Tag 会重新上传整个 Image。

检查 Tag：

```bash
docker images
```

---

# Exercise Five：Image size and the build context（Optional）

## 目标

比较合适 Base image 对 Image size 的影响，并理解 `COPY .`、Build context 与 `.dockerignore` 的性能和 Security（安全）意义。

## Task 1：使用官方 nginx:alpine

先比较现有 Image：

```bash
docker images mynginx
```

在新目录只放 `index.html` 和新的 Dockerfile：

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
	```

Build 并在 8081 运行：

```bash
docker build -t mynginx:alpine .
docker run -d -p 8081:80 --name webalpine mynginx:alpine
curl http://localhost:8081
```

### 问题：为什么没有 apt-get、配置文件和 CMD 仍能运行？

Official image（官方镜像）`nginx:alpine` 已经包含：

- nginx executable。
- 默认 nginx configuration。
- 默认 `ENTRYPOINT` / `CMD` 启动逻辑。
- 适合容器运行的 Foreground 配置。

你的 Dockerfile 只需覆盖网页内容。通常它明显小于 Ubuntu + 手动安装 nginx 的 Image，说明选择 Purpose-built base image（专用基础镜像）能减少 Size、Build time 和 Attack surface（攻击面）。实际大小以 `docker images mynginx` 为准。

## Task 2：改用 COPY .

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html/
```

这会把 Build context 中未被忽略的所有内容复制到 Web root（网站根目录）。

## Task 3：制造无关大文件并观察 Context

macOS / Linux：

```bash
mkdir .git
dd if=/dev/zero of=.git/objects.pack bs=1024 count=51200
dd if=/dev/zero of=debug.log bs=1024 count=20480
docker build -t mynginx:alpine .
```

Windows PowerShell：

```powershell
mkdir .git
fsutil file createnew .git\objects.pack 52428800
fsutil file createnew debug.log 20971520
docker build -t mynginx:alpine .
```

这会产生约 50 MB 假 Git 数据和 20 MB Log。记录 Build output 中 `transferring context` 的大小。

## Task 4：检查错误暴露的文件

```bash
docker rm -f webalpine
docker run -d -p 8081:80 --name webalpine mynginx:alpine
curl http://localhost:8081/Dockerfile
```

预期：服务器会返回 Dockerfile 内容，而不是 404。

### 问题：哪条指令把 Dockerfile 放进 Image？还放进了什么？

是：

```dockerfile
COPY . /usr/share/nginx/html/
```

它会把 Dockerfile、`.git/objects.pack`、`debug.log` 和目录中其他未忽略内容全部复制进 Web root，因此可能被访问。这既增加 Image size，也可能泄漏 Source history、Logs、Credentials（凭据）和 Secrets（秘密数据）。

## Task 5：添加 .dockerignore

创建与 Dockerfile 同级的 `.dockerignore`：

```dockerignore
.git
*.log
Dockerfile
.dockerignore
```

重新 Build：

```bash
docker build -t mynginx:alpine .
```

比较 `transferring context`，应显著缩小。重新启动：

```bash
docker rm -f webalpine
docker run -d -p 8081:80 --name webalpine mynginx:alpine
curl -i http://localhost:8081/Dockerfile
```

预期 HTTP status 为 `404 Not Found`。

### 问题：.dockerignore 改变了哪两个方面？

1. 匹配的文件不再传给 Builder（构建器），缩小 Build context，减少传输、Hashing（哈希计算）和误暴露风险。
2. 因为这些文件不在 Builder 可见的 Context 中，`COPY .` 也不会把它们放入 Image。

对于 Private keys（私钥），第一个效果最关键：文件根本不进入 Build context，也不会进入任何 Layer 或 Build cache。若先 `COPY` 再在后续 `RUN rm` 删除，Secret 仍保留在较早的 Immutable layer（不可变层）中，可能被恢复。

### BuildKit 注意事项

本实验用 `COPY .`，因此整个有效 Context 都是构建所需输入，加入大文件会明显影响 Context。现代 BuildKit 对只写 `COPY index.html ...` 的 Dockerfile 可能只传实际使用的文件，所以差异很小。这不改变 `.dockerignore` 对 `COPY .` 和 Secrets 的重要性。

### 常见错误

- `.dockerignore` 文件名写错或被保存成 `.dockerignore.txt`。
- `.dockerignore` 不在 Build context 根目录。
- 误以为 `.gitignore` 会自动对 Docker Build 生效。
- 只关注 Image size，忽略不必要文件被 Web server 公开的安全问题。
- 认为在后续 Layer 删除 Secret 就等同于从 Image history 中删除。

---

# Clean up：实验结束清理

先检查：

```bash
docker ps -a
```

删除本实验 Containers：

```bash
docker rm -f webserver webalpine
```

检查并删除本地 Images：

```bash
docker images
docker rmi ex1:v1.0 ex1:v2.0 mynginx:v1.0 mynginx:alpine
docker rmi <your-docker-username>/mynginx:v1.0 <your-docker-username>/mynginx:v1.1
docker system df
```

注意：

- 若 `docker rmi` 报 Image is in use，先用 `docker ps -a` 找到仍引用它的停止 Container 并删除。
- 删除 Local image（本地镜像）不会删除 Docker Hub 上的 Image。
- Exercise Four 推送的内容仍在 Docker Hub，除非进入 Docker Hub 删除对应 Repository 或 Tag。

---

# 需要提交什么？

PDF 明确说明：

> This practical is not graded and there is nothing to submit.

因此没有正式 Submission（提交物）。为了确认自己已经完成，建议保留以下 Evidence（证据），但这些不是 PDF 强制提交项：

- `docker history ex1:v1.0` 的输出。
- 第二次 Build 显示 `CACHED` 的输出。
- 改变 `message.txt` 前后 Cache 命中差异。
- `docker ps -a` 与 `docker logs webserver` 显示 nginx 第一版退出的原因。
- `curl http://localhost:8080` 返回自制网页。
- Docker Hub 中的 `v1.0` 和 `v1.1` Tags。
- `docker images mynginx` 的 Image size 对比。
- 添加 `.dockerignore` 前后的 `transferring context` 大小与 404 结果。

---

# 绿色问题答案速查

1. **哪些指令产生 Layer？** `RUN` 产生文件系统变化层；Base image 自带层；`CMD` 主要写 Metadata。
2. **运行时命令与 CMD 的关系？** Image name 后的命令会 Override `CMD`。
3. **修改 COPY 输入后的 Cache 规则？** 首个变化步骤 Cache miss，后续链重新处理。
4. **Dockerfile 排序原则？** 稳定且耗时的步骤靠前，频繁变化步骤靠后；先安装依赖，再复制应用源代码。
5. **为什么 update 与 install 同一 RUN？** 避免复用过期 Package index，并保持相关操作在同一 Cache unit（缓存单元）。
6. **nginx 为什么退出？** 它默认 Daemonize，使 PID 1 结束；`daemon off;` 让 nginx 前台运行。
7. **第二次 Push 为什么只传少量？** Registry 已有未变化 Layers 的 Digests，只上传内容改变的 Layer。
8. **nginx:alpine 为什么两行就能用？** Base image 已含 nginx、配置与启动命令。
9. **什么把无关文件放进 Web root？** `COPY . /usr/share/nginx/html/`。
10. **.dockerignore 的两个作用？** 减少传给 Builder 的 Context，并阻止匹配文件被 `COPY .` 放入 Image。
11. **为何不能后删 Secret？** Secret 已进入早期 Immutable layer，后续删除不能抹除历史 Layer。

# 最终自检

- [ ] 能区分 `RUN`、`CMD`、`ENTRYPOINT` 的执行阶段和作用。
- [ ] 能解释 Build context 与命令末尾 `.`。
- [ ] 能通过 `docker history` 查看 Layer history。
- [ ] 能说明 Cache invalidation 如何沿后续指令传播。
- [ ] 能编写能持续运行的 nginx Dockerfile。
- [ ] 能区分 `EXPOSE` 与 `-p`。
- [ ] 能 Build、Tag、Push 两个版本。
- [ ] 能说明 `.dockerignore` 的性能与 Security 价值。
- [ ] 已清理占用 8080、8081 的 Containers。
