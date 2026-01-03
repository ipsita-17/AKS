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
