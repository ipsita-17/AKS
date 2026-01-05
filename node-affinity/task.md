Task 1: Pod with nginx + NodeAffinity disktype=ssd

1️⃣ Create nginx pod with requiredDuringSchedulingIgnoredDuringExecution

nginx-node-affinity.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-node-affinity
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

Apply it:

kubectl apply -f nginx-node-affinity.yaml

2️⃣ Check pod status (why it is not scheduled):

kubectl get pod nginx-node-affinity
kubectl describe pod nginx-node-affinity

3️⃣ Add label to worker01:

kubectl label node worker01 disktype=ssd

Verify:

kubectl get nodes --show-labels

4️⃣ Check pod status again:

kubectl get pod nginx-node-affinity -o wide

✅ Task 2: Pod with redis + NodeAffinity disktype (key only, no value)

5️⃣ Create redis pod with key-only node affinity

redis-node-affinity.yaml

apiVersion: v1
kind: Pod
metadata:
  name: redis-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: Exists
  containers:
  - name: redis
    image: redis

Apply it:

kubectl apply -f redis-node-affinity.yaml

6️⃣ Add label to worker02 without value

kubectl label node worker02 disktype

Verify:

kubectl get nodes --show-labels

7️⃣ Check redis pod placement:

kubectl get pod redis-node-affinity -o wide
