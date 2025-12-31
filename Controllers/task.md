<a> ReplicaSet

1. Create a new Replicaset based on the nginx image with 3 replicas
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
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
        image: nginx
        ports:
        - containerPort: 80

kubectl apply -f nginx-replicaset.yml
kubectl get rs
kubectl get pods

2. Update the replicas to 4 from the YAML

kubectl apply -f nginx-replicaset.yml
kubectl get rs nginx-rs

3. Update the replicas to 6 from the command line

kubectl scale rs nginx-rs --replicas=6
kubectl get rs nginx-rs

<b> Deployment

1. Create a Deployment named nginx with 3 replicas. The Pods should use the nginx:1.23.0 image and the name nginx. The Deployment uses the label tier=backend. The Pod template should use the label app=v1.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: v1
  template:
    metadata:
      labels:
        app: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.0
        ports:
        - containerPort: 80

kubectl apply -f nginx-deployment.yml

kubectl get deployment nginx
kubectl get pods --show-labels

2. List the Deployment and ensure the correct number of replicas is running.

kubectl get deployment nginx
kubectl get pods

3. Update the image to nginx:1.23.4

kubectl set image deployment/nginx nginx=nginx:1.23.4

4. Verify the rollout to all replicas

Check rollout status:

kubectl rollout status deployment/nginx

Verify Pods are using the new image:

kubectl get pods -o wide

5. Assign the change cause

"Pick up patch version"

kubectl annotate deployment/nginx \
kubernetes.io/change-cause="Pick up patch version"

6. Scale the Deployment to 5 replicas

kubectl scale deployment nginx --replicas=5

Verify:

kubectl get deployment nginx
kubectl get pods

7. View the Deployment rollout history

kubectl rollout history deployment/nginx

To see details of a revision:

kubectl rollout history deployment/nginx --revision=2

8. Ensure Pods use image nginx:1.23.0

Verify:

kubectl get pods
kubectl describe pod <pod-name> | grep Image
