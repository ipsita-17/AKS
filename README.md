# Create an ngnix pod through ( Through command or API calls)

kubectl run nginx-pod(podname) --image=nginx:latest

## kubectl explain rs

1. kubectl get po
2. kubectl delete pod nginx-pod
3. kubectl delete pods --all
4. kubectl apply -f rc.yml
5. kubectl get po
6. kubectl get rc
7. kubectl describe po nginx-rc-rjmvc
8. kubectl apply -f deployment.yml
9. kubectl set image deploy/nginx-deploy \
> nginx=nginx:1.9.1
10. kubectl describe deploy/nginx-deploy
11. kubectl rollout history deploy/nginx-deploy
12. kubectl create deploy deploy/nginx-new --image=nginx --dry-run=client
13. kubectl create deployment nginx-new --image=nginx \ --dry-run=client -o yaml > deploy.yml
14. kubectl scale rs nginx-rs --replicas=0
15. kubectl get pod --show-labels

<b> Services Commands:

1. kubectl explain services
2. kubectl create -f nodeport.yml
3. kubectl get svc
4. kubectl get pod -o wide
5. kind create cluster --config=kind.yml --name=cka-cluster3
6. kubectl get nodes

<c> NameSpace Commands:

1. kubectl get ns
2. kubectl get all -n=default
3. kubectl get all -n=kube-system
4. cd ../namespace
5. vi ns.yml
6. kubectl apply -f ns.yml
7. imperative way: kubectl create ns demo
8. kubectl create deployment nginx-demo --image=nginx -n demo
9. kubectl create deployment nginx-test --image=nginx -n demo
10. kubectl get pods -n=demo
11. kubectl exec -it nginx-demo-5c86c7d69f-269ft -n demo -- sh
12. IP: kubectl get pods -n demo -o wide
13. kubectl scale --replicas=3 deploy/nginx-demo -n=demo
14. kubectl scale deployment nginx-test --replicas=3 -n demo
15. kubectl get pods -n=demo
16. kubectl expose deploy/nginx-demo --name=svc-demo --port 80 -n=demo
17. kubectl get svc -n=demo
18. cat /etc/resolv.conf

<d> Multi Container Pod:

1. pod.yml created: kubectl create -f pod.yml
2. kubectl get pod
3. kubectl describe pod myapp
4. kubectl logs myapp-pod
5. kubectl logs myapp-pod -c init-myservice
6. kubectl create deploy nginx-deploy --image nginx --port 80
8. kubectl expose deploy nginx-deploy --name myservice --port 80
9. kubectl get pod
10. kubectl exec -it myapp-pod -- printenv/kubectl exec -it myapp-pod -- sh
11. kubectl expose deploy mydb --name mydb --port 80
12. kubectl get po -w

<e> DaemonSet:

1. cd daemonset
2. kubectl create -f ds.yml
3. kubectl get ds
4. kubectl get ds -n kube-system
5. kind get clusters
6. kubectl config use-context kind-kind
7. kind get nodes
8. kubectl get ds -A
9. kubectl get pods -A
10. kubectl config use-context kind-cka-cluster3
11. kubectl get pods -n kube-system | grep proxy
12. kubectl get pods

<f> Managing Static Pods, Labels & Selectors:

1. SSH into the Node: You will gain access to the node where the static pod is defined.(Mostly the control plane node)
2. Modify the YAML File: Edit or create the YAML configuration file for the static pod.
3. Remove the Scheduler YAML: To stop the pod, you must remove or modify the corresponding file directly on the node.
4. Default location": is usually /etc/kubernetes/manifests/; you can place the pod YAML in the directory, and Kubelet will pick it for scheduling.

### Labels 🏷️
Labels are key-value pairs attached to Kubernetes objects like pods, services, and deployments. They help organize and group resources based on criteria that make sense to you.

**Examples of Labels:**
- `environment: production`
- `type: backend`
- `tier: frontend`
- `application: my-app`

### Selectors 🔍
Selectors filter Kubernetes objects based on their labels. This is incredibly useful for querying and managing a subset of objects that meet specific criteria.

**Common Usage:**
- **Pods**: `kubectl get pods --selector app=my-app`
- **Deployments**: Used to filter the pods managed by the deployment.
- **Services**: Filter the pods to which the service routes traffic.

### Labels vs. Namespaces 🌍
- **Labels**: Organize resources within the same or across namespaces.
- **Namespaces**: Provide a way to isolate resources from each other within a cluster.

<g> Taint: 
Think of taints as "only you are allowed" signs on your Kubernetes nodes. A taint marks a node with a specific characteristic, such as `"gpu=true"`. By default, pods cannot be scheduled on tainted nodes unless they have a special permission called toleration. When a toleration on a pod matches with the taint on the node then only that pod will be scheduled on that node.

1. kubectl taint node cka-cluster2-worker gpu=true:NoSchedule
2. kubectl describe node cka-cluster2-worker | grep -i taint

Tolerations: Toleration allows a pod to say, "Hey, I can handle that taint. Schedule me anyway!" You define tolerations in the pod specification to let them bypass the taints.

<h> Affinity:

1. kubectl create -f affinity.yml
2. kubectl get po
3. kubectl describe po
4. kubectl label node
5. kubectl get nodes
6. kubectl label node cka-cluster2-worker disktype=ssd
7. kubectl get nodes --show-labels
8. kubectl describe pod redis-3
9. 