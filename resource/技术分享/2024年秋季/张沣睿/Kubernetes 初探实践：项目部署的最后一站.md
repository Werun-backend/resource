**本文核心：**对于常见的 Go-Web 项目，分别给出 Docker-Compose 与 K8s 编排部署的一种**最佳实现方式，供实验室日后开发部署Golang项目进行准备**。本文只针对初识并要学习K8s的读者，只探索一种对小型项目最佳实践的方式，对为什么要用 Docker、Docker-Compose 与 K8s 不做解释，同时对原理部分并不进行深究。

**前置知识：**

- Docker 相关知识：https://zxlmdonnie.cn/archives/1706843165985
- 最基本的K8s理解 ：https://www.bilibili.com/video/BV1Se411r7vY

## 1. 问题背景

现在有一个单体博客系统（https://github.com/Werun-backend/bluebell-plus），技术栈选型如下：

- **项目名称：**bluebell_backend
- **Web框架：** Golang 1.14 + Gin 1.7.7
- **数据库：** MySQL 8.0.19 + Redis 5.0.7
- **操作数据库框架：** sqlx（MySQL）+ go-redis
- 雪花算法 zap日志库 Viper配置管理 swagger生成文档 令牌桶限流 Github热榜 畅言云评论系统……

### 1.1 任务目标

我们知道，运行 `go build bluebell_backend` 后即可编译出二进制文件之后运行，但这不满足跨平台的需要，我们需要虚拟化 Docker 的支持才能将其作为 Docker 容器在各个平台上运行。

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-yfgo.png)为了做到这一点，我们需要在项目根目录写一个 **Dockerfile** 以将项目打包成 Docker 镜像：

**Dockerfile：**

```Dockerfile
FROM golang:alpine AS builder

# 为我们的镜像设置必要的环境变量
ENV GO111MODULE=on \
GOPROXY=https://goproxy.cn,direct \
CGO_ENABLED=0 \
GOOS=linux \
GOARCH=amd64

# 移动到工作目录：/build
WORKDIR /build

# 复制项目中的 go.mod 和 go.sum文件并下载依赖信息
COPY go.mod .
COPY go.sum .
RUN go mod download

# 将代码复制到容器中
COPY . .

# 将我们的代码编译成二进制可执行文件 bubble_app
RUN go build -o bluebell_app .

###################
# 接下来创建一个小镜像
###################
FROM debian:stretch-slim
#FROM scratch

COPY ./wait-for.sh /
COPY ./templates /templates
COPY ./static /static
COPY ./conf /conf

# 从builder镜像中把可执行文件拷贝到当前目录
COPY --from=builder /build/bluebell_app /

# 声明服务端口
EXPOSE 8081

# 需要运行的命令
ENTRYPOINT ["/bluebell_app", "conf/config.yaml"]
```

然而打包成 Docker 镜像还不够，我们的项目中还有数据库等持久化存储的数据（在每个人的开发环境中不一致），并且可能用到分布式多实例高可用，只是打包成Docker，**里面访问数据库的配置信息**也不方便改变。

我们的目标如下：

- **开发阶段（或简单的生产环境）**，在追求简单的场景下，使用 Docker-Compose 编排项目
- **生产阶段**，在要求高可用的场景下，使用 Kubernetes 编排项目
- 尽可能使得开发机和部署机环境相似，**将配置信息在不同环境中尽可能保持一致**，同时使用各种工具简化开发过程



### 1.2 技术准备

本人是 Win11 系统，以下软件安装仅供参考：

- 本机安装 Docker Desktop（WSL2、Docker、Docker-Compose）
- Docker Hub / 华为云账号，用于托管镜像
- 本机安装 Goland 2023.3 以上（付费版），支持 Docker 即可
- 本机或远程安装 MySQL8.0 与 Redis5.0
- 安装K8s：最简单的方式是通过Docker Desktop安装，然后命令行就可以用 `kubectl` 了

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-mjqa.png)

### 1.3 镜像托管准备服务

这里以华为云为例，演示一下第三方镜像托管相关配置。

华为云容器镜像服务（SWR）：https://console.huaweicloud.com/swr

- 每个账户可以**免费**托管10个镜像
- 绝对放心的连接效率
- 不支持 Webhook，有触发器但是只支持把镜像部署到自家的k8s服务上

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-qteu.png)



首先，创建一个组织：


![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-qmjd.png)



然后按照“客户端上传”的方式将本地容器镜像推上去 / 拉下来，这里没有什么坑，照做就好。

注意把 docker login 的登录指令要保存下来，当然 Powershell 和 cmd 是不需要 sudo 的。

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-dgug.png)

图形化界面的配置如下即可：

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-jigq-1733832743447-14.png)

一般在生产中，都需要通过 API 来对镜像托管服务进行操作：https://support.huaweicloud.com/api-swr/swr_02_0065.html

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-daaw.png)

文档里也给出了各个语言（Java、Golang、Python等）的SDK。

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-yohv.png)



## 2. Docker-Compose 编排

### 2.1 方案说明

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-jtyg.png)

将 MySQL 与 Redis 都作为容器来编排，进行服务内部通信，既方便开发又保证环境统一。

- **服务结构**：
  - **bluebell-app**：前后端一体，容器内运行在 `8081`，外部通过端口映射 `8080:8081` 访问。
  - **MySQL**：容器名 `mysql8025`，端口映射为 `33061:3306`。
  - **Redis**：容器名 `redis507`，端口映射为 `23679:6379`。
- **服务连接方式**：
  - **容器内连接**：
    - `bluebell-app` 通过容器网络直接访问其他服务，MySQL 的地址为 `mysql8025:3306`，Redis 的地址为 `redis507:6379`。
    - 容器内通过服务名称解析（如 `mysql8025` 和 `redis507`）建立通信，无需关心主机端口。
  - **容器外访问**：
    - 外部工具如数据库客户端可通过主机的 `localhost:33061` 连接 MySQL，`localhost:23679` 连接 Redis。
- **通信流程**：
  - 用户请求通过 `http://localhost:8080` 访问 `bluebell-app`。
  - 应用通过容器网络直接连接 MySQL 和 Redis，分别处理持久化和缓存。

### 2.2 配置文件

docker-compose.yaml 文件与 Dockerfile 文件一样，放到项目根目录下。

```yaml
# yaml 配置
version: "3.7"
services:
  redis507:
    image: "redis:5.0.7"
    ports:
      - "26379:6379"  # 外部端口:内部端口
  mysql8019:
    image: "mysql:8.0.19"
    ports:
      - "33061:3306"
    command: "--default-authentication-plugin=mysql_native_password --init-file /data/application/init.sql"  # 初始化命令
    environment:
      MYSQL_ROOT_PASSWORD: "root"
      MYSQL_DATABASE: "bluebell-plus"
      MYSQL_PASSWORD: "root"
    volumes:
      - ./init.sql:/data/application/init.sql
  bluebell_app:
    build: .
    command: sh -c "./wait-for.sh redis507:6379 mysql8019:3306 -- ./bluebell_app ./conf/config.yaml"
    depends_on:
      - mysql8019
      - redis507
    ports:
      - "8081:8081"
```

数据库配置文件（注意 host 主机名）：

```yaml
mode: "development"
port: 8081
name: "MyApp"
version: "1.0.0"
start_time: "2024-01-01T00:00:00Z"
machine_id: 1
log:
  level: "info"
  filename: "app.log"
  max_size: 10
  max_age: 30
  max_backups: 5
mysql:
  host: "mysql8019"
  user: "root"
  password: "root"
  dbname: "bluebell-plus"
  port: 3306
  max_open_conns: 100
  max_idle_conns: 50
redis:
  host: "redis507"
  password: ""
  port: 6379
  db: 0
  pool_size: 10
  min_idle_conns: 5
```

### 2.3 实际开发

从 GitHub 拉取代码后，就可以在项目根目录（含有 docker-compose.yaml）下，运行：

```
docker-compose up -d
```

- **启动所有服务**：
  - 按照 YAML 文件中定义的服务（`redis507`、`mysql8019`、`bluebell_app`）逐一启动容器，并保证依赖关系正确（`bluebell_app` 会等待 `mysql8019` 和 `redis507` 启动成功后再启动）。
- **以守护模式运行**：
  - 参数 `-d` 表示 **Detached Mode（后台运行模式）**，运行完成后命令行会返回，而容器会在后台持续运行。
- **构建镜像**：
  - 对于定义了 `build: .` 的服务（如 `bluebell_app`），会先根据当前目录下的 Dockerfile 构建镜像。



## 3. Kubernetes 编排

### 3.1 K8s 架构：Node 和 pod

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-hxot.png)

Kubernetes 架构

K8s 架构可以分三部分：

1. **控制平面（Control Plane）**：负责管理和调度整个Kubernetes集群的资源和操作。
2. **工作节点（Nodes）**：执行容器化应用，由控制平面分配和管理。
3. **云服务提供商（Cloud Service Providers）**：提供基础设施支持，使得Kubernetes集群可以在云环境中运行。



下面来看 Node：

![img](http://60.205.137.2:8080/upload/tmpB604-tyup.png)

- **Node**：这是Kubernetes集群中的一个工作节点，可以是物理机或虚拟机，负责运行Pods。
- **svc (Service)**：服务是Kubernetes中的一个抽象，定义了一种访问Pod的方式。它允许外部流量通过一个固定的IP地址和端口访问Pod，即使Pod的IP地址发生变化。
- **pod**：Pod是Kubernetes中的基本部署单元，可以包含一个或多个容器。Pods是短暂的，它们可以被创建、销毁和替换。

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/tmpDCD8.png)

- 服务（svc）连接了不同节点上的Pods，使得它们可以通过服务的IP地址和端口进行通信。
- 服务可以负载均衡流量到后端的多个Pods，提高了应用的可用性和扩展性。

![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-mgss.png)

- **Deployment**：用于管理Pod的声明式更新，可以自动处理Pod的创建、删除和更新。
- **StatefulSet**：用于管理有状态应用，确保每个Pod都有一个唯一的网络标识和存储。

Kubernetes的StatefulSet可以用来管理需要持久化状态的应用程序，但通常不建议用它来部署数据库。原因包括：

1. **性能和资源管理限制**：Kubernetes可能无法提供数据库所需的最佳性能和资源管理。
2. **运维复杂性**：在Kubernetes上管理数据库会增加运维的难度。
3. **最佳实践**：更好的做法是将数据库部署在Kubernetes集群之外。



### 3.2 K8s 对象：声明式 API

> 建议直接阅读官方中文文档：https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/
>
> K8s 这官方文档写的是挺不错就是有点找不着……

**Kubernetes 对象** 是 Kubernetes 系统中持久化的实体，用于表示整个集群的状态。它们描述了以下信息：

- 哪些容器化应用正在运行（以及在哪些节点上运行）
- 可以被应用使用的资源
- 关于应用运行时行为的策略，比如重启策略、升级策略以及容错策略

**Kubernetes 对象** 是一种“意向表达（Record of Intent）”。一旦创建该对象，Kubernetes 系统将不断工作以确保该对象存在。通过创建对象，你本质上是在告知 Kubernetes 系统，你想要的集群工作负载状态看起来应是什么样子的，这就是 Kubernetes 集群所谓的期望状态（Desired State）。

操作 Kubernetes 对象——无论是创建、修改或者删除——需要使用 Kubernetes API。例如，当使用 kubectl 命令行接口（CLI）时，CLI 会调用必要的 Kubernetes API；也可以在程序中使用客户端库，来直接调用 Kubernetes API。

#### **3.2.1 对象规约（Spec）与状态（Status）**

几乎每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置：对象 spec（规约）和对象 status（状态）。

- **Spec**：对于具有 spec 的对象，你必须在创建对象时设置其内容，描述你希望对象所具有的特征：期望状态（Desired State）。
- **Status**：描述了对象的当前状态（Current State），它是由 Kubernetes 系统和组件设置并更新的。Kubernetes 控制平面都一直在积极地管理着对象的实际状态，以使之达成期望状态。

例如，Kubernetes 中的 Deployment 对象能够表示运行在集群中的应用。当创建 Deployment 时，你可能会设置 Deployment 的 spec，指定该应用要有 3 个副本运行。Kubernetes 系统读取 Deployment 的 spec，并启动我们所期望的应用的 3 个实例——更新状态以与规约相匹配。如果这些实例中有的失败了（一种状态变更），Kubernetes 系统会通过执行修正操作来响应 spec 和 status 间的不一致——意味着它会启动一个新的实例来替换。

#### **3.2.2 描述 Kubernetes 对象**

创建 Kubernetes 对象时，必须提供对象的 spec，用来描述该对象的期望状态，以及关于对象的一些基本信息（例如名称）。当使用 Kubernetes API 创建对象时（直接创建或经由 kubectl 创建），API 请求必须在请求主体中包含 JSON 格式的信息。大多数情况下，你会通过清单（Manifest）文件为 kubectl 提供这些信息。按照惯例，清单是 YAML 格式的（你也可以使用 JSON 格式）。像 kubectl 这样的工具在通过 HTTP 进行 API 请求时，会将清单中的信息转换为 JSON 或其他受支持的序列化格式。

**清单示例文件**：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # 告知 Deployment 运行 2 个与该模板匹配的 Pod
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

使用 kubectl 命令行接口（CLI）的 `kubectl apply` 命令，可以将 .yaml 文件作为参数：

```shell
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

输出类似下面这样：

```plaintext
deployment.apps/nginx-deployment created
```

#### **3.2.3 必需字段**

在想要创建的 Kubernetes 对象所对应的清单（YAML 或 JSON 文件）中，需要配置的字段如下：

- **apiVersion**：创建该对象所使用的 Kubernetes API 的版本。
- **kind**：想要创建的对象的类别。
- **metadata**：帮助唯一标识对象的一些数据，包括一个 name 字符串、UID 和可选的 namespace。
- **spec**：你所期望的该对象的状态。

对每个 Kubernetes 对象而言，其 spec 之精确格式都是不同的，包含了特定于该对象的嵌套字段。Kubernetes API 参考可以帮助你找到想要使用 Kubernetes 创建的所有对象的规约格式。

例如，参阅 Pod API 参考文档中 spec 字段。对于每个 Pod，其 `.spec` 字段设置了 Pod 及其期望状态（例如 Pod 中每个容器的容器镜像名称）。另一个对象规约的例子是 StatefulSet API 中的 spec 字段。对于 StatefulSet 而言，其 `.spec` 字段设置了 StatefulSet 及其期望状态。在 StatefulSet 的 `.spec` 内，有一个为 Pod 对象提供的模板。该模板描述了 StatefulSet 控制器为了满足 StatefulSet 规约而要创建的 Pod。不同类型的对象可以有不同的 `.status` 信息。API 参考页面给出了 `.status` 字段的详细结构，以及针对不同类型 API 对象的具体内容。





### 3.3 方案说明



![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-calc.png)

**Kubernetes集群（K8s）**：图片中间部分展示了Kubernetes集群的内部结构，包括：

- **[deploy] bluebell-app**：这是一个Deployment对象，用于管理Pod的生命周期和更新策略。
- **[pod] bluebell**：这是由Deployment创建的Pod，运行着应用程序，并且暴露了8081端口。
- **[svc] NodePort 30001:8081**：这是一个Service对象，类型为NodePort，它将外部的30001端口映射到Pod的8081端口，允许从集群外部访问Pod。
- **[svc] ExternalName mysql8019** 和 **[svc] ExternalName redis507**：这两个Service对象使用了ExternalName类型，它们分别指向MySQL和Redis服务的外部地址，允许Pod通过这些服务名访问数据库。



### 3.4 kubectl 操作

#### 3.4.1 创建对象 `kubectl apply`

Kubernetes 配置可以用 YAML 或 JSON 定义。可以使用的文件扩展名有 `.yaml`、`.yml` 和 `.json`。

```shell
kubectl apply -f ./my-manifest.yaml                  # 创建资源
kubectl apply -f ./my1.yaml -f ./my2.yaml            # 使用多个文件创建
kubectl apply -f ./dir                               # 基于目录下的所有清单文件创建资源
kubectl apply -f https://example.com/manifest.yaml   # 从 URL 中创建资源（注意：这是一个示例域名，不包含有效的清单）
kubectl create deployment nginx --image=nginx        # 启动单实例 nginx

kubectl explain pods                          # 获取 Pod 清单的文档说明

# 从标准输入创建多个 YAML 对象
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    args:
    - sleep
    - "1000"
EOF
```



#### 3.4.2 查看对象 `kubectl get`



```shell
# get 命令的基本输出
kubectl get services                          # 列出当前命名空间下的所有 Service
kubectl get pods --all-namespaces             # 列出所有命名空间下的全部的 Pod
kubectl get pods -o wide                      # 列出当前命名空间下的全部 Pod 并显示更详细的信息
kubectl get deployment my-dep                 # 列出某个特定的 Deployment
kubectl get pods                              # 列出当前命名空间下的全部 Pod
kubectl get pod my-pod -o yaml                # 获取一个 Pod 的 YAML

# describe 命令的详细输出
kubectl describe nodes my-node
kubectl describe pods my-pod
```



#### 3.4.3 查看 pod 日志 `kubectl logs`

```shell
kubectl logs my-pod                                 # 获取 Pod 日志（标准输出）
kubectl logs -l name=myLabel                        # 获取含 name=myLabel 标签的 Pod 的日志（标准输出）
kubectl logs my-pod --previous                      # 获取上个容器实例的 Pod 日志（标准输出）
kubectl logs -f my-pod                              # 流式输出 Pod 的日志（标准输出）
```



### 3.5 实际操作

假如我们已经做好了关于镜像服务托管的准备工作，我们就可以创建这几个对象了：

【Deployment 对象】bluebell-app-deployment.yaml 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bluebell-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bluebell-app
  template:
    metadata:
      labels:
        app: bluebell-app
    spec:
      containers:
        - name: bluebell-app
          image: swr.cn-north-4.myhuaweicloud.com/zxlm/bluebell_backend-bluebell_app:latest
      imagePullSecrets:
        - name: my-huawei-registry-secret  # 需要提前创建 Secret 以访问华为云镜像仓库
```

【Service ExternalName 对象】mysql0819-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql8019
spec:
  type: ExternalName
  externalName: host.docker.internal # 宿主机
  ports:
    - port: 3306
```

【Service ExternalName 对象】redis507-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis507  # 与 bluebell_app 的 redis 主机名保持一致
spec:
  type: ExternalName
  externalName: host.docker.internal
  ports:
    - port: 6379
```

【Service NodePort 对象】nodeport.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: your-service-name  # 名字忘改了……
spec:
  type: NodePort
  selector:
    app: bluebell-app
  ports:
    - protocol: TCP
      port: 30001
      targetPort: 8081
```

最后执行：

```shell
kubectl apply -f .\mysql0819-service.yaml
kubectl apply -f .\redis507-service.yaml
kubectl apply -f .\bluebell-app-deployment.yaml
kubectl apply -f .\nodeport.yaml
```



![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-vgsv.png)![img](./Kubernetes%20%E5%88%9D%E6%8E%A2%E5%AE%9E%E8%B7%B5%EF%BC%9A%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2%E7%9A%84%E6%9C%80%E5%90%8E%E4%B8%80%E7%AB%99/image-pswk.png)





## 4. 待改进之处

- 热更新 / 热部署：Docker镜像要手动删除
- 微服务场景下的 Docker-compose 和 k8s
- pods 连接 MySQL 和 Redis 的方式可能需要进一步改进
- 对 pods 的进一步编排和操作，当然，这就是云原生的故事了……



## 参考

1. **Kubernetes一小时轻松入门** https://www.bilibili.com/video/BV1Se411r7vY
2. **Kubernetes一小时入门课程 - 视频配套笔记** https://geekhour.net/2023/12/23/kubernetes/
3. **Kubernetes-bearsattack** http://bearsattack.top/archives/kubernetes
4. 官方中文文档 https://kubernetes.io/zh-cn/docs/reference/kubectl/quick-reference/



 