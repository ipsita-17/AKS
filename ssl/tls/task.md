1. Generate a private key and CSR
# Generate private key
openssl genrsa -out learner.key 2048

# Generate CSR
openssl req -new -key learner.key -out learner.csr -subj "/CN=learner"


Verify:

ls -l learner.key learner.csr
openssl req -in learner.csr -noout -text

2. Base64-encode the CSR (for Kubernetes)
cat learner.csr | base64 | tr -d '\n'


Copy the full output — you’ll paste it into the CSR YAML.

3. Create the Kubernetes CertificateSigningRequest

Create a file: learner-csr.yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: learner
spec:
  request: <PASTE_BASE64_CSR_HERE>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 604800   # 1 week = 7 * 24 * 60 * 60
  usages:
    - client auth


Apply it:

kubectl apply -f learner-csr.yaml


Verify:

kubectl get csr learner
kubectl describe csr learner


Status should show Pending.

4. Approve the CSR
kubectl certificate approve learner


Verify approval:

kubectl get csr learner


You should see Approved,Issued.

5. Retrieve the issued certificate

Export the CSR object to YAML:

kubectl get csr learner -o yaml > learner-issued.yaml


Confirm the cert is present:

grep "certificate:" -A 2 learner-issued.yaml

6. Decode the certificate into learner.crt
kubectl get csr learner -o jsonpath='{.status.certificate}' \
  | base64 --decode > learner.crt


Verify the certificate:

ls -l learner.crt
openssl x509 -in learner.crt -noout -text


Check:

Subject: CN=learner

Validity: ~7 days

Issuer: Kubernetes CA

7. Final verification checklist

Run these to confirm everything is correct:

# Files exist
ls learner.key learner.csr learner.crt learner-issued.yaml

# Private key sanity check
openssl rsa -in learner.key -check

# CSR matches key
openssl req -in learner.csr -noout -modulus | openssl md5
openssl rsa -in learner.key -noout -modulus | openssl md5

# Certificate matches key
openssl x509 -in learner.crt -noout -modulus | openssl md5