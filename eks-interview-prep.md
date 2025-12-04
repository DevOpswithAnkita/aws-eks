# Complete AWS EKS Interview Guide & Architecture

## What is Amazon EKS?

**Amazon Elastic Kubernetes Service (EKS)** is a fully managed Kubernetes service that makes it easy to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane or nodes.

### Key Features
- **Managed Control Plane**: AWS manages the Kubernetes control plane (API server, etcd, scheduler)
- **High Availability**: Control plane runs across multiple AZs
- **Automatic Updates**: AWS handles patching and upgrading
- **AWS Integration**: Native integration with AWS services (IAM, VPC, ELB, EBS, etc.)
- **Kubernetes Conformant**: Runs standard Kubernetes applications

---

## EKS Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AWS Account                          │
│                                                         │
│  ┌───────────────────────────────────────────────────   │
│  │         EKS Control Plane (AWS Managed)           │  │
│  │  ┌──────────────┐  ┌──────────────┐               │  │
│  │  │ API Server   │  │   etcd       │               │  │
│  │  └──────────────┘  └──────────────┘               │  │
│  │  ┌──────────────┐  ┌──────────────┐               │  │
│  │  │  Scheduler   │  │ Controller   │               │  │
│  │  │              │  │  Manager     │               │  │
│  │  └──────────────┘  └──────────────┘               │  │
│  │           (Multi-AZ for HA)                       │  │
│  └───────────────────────────────────────────────────   │
│                         │                               │ 
│                    API calls                            │
│                         │                               │
│  ┌───────────────────────────────────────────────────   │
│  │      VPC (Customer Account)                      │   │
│  │                                                  │   │
│  │  ┌─────────────────────────────────────────┐     │   │
│  │  │  Availability Zone 1                    │     │   │
│  │  │  ┌────────────┐  ┌────────────┐         │     │   │
│  │  │  │ Worker     │  │ Worker     │         │     │   │
│  │  │  │ Node 1     │  │ Node 2     │         │     │   │
│  │  │  │  (EC2)     │  │  (EC2)     │         │     │   │
│  │  │  └────────────┘  └────────────┘         │     │   │
│  │  └─────────────────────────────────────────┘     │   │
│  │                                                  │   │
│  │  ┌─────────────────────────────────────────┐     │   │
│  │  │  Availability Zone 2                    │     │   │
│  │  │  ┌────────────┐  ┌────────────┐         │     │   │
│  │  │  │ Worker     │  │ Fargate    │         │     │   │
│  │  │  │ Node 3     │  │ Pod        │         │     │   │
│  │  │  │  (EC2)     │  │            │         │     │   │
│  │  │  └────────────┘  └────────────┘         │     │   │
│  │  └─────────────────────────────────────────┘     │   │
│  │                                                  │   │
│  │  ┌─────────────────────────────────────────┐     │   │
│  │  │        AWS Services Integration         │     │   │
│  │  │  • ALB/NLB • EBS • EFS • ECR            │     │   │
│  │  │  • IAM • CloudWatch • Route53           │     │   │
│  │  └─────────────────────────────────────────┘     │   │
│  └───────────────────────────────────────────────────   │
└─────────────────────────────────────────────────────────┘
```

### Components Explained

#### 1. Control Plane (AWS Managed)
- **API Server**: Entry point for all REST commands to control the cluster
- **etcd**: Distributed key-value store for cluster data
- **Scheduler**: Assigns pods to nodes
- **Controller Manager**: Runs controller processes
- Runs across 3 AZs for high availability
- AWS handles all maintenance, patching, and scaling

#### 2. Data Plane (Worker Nodes)
- **EC2 Nodes**: Self-managed or EKS-managed node groups
- **Fargate**: Serverless compute for pods
- **Auto Mode Nodes**: Fully AWS-managed with automatic lifecycle

#### 3. Networking
- **VPC CNI**: Assigns VPC IP addresses to pods
- **CoreDNS**: Service discovery and DNS
- **Network Policies**: Pod-to-pod communication control
- **Load Balancers**: ALB/NLB for external access

#### 4. Storage
- **EBS CSI Driver**: Persistent block storage
- **EFS CSI Driver**: Shared file storage
- **FSx for Lustre**: High-performance computing

#### 5. Security
- **IAM Roles for Service Accounts (IRSA)**: Pod-level IAM permissions
- **Pod Identity**: Simplified IAM authentication
- **Secrets Manager**: Credential storage
- **KMS**: Encryption at rest

---

## EKS Auto Mode

EKS Auto Mode, announced at AWS re:Invent 2024, fully automates compute, storage, and networking management for Kubernetes clusters.

### What is EKS Auto Mode?

EKS Auto Mode streamlines EKS operations by automating key infrastructure components including compute autoscaling, pod and service networking, application load balancing, cluster DNS, block storage, and GPU support.

### Key Capabilities

1. **One-Click Cluster Setup**: Deploy production-ready clusters with minimal operational overhead

2. **Automated Compute Management**:
   - Dynamically adds or removes nodes based on application demands
   - Automatically selects optimal EC2 instances for your workloads

3. **Enhanced Security**:
   - Uses immutable AMIs with locked-down software
   - SELinux mandatory access controls and read-only root file systems
   - Nodes have a maximum lifetime of 21 days and are automatically replaced

4. **Automated Upgrades**:
   - Keeps cluster, nodes, and components up to date
   - Respects Pod Disruption Budgets and NodePool Disruption Budgets

5. **Built-in Components**:
   - Includes Kubernetes and AWS features as core components
   - Pod IP assignments, network policies, local DNS, GPU plugins, health checkers, and EBS CSI storage

6. **Cost Optimization**:
   - Continuously selects and refines the mix of EC2 instances
   - Maximizes performance and minimizes costs

### EKS Auto Mode vs Traditional EKS

| Aspect | Traditional EKS | EKS Auto Mode |
|--------|----------------|---------------|
| Control Plane | AWS Managed | AWS Managed |
| Data Plane | Customer Managed | AWS Managed |
| Compute Scaling | Manual (CA/Karpenter) | Automatic |
| Add-ons | Manual Installation | Pre-installed |
| Node Patching | Manual | Automatic |
| Node Lifetime | Indefinite | Max 21 days |
| Setup Time | Hours | Minutes |

---

## Karpenter vs Cluster Autoscaler

### What is Karpenter?

Karpenter is an intelligent, fast autoscaler for EKS that creates new EC2 nodes automatically when pods are unschedulable. It selects the correct instance type based on pod requirements, reducing cost and improving cluster performance.

### Comparison

**Cluster Autoscaler (Traditional)**
- Works with fixed node groups (ASG / Managed Nodegroups)
- Scales only the number of nodes inside those node groups
- Limited to predefined instance types
- Slower scaling (10–15 minutes)
- Requires manual configuration and management of node groups

**Karpenter**
- Pod-driven scaling (creates nodes based on pending pods)
- Does not depend on node groups – can launch any EC2 instance type dynamically
- Much faster scaling (seconds to a couple of minutes)
- Automatically picks optimal instance types
- Better cost optimization and bin-packing

**Karpenter "Auto Mode" (AWS managed version)**
- Fully managed by AWS
- No installation or upgrades required
- Automatically integrates with EKS
- Automatically selects and places nodes efficiently

---

## Deploying Applications on EKS

### Option 1: EKS with EC2 Nodes (Traditional)

**Step 1: Create Cluster**
```bash
# Using eksctl
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

**Step 2: Configure kubectl**
```bash
aws eks update-kubeconfig --region us-east-1 --name my-cluster
```

**Step 3: Create Deployment**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

**Step 4: Create Service (LoadBalancer)**
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Step 5: Deploy**
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Get Load Balancer URL
kubectl get service nginx-service
```

### Option 2: EKS with Fargate (Serverless)

**Step 1: Create Fargate Profile**
```bash
eksctl create fargateprofile \
  --cluster my-cluster \
  --name my-fargate-profile \
  --namespace default
```

**Step 2: Deploy Application**
```yaml
# Same deployment.yaml and service.yaml as above
# Pods will automatically run on Fargate
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Option 3: EKS Auto Mode (Newest)

**Step 1: Create Auto Mode Cluster**
```bash
eksctl create cluster \
  --name auto-cluster \
  --region us-east-1 \
  --compute-config autoMode=ENABLED
```

**Step 2: Deploy Application**
```bash
# Auto Mode handles everything automatically
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# That's it! Auto Mode will:
# - Provision optimal EC2 instances
# - Scale automatically
# - Handle networking and load balancing
# - Update and patch nodes
```

### Complete Application Deployment Example

**1. Dockerfile**
```dockerfile
FROM node:18
WORKDIR /app
COPY server.js .
RUN npm install express
CMD ["node", "server.js"]
```

**2. Build and Push to ECR**
```bash
# Authenticate to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com

# Build image
docker build -t my-app:latest .

# Tag and push
docker tag my-app:latest \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
```

**3. Kubernetes Manifests**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: customer-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: customer-prod
  template:
    metadata:
      labels:
        app: customer-prod
    spec:
      containers:
      - name: customer-container
        image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "256Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"
            cpu: "1000m"

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: customer-service
spec:
  type: LoadBalancer
  selector:
    app: customer-prod
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
```

**4. Deploy**
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Wait and get external URL
kubectl get service customer-service
```

**5. Map to Domain (Route53)**
```bash
# Get Load Balancer DNS
LB_DNS=$(kubectl get service customer-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

# Create Route53 record (manual or via CLI)
# customer.yourdomain.com -> $LB_DNS (CNAME or Alias)
```

---

## Interview Questions & Answers

### Q1: What is the difference between ECS, EKS, and Fargate?

**Answer:**
- **ECS**: AWS-native container orchestration service. Simpler, AWS-specific
- **EKS**: Managed Kubernetes service. Open-source standard, portable
- **Fargate**: Serverless compute engine that works with both ECS and EKS. You don't manage servers

---

### Q2: How do you expose an EKS application to the internet?

**Answer:**

Three main methods:

1. **Service of type LoadBalancer**: Automatically creates an AWS ELB
2. **Ingress with ALB**: Use AWS Load Balancer Controller for ALB
3. **NodePort**: Expose on node IP (not recommended for production)

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer  # Creates AWS ELB
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
```

**Traffic Flow:** User → Route53 (DNS) → ELB → EKS Service → Pods

---

### Q3: How do you secure an EKS cluster?

**Answer:**

1. **IAM + RBAC**: Map IAM roles to Kubernetes users with fine-grained permissions
2. **Secrets Management**: Use AWS Secrets Manager or SSM for credentials
3. **MFA**: Enable Multi-Factor Authentication for all AWS IAM users
4. **Private API Endpoint**: Restrict public access to EKS control plane
5. **Network Policies**: Control pod-to-pod communication
6. **Pod Security**: Use PodSecurityPolicy or OPA Gatekeeper
7. **Encryption**: Use KMS for secret encryption and encrypt etcd
8. **Rolling Updates**: Use rolling updates to avoid zero-day issues
9. **Monitoring**: Use CloudTrail, GuardDuty, and audit logs

---

### Q4: How do you handle cluster upgrades in EKS?

**Answer:**

**Strategy 1: Control Plane First**
1. Upgrade control plane (API server, scheduler, controller)
2. Ensure compatibility with current node version
3. Create new node group with upgraded version
4. Drain old nodes: `kubectl drain <node-name> --ignore-daemonsets`
5. Pods reschedule automatically to new nodes
6. Delete old node group after verification

**Strategy 2: Blue-Green Deployment**
- Create new node group with upgraded AMI
- Gradually migrate workloads
- Delete old node group

**Strategy 3: Taint + Cordon + Drain**
- Cordon node: `kubectl cordon <node>` (prevent new pods)
- Drain node: `kubectl drain <node>` (evict existing pods)
- Upgrade, uncordon, and remove taint

**Strategy 4: Use Auto Mode**
- One-click cluster upgrade
- Auto Mode handles node migration automatically

**Best Practices:**
- Use PodDisruptionBudgets (PDBs)
- Use node affinity and tolerations carefully
- Respect anti-affinity to spread pods across nodes

**A PodDisruptionBudget (PDB)** protects your application pods from going below a minimum number during voluntary disruptions, such as:

- Node draining
- Cluster upgrades
- Karpenter scaling events
- A human running: kubectl drain <node>

**PDB ensures your application always has enough pods running, so the service does not go down.**

---

### Q5: What is IAM Roles for Service Accounts (IRSA)?

**Answer:**

IRSA allows Kubernetes pods to assume IAM roles for accessing AWS services without using node-level permissions or hardcoded credentials.

**How it works:**
1. Create IAM role with trust policy for OIDC provider
2. Annotate Kubernetes ServiceAccount with IAM role ARN
3. Pod automatically gets temporary AWS credentials

**Example:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-reader
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/S3ReaderRole
---
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: s3-reader
  containers:
  - name: app
    image: my-app:latest
```

Now this pod can access S3 without hardcoded credentials!

---

### Q6: What is the VPC CNI plugin in EKS?

**Answer:**

The VPC CNI (Container Network Interface) assigns AWS VPC IP addresses directly to Kubernetes pods, enabling pod-level networking.

**Benefits:**
- Pods get real VPC IPs
- Security groups can be applied at pod level
- No NAT required for pod-to-pod communication
- Better performance than overlay networks

**How it works:**
- Uses Elastic Network Interfaces (ENIs) on EC2 instances
- Secondary IP addresses from ENI are assigned to pods
- Maximum pods per node = (ENI limit × IPs per ENI) - reserved

---

### Q7: How do you implement CI/CD for EKS applications?

**Answer:**

**Pipeline Flow:**
```
Developer Push → GitHub → CodePipeline → CodeBuild 
  → Build Docker Image → Push to ECR 
  → Update Kubernetes Manifests → kubectl apply 
  → Rolling Update in EKS
```

**Example using GitOps (ArgoCD):**
1. Developer pushes code to Git
2. CI builds Docker image and pushes to ECR
3. CI updates Kubernetes manifests in Git repo (new image tag)
4. ArgoCD detects Git changes
5. ArgoCD applies changes to EKS cluster automatically

**Deployment Strategies:**
- **Rolling Update**: Default, gradual replacement
- **Blue-Green**: Deploy new version, switch traffic
- **Canary**: Gradually shift traffic to new version

---

### Q8: What is the difference between Deployment, StatefulSet, and DaemonSet?

**Answer:**

| Type | Use Case | Pod Identity | Storage | Example |
|------|----------|--------------|---------|---------|
| **Deployment** | Stateless apps | Interchangeable | Ephemeral | Web servers, APIs |
| **StatefulSet** | Stateful apps | Unique, stable | Persistent | Databases, Kafka |
| **DaemonSet** | One pod per node | One per node | Usually none | Log collectors, monitoring agents |

---

### Q9: How do you troubleshoot a pod that won't start?

**Answer:**

**Step-by-step debugging:**

1. **Check pod status:**
```bash
kubectl get pods
kubectl describe pod <pod-name>
```

2. **Check events:**
```bash
kubectl get events --sort-by='.lastTimestamp'
```

3. **Check logs:**
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # Previous crashed container
```

4. **Check resources:**
```bash
kubectl top nodes
kubectl top pods
```

5. **Common issues:**
- **ImagePullBackOff**: Wrong image name or ECR authentication issue
- **CrashLoopBackOff**: Application error, check logs
- **Pending**: Insufficient resources or node selector mismatch
- **ErrImagePull**: Image doesn't exist or registry issues

6. **Debug interactively:**
```bash
kubectl exec -it <pod-name> -- /bin/bash
```
---

### Q10: Your EKS application needs to access S3. What's the best approach?

**Answer:**

Use **IRSA (IAM Roles for Service Accounts)** - it's the most secure method.

**Steps:**
1. Create IAM role with S3 access policy
2. Create OIDC provider for EKS cluster
3. Create ServiceAccount with IAM role annotation
4. Use ServiceAccount in pod spec

**Why not alternatives?**
- Node IAM role: Too broad, all pods get access
- Hardcoded credentials: Security risk, rotation issues
- IRSA: Pod-level permissions, automatic credential rotation
  
---

### Q11: How would you handle a sudden traffic spike in your EKS application?

**Answer:**

**Three-layer autoscaling:**

1. **Pod Autoscaling (HPA)**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

2. **Node Autoscaling**:
- Traditional: Cluster Autoscaler
- Modern: Karpenter
- Auto Mode: Automatic

3. **Application Load Balancer**: Handles traffic distribution, built-in with Auto Mode

**Result:** App scales from 2 to 10 pods, nodes auto-provision, traffic handled gracefully!

---

### Q12: Your company requires encryption at rest for all data. How do you implement this in EKS?

**Answer:**

**1. Encrypt etcd (Control Plane):**
```bash
aws eks create-cluster \
  --name my-cluster \
  --encryption-config \
    '[{"resources":["secrets"],"provider":{"keyArn":"arn:aws:kms:region:account:key/key-id"}}]'
```

**2. Encrypt EBS volumes:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: encrypted-gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
  kmsKeyId: arn:aws:kms:region:account:key/key-id
```

**3. Encrypt EFS:**
- Enable encryption when creating EFS file system
- Use KMS key

**Auto Mode:** Encryption is enabled by default!

---
### Q13: What is an OIDC Provider in EKS?

### **Definition**
An **OIDC Provider (OpenID Connect Provider)** in EKS is an identity service that verifies the **JWT tokens** issued for Kubernetes service accounts.

It allows AWS IAM to trust Kubernetes service accounts so that pods can securely assume IAM roles **without using access keys**.


**Why Do We Need OIDC?**

OIDC is required for **IRSA (IAM Roles for Service Accounts)**.

IRSA allows Kubernetes pods to access AWS services securely:

- S3  
- DynamoDB  
- SQS  
- Secrets Manager  
- ECR  
- CloudWatch  
- Parameter Store  

All this happens **without storing AWS credentials** inside containers.


**What Does an OIDC Provider Do?**

- Verifies Kubernetes service account tokens  
- Ensures the token is issued by the cluster  
- Allows IAM roles to trust Kubernetes  
- Enables pods to assume IAM roles securely  



**Command to Enable OIDC Provider**

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster <cluster-name> \
  --approve
```
---

### Q14: How Do You Configure Monitoring in EKS?

Monitoring in EKS has two major parts:

1. **Control Plane Monitoring (CloudWatch Logs)**
2. **Worker Node and Pod Monitoring (Prometheus + Grafana)**


**2.1 Control Plane Monitoring (CloudWatch)**

EKS control plane logs include:

- API Server  
- Audit Logs  
- Controller Manager  
- Scheduler  

 **Enable Control Plane Logs**

```bash
eksctl utils update-cluster-logging \
  --cluster <cluster-name> \
  --enable-types all \
  --approve
  ```

**2.2 Worker Node + Pod Monitoring in EKS**

     **A) Install Metrics Server**

Metrics Server provides resource metrics such as:

- Pod CPU/Memory  
- Node CPU/Memory  
- Required for Horizontal Pod Autoscaler (HPA)

 **Command:**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
  **B) Install Prometheus + Grafana (Best Practice)**

Install using Helm:
      
      ```bash
      helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
      helm install kube-prometheus prometheus-community/kube-prometheus-stack \
        -n monitoring --create-namespace
      ```
  **This installs:**

    - Prometheus (metrics collection)
    - Grafana (dashboard visualization)
    - Node Exporter
    - Kube State Metrics

  **Dashboards provided:**

    - Node performance
    - Pod CPU/Memory
    - API server metrics
    - Cluster health
    - Workload status
---

### Q15: What are the prerequisites for setting up and managing an EKS cluster?
- kubectl – Command-line tool for interacting with Kubernetes clusters.
- eksctl – CLI tool that simplifies creation and management of EKS clusters.
- AWS CLI – Command-line tool for working with AWS services; used to configure AWS credentials and interact with EKS and related resources.

---

### Q16: How do you set up an AWS Application Load Balancer (ALB) on an EKS cluster using the AWS Load Balancer Controller?

*Download IAM policy*

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

*Create IAM Policy*

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

*Create IAM Role*

```
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

**Deploy ALB controller**

*Add helm repo*

```
helm repo add eks https://aws.github.io/eks-charts
```

*Update the repo*

```
helm repo update eks
```

*Install*

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id>
```

**Verify that the deployments are running.**

```
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl logs -n kube-system deployment/aws-load-balancer-controller
```

*Note*
- Managed Kubernetes (EKS) → Use AWS Load Balancer Controller (AWS-specific Ingress Controller with ALBs). 
- Self-managed Kubernetes → Use generic Ingress Controller (NGINX, Traefik, etc.) and optionally a LoadBalancer solution.

---

## Scenario-Based Questions

### Scenario 1: Pod Communication Issues

**Question:** Pods in namespace A cannot communicate with services in namespace B. What could be wrong?

**Answer:**
- Check if Network Policies are blocking cross-namespace traffic
- Verify Security Groups for Pods configuration
- Ensure services are exposed correctly with ClusterIP
- Check DNS resolution: `nslookup service-name.namespace-b.svc.cluster.local`

---

### Scenario 2: Nodes Not Scaling

**Question:** Cluster Autoscaler is not adding nodes even though pods are pending. Why?

**Answer:**
- Check Auto Scaling Group max limits
- Verify IAM permissions for Cluster Autoscaler
- Ensure pods have resource requests defined
- Check if node selectors or taints are preventing scheduling
- Review Cluster Autoscaler logs: `kubectl logs -n kube-system deployment/cluster-autoscaler`

---

### Scenario 3: Authentication Failure

**Question:** Developers get "Unauthorized" error when accessing EKS cluster. How to fix?

**Answer:**
- Add their IAM role/user to aws-auth ConfigMap
- Command: `kubectl edit configmap aws-auth -n kube-system`
- Add entry under `mapRoles` or `mapUsers`
- Verify kubeconfig: `aws eks update-kubeconfig --name cluster-name`
- Check RBAC permissions with Role/RoleBinding

*mapRoles*

- Mapping worker node IAM Role → allows nodes to join the cluster and get proper permissions.
- Mapping IRSA IAM Role → allows pods to assume a role and access AWS services.

*mapUsers*

- Give direct access to a human IAM user to the EKS cluster.
- Example: admin user or DevOps engineer needing full access.

---

### Scenario 4: High Data Transfer Costs

**Question:** AWS bill shows high data transfer costs in EKS. How to reduce?

**Answer:**
- Enable topology-aware routing to keep traffic in same AZ
- Use `service.kubernetes.io/aws-load-balancer-type: nlb-ip` for internal traffic
- Deploy pods across AZs strategically
- Use VPC endpoints for AWS services
- Consider using NAT Gateway per AZ instead of single NAT Gateway

---

### Scenario 5: Cluster Upgrade Failed

**Question:** Control plane upgraded to 1.28 but nodes won't update. What's wrong?

**Answer:**
- EKS supports max 2 minor version difference between control plane and nodes
- Upgrade nodes incrementally (can't skip versions)
- Update node group AMI to compatible version
- Check for deprecated API versions in workloads
- Upgrade add-ons (VPC CNI, CoreDNS, kube-proxy) before node upgrade

---

### Scenario 6: EBS Volume Not Attaching

**Question:** StatefulSet pods stuck in Pending with "failed to provision volume" error. Solution?

**Answer:**
- Install EBS CSI driver: `aws eks create-addon --cluster-name my-cluster --addon-name aws-ebs-csi-driver`
- Create IAM policy for EBS CSI driver
- Attach policy to node role or use IRSA
- Verify StorageClass exists and is set as default
- Check if AZ where pod is scheduled has EBS availability

---

### Scenario 7: DNS Not Working

**Question:** Pods can't resolve service names but IP communication works. Fix?

**Answer:**
- Check CoreDNS pods: `kubectl get pods -n kube-system -l k8s-app=kube-dns`
- Increase CoreDNS resources if under load
- Verify CoreDNS ConfigMap is correct
- Test DNS: `kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default`
- Check if Network Policies are blocking DNS (port 53)

---

### Scenario 8: Suspicious Container Activity

**Question:** A container is making suspicious outbound connections. What to do?

**Answer:**
1. **Immediate**: Delete the pod: `kubectl delete pod <pod-name>`
2. **Investigate**: Check logs and events
3. **Review**: Check AWS GuardDuty and CloudTrail for unusual API calls
4. **Prevent**: Implement Network Policies to restrict egress traffic
5. **Secure**: Review IAM roles via IRSA, scan container images

---

### Scenario 9: Application Slow Performance

**Question:** Application response time suddenly increased 3x. Pods look healthy. Why?

**Answer:**
- Check if node has hit ENI/IP address limits: `kubectl describe node`
- Review VPC CNI metrics for IP exhaustion
- Check API server throttling in CloudWatch
- Verify EBS volume IOPS limits aren't exceeded
- Look for network throttling at instance level
- Check load balancer health and target health in AWS console

---

### Scenario 10: Regional Outage Failover

**Question:** Primary region is down. How to failover to DR cluster?

**Answer:**
1. **DNS**: Update Route53 to point to DR cluster load balancer
2. **Data**: Ensure RDS read replicas are promoted or restore from backup
3. **Application**: Deploy latest version using GitOps (ArgoCD/Flux)
4. **Verify**: Check all services: `kubectl get all --all-namespaces`
5. **Storage**: Restore PVs from Velero backup if needed
6. **Monitor**: Watch CloudWatch and application logs

---

### Scenario 11: Pod Evicted Frequently

**Question:** Pods keep getting evicted with "The node was low on resource: memory" message.

**Answer:**
- Check pod resource requests and limits are properly set
- Review node memory usage: `kubectl top nodes`
- Identify memory-hungry pods: `kubectl top pods --all-namespaces`
- Consider using larger instance types
- Implement Pod Disruption Budgets (PDB)
- Use Vertical Pod Autoscaler to right-size resources

---

### Scenario 12: LoadBalancer Service Timeout

**Question:** Service type LoadBalancer created but External-IP shows `<pending>` forever.

**Answer:**
- Check AWS Load Balancer Controller is installed
- Verify IAM permissions for Load Balancer Controller
- Check subnet tags: `kubernetes.io/role/elb=1` for public subnets
- Review security groups allow traffic on service ports
- Check service annotations are correct
- View controller logs: `kubectl logs -n kube-system deployment/aws-load-balancer-controller`

---

### Scenario 13: IRSA Not Working

**Question:** Pod can't access S3 even with IRSA configured. What's missing?

**Answer:**
- Verify OIDC provider is created for cluster
- Check ServiceAccount annotation: `eks.amazonaws.com/role-arn`
- Ensure IAM role trust policy includes OIDC provider
- Confirm IAM role has required S3 permissions
- Verify pod uses correct ServiceAccount
- Test: `kubectl exec <pod> -- aws sts get-caller-identity`

---

### Scenario 14: Image Pull Errors

**Question:** Pods show "ImagePullBackOff" error. How to resolve?

**Answer:**
- Check image name and tag are correct
- Verify ECR repository exists and image is pushed
- Ensure nodes have IAM role with ECR permissions (`AmazonEC2ContainerRegistryReadOnly`)
- For private registries, create imagePullSecrets
- Check image exists: `aws ecr describe-images --repository-name <repo>`
- Review pod events: `kubectl describe pod <pod-name>`

---

### Scenario 15: Ingress Not Working

**Question:** Created Ingress but can't access application. What to check?

**Answer:**
- Install AWS Load Balancer Controller for ALB Ingress
- Verify IngressClass is set correctly
- Check Ingress annotations match your setup
- Ensure backend service and pods are running
- Review security groups on ALB allow inbound traffic
- Check subnet tags: `kubernetes.io/role/elb=1`
- View Ingress status: `kubectl describe ingress <name>`
- Check ALB target group health in AWS console

---

## Quick Reference Commands

### Cluster Management
```bash
# Create cluster
eksctl create cluster --name my-cluster --region us-east-1

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-east-1

# Get contexts
kubectl config get-contexts
```

### Pod Management
```bash
# List all pods
kubectl get pods -A

# Describe pod
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name> -f
```
---

