# Kubernetes Learning Lab with Minikube on Windows

A hands-on guide to learning Kubernetes locally using **Docker Desktop**, **Minikube**, and **kubectl** on Windows.

---

# Table of Contents

* Prerequisites
* Install kubectl
* Install Minikube
* Start a Kubernetes Cluster
* Open Kubernetes Dashboard
* Explore the Cluster
* Deploy Your First Pod
* Deployments, ReplicaSets & Self-Healing
* Services & Networking
* YAML-Based Deployments
* Minikube Management Commands
* Useful Commands

---

# Prerequisites

Before starting, ensure the following are installed:

* Docker Desktop
* Windows 10/11
* PowerShell (Run as Administrator when required)

---

# Install kubectl

## Step 1: Download kubectl

Official Documentation:

https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

```powershell
curl.exe -LO "https://dl.k8s.io/release/v1.36.0/bin/windows/amd64/kubectl.exe"
```

## Step 2: Move kubectl.exe

Copy `kubectl.exe` to:

```text
C:\Windows\System32
```
or Add into `PATH` Environment variable.
## Step 3: Verify Installation

```powershell
kubectl version --client
```

or

```powershell
kubectl version --client --output=yaml
```

Expected output:

```yaml
clientVersion:
  buildDate: "2026-04-22T13:54:45Z"
  compiler: gc
  gitCommit: ecf6decece6a6de25a57aad9ba90b6ce580f6f78
  gitTreeState: clean
  gitVersion: v1.36.0
  goVersion: go1.26.2
  major: "1"
  minor: "36"
  platform: windows/amd64
kustomizeVersion: v5.8.1
```

---

# Install Minikube

## Step 1: Download Minikube

Official Documentation:

https://minikube.sigs.k8s.io/docs/start/

Create the installation directory and download Minikube:

```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force

$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest `
-OutFile 'c:\minikube\minikube.exe' `
-Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' `
-UseBasicParsing
```

---

## Step 2: Add Minikube to PATH

Run PowerShell as Administrator:

```powershell
$oldPath = [Environment]::GetEnvironmentVariable(
'Path',
[EnvironmentVariableTarget]::Machine
)

if ($oldPath.Split(';') -inotcontains 'C:\minikube')
{
    [Environment]::SetEnvironmentVariable(
    'Path',
    "$oldPath;C:\minikube",
    [EnvironmentVariableTarget]::Machine
    )
}
```

Open a new terminal and verify:

```powershell
minikube version
```

---

# Start a Kubernetes Cluster

Start Minikube using Docker Desktop:

```powershell
minikube start --driver=docker
```

```text
minikube start --driver=docker
* minikube v1.38.1 on Microsoft Windows 11 Home Single Language 24H2
* Using the docker driver based on user configuration
! Starting v1.39.0, minikube will default to "containerd" container runtime. See #21973 for more info.
* Using Docker Desktop driver with root privileges
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.50 ...
* Downloading Kubernetes v1.35.1 preload ...
    > preloaded-images-k8s-v18-v1...:  272.45 MiB / 272.45 MiB  100.00% 1.23 Mi
    > gcr.io/k8s-minikube/kicbase...:  519.58 MiB / 519.58 MiB  100.00% 1.58 Mi
* Creating docker container (CPUs=2, Memory=6000MB) ...
* Preparing Kubernetes v1.35.1 on Docker 29.2.1 ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Configure Docker as the default driver:

```powershell
minikube config set driver docker
```
```text
C:\Users\pande>minikube config set driver docker
! These changes will take effect upon a minikube delete and then a minikube start
```
> Changes take effect after running `minikube delete` followed by `minikube start`.

---

# Verify Cluster Creation

Check cluster status:

```powershell
minikube status
```

Expected:

```text
host: Running
kubelet: Running
apiserver: Running
```

Verify node:

```powershell
kubectl get nodes
```

Output:

```text
NAME       STATUS   ROLES           VERSION
minikube   Ready    control-plane   v1.35.1
```

---

# Docker Desktop Integration

Open Docker Desktop:

```text
Docker Desktop
 └── Containers
      └── minikube
```

You should see a running container named:

```text
minikube
```

---

# Kubernetes Dashboard

Minikube includes a built-in Kubernetes Dashboard.

Launch Dashboard:

```powershell
minikube dashboard
```

Get Dashboard URL:

```powershell
minikube dashboard --url
```

Example:

```text
http://127.0.0.1:51704/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

---

# Explore the Cluster

## List Nodes

```powershell
kubectl get nodes
```
```text
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   6m59s   v1.35.1
```

---

## List Namespaces

```powershell
kubectl get namespaces
```
```text
kubectl get namespaces
NAME              STATUS   AGE
default           Active   10m
kube-node-lease   Active   10m
kube-public       Active   10m
kube-system       Active   10m
```
---

## List All Pods

```powershell
kubectl get pods -A
```
```text
kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-7d764666f9-2jzfv           1/1     Running   0             11m
kube-system   etcd-minikube                      1/1     Running   0             11m
kube-system   kube-apiserver-minikube            1/1     Running   0             11m
kube-system   kube-controller-manager-minikube   1/1     Running   0             11m
kube-system   kube-proxy-7x2tm                   1/1     Running   0             11m
kube-system   kube-scheduler-minikube            1/1     Running   0             11m
kube-system   storage-provisioner                1/1     Running   1 (10m ago)   11m
```
---

## Cluster Information

```powershell
kubectl cluster-info
```

```text
kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:50118
CoreDNS is running at https://127.0.0.1:50118/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

---

# Deploy Your First Pod

## Create an NGINX Pod

```powershell
kubectl run nginx --image=nginx --restart=Never
```

---

## Verify Pod

```powershell
kubectl get pods
```

---

## Describe Pod

```powershell
kubectl describe pod nginx
```

---

## View Logs

```powershell
kubectl logs nginx
```

---

## To edit index.html

```powershell
kubectl exec -it nginx -- /bin/bash
apt-get update && apt-get install vim -y
vi /usr/share/nginx/html/index.html
exit

minikube service nginx-service --url
```

---

## Delete Pod

```powershell
kubectl delete pod nginx
```

```text
kubectl run nginx --image=nginx --restart=Never
pod/nginx created

C:\Users\pande>kubectl get pods -A
NAMESPACE              NAME                                         READY   STATUS              RESTARTS      AGE
default                nginx                                        0/1     ContainerCreating   0             24s
kube-system            coredns-7d764666f9-2jzfv                     1/1     Running             0             24m
kube-system            etcd-minikube                                1/1     Running             0             24m
kube-system            kube-apiserver-minikube                      1/1     Running             0             24m
kube-system            kube-controller-manager-minikube             1/1     Running             0             24m
kube-system            kube-proxy-7x2tm                             1/1     Running             0             24m
kube-system            kube-scheduler-minikube                      1/1     Running             0             24m
kube-system            storage-provisioner                          1/1     Running             1 (23m ago)   24m
kubernetes-dashboard   dashboard-metrics-scraper-5565989548-9l256   1/1     Running             0             10m
kubernetes-dashboard   kubernetes-dashboard-b84665fb8-c5lwl         1/1     Running             0             10m

C:\Users\pande>kubectl describe pod nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Tue, 16 Jun 2026 10:11:08 +0530
Labels:           run=nginx
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Containers:
  nginx:
    Container ID:
    Image:          nginx
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4vjj6 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   False
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-4vjj6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  35s   default-scheduler  Successfully assigned default/nginx to minikube
  Normal  Pulling    34s   kubelet            spec.containers{nginx}: Pulling image "nginx"

C:\Users\pande>kubectl logs nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/06/16 04:41:53 [notice] 1#1: using the "epoll" event method
2026/06/16 04:41:53 [notice] 1#1: nginx/1.31.1
2026/06/16 04:41:53 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/06/16 04:41:53 [notice] 1#1: OS: Linux 6.6.114.1-microsoft-standard-WSL2
2026/06/16 04:41:53 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/06/16 04:41:53 [notice] 1#1: start worker processes
2026/06/16 04:41:53 [notice] 1#1: start worker process 29
2026/06/16 04:41:53 [notice] 1#1: start worker process 30
2026/06/16 04:41:53 [notice] 1#1: start worker process 31
2026/06/16 04:41:53 [notice] 1#1: start worker process 32
2026/06/16 04:41:53 [notice] 1#1: start worker process 33
2026/06/16 04:41:53 [notice] 1#1: start worker process 34
2026/06/16 04:41:53 [notice] 1#1: start worker process 35
2026/06/16 04:41:53 [notice] 1#1: start worker process 36
2026/06/16 04:41:53 [notice] 1#1: start worker process 37
2026/06/16 04:41:53 [notice] 1#1: start worker process 38
2026/06/16 04:41:53 [notice] 1#1: start worker process 39
2026/06/16 04:41:53 [notice] 1#1: start worker process 40
2026/06/16 04:41:53 [notice] 1#1: start worker process 41
2026/06/16 04:41:53 [notice] 1#1: start worker process 42
2026/06/16 04:41:53 [notice] 1#1: start worker process 43
2026/06/16 04:41:53 [notice] 1#1: start worker process 44
2026/06/16 04:41:53 [notice] 1#1: start worker process 45
2026/06/16 04:41:53 [notice] 1#1: start worker process 46

C:\Users\pande>kubectl delete pod nginx
pod "nginx" deleted from default namespace
```

---

# Deployments, ReplicaSets & Self-Healing

Deployments manage Pods and automatically recreate failed Pods.

## Create Deployment

```powershell
kubectl create deployment web --image=nginx
```

---

## Scale Deployment

```powershell
kubectl scale deployment web --replicas=2
```

---

## Verify

```powershell
kubectl get deployments

kubectl get pods
```

Expected:

```text
NAME   READY
web    2/2
```

---

## Test Self-Healing

Delete a Pod:

```powershell
kubectl delete pod <pod-name>
```

Example:

```powershell
kubectl delete pod web-68d995574f-h2vxw
```

Watch Kubernetes recreate it:

```powershell
kubectl get pods -w
```
```text
C:\Users\pande>kubectl create deployment web --image=nginx
deployment.apps/web created

C:\Users\pande>kubectl scale deployment web --replicas=2
deployment.apps/web scaled

C:\Users\pande>kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    2/2     2            2           35s

C:\Users\pande>kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
web-68d995574f-h2vxw   1/1     Running   0          32s
web-68d995574f-pz5kd   1/1     Running   0          44s

C:\Users\pande>kubectl delete pod web-68d995574f-h2vxw
pod "web-68d995574f-h2vxw" deleted from default namespace

C:\Users\pande>kubectl get pods -w
NAME                   READY   STATUS    RESTARTS   AGE
web-68d995574f-lc5hx   1/1     Running   0          20s
web-68d995574f-pz5kd   1/1     Running   0          112s
```
---

# Services & Networking

Expose the Deployment using a Service.

## Create Service

```powershell
kubectl expose deployment web --port=80 --type=NodePort
```

---

## Verify Service

```powershell
kubectl get svc
```

Example:

```text
NAME   TYPE       PORT(S)
web    NodePort   80:31514/TCP
```

---

## Access Application

```powershell
minikube service web
```

```text
C:\Users\pande>kubectl expose deployment web --port=80 --type=NodePort
service/web exposed

C:\Users\pande>kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        52m
web          NodePort    10.96.243.34   <none>        80:31514/TCP   35s

C:\Users\pande>minikube service web
┌───────────┬──────┬─────────────┬───────────────────────────┐
│ NAMESPACE │ NAME │ TARGET PORT │            URL            │
├───────────┼──────┼─────────────┼───────────────────────────┤
│ default   │ web  │ 80          │ http://192.168.49.2:31514 │
└───────────┴──────┴─────────────┴───────────────────────────┘
* Starting tunnel for service web.
┌───────────┬──────┬─────────────┬────────────────────────┐
│ NAMESPACE │ NAME │ TARGET PORT │          URL           │
├───────────┼──────┼─────────────┼────────────────────────┤
│ default   │ web  │             │ http://127.0.0.1:55186 │
└───────────┴──────┴─────────────┴────────────────────────┘
* Opening service default/web in default browser...
! Because you are using a Docker driver on windows, the terminal needs to be open to run it.
```

Open the URL in your browser to view the NGINX welcome page.

---

# YAML-Based Deployments

Kubernetes is typically managed using YAML manifests.

Create a file named:

```text
deployment.yaml
```

Contents:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: myapp

spec:
  replicas: 3

  selector:
    matchLabels:
      app: myapp

  template:
    metadata:
      labels:
        app: myapp

    spec:
      containers:
      - name: nginx
        image: nginx:latest

        ports:
        - containerPort: 80
```

---

## Apply Configuration

```powershell
kubectl apply -f deployment.yaml
```

---

## Verify

```powershell
kubectl get deploy

kubectl get pods
```

Expected:

```text
myapp   3/3
```

---

## Delete Resources

```powershell
kubectl delete -f deployment.yaml
```
```text
C:\Docker_&k8s>kubectl apply -f deployment.yaml
deployment.apps/myapp created

C:\Docker_&k8s>kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
myapp   0/3     3            0           9s
web     2/2     2            2           56m

C:\Docker_&k8s>kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-7c5c65b47b-5wnbc   1/1     Running   0          17s
myapp-7c5c65b47b-csx5b   1/1     Running   0          17s
myapp-7c5c65b47b-q7zk9   1/1     Running   0          17s
web-68d995574f-lc5hx     1/1     Running   0          54m
web-68d995574f-pz5kd     1/1     Running   0          56m

C:\Docker_&k8s>kubectl delete -f deployment.yaml
deployment.apps "myapp" deleted from default namespace

C:\Docker_&k8s>kubectl get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    2/2     2            2           57m

C:\Docker_&k8s>kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
web-68d995574f-lc5hx   1/1     Running   0          55m
web-68d995574f-pz5kd   1/1     Running   0          57m
```
---

# Minikube Management Commands

## Check Status

```powershell
minikube status
```
```text
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
---

## Stop Cluster

```powershell
minikube stop
```

Expected:

```text
host: Stopped
kubelet: Stopped
apiserver: Stopped
```

---

## Start Cluster Again

```powershell
minikube start
```

---

## Delete Cluster

```powershell
minikube delete
```
```text
* Deleting "minikube" in docker ...
* Deleting container "minikube" ...
* Removing C:\Users\pande\.minikube\machines\minikube ...
* Removed all traces of the "minikube" cluster.
```
> This removes the entire Kubernetes cluster.

---


# Useful Commands Cheat Sheet

```powershell
kubectl get nodes

kubectl get namespaces

kubectl get pods -A

kubectl cluster-info

kubectl logs <pod-name>

kubectl describe pod <pod-name>

kubectl delete pod <pod-name>

kubectl get deployments

kubectl scale deployment <name> --replicas=2

kubectl get svc

minikube dashboard

minikube service <service-name>

minikube status

minikube stop

minikube start

minikube delete
```

---

Happy Learning Kubernetes 🚀
