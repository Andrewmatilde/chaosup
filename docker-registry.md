## Why Not Use Self-Signed TLS Certificates

### Background

When implementing HTTPS encryption, TLS certificates are central to ensuring secure communication. These certificates, issued by trusted Certificate Authorities (CAs), provide authentication and data protection for encrypted connections between servers and clients. While it's possible to generate self-signed TLS certificates, doing so is generally not recommended.

This article explores the limitations of self-signed TLS certificates and why using TLS certificates issued by trusted CAs is advised.

---

### 1. **Limitations of Self-Signed TLS Certificates**

#### 1.1 **Trust Issues**

- **Lack of Public Trust Chain**: Self-signed certificates cannot be validated through a public trust chain.
- **Security Warnings**: Browsers and clients will display security warnings for self-signed certificates, indicating that the connection is not secure. This negatively impacts user experience and may lead users to distrust your service.

#### 1.2 **Non-Compliance with Modern Standards**

- **Deprecation of the Common Name (CN) Field**:
  - Modern browsers and TLS clients no longer use the CN field for domain validation; they rely on the `Subject Alternative Name` (SAN) field instead.
  - When manually generating self-signed certificates, it's easy to overlook configuring the SAN field, causing certificates to be rejected by modern clients.

#### 1.3 **Complex and Error-Prone Manual Management**

- **Manual Distribution and Installation**:
  - Clients must manually add the self-signed certificate to their trusted store, which can lead to omissions, configuration errors, or broken trust chains.
- **Manual Renewal and Distribution**:
  - Managing certificate renewals and distribution manually increases complexity and the risk of security vulnerabilities.

#### 1.4 **Lack of Trust Chain and Auditing**

- **No Third-Party Validation**:
  - Self-signed certificates lack validation and endorsement from trusted third-party authorities, so they cannot provide a traceable trust chain.
- **Non-Compliance with Security Standards**:
  - In scenarios involving regulatory requirements or corporate compliance audits, the absence of a trusted certificate management mechanism may not meet necessary security standards.

---

### Configuring an Insecure Docker Registry for Pushing and Pulling Images

To push images from the master node and pull them from an insecure Docker Registry on virtual machine child nodes, follow these steps:

#### 1. Start an Insecure Docker Registry on the Master Node

On the master node, start a Docker Registry without TLS using the following command:

```bash
docker run -d \
  --restart=always \
  --name registry \
  -p 5000:5000 \
  registry:2
```

This command launches an insecure Docker Registry on port `5000` of the master node.

#### 2. Configure the Docker Daemon on the Master Node

To allow Docker to communicate with an insecure registry, configure the Docker daemon on the master node:

1. **Edit the Docker Daemon Configuration File**:

   Open the `/etc/docker/daemon.json` file for editing (create it if it doesn't exist):

   ```bash
   sudo nano /etc/docker/daemon.json
   ```

2. **Add the Insecure Registry Configuration**:

   Add the following content to the file:

   ```json
   {
     "insecure-registries": ["<MASTER_NODE_IP>:5000"]
   }
   ```

   Replace `<MASTER_NODE_IP>` with the actual IP address of your master node.

3. **Restart the Docker Service**:

   Save the file and restart the Docker service to apply the changes:

   ```bash
   sudo systemctl restart docker
   ```

#### 3. Push Images to the Insecure Registry on the Master Node

On the master node, push your local images to the insecure registry:

1. **Tag the Image**:

   Suppose you have a local image named `my-image:latest`. Tag it with the registry address:

   ```bash
   docker tag my-image:latest <MASTER_NODE_IP>:5000/my-image:latest
   ```

   Replace `<MASTER_NODE_IP>` with the actual IP address of your master node.

2. **Push the Image to the Registry**:

   Use the following command to push the image:

   ```bash
   docker push <MASTER_NODE_IP>:5000/my-image:latest
   ```

#### 4. Configure Containerd on the Virtual Machine Child Nodes

The configuration for `containerd` on the virtual machine child nodes is already set up in your `bootstrap.sh` script. Ensure that the child nodes are configured to allow pulling images from the insecure registry:

1. **Edit the Containerd Configuration File**:

   On each child node, edit the `/etc/containerd/config.toml` file:

   ```bash
   sudo nano /etc/containerd/config.toml
   ```

2. **Add Insecure Registry Settings**:

   Find the `[plugins."io.containerd.grpc.v1.cri".registry]` section and add the following:

   ```toml
   [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
     [plugins."io.containerd.grpc.v1.cri".registry.mirrors."<MASTER_NODE_IP>:5000"]
       endpoint = ["http://<MASTER_NODE_IP>:5000"]
   ```

   Replace `<MASTER_NODE_IP>` with the master node's IP address.

3. **Restart the Containerd Service**:

   Save the file and restart the containerd service:

   ```bash
   sudo systemctl restart containerd
   ```

Now, your virtual machine child nodes can pull images from the insecure Docker Registry running on the master node without encountering security errors.

---

By following these steps, you streamline the process of pushing and pulling Docker images in a development or testing environment where using trusted TLS certificates may not be practical. This setup allows for efficient image distribution while acknowledging the limitations and risks associated with using an insecure registry.