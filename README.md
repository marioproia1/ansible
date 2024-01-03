# Event-driven Ansible demo
## AWX installation steps on Kubernetes
### Reference guide: https://www.linuxtechi.com/install-ansible-awx-on-kubernetes-cluster/?utm_content=cmp-true

### 1) Install helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```
```
chmod +x get_helm.sh
```
```
./get_helm.sh
```
```
helm version
```

Alternative on Powershell (run as administrator):
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
```
choco install openshift-cli 
```
```
choco install kubernetes-helm 
```
```
helm version 
```
### 2) Install the AWX operator chart
```
helm repo add awx-operator https://ansible.github.io/awx-operator/
```
```
helm install ansible-awx-operator awx-operator/awx-operator -n awx --create-namespace
```

### 3) Verify AWX operator installation
```
kubectl get pods -n awx
```

### 4) Deploy AWX
Create a file awx.yml:
```
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: ansible-awx
  namespace: awx
spec:
  service_type: ClusterIP
  postgres_storage_class: default
```
Then:
```
kubectl apply -f awx.yml
```

### 5) Retrieve awx services:
```
kubectl get svc -n awx
```

Example output:
NAME                                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
ansible-awx-postgres-13                           ClusterIP   None           <none>        5432/TCP   5m36s
ansible-awx-service                               ClusterIP   10.0.223.152   <none>        80/TCP     4m5s
awx-operator-controller-manager-metrics-service   ClusterIP   10.0.167.149   <none>        8443/TCP   38m

### 6) Retrieve awx service port:
```
kubectl get svc -o yaml ansible-awx-service
```

Example output:
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: '{"apiVersion":"v1","kind":"Service","metadata":{"labels":{"app.kubernetes.io/component":"awx","app.kubernetes.io/managed-by":"awx-operator","app.kubernetes.io/operator-version":"2.10.0","app.kubernetes.io/part-of":"ansible-awx"},"name":"ansible-awx-service","namespace":"awx"},"spec":{"ports":[{"name":"http","port":80,"protocol":"TCP","targetPort":8052}],"selector":{"app.kubernetes.io/component":"awx","app.kubernetes.io/managed-by":"awx-operator","app.kubernetes.io/name":"ansible-awx-web"},"type":"ClusterIP"}}'
  creationTimestamp: "2024-01-03T14:15:59Z"
  labels:
    app.kubernetes.io/component: awx
    app.kubernetes.io/managed-by: awx-operator
    app.kubernetes.io/operator-version: 2.10.0
    app.kubernetes.io/part-of: ansible-awx
  name: ansible-awx-service
  namespace: awx
  ownerReferences:
  - apiVersion: awx.ansible.com/v1beta1
    kind: AWX
    name: ansible-awx
    uid: 1ea06c40-c29f-4653-a01a-25678a916e5e
  resourceVersion: "458018"
  uid: 8f88d1ae-56f8-416b-82a9-f0eda4d57c1d
spec:
  clusterIP: 10.0.223.152
  clusterIPs:
  - 10.0.223.152
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 8052
  selector:
    app.kubernetes.io/component: awx
    app.kubernetes.io/managed-by: awx-operator
    app.kubernetes.io/name: ansible-awx-web
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

### 7) Deploy awx ingressroute on traefik:
Create a file awx-ingress.yml:
```
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  labels:
    app: awx
  name: awx
  namespace: awx
spec:
  routes:
  - kind: Rule
    match: Host(`awx.rhsummit.cloud`)
    services:
    - kind: Service
      name: ansible-awx-service
      namespace: awx
      port: http
```	  
Then:
```
kubectl apply -f awx-ingress.yaml
```

### 7) Retrieve admin password secret key for AWX web interface:
```
oc get secret -o yaml ansible-awx-admin-password
```

### 8) Add to hosts file:
20.113.125.177 awx.rhsummit.cloud

### 9) Log into AWX:
Type the host name in a search bar, then use "admin" as username and the decoded secret password
