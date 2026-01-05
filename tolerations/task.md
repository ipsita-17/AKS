1️⃣ Taint both worker nodes
Taint worker01:
kubectl taint nodes worker01 gpu=true:NoSchedule

Taint worker02:
kubectl taint nodes worker02 gpu=false:NoSchedule

Verify taints:
kubectl describe node worker01 | grep -i taint
kubectl describe node worker02 | grep -i taint

2️⃣ Create an nginx pod (without tolerations)
kubectl run nginx-pod --image=nginx

Check pod status:
kubectl get pods

3️⃣ Create nginx pod WITH toleration for worker01:

nginx pod YAML with toleration

apiVersion: v1
kind: Pod
metadata:
  name: nginx-gpu-pod
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx


Apply it:

kubectl apply -f nginx-gpu-pod.yaml

Verify scheduling:
kubectl get pod nginx-gpu-pod -o wide

4️⃣ Delete taint from control plane node
Identify control plane node name
kubectl get nodes

Assume name is control-plane

Remove taint:
kubectl taint nodes control-plane node-role.kubernetes.io/control-plane:NoSchedule-

5️⃣ Create redis pod (no tolerations):
kubectl run redis-pod --image=redis

Verify
kubectl get pod redis-pod -o wide

6️⃣ Add the taint back on control plane node
kubectl taint nodes control-plane node-role.kubernetes.io/control-plane:NoSchedule

Verify:

kubectl describe node control-plane | grep -i taint
