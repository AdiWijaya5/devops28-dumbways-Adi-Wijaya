## Setup Kubernetes


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

  - Buat Namespace production

```bash

kubectl create namespace production

```
  
<p align="center"><img width="1919" height="366" alt="image" src="https://github.com/user-attachments/assets/d7366087-a0d4-47ff-83a0-c42d826b0ff7" />
</p>


<p align="center"><img width="1154" height="260" alt="image" src="https://github.com/user-attachments/assets/ea3e889f-83cc-4b71-b6f3-20fae898c60a" />
</p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
