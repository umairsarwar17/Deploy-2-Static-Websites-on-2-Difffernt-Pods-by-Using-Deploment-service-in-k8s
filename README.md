## Step 1: System update
```
sudo apt update && sudo apt upgrade -y
```
---

## Step 2: Docker install (Kind ke liye required)

```
sudo apt install -y docker.io
```

Docker start karo:

```
sudo systemctl enable docker  
sudo systemctl start docker
```

Check:

```
docker --version
```

---

##  Step 3: user ko docker group me add

```
sudo usermod -aG docker $USER  
newgrp docker
```

---

##  Step 4: kubectl install

```
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"  
  
chmod +x kubectl  
sudo mv kubectl /usr/local/bin/
```

Check:

```
kubectl version --client
```

---

## Step 5: Kind install

```
curl -Lo kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64  
chmod +x kind  
sudo mv kind /usr/local/bin/
```

Check:

```
kind version
```
##Part 2: Kubernetes Cluster Create (Kind)
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  # ================= MASTER NODE =================
  - role: control-plane
    extraPortMappings:
      - containerPort: 80
        hostPort: 8080
        protocol: TCP

      - containerPort: 443
        hostPort: 8443
        protocol: TCP

    extraMounts:
      - hostPath: /mnt/kind-data/master
        containerPath: /data

  # ================= WORKER 1 =================
  - role: worker
    extraPortMappings:
      - containerPort: 80
        hostPort: 8081
        protocol: TCP

    extraMounts:
      - hostPath: /mnt/kind-data/worker1
        containerPath: /data

  # ================= WORKER 2 =================
  - role: worker
    extraPortMappings:
      - containerPort: 80
        hostPort: 8082
        protocol: TCP

    extraMounts:
      - hostPath: /mnt/kind-data/worker2
        containerPath: /data

  # ================= WORKER 3 =================
  - role: worker
    extraPortMappings:
      - containerPort: 80
        hostPort: 8083
        protocol: TCP

    extraMounts:
      - hostPath: /mnt/kind-data/worker3
        containerPath: /data
```
Volume create krna in pods kay liya 
```
sudo mkdir -p /mnt/kind-data/master
sudo mkdir -p /mnt/kind-data/worker1
sudo mkdir -p /mnt/kind-data/worker2
sudo mkdir -p /mnt/kind-data/worker3
```

ab cluster bnana is config file ka:
```
kind create cluster --config config.yml
```
Check nodes:

```
kubectl get nodes
```

Else we also can create it by yml file like the file name is namespace.yml
```
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```
 now just need to apply here:
 ```
 kubectl apply -f namespace.yml
 ```

Now need to create 2 docker images for 
# Step 1: 2 HTML apps banao

```
mkdir multi-html && cd multi-html
```

---

## 📄 App 1

```
mkdir app1 && cd app1  
echo "<h1>This is App 1 - Pod 1</h1>" > index.html
```

...........
Dockerfile:

``` 
vi Dockerfile 
```
Then
```
FROM nginx:alpine  
COPY index.html /usr/share/nginx/html/index.html
```

Build:

```
docker build -t html-app1 .
```

---

## 📄 App 2

```
cd ..  
mkdir app2 && cd app2  
echo "<h1>This is App 2 - Pod 2</h1>" > index.html
```

Dockerfile:

```
FROM nginx:alpine  
COPY index.html /usr/share/nginx/html/index.html
```

Build:

```
docker build -t html-app2 .
```

---

##  Kind me images load karo

```
kind load docker-image html-app1 --name kind
kind load docker-image html-app2 --name kind
```
After that Create  Pods:
```
vi pod1.yaml
```
Write config file:
```
apiVersion: v1
kind: Pod
metadata:
  name: html-pod-2
  namespace: my-namespace
spec:
  containers:
  - name: app2
    image: html-app2
    ports:
    - containerPort: 80
```

check pods
```
kubectl get pods -n my-namespace
```
its time to create Deployment 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: html-app
  namespace: my-namespace
spec:
  replicas: 2
  selector:
    matchLabels:
      app: html
  template:
    metadata:
      labels:
        app: html
    spec:
      containers:
      - name: html
        image: html-app1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```
