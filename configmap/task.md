1) Create a Secret Object

Here we create a secret called test-secret that holds a username and password.

Create secret.yaml:

apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  username: bXktYXBw         # base64 for "my-app"
  password: Mzk1MjgkdmRnN0pi # base64 for some secret


Apply it:

kubectl apply -f secret.yaml

Check it exists:

kubectl get secret test-secret
kubectl describe secret test-secret

2) Create a Pod That Uses the Secret as Environment Variables

Create secret-env-pod.yaml:

apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo $USERNAME && echo $PASSWORD && sleep 3600"]
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: test-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: test-secret
          key: password

Apply it:

kubectl apply -f secret-env-pod.yaml

3) Verify the Pod and Log Output

Check that the pod starts:

kubectl get pods

View logs:

kubectl logs secret-env-pod
