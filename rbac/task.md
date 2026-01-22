1. Switch to the krishna context

(Use the context you created from learner.key / learner.crt — rename if needed)

kubectl config use-context krishna

Verify:

kubectl config current-context

2. Try to create a Pod (before granting permissions)

kubectl run test-pod --image=nginx

3. Switch back to admin context

kubectl config use-context kubernetes-admin

(or your cluster’s admin context name)

4. Create a Role: pod-reader

This Role only allows read access to Pods.

kubectl create role pod-reader \
  --verb=get,watch,list \
  --resource=pods


Verify:

kubectl get role pod-reader
kubectl describe role pod-reader

5. Create a RoleBinding: read-pods

Bind user krishna to the Role pod-reader.

kubectl create rolebinding read-pods \
  --role=pod-reader \
  --user=krishna


Verify:

kubectl get rolebinding read-pods
kubectl describe rolebinding read-pods

6. Switch back to krishna context
kubectl config use-context krishna

7. Try to create a Pod again
kubectl run nginx --image=nginx

8. List Pods in the namespace
kubectl get pods

9. Try to create a Deployment
kubectl create deployment nginx-deploy --image=nginx