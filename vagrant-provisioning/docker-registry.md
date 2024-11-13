## 为什么不使用自签 TLS 证书

### 背景

在实现 HTTPS 加密时，TLS 证书是确保通信安全的核心。TLS 证书由可信任的证书颁发机构（Certificate Authority, CA）签发，为服务器和客户端之间的加密连接提供身份验证和数据保护。虽然可以通过自签名的方式生成 TLS 证书（简称自签证书），但在大多数情况下，这种做法并不可取。

本文将探讨自签 TLS 的局限性以及为什么推荐使用由受信任 CA 签发的 TLS 证书。

---

### 1. **自签 TLS 的局限性**

#### 1.1 **信任问题**
- 自签证书无法通过公共信任链验证。
- 浏览器和客户端会对自签证书显示安全警告，提示用户连接不安全。这对用户体验极为不利，可能导致用户对服务的不信任。

#### 1.2 **不符合现代浏览器和客户端标准**
- **通用名称（CN）字段的废弃**：
  - 现代浏览器和 TLS 客户端已经不再使用 CN 字段进行域名验证，而是依赖于 `Subject Alternative Name`（SAN）字段。
  - 手动生成自签证书时，可能容易忽略 SAN 字段配置，从而导致证书被现代客户端拒绝。

#### 1.3 **手动管理复杂且容易出错**
- 自签证书需要手动分发和安装：
  - 客户端必须手动添加信任，容易导致漏装、配置错误或信任链中断。
- 手动续期和分发证书可能增加管理复杂度，并提高出现安全漏洞的风险。

#### 1.4 **缺乏信任链和审计**
- 自签证书没有第三方机构的验证和背书，因此无法提供可追溯的信任链。
- 在某些场景下（如法规要求或企业内部合规性审核），缺乏可信赖的证书管理机制可能不符合安全规范。

要在主节点上推送镜像，并在虚拟机子节点上从不安全的 Docker Registry 拉取镜像，您可以按照以下步骤进行配置：

### 1. 在主节点上启动不安全的 Docker Registry

在主节点上，使用以下命令启动一个不使用 TLS 的 Docker Registry：

```bash
docker run -d \
  --restart=always \
  --name registry \
  -p 5000:5000 \
  registry:2
```

此命令将在主节点的 5000 端口启动一个不安全的 Docker Registry。

### 2. 配置主节点的 Docker Daemon

为了允许 Docker 与不安全的 Registry 通信，需要在主节点上配置 Docker Daemon：

1. **编辑 Docker Daemon 配置文件**：

   在主节点上，编辑 `/etc/docker/daemon.json` 文件（如果不存在，请创建）：

   ```bash
   sudo nano /etc/docker/daemon.json
   ```

2. **添加不安全的 Registry 配置**：

   在文件中添加以下内容：

   ```json
   {
     "insecure-registries": ["<主节点IP>:5000"]
   }
   ```

   将 `<主节点IP>` 替换为主节点的实际 IP 地址。

3. **重启 Docker 服务**：

   保存文件后，重启 Docker 服务以应用更改：

   ```bash
   sudo systemctl restart docker
   ```

### 3. 在主节点上推送镜像到不安全的 Registry

在主节点上，按照以下步骤将本地镜像推送到不安全的 Registry：

1. **为镜像打标签**：

   假设您有一个名为 `my-image:latest` 的本地镜像，使用以下命令为其打标签：

   ```bash
   docker tag my-image:latest <主节点IP>:5000/my-image:latest
   ```

   将 `<主节点IP>` 替换为主节点的实际 IP 地址。

2. **推送镜像到 Registry**：

   使用以下命令将镜像推送到不安全的 Registry：

   ```bash
   docker push <主节点IP>:5000/my-image:latest
   ```

### 3. 在虚拟机子节点上配置 ~~Containerd | cri-o

在 bootstrap.sh 里已经有配置