
### 📌 What We're Building

We’re setting up **HashiCorp Vault** on a **Kubernetes cluster** (via Kind), using **Consul** as the storage backend. Vault will be deployed with TLS and Kubernetes auth enabled, ready to inject secrets into workloads.

---

### 🧰 Prerequisites

| Tool                | Purpose                                                  |
| ------------------- | -------------------------------------------------------- |
| **Kubernetes 1.21** | Container orchestration platform                         |
| **Kind**            | Lightweight tool to run Kubernetes clusters in Docker    |
| **kubectl**         | CLI to interact with Kubernetes clusters                 |
| **Helm**            | Kubernetes package manager                               |
| **Docker**          | Container runtime used by Kind and for our CLI container |

---

### 🌀 Step 1: Set Up a Local Kubernetes Cluster (Kind)

We’ll create a Kubernetes cluster using [Kind](https://kind.sigs.k8s.io/), which runs clusters in Docker containers.

```bash
cd hashicorp/vault-2022

kind create cluster --name vault \
  --image kindest/node:v1.21.1 \
  --config kind.yaml
```

🧠 **Why?**
We're using Kind to quickly spin up a disposable, local K8s cluster for development and testing.

---

### 🧪 Step 2: Optional Dev Container for Tools

You can run a temporary Alpine container to install tools like `kubectl` and `helm` if you don’t want to pollute your local machine.

```bash
docker run -it --rm --net host \
  -v ${HOME}/.kube/:/root/.kube/ \
  -v ${PWD}:/work -w /work alpine sh
```

---

### 🧰 Step 3: Install kubectl and Helm (Inside Container)

#### 🧩 Install `kubectl`

```bash
apk add --no-cache curl
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
```

#### 📦 Install `helm`

```bash
curl -LO https://get.helm.sh/helm-v3.7.2-linux-amd64.tar.gz
tar -C /tmp/ -zxvf helm-v3.7.2-linux-amd64.tar.gz
mv /tmp/linux-amd64/helm /usr/local/bin/helm
chmod +x /usr/local/bin/helm
rm helm-v3.7.2-linux-amd64.tar.gz
```

---

### ✅ Step 4: Verify Cluster Access

```bash
kubectl get nodes
```

Expected output:

```
NAME                  STATUS   ROLES                  AGE   VERSION
vault-control-plane   Ready    control-plane,master   30s   v1.21.1
```

---

### 🔗 Step 5: Add Helm Repositories

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
```

🧠 **Why?**
This gives us access to pre-packaged charts for Vault and Consul maintained by HashiCorp.

---

### 💾 Step 6: Deploy Consul as Vault’s Storage Backend

#### 🧠 Why Consul?

Vault needs persistent storage for secrets and operational state. While it supports many backends (Raft, S3, Postgres), **Consul is the most mature and battle-tested** backend with HA support.

#### 📋 Choose a chart version:

```bash
helm search repo hashicorp/consul --versions
```

We’ll use version **0.39.0**.

#### 📁 Create Consul Manifest:

```bash
mkdir -p manifests

helm template consul hashicorp/consul \
  --namespace vault \
  --version 0.39.0 \
  -f consul-values.yaml \
  > ./manifests/consul.yaml
```

> `consul-values.yaml` contains custom settings for the deployment, e.g. disabling persistence or tweaking replicas.

#### 🚀 Deploy Consul to the Cluster:

```bash
kubectl create ns vault
kubectl -n vault apply -f ./manifests/consul.yaml
```

Check if pods are running:

```bash
kubectl -n vault get pods
```

---

### 🔒 Step 7: Set Up TLS for Vault

Vault needs TLS for secure communication.

#### 📜 Generate Certificates

Use the script in:

```bash
./tls/ssl_generate_self_signed.md
```

Or bring your own certs.

⚠️ Don’t check in `.pem` files into Git!

#### 🧱 Create Secrets in Kubernetes:

```bash
kubectl -n vault create secret tls tls-ca \
  --cert ./tls/ca.pem \
  --key ./tls/ca-key.pem

kubectl -n vault create secret tls tls-server \
  --cert ./tls/vault.pem \
  --key ./tls/vault-key.pem
```

---

### 🧭 Step 8: Deploy Vault

#### 🔍 Check available chart versions:

```bash
helm search repo hashicorp/vault --versions
```

We’ll use version **0.19.0**.

#### 📁 Generate Vault Manifest:

```bash
helm template vault hashicorp/vault \
  --namespace vault \
  --version 0.19.0 \
  -f vault-values.yaml \
  > ./manifests/vault.yaml
```

> `vault-values.yaml` controls configuration like HA, TLS usage, storage backend, etc.

#### 🚀 Apply Manifest:

```bash
kubectl -n vault apply -f ./manifests/vault.yaml
kubectl -n vault get pods
```

---

### 🧪 Step 9: Initialize and Unseal Vault

Vault must be initialized and unsealed manually the first time.

#### 🔁 Connect to Vault pods:

```bash
kubectl -n vault exec -it vault-0 -- sh
```

#### 🔐 Initialize Vault:

```bash
vault operator init
```

Save the unseal keys and root token securely.

#### 🔓 Unseal:

```bash
vault operator unseal
```

Repeat for other pods:

```bash
kubectl -n vault exec -it vault-1 -- sh
vault operator unseal

kubectl -n vault exec -it vault-2 -- sh
vault operator unseal
```

Check status:

```bash
vault status
```

---

### 🌐 Step 10: Access Vault Web UI

```bash
kubectl -n vault get svc
kubectl -n vault port-forward svc/vault-ui 443:8200
```

Access at:
➡️ [https://localhost:443](https://localhost:443)

Use your root token to log in.

---

### 🔐 Step 11: Enable Kubernetes Authentication

This allows Kubernetes apps to authenticate and fetch secrets from Vault.

```bash
kubectl -n vault exec -it vault-0 -- sh

vault login  # Use your root token

vault auth enable kubernetes

vault write auth/kubernetes/config \
  token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  issuer="https://kubernetes.default.svc.cluster.local"

exit
```

---

### ✅ Final Result

You now have:

* A running Vault cluster with TLS
* Consul as the backend
* Kubernetes auth enabled
* Helm-managed deployment
* Web UI access
* Secure foundation for injecting secrets into apps

---

### 📦 What's Next?

#### ➕ Secret Injection Examples

* **Static Secrets**: Manually store key-values and inject them into pods.
* **Dynamic Secrets**: Set up integrations (e.g. database or AWS IAM secrets).
* **Vault Agent Injector**: Auto-inject secrets into containers via annotations.

---
