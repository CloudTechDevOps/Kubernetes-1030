# Day 3: Kubernetes Services - Network Exposure Patterns

This directory demonstrates the three main Kubernetes Service types: ClusterIP, NodePort, and LoadBalancer. Services provide stable network endpoints to access pods, enabling reliable communication between components.

## 📋 Service Types Overview

| Service Type | Accessibility | Use Case | External IP |
|-------------|---------------|----------|-------------|
| **ClusterIP**    | Internal only | Microservice communication | ❌ |
| **NodePort**     | External (via Node IP) | Development, limited external access | ❌ |
| **LoadBalancer** | External (via cloud LB) | Production external access | ✅ |

## 🔍 Detailed Service Explanations

### 3.1 ClusterIP Service (Internal Cluster Access)

**Purpose**: Default service type that exposes applications internally within the cluster.

#### YAML Structure Analysis (ClusterIP.yml)

```yaml
apiVersion: v1              # Core API for Service resources
kind: Service               # Resource type for network abstraction
metadata:
  name: my-app-svc         # Service name (used for DNS)
  labels:
    app: my-app           # Labels for organization
spec:
  type: ClusterIP          # Service type (default)
  ports:
  - port: 80              # Service port (cluster-internal)
    targetPort: 80        # Pod container port
    protocol: TCP         # Network protocol
  selector:
    app: my-app           # Pod selector labels
```

#### How ClusterIP Works

1. **IP Assignment**: Kubernetes assigns a virtual IP from cluster's internal range (10.0.0.0/24 by default)
2. **Internal Routing**: Traffic only routable within cluster via kube-proxy
3. **Load Balancing**: Distributes requests across matching pods using iptables/ipvs rules
4. **DNS Resolution**: Automatic DNS record creation (e.g., `my-app-svc.default.svc.cluster.local`)
5. **Service Discovery**: Pods access service by name, resolved by cluster DNS

#### ClusterIP Advantages ✅
- **Secure**: Not exposed externally, reducing attack surface
- **Stable**: Virtual IP doesn't change during pod lifecycle
- **Load Balanced**: Automatic distribution across healthy pods
- **DNS Integration**: Built-in service discovery
- **Resource Efficient**: No external IP allocation needed

#### ClusterIP Disadvantages ❌
- **No External Access**: Cannot be reached from outside cluster
- **Limited Exposure**: Requires Ingress or other services for external traffic
- **Debugging Challenges**: Harder to access directly for testing

#### ClusterIP Use Cases
- **Microservice Communication**: API calls between internal services
- **Database Access**: Internal database connections
- **Backend Services**: Application components not needing external access

### 3.2 NodePort Service (Node-Level External Access)

**Purpose**: Exposes service on each node's IP at a static port, making it accessible externally.

#### YAML Structure Analysis (Nodeport.yml)

```yaml
# First resource: Deployment to create pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment-np
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app-np
  template:
    metadata:
      labels:
        app: my-app-np
    spec:
      containers:
      - name: my-container
        image: nginx:latest
        ports:
        - containerPort: 80
---
# Second resource: NodePort Service
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort           # Service type for node-level exposure
  selector:
    app: my-app-np        # Must match deployment labels
  ports:
    - port: 80            # Service port (internal cluster access)
      targetPort: 80      # Pod container port
      nodePort: 30007     # Static port on each node (30000-32767)
```
#### NodePort Advantages ✅
- **Simple External Access**: Direct access via node IP and port
- **Easy Testing**: Quick way to expose applications for development
- **Port Predictability**: Can specify exact port numbers

#### NodePort Disadvantages ❌
- **Limited Ports**: Only 2768 ports available (30000-32767)
- **Security Concerns**: Opens ports on all nodes, potential attack vector
- **Load Balancing**: No intelligent load balancing, just round-robin
- **Firewall Management**: Requires opening ports on all nodes

#### NodePort Use Cases
- **Development Environments**: Quick access for testing and debugging
- **Temporary Exposure**: Short-term external access needs

### 3.3 LoadBalancer Service (Cloud-Native External Access)

**Purpose**: Creates an external load balancer to distribute traffic across pods automatically.

#### YAML Structure Analysis (LoadBalancer.yml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb         # Service name
  labels:
    app: my-app-lb       # Labels for organization
spec:
  type: LoadBalancer      # Service type requiring cloud integration
  ports:
  - port: 80             # Service port (load balancer port)
    targetPort: 80       # Pod container port
    protocol: TCP        # Network protocol
  selector:
    app: my-app-lb       # Pod selector (must match pod labels)
```

#### How LoadBalancer Works

1. **Cloud Integration**: Interacts with cloud provider's load balancer service
2. **External IP**: Automatically provisions external IP/load balancer
3. **Traffic Distribution**: Cloud LB distributes traffic across healthy pods
4. **Health Checks**: Automatic health monitoring of pods

#### LoadBalancer Advantages ✅
- **Production Ready**: Designed for high-traffic production workloads
- **Intelligent Balancing**: Advanced load balancing algorithms
- **Health Monitoring**: Automatic pod health checking and failover
- **Cloud Integration**: Native integration with cloud provider features

#### LoadBalancer Disadvantages ❌
- **Cloud Dependent**: Only works on supported cloud platforms
- **Cost**: Load balancers incur additional cloud costs
- **Provisioning Time**: May take time to provision external resources

## 🌐 How Services Work Across Nodes (Cross-Node Architecture)

### The Challenge: Pod on Node-1, Accessed from Node-2

**Scenario**: You have a pod running on **Node-1** with IP `10.244.0.5:80`, but you want to access it from **Node-2** (or from outside the cluster). How does this work?

### Answer: kube-proxy Magic on Every Node!

The key is that **kube-proxy runs on EVERY node** and implements the service networking rules. Here's how it works:

### Step-by-Step: How a Service Bridges Nodes

#### Example Setup:
```
EKS Cluster with 3 Nodes:
├── Node-1: Running Pod-A (app: nginx) - Pod IP: 10.244.0.5
├── Node-2: Running Pod-B (app: nginx) - Pod IP: 10.244.1.3
└── Node-3: No pods, but can still route traffic

ClusterIP Service: my-app-svc
├── Service IP (Virtual): 10.96.0.10
├── Port: 80
└── Selector: app: nginx
```

### How Traffic Flows: Client on Node-3 → Pod on Node-1

```
1. Client on Node-3 tries to access 10.96.0.10:80

2. Packet reaches Node-3's kernel network stack

3. Node-3's kube-proxy intercepts the packet:
   ├── Has iptables rules that match destination 10.96.0.10:80
   ├── Rule says: "This is a service, translate it"
   └── Performs NAT (Network Address Translation)

4. Packet destination is rewritten:
   ├── Original: 10.96.0.10:80
   └── Rewritten to: 10.244.0.5:80 (Pod-A on Node-1)

5. Packet travels through cluster network to Node-1

6. Node-1 receives packet destined for 10.244.0.5:80

7. Node-1's kube-proxy recognizes the destination

8. Kernel routes packet to Pod-A running on Node-1

9. Pod-A responds, and response travels back through Node-3

10. Node-3's kube-proxy reverses NAT (back to Service IP)

11. Client receives response from 10.96.0.10:80
```

### The Infrastructure Components

#### 1. **kube-proxy** (On Every Node)
- Small agent running on every node
- Maintains network rules using:
  - **iptables**: Traditional Linux firewall (still most common)
  - **IPVS**: IP Virtual Server (for high-performance)
  - **Windows HNS**: On Windows nodes
- Updates rules automatically when services change

#### 2. **Cluster Network Plugin** (Flannel, Calico, WeaveNet, etc.)
- Ensures pod-to-pod communication across nodes
- Provides overlay network where all pods can reach each other
- Manages IP routing between nodes

#### 3. **API Server** (Control Plane)
- Watches for service creation/updates
- Triggers kube-proxy to update rules

#### 4. **Endpoints Controller** (Control Plane)
- Maintains list of healthy pods matching service selector
- Automatically updates as pods are added/removed
- Only includes pods passing readiness probes

### Detailed iptables Rules Example

When you create a service, kube-proxy creates rules like:

```bash
# Rule 1: Match traffic to Service IP
iptables -t nat -A KUBE-SERVICES -d 10.96.0.10/32 -p tcp -m tcp --dport 80 \
    -j KUBE-SVC-ABCD1234

# Rule 2: Load balance to endpoint pods (round-robin)
iptables -t nat -A KUBE-SVC-ABCD1234 -m statistic --mode random --probability 0.5000000000 \
    -j KUBE-SEP-POD1
iptables -t nat -A KUBE-SVC-ABCD1234 \
    -j KUBE-SEP-POD2

# Rule 3: NAT to actual pod IP
iptables -t nat -A KUBE-SEP-POD1 -p tcp -m tcp -j DNAT --to-destination 10.244.0.5:80
iptables -t nat -A KUBE-SEP-POD2 -p tcp -m tcp -j DNAT --to-destination 10.244.1.3:80
```

### Why Don't You Need Pods on Every Node?

**Important Insight**: The service IP (10.96.0.10) is **virtual** - no pod actually needs to bind to it!

- **Virtual IP**: Exists only in kernel network rules
- **Network Address Translation**: Converts service traffic to actual pod IPs
- **Cross-Node Routing**: Cluster network routes packets to the actual pod location
- **Load Balancing**: kube-proxy decides which pod to route to

### Example: Access Patterns Across Nodes

#### Pattern 1: Pod-to-Pod (Node-1 → Node-3)
```
Pod on Node-1 (10.244.0.100)
    ↓ sends request to Service IP 10.96.0.10:80
Node-1's kube-proxy intercepts (iptables rule)
    ↓ translates to Pod-C (10.244.2.50) on Node-3
    ↓ network plugin routes across nodes
Pod-C on Node-3 receives request
```

#### Pattern 2: External Client (Outside EKS → Node-1)
```
External Client (203.0.113.5)
    ↓ connects to Node IP 1.2.3.4 on NodePort 30007
Node-1's kube-proxy (NodePort mode)
    ↓ opens port 30007 on all nodes
    ↓ translates to Pod-A (10.244.0.5:80)
Pod-A on Node-1 receives request
```

#### Pattern 3: Service Discovery (DNS → ClusterIP)
```
Client pod queries DNS for "my-app-svc"
    ↓ CoreDNS returns Service IP: 10.96.0.10
Client connects to 10.96.0.10:80
    ↓ kube-proxy on client's node translates
    ↓ connection reaches actual pod on any node
```

### Visualizing the Service Network

```
Kubernetes Cluster Network:
┌─────────────────────────────────────────────────┐
│  EKS Cluster (172.0.0.0/16)                    │
├─────────────────────────────────────────────────┤
│ ┌──────────────┐  ┌──────────────┐  ┌────────┐ │
│ │   Node-1     │  │   Node-2     │  │ Node-3 │ │
│ │ 172.20.0.21  │  │ 172.20.0.22  │  │172.20..│ │
│ │              │  │              │  │        │ │
│ │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌────┐ │ │
│ │ │ Pod-A    │ │  │ │ Pod-B    │ │  │ │App │ │ │
│ │ │10.244.0.5│ │  │ │10.244.1.3│ │  │ │Pod │ │ │
│ │ │ :80      │ │  │ │ :80      │ │  │ │    │ │ │
│ │ └──────────┘ │  │ └──────────┘ │  │ └────┘ │ │
│ │              │  │              │  │        │ │
│ │ kube-proxy   │  │ kube-proxy   │  │k-prox  │ │
│ │(iptables)    │  │(iptables)    │  │ (rule) │ │
│ └──────────────┘  │ └──────────────┘  │        │ │
│                                       └────────┘ │
│                                                   │
│  Service: my-app-svc = 10.96.0.10                 │
│  (Virtual IP - managed by all kube-proxy instances)
└─────────────────────────────────────────────────┘

All nodes can route:
- Node-1 → Pod-A (local)
- Node-1 → Pod-B (routed via network plugin to Node-2)
- Node-2 → Pod-A (routed via network plugin to Node-1)
- Node-2 → Pod-B (local)
- Node-3 → Pod-A (routed to Node-1)
- Node-3 → Pod-B (routed to Node-2)
```

### Key Concepts Summary

| Concept | Explanation |
|---------|------------|
| **Virtual IP** | Service IP that exists only in network rules, not on any real interface |
| **kube-proxy** | Agent on every node that implements service networking via iptables |
| **iptables Rules** | Kernel-level rules that intercept and NAT service traffic |
| **Endpoints** | List of healthy pod IPs that match service selector |
| **Load Balancing** | Random/round-robin selection from endpoints |
| **Cross-Node Routing** | Cluster network plugin ensures packets reach pods on other nodes |
| **NAT** | Network Address Translation that rewrites destination IP |

### Debugging Cross-Node Service Access

```bash
# Check service and endpoints
kubectl get svc my-app-svc
kubectl get endpoints my-app-svc

# Check iptables rules on a node
kubectl run debug-pod --image=ubuntu --overrides='{"spec":{"hostNetwork":true}}' -it -- /bin/bash
# Inside: iptables -t nat -L -n | grep 10.96.0.10

# Check if pod is reachable from another pod
kubectl run debug-pod --image=busybox --rm -it -- wget -O- my-app-svc:80

# Trace network path
kubectl run trace-pod --image=nicolaka/netshoot --rm -it -- traceroute <service-ip>

# Check kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy -f
```

### Why This Architecture Matters

1. **Scalability**: Nodes don't need to know about every pod
2. **Resilience**: Pod failures don't break service routing
3. **Load Distribution**: Traffic automatically spreads across healthy pods
4. **Simplicity**: Applications see stable service IP regardless of pod location
5. **Multi-Cloud**: Same architecture works on any cloud or on-premises

## 🛠️ Practical Commands & Testing

### 3.1 ClusterIP Service Testing

```bash
# Deploy ClusterIP service
kubectl apply -f ClusterIP.yml

# Check service details
kubectl get svc my-app-svc
kubectl describe svc my-app-svc

# Test internal access from another pod
kubectl run test-pod --image=busybox --rm -it -- /bin/sh
# Inside pod: wget my-app-svc:80

# Check DNS resolution
kubectl run dns-test --image=busybox --rm -it -- nslookup my-app-svc

# Clean up
kubectl delete -f ClusterIP.yml
```

### 3.2 NodePort Service Testing

```bash
# Deploy NodePort service (includes deployment)
kubectl apply -f Nodeport.yml

# Check service (note the NodePort)
kubectl get svc my-service

# Find node IP
kubectl get nodes -o wide

# Test external access
curl http://<node-ip>:30007

# Check which pods are receiving traffic
kubectl get pods -l app=my-app-np
kubectl logs -l app=my-app-np --follow

# Clean up
kubectl delete -f Nodeport.yml
```

### 3.3 LoadBalancer Service Testing

```bash
# Deploy LoadBalancer service (requires cloud environment)
kubectl apply -f LoadBalancer.yml

# Wait for external IP provisioning
kubectl get svc my-app-lb --watch

# Check service details
kubectl describe svc my-app-lb

# Test external access (once EXTERNAL-IP is assigned)
curl http://<external-ip>

# Monitor load balancing
kubectl get pods -l app=my-app-lb
kubectl logs -l app=my-app-lb --follow

# Clean up
kubectl delete -f LoadBalancer.yml
```

## 🔄 Service Evolution & Best Practices

### Service Type Selection Guide

```
Development → NodePort (quick access, testing)
Staging → ClusterIP + Ingress (controlled external access)
Production → LoadBalancer (cloud-native, scalable)
```

### Common Patterns

1. **Internal Services**: Use ClusterIP for microservice communication
2. **External APIs**: Use LoadBalancer for public APIs
3. **Development**: Use NodePort for quick testing
4. **Hybrid**: Combine multiple service types in same application

### Service Best Practices

- **Use Labels Effectively**: Consistent labeling for service discovery
- **Resource Limits**: Set appropriate resource requests/limits
- **Health Checks**: Implement readiness and liveness probes
- **Security**: Use NetworkPolicies to control traffic
- **Monitoring**: Monitor service metrics and pod health



## 🔧 Troubleshooting Services

### Common Issues

1. **Service Not Accessible**
   ```bash
   # Check service exists
   kubectl get svc

   # Verify selector matches pods
   kubectl get pods --show-labels

   # Check endpoints
   kubectl get endpoints
   ```

2. **LoadBalancer Pending**
   ```bash
   # Check cloud provider integration
   kubectl describe svc <service-name>

 



## 🐛 Troubleshooting

### Common Issues

1. **Image Pull Errors**
   ```bash
   kubectl describe pod <pod-name>
   # Check image name and registry access
   ```

2. **Pod Pending Status**
   ```bash
   kubectl describe pod <pod-name>
   # Check node resources and scheduling constraints
   ```

3. **Service Not Accessible**
   ```bash
   kubectl get endpoints
   # Verify pods are matching service selectors
   ```

4. **DNS Resolution Issues**
   ```bash
   kubectl exec -it <pod-name> -- nslookup <service-name>
   # Check CoreDNS pods: kubectl get pods -n kube-system
   ```

### Health Checks

```bash
# Check node status
kubectl get nodes

# Check system pods
kubectl get pods -n kube-system

# Check resource usage
kubectl top nodes
kubectl top pods
```
