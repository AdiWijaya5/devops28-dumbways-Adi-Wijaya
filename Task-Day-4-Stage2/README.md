# Kubernetes install and apply to app wayshub

## 1. Create server (Node-master server) and (Node-worker server 2) terrafrom and ansibel

- create key pairs to region indonesia
  + name k8s-key
        
<p align="center"><img width="940" height="893" alt="image" src="https://github.com/user-attachments/assets/28284b1e-fb73-4d62-bb69-5506e29ac23e" /></p>

- create server with terrafrom and ansibel create user
    + server Node-1 (master)
    + Server Node-2 (worker-1)
    + Server Node-3 (worker-2)
- run terrafrom
    ```
    + terrafrom init
    + terrafrom plan
    + terrafrom Apply
    ```
      
<p align="center"><img width="639" height="361" alt="image" src="https://github.com/user-attachments/assets/6dcdb9bd-7113-4514-b5cd-6d2e8d8e89a9" /></p>

- success terraform apply to server

<P align="center"><img width="1906" height="909" alt="image" src="https://github.com/user-attachments/assets/17f72058-1411-4158-b13c-c00958f14802" /></P>

- Open terminal, navigate to the folder containing your key, and run
  ```
  bash
  
  chmod 400 key.pem

  ```

- Testing the login with the previously created SSH key

<p align="center"><img width="953" height="816" alt="image" src="https://github.com/user-attachments/assets/bd62b5a7-be13-4fd0-ab01-573b96eb565c" /></p>

- create Invebtory
  ```
  [master]
  15.232.17.54
  
  [worker-1]
  43.218.48.54
  
  [worker-2]
  16.79.106.228
  ```
- create group_vers folder and file all

<p align="center"><img width="868" height="322" alt="image" src="https://github.com/user-attachments/assets/d761a52e-a70f-437d-b58a-14ac61de4c26" /></p>

- Create new users and group them using Ansible
  
  ```
  - name: Create New User and Setup Access
  hosts: all
  become: true

  tasks:
    - name: Create user new with password enkripsi and group
      ansible.builtin.user:
        name: "{{ app_user }}"
        password: "{{ app_user_password_hash }}"
        shell: /bin/bash
        create_home: true
        groups: sudo
        append: true

    - name: Ensure .ssh directory exists with correct permissions
      ansible.builtin.file:
        path: "/home/{{ app_user }}/.ssh"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: "0700"

    - name: Add SSH Public Key for secure login (.pem key)
      ansible.builtin.authorized_key:
        user: "{{ app_user }}"
        state: present
        key: "{{ ssh_public_key }}"

    - name: Enable password authentication in SSH configuration
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^PasswordAuthentication"
        line: "PasswordAuthentication yes"
        state: present
      notify: Restart SSH

    - name: Enable SSH kayboard-interactive authentication in SSH configuration
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^KbdInteractiveAuthentication"
        line: "KbdInteractiveAuthentication yes"
        state: present
      notify: Restart SSH

  handlers:
    - name: Validate SSH configuration
      ansible.builtin.command: sshd -t
      changed_when: false
      listen: Restart SSH

    - name: Restart SSH
      ansible.builtin.service:
        name: ssh
        state: restarted
      become: true

  ```

- test acces server master with password user adi

<p align="center"><img width="960" height="620" alt="image" src="https://github.com/user-attachments/assets/fe25599a-b0e3-4453-b49f-6bf4955672b9" /></p>

## 2. Install k3s (kubernetes) to server

- Install k3s server to root@master
  + Install K3S using the install script

  ```
  bash
  
  curl -sfL https://get.k3s.io | sh - 

  ```
  
<p align="cenetr"><img width="958" height="518" alt="image" src="https://github.com/user-attachments/assets/6bc78291-697a-4281-b941-315d03624a08" /></p>

+ After installing K3s, check the status with the following command

```
bash

systemctl status k3s

```

- Check the status of the K3s cluster running on this master node

```
bash

k3s kubectl get node

```

- Make sure you have the /usr/local/bin directory in your path to directly access the installed binaries

```
bash

export PATH=/usr/local/bin:$PATH
k3s kubectl get nodes

kubectl get nodes
kubectl get pods -A

```

- Customise the server configuration

```
bash

export PATH=/usr/local/bin:$PATH
k3s kubectl get nodes

kubectl get nodes
kubectl get pods -A

```

### Install NGINX Ingress Controller

- Disable Traefik Ingress Controller
  + Edit the configuration file

```
bash

vim /etc/rancher/k3s/config.yaml
cluster-init: true
disable: servicelb
disable: traefik


# Restart k3s
systemctl restart k3s
```

<p align="center"><img width="950" height="187" alt="image" src="https://github.com/user-attachments/assets/95fc8527-6ff9-4da1-911a-f16e7d070b10" /></p>

- create the nginx ingress manifest

```
cat <<EOF > /var/lib/rancher/k3s/server/manifests/nginx-ingress.yaml
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
EOF

#Verify
kubectl -n ingress-nginx get pods

```

<p align="center"><img width="957" height="607" alt="image" src="https://github.com/user-attachments/assets/7658fa14-1af1-4cd9-896c-418b09f146b9" /></p>

- Configure a Private Registry
  + Create the registries configuration (Create the registries.yaml file)


  ```
  cat <<EOF > /etc/rancher/k3s/registries.yaml
  mirrors:
    registry.home-k8s.lab:
      endpoint:
        - "https://registry.home-k8s.lab"
  
  configs:
    "registry.home-k8s.lab":
      auth:
        username: admin
        password: Harbor12345
      tls:
        insecure_skip_verify: true
  EOF
  
  # Restart k3s
  systemctl restart k3s
  ```

- Configure default storage
  + Set the default local storage path
  
  ```
  cat /etc/rancher/k3s/config.yaml
  cluster-init: true
  disable: servicelb
  disable: traefik
  default-local-storage-path: /mnt/disk1
  
  # Restart k3s
  systemctl restart k3s
  ```

- Adding an agent node
  + Retrieve token from etcd capable server node

  ```
  cat /var/lib/rancher/k3s/server/token
  ```

  + Login to worker-1 and worker-1 server node and install K3S

  ```
  curl -sfL https://get.k3s.io | K3S_URL=https://node1:6443 K3S_TOKEN=node1token sh -
  ```

  <p align="center"><img width="958" height="497" alt="image" src="https://github.com/user-attachments/assets/30485c29-398e-4d4a-b85e-5986fb2e58d2" />
</p>

- Configure default storage
  + Set the default local storage path

```
cat <<EOF > /etc/rancher/node/congig.yaml
disable: traefik
default-local-storage-path: /mnt/disk1
EOF

# Restart k3s
systemctl restart k3s-agent
```

- Install helm kubenetes

  <p align="center"><img width="958" height="492" alt="image" src="https://github.com/user-attachments/assets/06f00e88-7bd3-462d-968e-f7bed550e851" /></p>

- Verify helm kubernetes

  <p align="center"><img width="956" height="374" alt="image" src="https://github.com/user-attachments/assets/56d4b79b-1138-4b08-943e-21611d6bf5d5" /></p>


## Build and Deploy Wayshub 

- create namespace app-wayshub
    
<p align="center"><img width="816" height="273" alt="image" src="https://github.com/user-attachments/assets/12ec46fa-2985-47c8-9faa-d137d4a13bcd" /></p>

- create containerd name file frontend.yaml

  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: frontend-deployment
    namespace: app-wayshub
    labels:
      app: frontend
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: frontend
    template:
      metadata:
        labels:
          app: frontend
      spec:
        nodeName: worker-1
        containers:
        - name: frontend
          image: adiwijayajy/wayshub:frontend-deploy
          ports:
          - containerPort: 3000
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: frontend-service
    namespace: app-wayshub
  spec:
    selector:
      app: frontend
    ports:
      - protocol: TCP
        port: 80
        targetPort: 3000

  ```

- create containerd name file backend.yaml

  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: backend-deployment
    namespace: app-wayshub
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: backend
    template:
      metadata:
        labels:
          app: backend
      spec:
        nodeName: worker-2
        containers:
        - name: backend
          image: adiwijayajy/wayshub:backend-deploy
          ports:
          - containerPort: 5000
          env:
          - name: DB_HOST
            value: "mysql-service"
          - name: DB_PORT
            value: "3306"
          - name: DB_USER
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: MYSQL_USER
          - name: DB_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: MYSQL_PASSWORD
          - name: DB_NAME
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: MYSQL_DATABASE
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: backend-service
    namespace: app-wayshub
  spec:
    selector:
      app: backend
    ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
    type: ClusterIP
  ```
 
- create containerd name file mysql-secret.yaml

  ```
  apiVersion: v1
  kind: Secret
  metadata:
    name: mysql-secret
    namespace: app-wayshub
  type: Opaque
  stringData:
    MYSQL_ROOT_PASSWORD: fire23421
    MYSQL_DATABASE: wayshub
    MYSQL_USER: adi
    MYSQL_PASSWORD: bahas3241

  ```

- create containerd name file mysql-statefulset.yaml

  ```
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: mysql
    namespace: app-wayshub
  spec:
    serviceName: "mysql-service"
    replicas: 1
    selector:
      matchLabels:
        app: mysql
    template:
      metadata:
        labels:
          app: mysql
      spec:
        containers:
        - name: mysql
          image: mysql:8.0
          ports:
          - containerPort: 3306
            name: mysql
          env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: MYSQL_ROOT_PASSWORD
          - name: MYSQL_DATABASE
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: MYSQL_DATABASE
          - name: MYSQL_USER
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: MYSQL_USER
          - name: MYSQL_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: MYSQL_PASSWORD
          volumeMounts:
          - name: mysql-data
            mountPath: /var/lib/mysql
    volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "local-path"
        resources:
          requests:
            storage: 1Gi
  ```

   - edit config/config.json backend and docker build , docker push, apply again containerd baceknd

  <p align="center"><img width="671" height="477" alt="image" src="https://github.com/user-attachments/assets/8688ccd8-54b0-4ee7-8065-11c347d0d080" /></p>

  - mysql npx sequelize-cli db:migrate

  ```
  Bash

  kubectl exec -it <pod/..> -n <namespase> -- npx sequelize-cli db:migrate
  kubectl exec -it <pod/..> -n <namespase> -- npm run migrate
  
  ```

  <p align="center"><img width="1287" height="469" alt="image" src="https://github.com/user-attachments/assets/48143227-da93-4009-ba14-ec3f566e4d86" /></p>
  
## Instal ingress wayshub

  - Install Cert-Manager ke Kubernetes Cluster

  ```
  Bash

  kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.2/cert-manager.yaml
  ```

  - Wait for a few moments, then verify that all cert-manager pods are Running with the command

  ```
  Bash

  kubectl get pods -n cert-manager
  ```

  - generate token cloudefire and apply token

  <p align="center"><img width="956" height="993" alt="image" src="https://github.com/user-attachments/assets/484b5f6d-25bb-496b-a4c2-182c36339847" /></p>
  
  <p align="center"><img width="955" height="329" alt="image" src="https://github.com/user-attachments/assets/7fe4da49-9d10-4ce2-8904-9b0610555e17" /></p>
  
  - ClusterIssuer (Let's Encrypt)
    + create file issuer-wildcard.yaml

  ```
  apiVersion: cert-manager.io/v1
  kind: ClusterIssuer
  metadata:
    name: letsencrypt-wildcard-prod
  spec:
    acme:
      server: https://acme-v02.api.letsencrypt.org/directory
      email: adiwijaya5699@gmail.com
      privateKeySecretRef:
        name: letsencrypt-wildcard-prod-key
      solvers:
      - dns01:
          cloudflare:
            email: adiwijaya5699@gmail.com.com
            apiTokenSecretRef:
              name: cloudflare-api-token-secret
              key: api-token
  ```

     
  +  run ClusterIssuer
  ```
  Bash
  
  kubectl apply -f issuer.yaml
  ```

  + Update Ingress YAML to acctiv SSL file name wayshub-ingress.yaml
    + kubectl apply -f wayshub-ingress.yaml -n app-wayshub

  ```
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: wayshub-ingress
    namespace: app-wayshub
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-wildcard-prod"
  spec:
    ingressClassName: nginx
    tls:
    - hosts:
      - "*.adiwijaya.kubernetes.studentdumbways.my.id"
      - adiwijaya.kubernetes.studentdumbways.my.id
      secretName: wayshub-wildcard-tls-secret
    rules:
    # Host untuk Frontend
    - host: adiwijaya.kubernetes.studentdumbways.my.id
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: frontend-service
              port:
                number: 3000
    # Host untuk Backend (API)
    - host: api.adiwijaya.kubernetes.studentdumbways.my.id
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
  
  
  - Verifikasi SSL Certificate

  ```
  Bash

  kubectl get certificate -n app-wayshub
  ```

  <p align="center"><img width="953" height="261" alt="image" src="https://github.com/user-attachments/assets/a35cbc5f-84a5-4bf4-9296-c9252cec5c33" /></p>
  

  - edit app frontend configure api.js to URL new 

  <p align="center"><img width="949" height="665" alt="image" src="https://github.com/user-attachments/assets/4f1ab6a9-a05f-44d2-9365-8490729d42ab" /></p>

  - docker build and docker push again

  <p align="center"><img width="957" height="668" alt="image" src="https://github.com/user-attachments/assets/3dfddcde-7a5a-4dc9-949a-604f92fc8a69" /></p>

  <p align="center"><img width="951" height="279" alt="image" src="https://github.com/user-attachments/assets/d94f4a32-fea5-481b-a839-7406ed12e811" /></p>

  <p align="center"><img width="1204" height="942" alt="image" src="https://github.com/user-attachments/assets/e28dee5b-a8bc-4a8e-997e-b37b54fd2775" /></p>

  - edit image frontend.yaml containerd and apply again
  
  ```
  Bash

  kubectl apply -f frontend.yaml -n app-wayshub
  ```

  - SSL Certificate Succes apply

  <p align="center"><img width="1919" height="1040" alt="image" src="https://github.com/user-attachments/assets/9cdd8055-e57f-416f-bfe1-0c026b54db0e" /></p>

  <p align="center"><img width="1919" height="1040" alt="image" src="https://github.com/user-attachments/assets/d57ae3ac-3743-4d66-8377-e7c4c9e3c5c4" /></p>
  

  - Deploy wayshub app

  ```
  Bash

  kubectl get all -n app-wayshub
  ```
  
  <p align="center"><img width="956" height="569" alt="image" src="https://github.com/user-attachments/assets/86ad7067-964b-40ca-b6c3-a455260d1beb" /></p>

  - Test create chanel Webseite Wayshub appp succes add chanel

  <p align="center"><img width="1918" height="992" alt="image" src="https://github.com/user-attachments/assets/db7408b4-9297-4a05-b0f8-a3914db682a6" /></p>


  - Ensuring the data can be properly managed, retrieved, or secured

  <p align="center"><img width="956" height="861" alt="image" src="https://github.com/user-attachments/assets/b7d2889c-0ec4-4df1-8bbd-c725b87fc953" />
</p>


















  



