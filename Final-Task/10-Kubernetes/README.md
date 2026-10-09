## Setup Kubernetes mengunakan ansible

semua dikerjakan mengunakan ansible automation mulai dari install kubernetes setup containerd fe/be app dan database serta setup gnerate ssl wallcard melalui ansible

```yaml

---
- name: Setup K3s Master and Deploy Ingress
  hosts: master
  become: true
  tasks:
    - name: Ensure K3s config directory exists on Master
      ansible.builtin.file:
        path: /etc/rancher/k3s
        state: directory
        mode: "0755"

    - name: Configure K3s custom config yaml (Master)
      ansible.builtin.copy:
        dest: /etc/rancher/k3s/config.yaml
        content: |
          cluster-init: true
          disable:
            - servicelb
            - traefik
          default-local-storage-path: /mnt/disk1
        mode: "0600"

    - name: Check if K3s binary exists
      ansible.builtin.stat:
        path: /usr/local/bin/k3s
      register: k3s_binary

    - name: Install K3s Master Node
      ansible.builtin.shell: curl -sfL https://get.k3s.io | sh -
      when: not k3s_binary.stat.exists

    - name: Ensure K3s service is enabled, reloaded, and started via systemctl
      ansible.builtin.shell: |
        systemctl daemon-reload
        systemctl enable k3s
        systemctl restart k3s
      changed_when: true

    - name: Wait for node-token and read it directly via shell
      ansible.builtin.shell: |
        while [ ! -s /var/lib/rancher/k3s/server/node-token ]; do
          sleep 2
        done
        cat /var/lib/rancher/k3s/server/node-token
      args:
        executable: /bin/bash
      register: k3s_token_output
      changed_when: false

    - name: Set K3s Token fact globally
      ansible.builtin.set_fact:
        k3s_token: "{{ k3s_token_output.stdout | trim }}"

    - name: Verify K3s nodes from Master
      ansible.builtin.shell: /usr/local/bin/k3s kubectl get nodes
      register: k3s_nodes
      changed_when: false

    - name: Deploy NGINX Ingress Controller via K3s Manifest
      ansible.builtin.copy:
        dest: /var/lib/rancher/k3s/server/manifests/nginx-ingress.yaml
        content: |
          apiVersion: v1
          kind: Namespace
          metadata:
            name: ingress-nginx
          ---
          apiVersion: helm.cattle.io/v1
          kind: HelmChart
          metadata:
            name: ingress-nginx
            namespace: ingress-nginx
          spec:
            repo: https://kubernetes.github.io/ingress-nginx
            chart: ingress-nginx
            targetNamespace: ingress-nginx
            valuesContent: |-
              controller:
                image:
                  tag: "v1.8.1"
                service:
                  type: LoadBalancer

    - name: Verify NGINX Ingress Controller pods
      ansible.builtin.shell: /usr/local/bin/k3s kubectl -n ingress-nginx get pods
      register: ingress_pods
      changed_when: false

# PLAY 2: WORKER NODES SETUP 
- name: Setup K3s Worker Nodes
  hosts: worker-1,worker-2
  become: true
  tasks:
    - name: Create K3s configuration directory on Workers
      ansible.builtin.file:
        path: /etc/rancher/k3s
        state: directory
        mode: "0755"

    - name: Write K3s agent config.yaml on Workers
      ansible.builtin.copy:
        dest: /etc/rancher/node/config.yaml
        content: |
          disable:
            - servicelb
            - traefik
          default-local-storage-path: /mnt/disk1
        mode: "0600"

```

- instalasi Helm di Master Node:

  ```Bash

  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

  export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
  ```

### - Cek Token dan IP Master:
Pastikan token asli dari master bisa dilihat di master via:

```bash

sudo cat /var/lib/rancher/k3s/server/node-token

```
<p align="center"><img width="1919" height="368" alt="image" src="https://github.com/user-attachments/assets/476c8474-18cd-4b38-ac39-68d3bf12967a" /></p>

### - Join Manual di Worker (untuk verifikasi):
Ganti <IP_MASTER> dengan IP public master  (15.232.150.220) dan <TOKEN> dengan token dari master:

```bash

curl -sfL https://get.k3s.io | K3S_URL=https://<IP_MASTER>:6443 K3S_TOKEN=<TOKEN> sh -


sudo systemctl restart k3s-agent
```
 + Worker-1
   
<p align="center"><img width="1919" height="311" alt="image" src="https://github.com/user-attachments/assets/e4d2446b-1ec6-450d-8f95-9b6be99a2699" /></p>

  + Worker-2

<p align="center"><img width="1919" height="309" alt="image" src="https://github.com/user-attachments/assets/f43a8ea4-251d-4648-a804-578d2b527148" /></p>

### - Cek Setatus k3s-agent


```bash

# Restart k3s
---
sudo systemctl status k3s-agent

```
<p align="center"></p>

<p align="center"><img width="1919" height="413" alt="image" src="https://github.com/user-attachments/assets/3e8097ca-be96-482c-a249-29c70079dbc4" />
</p>

<p align="center"><img width="1919" height="408" alt="image" src="https://github.com/user-attachments/assets/f92947cd-3e24-4996-ad07-50688c02de31" />
</p>

### Melihat status seluruh node (Master & Worker)

```bash

# Melihat penggunaan CPU dan RAM pada Node
---
kubectl top nodes

# Melihat status seluruh node (Master & Worker)
---
kubectl get nodes 

```

<p align="center"><img width="1919" height="410" alt="image" src="https://github.com/user-attachments/assets/34310b38-d582-4a1e-8592-38980b35e842" /></p>

## Setup kubernetes

  - folder manifests

<p align="center"><img width="459" height="220" alt="image" src="https://github.com/user-attachments/assets/6771a377-1e27-4be5-a1e0-235da564aa11" /></p>



### - Buat Namespace production di dalam folder manifests namespace.yaml
  - membuat namespace
```yaml

apiVersion: v1
kind: Namespace
metadata:
  name: production

```
  
### - Buat file di dalam folder manifests frontend-deployment.yaml
   - untuk membuat containerd frontend app

```yaml


apiVersion: apps/v1
kind: Deployment
metadata:
  name: fe-dumbmerch
  namespace: production
  labels:
    app: fe-dumbmerch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fe-dumbmerch
  template:
    metadata:
      labels:
        app: fe-dumbmerch
    spec:
      containers:
        - name: frontend
          image: registry.adi.studentdumbways.my.id/fe-dumbmerch:production
          ports:
            - containerPort: 80
              name: http
          resources:
            limits:
              cpu: "500m"
              memory: "512Mi"
            requests:
              cpu: "100m"
              memory: "128Mi"

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: production
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
  selector:
    app: fe-dumbmerch


```

---

### - Buat file di dalam folder manifests backend-deployment.yaml
  - untuk membuat containerd backend app

```yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backend-pvc
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-path

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: be-dumbmerch
  namespace: production
  labels:
    app: be-dumbmerch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: be-dumbmerch
  template:
    metadata:
      labels:
        app: be-dumbmerch
    spec:
      containers:
        - name: backend
          image: registry.adi.studentdumbways.my.id/be-dumbmerch:production
          ports:
            - containerPort: 5000
              name: http
          envFrom:
            - secretRef:
                name: backend-env-secret
            - secretRef:
                name: postgres-secret
          volumeMounts:
            - name: backend-storage
              mountPath: /app/uploads
          resources:
            limits:
              cpu: "500m"
              memory: "512Mi"
            requests:
              cpu: "100m"
              memory: "128Mi"
      volumes:
        - name: backend-storage
          persistentVolumeClaim:
            claimName: backend-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: production
spec:
  type: ClusterIP
  ports:
    - port: 5000
      targetPort: 5000
      protocol: TCP
      name: http
  selector:
    app: be-dumbmerch

```

---

### - Buat file di dalam folder manifests postgres-deployment.yaml
  - untuk membuat containerd database postgres

```yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-path

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: production
  labels:
    app: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15-alpine
          ports:
            - containerPort: 5432
              name: postgres
          env:
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_PASSWORD
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_DB
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
          resources:
            limits:
              cpu: "500m"
              memory: "512Mi"
            requests:
              cpu: "100m"
              memory: "128Mi"
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: production
spec:
  type: ClusterIP
  ports:
    - port: 5432
      targetPort: 5432
      protocol: TCP
      name: postgres
  selector:
    app: postgres

```

---

### - Buat file di dalam folder manifests postgres-secret.yaml
  - user dan password database

```yaml

apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: production
type: Opaque
stringData:
  POSTGRES_PASSWORD: "E5uiExSg7jDSHihGrzCKsMRKIRdNjk8U"
  POSTGRES_USER: "merch"
  POSTGRES_DB: "dumbmerch_db"

```

---

### - Buat file di dalam folder manifests dumbmerch-ingress.yaml
  - setup domain Deployment
  
```yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dumbmerch-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-wildcard-issuer"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - "adi.studentdumbways.my.id"
        - "api.adi.studentdumbways.my.id"
        - "*.adi.studentdumbways.my.id"
      secretName: wildcard-tls-secret
  rules:
    - host: "adi.studentdumbways.my.id"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
    - host: "api.adi.studentdumbways.my.id"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 5000



```

---

### - Buat file di dalam folder manifests cluster-issuer.yaml
  - untuk membuat wildcard manager 

```yaml

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-wildcard-issuer
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: adiwijaya5699@gmail.com
    privateKeySecretRef:
      name: letsencrypt-wildcard-issuer-key
    solvers:
      - dns01:
          cloudflare:
            email: adiwijaya@studentdumbways.my.id
            apiTokenSecretRef:
              name: cloudflare-api-token-secret
              key: api-token

```

---

### - Buat file di dalam folder manifests cloudflare-secret.yaml
  - untuk token cloudflare

```yaml

cloudflare_token: "token"

```


### - Buat file di deploy-app-production.yaml untuk mendeploy semua file di dalam folder manifests secara otomatis mengunakan asible

```yaml

---
- hosts: master
  become: true
  vars:
    remote_dir: "/home/finaltask-adi/dumbmerch"

  tasks:
    - name: Ensure remote manifest directory exists on master node
      ansible.builtin.file:
        path: "{{ remote_dir }}/manifests"
        state: directory
        mode: "0755"

    - name: Copy all manifests to master node
      ansible.builtin.copy:
        src: "manifests/"
        dest: "{{ remote_dir }}/manifests/"
        mode: "0644"

    - name: Create Production Namespace
      ansible.builtin.command:
        cmd: "kubectl apply -f {{ remote_dir }}/manifests/namespace.yaml"
      environment:
        KUBECONFIG: "/etc/rancher/k3s/k3s.yaml"
      changed_when: true

    - name: Apply Database Secret & Deployment
      ansible.builtin.command:
        cmd: "kubectl apply -f {{ remote_dir }}/manifests/postgres-secret.yaml -f {{ remote_dir }}/manifests/postgres-deployment.yaml"
      environment:
        KUBECONFIG: "/etc/rancher/k3s/k3s.yaml"
      changed_when: true

    - name: Apply Backend Secret, PVC, Deployment, and Service
      ansible.builtin.command:
        cmd: "kubectl apply -f {{ remote_dir }}/manifests/backend-env-secret.yaml -f {{ remote_dir }}/manifests/backend-deployment.yaml"
      environment:
        KUBECONFIG: "/etc/rancher/k3s/k3s.yaml"
      changed_when: true

    - name: Apply Frontend Deployment and Service
      ansible.builtin.command:
        cmd: "kubectl apply -f {{ remote_dir }}/manifests/frontend-deployment.yaml"
      environment:
        KUBECONFIG: "/etc/rancher/k3s/k3s.yaml"
      changed_when: true

    - name: Wait for PostgreSQL rollout to complete in production namespace
      ansible.builtin.command:
        cmd: "kubectl rollout status deployment/postgres --namespace=production"
      environment:
        KUBECONFIG: "/etc/rancher/k3s/k3s.yaml"
      register: pg_status
      until: pg_status.rc == 0
      retries: 15
      delay: 5
      changed_when: false

    - name: Wait for Backend rollout to complete in production namespace
      ansible.builtin.command:
        cmd: "kubectl rollout status deployment/backend-dumbmerch --namespace=production"
      environment:
        KUBECONFIG: "/etc/rancher/k3s/k3s.yaml"
      register: be_status
      until: be_status.rc == 0
      retries: 15
      delay: 5
      changed_when: false

    - name: Wait for Frontend rollout to complete in production namespace
      ansible.builtin.command:
        cmd: "kubectl rollout status deployment/frontend-dumbmerch --namespace=production"
      environment:
        KUBECONFIG: "/etc/rancher/k3s/k3s.yaml"
      register: fe_status
      until: fe_status.rc == 0
      retries: 15
      delay: 5
      changed_when: false

```
---

### - Buat file di deploy-app-production.yaml untuk generate ssl 

```yaml

- hosts: master
  become: true
  vars_files:
    - manifests/cloudflare-secret.yaml

  tasks:
    - name: Install Cert-Manager ke klaster K3s
      ansible.builtin.shell: |
        kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.3/cert-manager.yaml
      args:
        executable: /bin/bash

    - name: Tunggu hingga Pod Cert-Manager siap (Ready)
      ansible.builtin.shell: |
        kubectl wait --for=condition=Available deployment --all -n cert-manager --timeout=120s
      args:
        executable: /bin/bash

    - name: Buat Namespace production jika belum ada
      ansible.builtin.shell: |
        kubectl create namespace production --dry-run=client -o yaml | kubectl apply -f -
      args:
        executable: /bin/bash

    - name: Buat Kubernetes Secret untuk Cloudflare API Token di namespace production
      ansible.builtin.shell: |
        kubectl create secret generic cloudflare-api-token-secret \
          --namespace=production \
          --from-literal=api-token="{{ cloudflare_token }}" \
          --dry-run=client -o yaml | kubectl apply -f -
      args:
        executable: /bin/bash

    - name: Salin folder manifests ke server master
      ansible.builtin.copy:
        src: manifests/
        dest: /manifests/

    - name: Terapkan ClusterIssuer dan Ingress ke Klaster K3s
      ansible.builtin.shell: |
        kubectl apply -f /manifests/cluster-issuer.yaml
        kubectl apply -f /manifests/ingress.yaml
      args:
        executable: /bin/bash


```

### Melihat status seluruh node (Master & Worker)

```bash

kubectl get nodes

```

<p align="center"><img width="514" height="102" alt="image" src="https://github.com/user-attachments/assets/5bc37cdd-f2d1-43b1-b140-56ec13bb6b64" /></p>

---

### Melihat semua resource (Pod, Service, Deployment) di namespace production

```bash

kubectl get all -n production
```

<p align="center"><img width="1295" height="362" alt="image" src="https://github.com/user-attachments/assets/500c9326-3b0f-4ed7-9aa2-08b7c070f30e" /></p>

### Melihat penggunaan CPU dan RAM pada Node atau Pod

```bash

kubectl top pods -n production

```

<p align="center"><img width="894" height="126" alt="image" src="https://github.com/user-attachments/assets/32a3e59c-4a81-4b8d-bc51-e246b280a487" /></p>

---

### Melihat penggunaan production menggunakan Persistent Volume dan Persistent Volume Claim

```bash

kubectl get pv,pvc -n production

```


<p align="center"><img width="1296" height="295" alt="image" src="https://github.com/user-attachments/assets/1d85cec1-8720-4a52-8d78-acefabcbb945" /></p>

---

### berhasil mengunakan ssl cloudfiree wellcard dan bisa login 

<p align="center"><img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a20cfff2-3e5e-4713-9357-a99a529b3682" /></p>


### Hasil dari Image ini berhasil dari Deploy Jenkins cicd dari github yang sudah di buat sebelumnya

<p align="center"><img width="1919" height="475" alt="image" src="https://github.com/user-attachments/assets/54a8e189-417c-4f72-a409-12702e36cc56" />
</p>
