busybox-liveness.yaml:

apiVersion: v1
kind: Pod
metadata:
  name: busybox-liveness
spec:
  containers:
  - name: busybox
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600

    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

Apply & verify:

kubectl apply -f busybox-liveness.yaml
kubectl get pod busybox-liveness -w

Agnhost pod with HTTP liveness + readiness probes:

agnhost-probes.yaml
apiVersion: v1
kind: Pod
metadata:
  name: agnhost-probes
spec:
  containers:
  - name: agnhost
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - netexec
    - --http-port=8080

    ports:
    - containerPort: 8080

    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10

    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10

Apply & verify:

kubectl apply -f agnhost-probes.yaml
kubectl describe pod agnhost-probes