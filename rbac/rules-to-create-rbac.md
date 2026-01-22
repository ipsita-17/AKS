| Step                           | Purpose                    |
| ------------------------------ | -------------------------- |
| openssl genrsa                 | Create private key         |
| openssl req                    | Create CSR                 |
| base64                         | Encode CSR for Kubernetes  |
| CertificateSigningRequest      | Ask cluster for cert       |
| kubectl certificate approve    | Approve cert               |
| kubectl config set-credentials | Add user to kubeconfig     |
| kubectl config set-context     | Create login context       |
| kubectl config use-context     | Switch user                |
| Role                           | Define permissions         |
| RoleBinding                    | Attach permissions to user |
| kubectl auth can-i             | Validate RBAC              |


Exam Memory Hook (CKA Style)

KEY → CSR → K8s CSR → APPROVE → CERT → KUBECONFIG → CONTEXT → ROLE → ROLEBINDING → TEST

PHASE 1 — Create a Kubernetes User (Cert-based Auth)
1. Generate private key
openssl genrsa -out adam.key 2048

2. Generate CSR
openssl req -new -key adam.key -out adam.csr -subj "/CN=adam"

3. Base64 encode CSR (single line)
base64 < adam.csr | tr -d '\n'


Copy output.

4. Create Kubernetes CSR object

Create file: adam-csr.yml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: adam
spec:
  request: <PASTE_BASE64_CSR_HERE>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 604800
  usages:
    - client auth


Apply:

kubectl apply -f adam-csr.yml

5. Approve CSR
kubectl certificate approve adam

6. Extract issued certificate
kubectl get csr adam -o jsonpath='{.status.certificate}' \
  | base64 --decode > adam.crt


Verify:

openssl x509 -in adam.crt -noout -text

🔹 PHASE 2 — Add User to kubeconfig
7. Add user credentials
kubectl config set-credentials adam \
  --client-key=adam.key \
  --client-certificate=adam.crt \
  --embed-certs=true

8. Create context for user
kubectl config set-context adam \
  --cluster=kind-lab-cluster \
  --user=adam

9. Switch to new user
kubectl config use-context adam

10. Verify authentication
kubectl auth whoami
kubectl cluster-info     # will FAIL (expected, no RBAC yet)

🔹 PHASE 3 — Test Default Permissions (Expected Failures)
kubectl get pods
kubectl run test-pod --image=nginx


Expected:

Forbidden

🔹 PHASE 4 — Grant RBAC Permissions
11. Switch back to admin
kubectl config use-context kind-lab-cluster

12. Create Role (read-only pods)
kubectl create role pod-reader \
  --verb=get,watch,list \
  --resource=pods

13. Bind Role to user
kubectl create rolebinding read-pods \
  --role=pod-reader \
  --user=adam

14. Verify RBAC objects
kubectl get role pod-reader
kubectl get rolebinding read-pods
kubectl describe role pod-reader
kubectl describe rolebinding read-pods

🔹 PHASE 5 — Validate RBAC Behavior
15. Switch back to user
kubectl config use-context adam

16. Test permissions
kubectl get pods                     # ✅ allowed
kubectl run nginx --image=nginx      # ❌ forbidden
kubectl create deployment nginx-deploy --image=nginx  # ❌ forbidden

17. RBAC sanity checks (optional)
kubectl auth can-i get pods --as=adam
kubectl auth can-i create pods --as=adam
kubectl auth can-i create deployments --as=adam


Expected:

yes
no
no
