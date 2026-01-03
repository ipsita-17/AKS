1. Create a Service named myapp of type ClusterIP that exposes port 80 and maps to the target port 80.

kubectl create service clusterip myapp \
  --tcp=80:80 \
  --dry-run=client -o yaml > service.yaml

service.yaml
------------
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80

kubectl apply -f service.yaml

2. Create a Deployment named myapp that creates 1 replica running the image nginx:1.23.4-alpine. Expose the container port 80.

deployment.yaml
----------------
kubectl create deployment myapp \
  --image=nginx:1.23.4-alpine \
  --replicas=1 \
  --dry-run=client -o yaml > deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
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
        image: nginx:1.23.4-alpine
        ports:
        - containerPort: 80

kubectl apply -f deployment.yaml

3. Scale the Deployment to 2 replicas.

kubectl scale deployment myapp --replicas=2
Check: kubectl get pods

4. Create a temporary Pod using the image busybox and run a wget command against the IP of the service.

kubectl get svc myapp

Run a temporary pod:
kubectl run test-pod \
  --image=busybox \
  --rm -it \
  --restart=Never \
  -- wget -qO- http://<SERVICE_CLUSTER_IP>:80

5. Run a wget command against the service outside the cluster.

This will NOT work because:

ClusterIP is internal-only
Not accessible outside the cluster

6. Change the service type so the Pods can be reached outside the cluster.

kubectl edit svc myapp

Change:

spec:
  type: NodePort

kubectl get svc myapp

7. Run a wget command against the service outside the cluster.

kubectl get nodes -o wide

localhost:30001

8. Discuss: Can you expose Pods as a Service without a Deployment?

Yes.

You can expose:

Standalone Pods

ReplicaSets

StatefulSets

DaemonSets

👉 A Service only needs labels, not a Deployment.

⚠️ But Deployments are recommended because:

Auto-healing

Scaling

Rolling updates

9. When to use different Service types?|

🔹 ClusterIP (default)

Internal communication only

Microservices talking to each other

Databases, backend APIs

🔹 NodePort

Simple external access

Dev / test environments

Exposes service on every node IP

🔹 LoadBalancer

Production workloads

Cloud environments (AKS, EKS, GKE)

Automatically provisions a cloud LB

🔹 ExternalName

Redirects traffic to external DNS

SaaS, external DBs

No pods involved
