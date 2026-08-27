# ☸️ Kubernetes — Complete Notes

> A comprehensive guide covering Kubernetes architecture, components, workloads, networking, storage, security, and tooling.

---

## 📋 Table of Contents

- [What is Kubernetes?](#what-is-kubernetes)
- [Cluster Architecture](#cluster-architecture)
  - [Control Plane Components](#control-plane-components)
  - [Worker Node Components](#worker-node-components)
  - [Core Components](#core-components)
- [Local Setup — Minikube](#local-setup--minikube)
- [kubectl](#kubectl)
- [eksctl & EKS Cluster Setup](#eksctl--eks-cluster-setup)
- [Kubernetes Workloads](#kubernetes-workloads)
  - [Pods](#pods)
- [Container Runtime Interface (CRI)](#container-runtime-interface-cri)
- [Kubernetes Services](#kubernetes-services)
- [Horizontal Pod Autoscaler (HPA)](#horizontal-pod-autoscaler-hpa)
- [Metrics Server](#metrics-server)
- [Ingress](#ingress)
- [RBAC](#rbac)
  - [Role](#role)
  - [RoleBinding](#rolebinding)
  - [ClusterRole and ClusterRoleBinding](#clusterrole-and-clusterrolebinding)
  - [Custom Permissions Commands](#custom-permissions-commands)
- [Service Accounts](#service-accounts)
- [Pod-to-AWS Communication (IRSA)](#pod-to-aws-communication-irsa)
- [Resource Requests and Limits](#resource-requests-and-limits)
- [Image Pull Policy](#image-pull-policy)
- [Scheduling](#scheduling)
  - [1. NodeSelector](#1-nodeselector)
  - [2. Node Affinity](#2-node-affinity)
  - [3. DaemonSet](#3-daemonset)
  - [4. Taints and Tolerations](#4-taints-and-tolerations)
  - [Node Drain](#node-drain)
- [Volumes and Storage](#volumes-and-storage)
- [Grafana and Prometheus](#grafana-and-prometheus)
- [Helm Charts](#helm-charts)
- [ArgoCD](#argocd)
- [StatefulSet](#statefulset)
- [Headless Service](#headless-service)
- [Probes](#probes)
  - [Startup Probe](#startup-probe)
  - [Readiness Probe](#readiness-probe)
  - [Liveness Probe](#liveness-probe)
- [EFK Stack](#efk-stack)

---

## What is Kubernetes?

Kubernetes is a container orchestration system that was initially designed by Google to help scale containerized applications in the cloud. Kubernetes can manage the lifecycle of containers, creating and destroying them depending on the needs of the application, as well as providing a host of other features.

Kubernetes has become one of the most discussed concepts in cloud-based application development, and the rise of Kubernetes signals a shift in the way that applications are developed and deployed.

In general, Kubernetes is formed by a cluster of servers, called Nodes, each running Kubernetes agent processes and communicating with one another. The Master Node contains a collection of processes called the control plane that helps enact and maintain the desired state of the Kubernetes cluster, while Worker Nodes are responsible for running the containers that form your applications and services.

### A Kubernetes cluster is composed of two separate planes:

- **Kubernetes control plane** — manages Kubernetes clusters and the workloads running on them. Includes components like the API Server, etcd, Scheduler, and Controller Manager.
- **Kubernetes Worker node** — runs containerized workloads. Each node is managed by the kubelet, an agent that receives commands from the control plane.

---

## Cluster Architecture

### Control Plane Components

#### API Server (control plane)

Provides an API that serves as the front end of a Kubernetes control plane. It is responsible for handling external and internal requests — determining whether a request is valid and then processing it. The API can be accessed via the `kubectl` command-line interface or other tools like `kubeadm`, and via REST calls.

#### Scheduler

This component is responsible for scheduling pods on specific nodes according to automated workflows and user defined conditions, which can include resource requests.

#### etcd

A key-value database that contains data about your cluster state and configuration. Etcd is fault tolerant and distributed.

#### Controller

It receives information about the current state of the cluster and objects within it, and sends instructions to move the cluster towards the cluster operator's desired state.

---

### Worker Node Components

#### kubelet

Each node contains a kubelet, which is a small application that can communicate with the Kubernetes control plane. The kubelet is responsible for ensuring that containers specified in pod configuration are running on a specific node, and manages their lifecycle. It executes the actions commanded by your control plane.

#### Kube Proxy

All compute nodes contain kube-proxy, a network proxy that facilitates Kubernetes networking services. It handles all network communications outside and inside the cluster, forwarding traffic or replying on the packet filtering layer of the operating system.

#### Container Runtime Engine

Each node comes with a container runtime engine, which is responsible for running containers. Docker is a popular container runtime engine, but Kubernetes supports other runtimes that are compliant with Open Container Initiative, including CRI-O and rkt.

---

### Core Components

#### Nodes

Nodes are physical or virtual machines that can run pods as part of a Kubernetes cluster. A cluster can scale up to **5000 nodes**. To scale a cluster's capacity, you can add more nodes.

#### Pod

Pods are the smallest unit provided by Kubernetes to manage containerized workloads. A pod typically includes several containers, which together form a functional unit or microservice.

---

## Local Setup — Minikube

Minikube is a tool that sets up a Kubernetes environment on a local PC or laptop. Minikube quickly sets up a local Kubernetes cluster on macOS, Linux, and Windows. It focuses on helping application developers and new Kubernetes users.

### Installation Process — Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

```bash
minikube start          # to start minikube
minikube status         # to check status
minikube update-context # to update
```

---

## kubectl

`kubectl` is the Kubernetes-specific command line tool that lets you communicate and control Kubernetes clusters. Whether you're creating, managing, or deleting resources on your Kubernetes platform, kubectl is an essential tool.

### kubectl Installation

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s \
  https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

---

## eksctl & EKS Cluster Setup

### eksctl Installation

a. Download and extract the latest release
b. Move the extracted binary to `/usr/local/bin`
c. Test that your eksctl installation was successful

```bash
curl --silent --location \
  "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
  | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### EKS Cluster Setup Process

#### Create an IAM Role and attach it to EC2 instance

> **Note:** Create IAM user with programmatic access if your bootstrap system is outside of AWS.

IAM user should have access to:
- IAM
- EC2
- VPC
- CloudFormation

#### Create your cluster and nodes

```bash
eksctl create cluster --name cluster-name  \
  --region region-name \
  --node-type instance-type \
  --nodes-min 2 \
  --nodes-max 2 \
  --zones <AZ-1>,<AZ-2>
```

**Example:**

```bash
eksctl create cluster --name test \
  --region us-east-1 \
  --node-type t2.medium
```

#### After creating cluster — update kubeconfig

```bash
aws eks update-kubeconfig --region <region> --name <cluster name>

# Example:
aws eks update-kubeconfig --region ap-south-1 --name naresh
```

#### To delete the EKS cluster

```bash
eksctl delete cluster naresh --region ap-south-1
```

---

## Kubernetes Workloads

### Imperative

```bash
kubectl run pod --image nginx
```

### Pods

```bash
kubectl apply -f pod.yaml

kubectl get pods

kubectl get pods -o wide    # to see wider output

kubectl delete pod <pod name>

kubectl describe pods <name of the pod>  # to know more details about pod

kubectl exec <pod-name> -it -- /bin/sh
```

### Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
      app: webapp
      type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

---

## Container Runtime Interface (CRI)

CRI (Container Runtime Interface) is an API layer that allows Kubernetes to talk to container runtimes. It defines how the kubelet communicates with the container runtime to start, stop, and manage containers.

> **Note:** Since Kubernetes no longer uses Docker directly (after v1.24), the `docker ps` command won't work unless you're explicitly using Docker (`cri-dockerd`) as your container runtime.

### Install crictl

```bash
VERSION="v1.30.0"
curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
```

```bash
sudo crictl ps   # to check only container details
kubectl get pods # to check pod details
```

### Runtime Architecture

**Below v1.24:**

```
Kubernetes (kubelet)
     ↓
  dockershim (translator)
     ↓
  Docker Engine
     ↓
  containerd → runc
```

**After v1.24:**

```
Kubernetes (kubelet)
     ↓
  CRI (native)
     ↓
  containerd / CRI-O / cri-dockerd
```

---

## Kubernetes Services

In Kubernetes, a Service is a method for exposing a network application that is running as one or more Pods in your cluster.
<!--
> 📁 GitHub link for service files: https://github.com/CloudTechDevOps/Kubernetes/tree/main/day-3-services
-->
### Types

#### ClusterIP

This is the default type for service in Kubernetes. As indicated by its name, this is just an address that can be used inside the cluster.

#### NodePort

A NodePort differs from the ClusterIP in the sense that it exposes a port in each Node. When a NodePort is created, kube-proxy exposes a port in the range **30000–32767**.

#### LoadBalancer

This service type creates load balancers in various Cloud providers like AWS, GCP, Azure, etc., to expose our application to the Internet.

#### Headless

Internal communications for Stateful applications (DB).

### Port Configurations

| Port | Description |
|---|---|
| **Port** | The port on which the service is exposed. Other pods can communicate with it via this port. |
| **TargetPort** | The actual port on which your container is deployed. The service sends requests to this port and the pod container must listen to the same port. |
| **NodePort** | Exposes a service externally to the cluster. The application can be accessed via this port externally. By default, it's automatically assigned during deployment. |

---

## Horizontal Pod Autoscaler (HPA)

In Kubernetes, a HorizontalPodAutoscaler automatically updates a workload resource (such as a Deployment or StatefulSet), with the aim of automatically scaling the workload to match demand.

Horizontal scaling means that the response to increased load is to deploy more Pods. This is different from vertical scaling, which for Kubernetes would mean assigning more resources (for example: memory or CPU) to the Pods that are already running for the workload.

If the load decreases, and the number of Pods is above the configured minimum, the HorizontalPodAutoscaler instructs the workload resource (the Deployment, StatefulSet, or other similar resource) to scale back down.
<!--
> 📁 GitHub link for HPA files: https://github.com/CloudTechDevOps/Kubernetes/tree/main/day-4-horizonalScaling
-->
### Cluster Autoscaler Setup

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/cluster-autoscaler-1.29.0/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

kubectl -n kube-system get pods -l app=cluster-autoscaler

kubectl -n kube-system edit deployment.apps/cluster-autoscaler
# add cluster name in above yml
```

Navigate to node group role and add → permissions autoscaler or admin

```bash
aws eks update-nodegroup-config \
  --cluster-name naresh \
  --nodegroup-name ng-af5ac006 \
  --scaling-config minSize=2,maxSize=6,desiredSize=3

kubectl -n kube-system logs -f deployment/cluster-autoscaler
```

---

## Metrics Server

The Kubernetes Metrics Server is an aggregator of resource usage data in your cluster, and it isn't deployed by default in Amazon EKS clusters. We need to deploy it using the following process.

### Deploy the Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Verify that the metrics-server deployment is running the desired number of Pods

```bash
kubectl get deployment metrics-server -n kube-system

kubectl top pod  # to see pod load
kubectl top node # to see node load
```

---

## Ingress

In Kubernetes, an Ingress is an object that allows access to your Kubernetes services from outside the Kubernetes cluster. You configure access by creating a collection of rules that define which inbound connections reach which services.

```bash
kubectl create namespace ingress-nginx  # create namespace for ingress-nginx

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.1/deploy/static/provider/cloud/deploy.yaml
# to install ingress controller yaml
```
<!--
> 📁 GitHub link for ingress files: https://github.com/CloudTechDevOps/Kubernetes/tree/main/day-5-ingress
-->
**Steps:**
1. Create deployment file for path-1
2. Create deployment file for path-2
3. Create ingress resource file

> **Note:** After deploying the above files just run:
> ```bash
> kubectl get ingress
> ```
> You will be able to see the load balancer link and access by giving path along with it.

---

## RBAC

RBAC (Role-Based Access Control) controls who can do what in your Kubernetes cluster.

- **Roles / ClusterRoles** → Define what actions are allowed.
- **RoleBindings / ClusterRoleBindings** → Define who can perform those actions.
- **IAM (AWS Identity and Access Management)** users/roles can be mapped to Kubernetes users via a ConfigMap: `aws-auth` in the `kube-system` namespace.
- You can then assign these identities Kubernetes permissions using RBAC.

| Component | Meaning | Example |
|---|---|---|
| **Role** | Says *what actions* are allowed. | "Can read pods" |
| **ClusterRole** | Like Role, but works across the *whole cluster*. | "Can read pods in all namespaces" |
| **RoleBinding** | Connects a *Role* to a *User* or *Group*. | "Give Alice the pod-reader role in the dev namespace" |
| **ClusterRoleBinding** | Connects a *ClusterRole* to a *User/Group* for the *whole cluster*. | "Give Bob admin access everywhere" |

### Role vs ClusterRole

| Feature | **Role** | **ClusterRole** |
|---|---|---|
| **Scope** | Works **only inside one namespace** | Works **across the whole cluster** |
| **Can manage namespaced resources** | ✅ Yes | ✅ Yes |
| **Can manage cluster-wide resources** (like nodes, namespaces) | ❌ No | ✅ Yes |
| **Binding type used** | RoleBinding | ClusterRoleBinding |
| **Example use** | Give a developer access to pods in `dev` namespace | Give admin access to all namespaces |

### Auth Flow

```
User
  │
  │ kubectl get nodes
  ▼
kubeconfig
  │
  ▼
aws eks get-token
  │
  ▼
IAM Authentication
  │
  ▼
aws-auth ConfigMap
  │
  ▼
Kubernetes User / Group
  │
  ▼
RBAC Authorization
  │
  ▼
API Server Response
```

### Step-by-Step RBAC Setup

- **Step 1** — Create IAM user with EKS cluster permission
- **Step 2** — `aws configure --profile IAMuser`
- **Step 3** — Create Kubernetes Role
- **Step 4** — Create Kubernetes RoleBinding to bind role and group
- **Step 5** — Add user ARN into ConfigMap file

---

### Role

A Role defines permissions (what actions can be performed) within a specific namespace. It cannot provide cluster-wide access; for that, use ClusterRole.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer-role
rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["configmap"]
    verbs: ["get", "list"]
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "list"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list"]
```

---

### RoleBinding

A RoleBinding assigns a Role to a specific user, group. It allows users to perform the actions specified in the Role within a namespace.

#### RoleBinding to map role and group

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
  - kind: Group
    name: "developer"
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

#### Edit aws-auth ConfigMap and add user details

```bash
kubectl edit cm aws-auth -n kube-system
```

```yaml
# aws-auth config
mapUsers: |
   - userarn: arn:aws:iam::730335657713:user/nareshit
     username: nareshit
     groups:
     - developer
```

---

#### Example 2 — Attach user to RoleBinding directly without group

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
  - kind: User
    name: developer1
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

#### Add user into aws-auth config

```yaml
mapUsers: |
  - userarn: arn:aws:iam::730335657713:user/developer1
    username: developer1
```

---

#### Example 3 — Specific resource access inside namespace (e.g. a specific pod)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-read-specific
  namespace: default
rules:
  - apiGroups: [""]       # core API group
    resources: ["pods"]
    resourceNames:        # restrict to specific pod(s)
      - my-pod
    verbs:
      - get
      - describe
```

---

### ClusterRole and ClusterRoleBinding

#### ClusterRole — All permissions

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-custom
rules:
  - apiGroups: ["*"]   # Allows access to all API groups
    resources: ["*"]   # Grants access to all resources (pods, services, deployments, etc.)
    verbs: ["*"]       # Grants all actions (get, list, create, update, delete, patch, watch, etc.)
```

#### ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-custom-binding
subjects:
- kind: Group
  name: admin-team           # group mapped in aws-auth
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-custom
  apiGroup: rbac.authorization.k8s.io
```

---

#### User with admin access (optional) — Full permissions using `system:masters`

```yaml
mapUsers: |
   - userarn: arn:aws:iam::381491944316:user/user-1
     username: user-1
     groups:
       - system:masters
```

> **Note:** In Kubernetes, `system:masters` is a built-in cluster-admin group that provides full administrative access to the cluster.

#### Full aws-auth block (mapRoles + mapUsers)

```yaml
mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::545009827818:role/eksctl-naresh-nodegroup-ng-2b5f9fe-NodeInstanceRole-Fifi7BsDcajz
      username: system:node:{{EC2PrivateDNSName}}
mapUsers: |
    - userarn: arn:aws:iam::545009827818:user/user-1
      username: user-1
      groups:
        - system:masters
```

---

#### Role with admin access — Add role to aws-auth

```yaml
# below block need to be added to aws-auth
      - groups:
        - system:masters
        rolearn: arn:aws:iam::545009827818:role/ec2-admin2
        username: ec2-admin2
```

#### Example — How to add to existing aws-auth

```yaml
mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::545009827818:role/eksctl-naresh-nodegroup-ng-2b5f9fe-NodeInstanceRole-Fifi7BsDcajz
      username: system:node:{{EC2PrivateDNSName}}
    - groups:
      - system:masters
      rolearn: arn:aws:iam::545009827818:role/ec2-admin2
      username: ec2-admin2
```

---

### Custom Permissions Commands

```bash
kubectl get rb
kubectl get rolebinding
kubectl api-resources

# to add user into aws-auth file
kubectl edit cm aws-auth -n kube-system

kubectl get cm -n kube-system
```

#### Full aws-auth ConfigMap reference example

```yaml
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::992382358200:role/eksctl-naresh-nodegroup-ng-bbb93ed-NodeInstanceRole-9GWNpfucPXRt
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
   - userarn: arn:aws:iam::483216680875:user/devops
     username: devops
     groups:
     - developer
kind: ConfigMap
metadata:
  creationTimestamp: "2024-03-22T02:22:59Z"
  name: aws-auth
  namespace: kube-system
  resourceVersion: "344678"
  uid: 8167447c-eb81-4108-8653-690369d98c4f
```

#### After editing the file, run below command to update cluster for created user

```bash
aws eks update-kubeconfig --name test --profile devops
# user updated permission with role and role binding with given access

aws eks update-kubeconfig --name naresh
# for default one — .kube updated by default creating cluster with admin permissions for inside cluster
```

---

## Service Accounts

Service Accounts allow pods to interact with the Kubernetes API.

**Use case:** Service accounts with pod-to-pod communications internally.

### Create a Service Account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
```

### Create a Role for the Service Account

Example: Allow this ServiceAccount to read pods.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

### Bind the Role to the Service Account

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: my-service-account
    namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Use the Service Account in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    app: webapp
    type: front-end
spec:
  serviceAccountName: my-service-account  # Attach the service account
  containers:
    - name: nginx-container
      image: nginx
```

Now, the pod runs using `my-service-account`, which has read-only access to pods.

### Verify Permissions

Exec into the pod:

```bash
kubectl exec -it pod /bin/bash
```

Try listing pods:

```bash
kubectl get pods
```

✅ It should work because the ServiceAccount has the `pod-reader` role.

### Conclusion

- 🔹 **ServiceAccount** → Used by pods to interact with the API.
- 🔹 **Role & RoleBinding** → Grants permissions to the ServiceAccount.
- 🔹 **Attach to Pod** → Use `serviceAccountName` in the pod spec.

---

## Pod-to-AWS Communication (IRSA)

### Kubernetes EKS — Pod Communication to AWS Services

### S3 Access Through Pod

#### Step 1 — Create a JSON permission file for S3 access

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Step 2 — Attach the policy

```bash
aws iam create-policy \
  --policy-name EKS_S3_Access_Policy \
  --policy-document file://s3-access-policy.json
```

Output:
```
arn:aws:iam::664418968609:policy/EKS_S3_Access_Policy
```

#### Step 3 — Create OIDC provider

Creating OIDC to connect node inside resource — able to get communication outside EKS cluster to AWS resources if permission is there.

```bash
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=project-eks --approve
```

#### Step 4 — Create an IAM Role for Service Account (using eksctl)

```bash
eksctl create iamserviceaccount \
  --name s3-access-sa \
  --namespace default \
  --cluster project-eks \
  --attach-policy-arn arn:aws:iam::975050030406:policy/EKS_S3_Access_Policy \
  --approve
```

#### Step 4 — Create a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: aws-cli-pod
spec:
  serviceAccountName: s3-access-sa
  containers:
    - name: aws-cli
      image: amazonlinux:2
      command: ["sleep", "3600"]
      tty: true
```

#### Step 5 — Apply and login

```bash
kubectl apply -f aws-cli-pod.yaml
kubectl exec -it aws-cli-pod /bin/bash
```

#### Step 6 — Install AWS CLI inside pod

```bash
yum install -y unzip curl
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install

aws --version
```

#### Step 7 — Access S3

```bash
aws s3 ls
```

Output:
```
2025-03-12 14:02:28 mysmh.co.in
2025-02-16 16:50:28 syedmujtaba
2025-04-24 09:50:11 syedredis
```

---

### Use IAM Roles for Service Accounts (IRSA)

AWS IAM Roles for Service Accounts allows each pod to assume its own IAM role securely.

**How it works:**
```
Pod → Kubernetes ServiceAccount → IAM Role → S3
```

**Why this is the best approach:**

- ✅ No hardcoded AWS keys in the pod
- ✅ Least privilege access (only that pod gets permission)
- ✅ Temporary credentials automatically rotated
- ✅ Recommended by AWS for production

### Other Approaches (Less Secure)

**1️⃣ Using Node IAM Role**
```
Pod → Node IAM Role → S3
```
Problems:
- All pods on the node share the same permissions
- Difficult to enforce least privilege
- Security risk in multi-tenant clusters

**2️⃣ Using AWS Access Keys in Pod**

Example using `AWS_ACCESS_KEY_ID`.

Problems:
- Keys stored in environment variables or secrets
- Risk of leakage
- Manual rotation required

> ❌ This is **not recommended** for production.

### Security Comparison

| Method | Security | Recommended |
|---|---|---|
| IAM Roles for Service Accounts (IRSA) | ⭐⭐⭐⭐⭐ | ✅ Yes |
| Node IAM Role | ⭐⭐ | ⚠️ Limited |
| Access Keys in Pod | ⭐ | ❌ No |

### Best Practice Architecture

```
EKS Pod
   ↓
ServiceAccount
   ↓
IAM Role (IRSA)
   ↓
Amazon S3
```

---

## Resource Requests and Limits

In Kubernetes, you use resources and requests in your pod or container specifications to manage and allocate resources effectively. These settings help ensure that your applications have the necessary resources to run efficiently and that the cluster's resources are used optimally.

### Kubernetes Pod Definition with Resource Requests and Limits

```yaml
# Pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx:latest
    resources:
      requests:
        cpu: "50m"
        memory: "100Mi"
      limits:
        cpu: "100m"
        memory: "200Mi"
```

### Explanation

| Field | Description |
|---|---|
| `apiVersion` | The version of the Kubernetes API you're using. |
| `kind` | The type of Kubernetes resource. In this case, it's a Pod. |
| `metadata` | Metadata about the pod, including the name. |
| `spec` | The specification of the pod. |
| `containers` | A list of containers in the pod. |
| `name` | The name of the container. |
| `image` | The container image to run. |

**resources:**
- `requests` — The minimum amount of resources required for the container to start.
  - `cpu` — The requested CPU (`50m` means 50 milliCPU).
  - `memory` — The requested memory (`100Mi` means 100 Mebibytes).
- `limits` — The maximum amount of resources the container is allowed to use.
  - `cpu` — The CPU limit (`100m` means 100 milliCPU).
  - `memory` — The memory limit (`200Mi` means 200 Mebibytes).

---

## Image Pull Policy

In Kubernetes, the `imagePullPolicy` is a field that determines when the kubelet should pull (download) the container image. There are three possible values for `imagePullPolicy`:

- **Always** — The image will always be pulled from the registry.
- **IfNotPresent** — The image will be pulled only if it is not already present on the node.
- **Never** — The image will never be pulled; it is assumed to be present on the node.

If the `imagePullPolicy` is not explicitly set in a Kubernetes Pod specification, Kubernetes uses default behaviors based on the image tag:

- If the image tag is `latest`, the default `imagePullPolicy` is `Always`.
- For all other tags (including when no tag is specified, which implicitly uses the `latest` tag), the default `imagePullPolicy` is `IfNotPresent`.

### Example Pod Specification (imagePullPolicy not set)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: nginx-container
    image: nginx
```

### Default Behavior for imagePullPolicy When Not Present

- If the image is `nginx:latest` or just `nginx` (implicitly `nginx:latest`): default `imagePullPolicy` will be `Always`.
- If the image has a specific tag, e.g., `nginx:1.19`: default `imagePullPolicy` will be `IfNotPresent`.

### Example with Specific imagePullPolicy

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: nginx-container
    image: nginx:1.19
    imagePullPolicy: IfNotPresent
```

### Explicitly Setting imagePullPolicy

**Always:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: nginx-container
    image: nginx:1.19
    imagePullPolicy: Always
```

**IfNotPresent:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: nginx-container
    image: nginx:1.19
    imagePullPolicy: IfNotPresent
```

**Never:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: nginx-container
    image: nginx:1.19
    imagePullPolicy: Never
```

### Summary

When `imagePullPolicy` is not specified, Kubernetes defaults to `Always` for images tagged as `latest` and `IfNotPresent` for other tags. Explicitly setting `imagePullPolicy` can help control image pull behavior and optimize deployment times and resource usage.

---

## Scheduling

### Scheduling Overview

A scheduler watches for newly created Pods that have no Node assigned. For every Pod that the scheduler discovers, the scheduler becomes responsible for finding the best Node for that Pod to run on. The scheduler reaches this placement decision taking into account the scheduling principles described below.

**Scheduling methods:**
1. Node Selector
2. Node Affinity
3. Daemon Set
4. Taint and Toleration

---

### 1. NodeSelector

NodeSelector is the simplest recommended form of node selection constraint. You can add the `nodeSelector` field to your Pod specification and specify the node labels you want the target node to have. Kubernetes only schedules the Pod onto nodes that have each of the labels you specify.

**Key rules:**
- ==> Unlabeled pod **can** schedule on labeled node
- ==> Labeled pod **cannot** schedule on unlabeled node
- ==> If we unlabel the node after the pod is already scheduled on the matched label node — **No impact** on running time; even if labels mismatch, pod already scheduled remains running

#### Label / Unlabel Commands

```bash
# to label the node
kubectl label nodes <node-name> <label-key>=<label-value>

# Examples:
kubectl label nodes ip-192-168-60-75.ec2.internal clr=black
kubectl label nodes ip-192-168-4-19.ec2.internal size=small

# to unlabel
kubectl label nodes <node-name> <label-key>=<label-value>-
kubectl label nodes ip-192-168-24-53.us-west-2.compute.internal clr-

# to list
kubectl get nodes --show-labels
```

> ⚠️ Labels are **case sensitive**.

#### Pod with NodeSelector

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
      app: webapp
      type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
  nodeSelector:
    size: Large
```

A Kubernetes node selector is a simple way to constrain which nodes your pods can be scheduled on, based on node labels. By using node selectors, you can ensure that certain pods run only on nodes that meet specific criteria. If the pod label is not matching it will not schedule on any node — always trying to schedule on the labeled node only, otherwise it will remain in **Pending** state.

---

### 2. Node Affinity

Node affinity is conceptually similar to `nodeSelector`, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

| Type | Behavior |
|---|---|
| `requiredDuringSchedulingIgnoredDuringExecution` | The scheduler can't schedule the Pod unless the rule is met. Functions like `nodeSelector`, but with a more expressive syntax. |
| `preferredDuringSchedulingIgnoredDuringExecution` | The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod on any other node. |
| `requiredDuringSchedulingRequiredDuringExecution` | Not directly supported, but similar behavior can be achieved with taints and tolerations to ensure pods are scheduled and evicted based on node conditions. |

**Additional notes:**
- `requiredDuringSchedulingIgnoredDuringExecution` — Ensures pods are scheduled only on nodes matching specific labels, but does not evict them if the labels change.
- `preferredDuringSchedulingIgnoredDuringExecution` — Prefers nodes with specific labels but can schedule pods on any available nodes if necessary.

#### a. requiredDuringSchedulingIgnoredDuringExecution

> ***It will schedule only if the pod and node label match, otherwise it will not schedule.***

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

#### b. preferredDuringSchedulingIgnoredDuringExecution

> ******If label matches it will create in matched node, otherwise it will schedule on another node.***

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

---

### 3. DaemonSet

A DaemonSet is another controller that manages pods like Deployments, ReplicaSets, and StatefulSets. It was created for one particular purpose: ensuring that the pods it manages run on **all the cluster nodes**. The pod is going to schedule on all available nodes.

> **Example:** If we have three nodes, the same pod is going to be scheduled on all three nodes.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: test-nginx
        image: nginx
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 50m
            memory: 100Mi
```

---

### 4. Taints and Tolerations

#### Taints

A taint is applied to a node to mark it as having some special property that only certain pods can tolerate. If a node has a taint, the scheduler will not place a pod on that node unless the pod has a matching toleration.

**Two types:**
- `NoSchedule`
- `NoExecute`

#### Node Taint and Untaint Commands

```bash
# Taint
kubectl taint nodes ip-192-168-3-253.ap-south-1.compute.internal app=blue:NoSchedule

kubectl taint nodes ip-192-168-40-106.ec2.internal app=blue:NoExecute

kubectl taint nodes ip-192-168-52-251.ec2.internal app=blue:NoExecute

kubectl taint nodes ip-192-168-60-135.ap-south-1.compute.internal app=green:NoExecute

kubectl taint node ip-192-168-43-22.ap-south-1.compute.internal app=blue:NoExecute

# Untaint
kubectl taint node ip-192-168-36-40.ec2.internal app=blue:NoSchedule-

kubectl taint node ip-192-168-43-22.ap-south-1.compute.internal app=blue:NoExecute-

# List tainted nodes
kubectl describe nodes ip-192-168-60-135.ap-south-1.compute.internal | grep Taints
```

#### Toleration Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

#### Important Notes

> - Toleration pod only creates on a specific tainted node if labels match. A tolerate pod **can** schedule on other nodes also, but an untolerate pod **cannot** schedule on a tainted node.
>
> - The taint effect defines how a tainted node reacts to a pod without appropriate toleration. It must be one of the following effects:
>
>   - **NoSchedule** — The pod will not get scheduled to the node without a matching toleration. *(Will not schedule new pods on tainted node, but running pods will not be deleted after enabling taint on nodes.)*
>
>   - **NoExecute** — This will immediately evict all the pods without the matching toleration from the node. *(No new pods will schedule AND running pods will also be deleted after enabling taint on nodes.)*

#### Check Tainted Node Info

```bash
kubectl describe nodes | grep -i taints       # to check any node tainted
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

---

### Node Drain

Draining a node in Kubernetes is a common operation for performing maintenance or decommissioning a node. The `kubectl drain` command safely evicts all the pods while ensuring minimal disruption to your applications.

Draining a Kubernetes node involves safely evicting all the pods from the node, so you can perform maintenance or remove the node from the cluster without disrupting services.

#### Step-by-Step Guide to Drain a Kubernetes Node

**1. Verify access by listing nodes:**

```bash
kubectl get nodes
```

**2. Drain the Node:**

Use the `kubectl drain` command to safely evict all the pods from the node. This command ensures that your applications are gracefully terminated, and no new pods are scheduled on the node.

Replace `ip-192-168-56-36.ap-south-1.compute.internal` with your node's name.

```bash
kubectl drain ip-192-168-56-36.ap-south-1.compute.internal --ignore-daemonsets --delete-emptydir-data
```

**Options explained:**

| Option | Description |
|---|---|
| `--ignore-daemonsets` | Allows the node to be drained even if there are daemonset-managed pods on it. |
| `--delete-emptydir-data` | Allows pods using emptyDir volumes to be deleted, as these volumes are tied to the node's filesystem and are ephemeral. |

**3. Verification:**

After running the drain command, verify that the node has been drained:

```bash
kubectl get nodes
```

The output should show that the node is in a `SchedulingDisabled` state, indicating that it has been drained and no new pods will be scheduled on it.

**4. Uncordon the Node (Optional):**

If you want to make the node schedulable again after performing maintenance:

```bash
kubectl uncordon ip-192-168-56-36.ap-south-1.compute.internal
```

> Remember to use `--ignore-daemonsets` and `--delete-emptydir-data` options to handle special cases. After maintenance, you can make the node schedulable again using the `kubectl uncordon` command.

---

## Volumes and Storage

### What is a Kubernetes Volume?

A Kubernetes volume is a directory containing data accessible to containers in a given pod, the smallest deployable unit in a Kubernetes cluster. Within the Kubernetes container orchestration and management platform, volumes provide a plugin mechanism that connects ephemeral containers with persistent data storage.

A Kubernetes volume persists until its associated pod is deleted. When a pod with a unique identification is deleted, the volume associated with it is destroyed. If a pod is deleted but replaced with an identical pod, a new and identical volume is also created.

### Setup Steps

**Step 1:**
- a) Create an EKS cluster with the clusterconfig file.
- b) Install Helm on your local machine. Below is the link → https://helm.sh/docs/intro/install/
- c) Connect to your EKS cluster. Check the connection.

### Three Types of Volume Approaches

| # | Approach | Description | Recommended? |
|---|---|---|---|
| 1 | **Static HostPath** (pv, pvc, deployment) | Local to the node. If node is deleted, volumes also deleted. | ❌ Not recommended |
| 2 | **Static Cloud EBS** (pv, pvc, deployment) | Static approach. No effect even if node is deleted — volume persists. But EC2 volume needs to be created manually. | ⚠️ Manual effort |
| 3 | **Dynamic Volume Provisioning via StorageClass** (pvc, storageclass) | PV is created automatically by StorageClass. | ✅ Recommended |

#### HostPath Limitation

> The `hostPath` volume is local to the node where it was created. If your pod is rescheduled on a different node, it will try to use the same path (`/mnt/data`) on the new node. However, this directory on the new node will not have the data from the original node, leading to **data loss**.

### Install Helm

```bash
# Releases: https://github.com/helm/helm/releases

wget https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz

tar -zxvf helm-v3.14.0-linux-amd64.tar.gz

mv linux-amd64/helm /usr/local/bin/helm

chmod 777 /usr/local/bin/helm  # give permissions

helm version
```

> **Note:** After creating the cluster, you have to give IAM EC2 full access or admin access to the node group IAM role — then only EBS volume can be created by node.

### Step 2 — Install CSI Driver in EKS Cluster

After connection, execute the below commands:

```bash
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm repo update

# Install AWS EBS driver to Kubernetes
helm upgrade --install aws-ebs-csi-driver --namespace kube-system aws-ebs-csi-driver/aws-ebs-csi-driver
```

### Access Modes

| Mode | Use Case |
|---|---|
| **ReadWriteMany** | If you need to write to the volume and may have multiple Pods needing to write, where you'd prefer the flexibility of those Pods being scheduled to different nodes, and `ReadWriteMany` is an option given the volume plugin for your K8s cluster, use `ReadWriteMany`. |
| **ReadWriteOnce** | If you need to write to the volume but either you don't have the requirement that multiple pods should be able to write to it, or `ReadWriteMany` simply isn't an available option for you, use `ReadWriteOnce`. |
| **ReadOnlyMany** | If you only need to read from the volume, and you may have multiple Pods needing to read from the volume where you'd prefer the flexibility of those Pods being scheduled to different nodes, and `ReadOnlyMany` is an option given the volume plugin for your K8s cluster, use `ReadOnlyMany`. If you only need to read from the volume but either you don't have the requirement that multiple pods should be able to read from it, or `ReadOnlyMany` simply isn't an available option for you, use `ReadWriteOnce`. In this case, you want the volume to be read-only but the limitations of your volume plugin have forced you to choose `ReadWriteOnce` (there's no `ReadOnlyOnce` option). As a good practice, consider the `containers.volumeMounts.readOnly`. |

### PVC Reclaim Policies

```
reclaim policy: delete   # this is the default one — whenever PVC is deleted, PV will delete automatically
                         # created by StorageClass, but EBS volume will not delete;
                         # if you want to delete it, delete it manually

reclaim policy: retain   # it will be persistent
```

### Types of Volume Bindings

```
volumeBinding: Immediate          # this is the default one — PV is created immediately
volumeBinding: WaitForFirstConsumer   # PV will be created only when any pod claims the storage
```

---

## Grafana and Prometheus

Prometheus collects and stores metric data as time-series data, while Grafana is an analytics and visualization web application that can ingest data from various sources and display it in customizable charts.

> 📖 For more details, refer blog:
> https://medium.com/@veerababu.narni232/deployment-of-prometheus-and-grafana-using-helm-in-eks-cluster-22caee18a872

---

## Helm Charts

Helm is a tool that automates the creation, packaging, configuration, and deployment of Kubernetes applications by combining your configuration files into a single reusable package.

### Install Helm

```bash
# Releases: https://github.com/helm/helm/releases

wget https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz

tar -zxvf helm-v3.14.0-linux-amd64.tar.gz

mv linux-amd64/helm /usr/local/bin/helm

chmod 777 /usr/local/bin/helm  # give permissions

helm version
```

### Usage

```bash
helm create helloworld
# create sample helm chart application named helloworld

helm install <FIRST_ARGUMENT_RELEASE_NAME> <SECOND_ARGUMENT_CHART_NAME>
# run helm
```

Whenever you create the sample project you will get a template of Kubernetes structure like below, so you can place your values and images:

```
helloworld
├── charts
├── Chart.yaml
├── templates
│  ├── deployment.yaml
│  ├── _helpers.tpl
│  ├── hpa.yaml
│  ├── ingress.yaml
│  ├── NOTES.txt
│  ├── serviceaccount.yaml
│  ├── service.yaml
│  └── tests
│      └── test-connection.yaml
└── values.yaml
```

```bash
helm list -a                          # to list helm charts

helm upgrade firstproject helloworld  # upgrade a release

helm delete <release> <chartname>     # to delete chart
```

> 📖 For more details, refer blog:
> https://medium.com/@veerababu.narni232/writing-your-first-helm-chart-for-hello-world-40c05fa4ac5a

---

## ArgoCD

Argo CD is a declarative continuous delivery tool for Kubernetes. It can be used as a standalone tool or as a part of your CI/CD workflow to deliver needed resources to your clusters.

### Install Process

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get pods -n argocd

kubectl get svc -n argocd

kubectl edit svc argocd-server -n argocd

kubectl get svc -n argocd

# access it via nodeIp:Nodeport

kubectl get secrets -n argocd
```

**User:** `admin`

```bash
kubectl edit secret argocd-initial-admin-secret -n argocd
# to get initial credential to login ArgoCD

echo bnFabGx3emtCNjB5dFZQSA== | base64 --decode
```

> 📖 For more details, refer blog:
> https://medium.com/@veerababu.narni232/a-complete-overview-of-argocd-with-a-practical-example-f4a9a8488cf9

---

## StatefulSet

StatefulSet is the workload API object used to manage stateful applications.

Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.

Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a **sticky identity** for each of its Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

### Using StatefulSets

StatefulSets are valuable for applications that require one or more of the following:

- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.

> 📖 For more details, refer blog:
> https://medium.com/@veerababu.narni232/what-are-stateful-applications-2a257d876187

---

## Headless Service

A Headless Service is a variation of the ClusterIP Service, where the `clusterIP` field is set to `None`. Unlike traditional services, Headless Services do not use a single Service IP to proxy connections to the Pods. Instead, they allow you to directly connect to Pods without any load balancing intermediary.

### Use Cases for Headless Services

Headless Services are particularly useful in the following scenarios:

- **Service Discovery** — Some service discovery mechanisms, such as Kubernetes DNS-based service discovery, require direct access to individual Pods. Headless Services provide a convenient way to achieve this.
- **Stateful Applications** — Applications that require direct access to individual Pods, such as databases or distributed storage systems, can benefit from using Headless Services.
- **Custom Load Balancing** — If you need to implement custom load balancing logic or use a specific load balancing mechanism, Headless Services allow you to directly access the Pods without relying on Kubernetes' built-in load balancing.

### Accessing Pods using Headless Services

The DNS name for each Pod follows the pattern: `<pod-ip>.<namespace>.pod.cluster.local`

**Example:** If you have a Pod with the IP address `10.0.0.5` in the default namespace, you can access it using the DNS name:

```
10-0-0-5.default.pod.cluster.local.
```

You can also access the Pods directly by their IP addresses, which can be useful for applications that require direct IP-based communication:

```
10-0-0-5   # ip of the target pod
```

> To access stateless to stateful — communicate through headless service using: `"podname.default.pod.cluster.local"` — here pod name is unique.

### Headless Service Conclusion

Headless Services in Kubernetes offer a unique approach to accessing individual Pods directly, without relying on a load balancer or a single Service IP. This type of service is invaluable in scenarios where direct Pod access is required, such as service discovery mechanisms, stateful applications like databases, or when implementing custom load balancing logic.

By setting the `clusterIP` field to `None`, Headless Services bypass the traditional load balancing layer and instead provide a direct connection to individual Pods. This is achieved through the assignment of DNS records for each Pod, allowing you to access them by their DNS names or IP addresses.

---

### StatefulSet + Headless Service Example

```yaml
# https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-normal
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx-headless"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: shared-volume
        emptyDir: {}
      initContainers:
      - name: busybox
        image: busybox
        volumeMounts:
        - name: shared-volume
          mountPath: /nginx-data
        command: ["/bin/sh", "-c"]
        args: ["echo Hello from container $HOSTNAME > /nginx-data/index.html"]
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: shared-volume
          mountPath: /usr/share/nginx/html
        - name: www
          mountPath: /usr/share/nginx
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Mi
```

### Testing StatefulSet with DNS

Run below command to create a stateless pod, then login and check nslookup and curl to stateful nginx normal service and headless service:

```bash
kubectl run -i --tty --image nginx:alpine dns-test
# create pod and access service from pod by using curl
# create pod and access service from pod by using nslookup
```

#### nslookup Normal Service

```bash
nslookup <normal service name>
```

Redirects to cluster IP — works like an internal load balancer. We can see the cluster IP, which means the stateless Pod connects to the stateful pod service cluster IP. In this case it also works like a load balancer — **not recommended**.

#### nslookup Headless Service

```bash
nslookup <headless service name>
```

Here the result shows we can directly connect to target pods without a service intermediary, so here we can define read pod and write pod:

```
Name:      mysql
Address 1: 192.168.11.161 mysql-0.mysql.default.svc.cluster.local
Address 2: 192.168.61.95 mysql-1.mysql.default.svc.cluster.local
```

#### Curl to Check Response

```bash
curl nginx-normal service
# will get response from any one of the pods — not recommended
# when reading, response should come from read pod
# when writing, response should come from write pod

curl nginx-headless service
# here also mostly request will go to first pod always

# to overcome this issue, call like below:
curl podname.nginx-headless service

# Examples:
curl web-0.nginx-headless   # always get response from web-0 only
                             # stateless application communicates to stateful pod web-0 for WRITE operation

curl web-1.nginx-headless   # always get response from web-1 only
                             # stateless application communicates to stateful pod web-1 for READ operation
```

> **Note:** If there is only one DB pod, we can use a regular deployment — no confusion for read and write. In that case, use ClusterIP service for internal communication from stateless to stateful DB pod.

---

## Probes

Kubernetes, the industry-leading container orchestration tool, offers mechanisms to implement robust health checks for your applications. These checks, termed "probes," act as guardians of your application's well-being, continuously monitoring the health status of your pods and their hosted applications.

### Pod Status Reference

| Status | Meaning |
|---|---|
| `1/1` | All containers are running and ready (healthy Pod) |
| `0/1` | Container is running but failing readiness or startup probe |
| `0/1` + `CrashLoopBackOff` | Container keeps crashing/restarting |
| `2/2` | Multi-container Pod (e.g., app + sidecar) where both containers are Ready |

### Probe Failure Effects

```
==> Startup probe failure   = endless restart loop.
==> Readiness probe failure = Pod runs but excluded from Service (no restarts).
==> Liveness probe failure  = Pod restarts, but only after startup has succeeded once.
```

---

### Startup Probe

A startup probe is a special type of probe designed to help Kubernetes handle whether an application has started or not — also helps with slow-starting applications.

Its main job is to tell Kubernetes: **"I'm still starting up, don't kill me yet!"**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-startup-only
spec:
  containers:
  - name: nginx
    image: nginx:latest
    command: ["sh", "-c", "sleep 40 && nginx -g 'daemon off;'"]
    ports:
    - containerPort: 80
    startupProbe:
      httpGet:
        path: /test
        port: 80
      failureThreshold: 12    # allow 12 failures
      periodSeconds: 5        # check every 5s (10 x 5 = 50s total grace time)
```

K8s tries to GET `http://<pod-ip>:80/test` every 5 seconds.

It will allow 10 consecutive failures = 50 seconds total grace (10 × 5).

If the probe doesn't succeed in that time, kubelet kills and restarts the container.

After many attempts pod will enter into **CrashLoopBackOff** error.

---

### Readiness Probe

A readiness probe is a check that Kubernetes performs on a container to determine whether it is ready to serve traffic.

- It does **not** restart the container.
- It only tells Kubernetes whether the Pod should be included in a Service's endpoints (i.e., receive traffic).

**Purpose:** *"Am I ready to serve traffic?"*

- Failing it → Pod is marked `NotReady`.
- K8s removes the Pod from the Service endpoints list.
- Pod keeps running, no restarts.

```yaml
# Pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx-readiness-test
  labels:
    app: nginx-test
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /path
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-readiness-svc
spec:
  selector:
    app: nginx-test
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

### Combination — Startup + Readiness

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-startup-readiness
  labels:
   app: nginx-startup-readiness
spec:
  containers:
  - name: nginx
    image: nginx:latest
    command: ["sh", "-c", "sleep 40 && nginx -g 'daemon off;'"]
    ports:
    - containerPort: 80
    startupProbe:
      httpGet:
        path: /te
        port: 80
      failureThreshold: 12
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb-svc
spec:
  selector:
    app: nginx-startup-readiness
  ports:
    - port: 80        # Service port
      targetPort: 80  # Container port
  type: LoadBalancer
```

---

### Liveness Probe

A liveness probe is a mechanism Kubernetes uses to check whether a container is still alive and functioning.

If the probe fails repeatedly, Kubernetes assumes the container is unhealthy and **restarts** it.

### Combination — Startup + Readiness + Liveness

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-startup-readiness
  labels:
    app: nginx-startup-readiness
spec:
  containers:
  - name: nginx
    image: nginx:latest
    command: ["sh", "-c", "sleep 40 && nginx -g 'daemon off;'"]
    ports:
    - containerPort: 80

    startupProbe:
      httpGet:
        path: /te
        port: 80
      failureThreshold: 12
      periodSeconds: 5

    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5

    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 10
      failureThreshold: 3
```

```
initialDelaySeconds = 5  → wait 5 seconds after container starts before first check.
periodSeconds = 10       → check every 10 seconds.
failureThreshold = 3     → if 3 consecutive checks fail, kubelet restarts the container.
```

### To Test Probes

Exec into the container and temporarily break the health endpoint:

```bash
kubectl exec -it <pod-name> -- sh
mv /usr/share/nginx/html/healthz /usr/share/nginx/html/healthz.bak
```

> 📖 For more details, refer blog:
> https://medium.com/@veerababu.narni232/probes-in-kubernetes-5133ebe03475

---

## EFK Stack

When it comes to log management in Kubernetes, the EFK stack stands out as a robust solution. EFK, short for **E**lasticsearch, **F**luent Bit, and **K**ibana, streamlines the process of collecting, processing and visualizing logs.

> 📖 For more details, refer blog:
> https://medium.com/@veerababu.narni232/setting-up-the-efk-stackwhat-is-efk-stack-7944fb4e56f0

---

## 📁 All YAML Files

> Please click the below GitHub link for all YAML files:
>
> ➡️ [https://github.com/CloudTechDevOps/](https://github.com/CloudTechDevOps/)

---

*For production use, always refer to the [official Kubernetes documentation](https://kubernetes.io/docs/).*
