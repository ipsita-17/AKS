

1) Log in to Your Cluster

Make sure your kubectl context is pointing to your cluster.

# view current contexts
kubectl config get-contexts

# switch to the desired context (example)
kubectl config use-context my-cluster-context

Verify you’re connected:

kubectl get nodes

2) Create the Namespace:

kubectl create namespace mem-example

Verify:

kubectl get namespaces

3) Install Metrics Server

You mentioned “using the YAML provided in this repo” — I assume it’s the official Metrics Server manifests or a repo you have locally.

If the repo is remote:

Replace <URL_TO_YAML> with the actual path/URL.

kubectl apply -f <URL_TO_YAML> -n kube-system


For example, official upstream:

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


Verify install:

kubectl get deployment metrics-server -n kube-system


Test:

kubectl top nodes
kubectl top pods -A

4) Deploy a Pod with Memory Requests & Limits

Based on the doc you referenced, create a YAML like this:

memory-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        memory: "64Mi"
      limits:
        memory: "128Mi"


Apply it:

kubectl apply -f memory-demo.yaml

5) Verify Resource Assignment

Check the Pod is created:

kubectl get pods -n mem-example


Describe it to see requests/limits:

kubectl describe pod memory-demo -n mem-example


Check metrics (after a minute, once metrics-server is ready):

kubectl top pod memory-demo -n mem-example
