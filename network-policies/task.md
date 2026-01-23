1) Create a Kind cluster without default CNI

Create a config file:

# kind-nocni.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  podSubnet: "192.168.0.0/16"
nodes:
  - role: control-plane
  - role: worker


Create the cluster:

kind create cluster --name calico-cluster --config kind-nocni.yaml
kubectl cluster-info --context kind-calico-cluster

2) Install Calico CNI
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml


Wait until Calico pods are ready:

kubectl get pods -n kube-system -w

3) Create the Deployments
Frontend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx
        ports:
        - containerPort: 80

kubectl apply -f frontend.yaml

Backend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: nginx
        ports:
        - containerPort: 80

kubectl apply -f backend.yaml

DB (MySQL)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: db
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: rootpass
        ports:
        - containerPort: 3306

kubectl apply -f db.yaml

4) Expose all Deployments via NodePort
Frontend Service
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080

Backend Service
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: NodePort
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30081

DB Service
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  type: NodePort
  selector:
    app: db
  ports:
  - port: 3306
    targetPort: 3306
    nodePort: 30306

kubectl apply -f frontend-svc.yaml
kubectl apply -f backend-svc.yaml
kubectl apply -f db-svc.yaml

5) Connectivity Test (Before NetworkPolicy)

Exec into frontend and backend and test DB:

kubectl exec -it deploy/frontend -- sh
apt update && apt install -y netcat
nc -zv db 3306
exit

kubectl exec -it deploy/backend -- sh
apt update && apt install -y netcat
nc -zv db 3306
exit


✅ Expected: Both should connect successfully.

6) Create NetworkPolicy (Allow only backend → db:3306)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-db
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 3306

kubectl apply -f db-netpol.yaml

7) Attach Policy Logic to Backend (Label already matches)

Your backend pods already have:

labels:
  app: backend


So no deployment change is required — the policy already targets backend.

8) Re-test Connectivity (After NetworkPolicy)

From backend:

kubectl exec -it deploy/backend -- sh
nc -zv db 3306
exit


✅ Should still work

From frontend:

kubectl exec -it deploy/frontend -- sh
nc -zv db 3306
exit

❌ Should FAIL

Final Validation
kubectl get pods -o wide
kubectl get svc
kubectl get networkpolicy