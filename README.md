# 🚀 Ansible Cluster in Docker Containers & Kubernetes Pods

---

## 🧭 Overview

This project covers:

- ✅ Building a Docker image for Ansible & SSH
- ✅ Running Ansible Master and Nodes as Docker containers
- ✅ Establishing SSH trust between Master and Nodes
- ✅ Deploying the same setup on Kubernetes Pods
- ✅ Running automation tasks using `ansible -m ping`

---

## 🔹 Phase 1: Ansible Cluster using Docker Containers

### 🛠️ Step 1: Create a Custom Docker Image

We begin with a base Ubuntu image and manually install required packages:

```dockerfile
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && \
    apt install -y python3 python3-pip ssh sudo ansible iputils-ping && \
    apt clean

# Create ansible user
RUN useradd -m ansible && \
    echo "ansible:ansible" | chpasswd && \
    usermod -aG sudo ansible && \
    mkdir -p /home/ansible/.ssh && \
    chmod 700 /home/ansible/.ssh && \
    chown -R ansible:ansible /home/ansible/.ssh

# SSH server setup
RUN mkdir /var/run/sshd && \
    echo "PermitRootLogin yes" >> /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
````

---

### 🧱 Step 2: Build & Run Containers

```bash
docker build -t ansible-base .
docker run -dit --name master ansible-base
docker run -dit --name node1 ansible-base
docker run -dit --name node2 ansible-base
```

---

### 🔐 Step 3: SSH Setup from Master to Nodes

```bash
docker exec -it master bash
su - ansible
ssh-keygen -t rsa
```

Find container IPs:

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' node1
```

Copy SSH key:

```bash
ssh-copy-id ansible@<node1-ip>
ssh-copy-id ansible@<node2-ip>
```

---

### 📁 Step 4: Ansible Inventory & Ping Test

Create `inventory.ini`:

```ini
[nodes]
node1 ansible_host=<node1-ip> ansible_user=ansible
node2 ansible_host=<node2-ip> ansible_user=ansible
```

Test connectivity:

```bash
ansible -i inventory.ini nodes -m ping
```

---

## ☸️ Phase 2: Ansible Cluster using Kubernetes Pods

### 🛠️ Step 1: Push Docker Image to Docker Hub

```bash
docker tag ansible-base yourdockerhub/ansible-k8s
docker push yourdockerhub/ansible-k8s
```

---

### 🧾 Step 2: Pod YAML Files

**master.yaml**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: master
spec:
  containers:
    - name: master
      image: yourdockerhub/ansible-k8s
      ports:
        - containerPort: 22
      securityContext:
        privileged: true
```

Create similar files for `node1.yaml` and `node2.yaml` by changing `metadata.name`.

---

### 📡 Step 3: SSH from Master Pod

```bash
kubectl exec -it master -- bash
su - ansible
ssh-keygen -t rsa
```

List pod IPs:

```bash
kubectl get pods -o wide
```

Copy SSH key:

```bash
ssh-copy-id ansible@<node1-pod-ip>
ssh-copy-id ansible@<node2-pod-ip>
```

---

### 📁 Step 4: Inventory & Ping

```ini
[nodes]
node1 ansible_host=<node1-pod-ip> ansible_user=ansible
node2 ansible_host=<node2-pod-ip> ansible_user=ansible
```

Run the ping test:

```bash
ansible -i inventory.ini nodes -m ping
```

---

## ✅ Final Result

🎉 You now have a fully working Ansible cluster:

* 🐳 Inside Docker containers
* ☸️ Inside Kubernetes Pods
* 🔧 Built from scratch using manual setup

---

## 📎 Project Repository

🔗 GitHub: [Ansible Cluster in Docker & Kubernetes](https://github.com/abhi-gadge1773/Ansible-Cluster-in-Docker-Containers-Kubernetes-Pods.git)

---

## 📧 Contact

* **Name**: Abhijeet Gadge
* **Email**: [abhijeetgadge100@gmail.com](mailto:abhijeetgadge100@gmail.com)
* **GitHub**: [abhi-gadge1773](https://github.com/abhi-gadge1773)
* **LinkedIn**: [linkedin.com/in/abhijeetgadge](https://www.linkedin.com/in/abhijeetgadge/)



