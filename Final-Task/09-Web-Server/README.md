# Web Server


## Cloudfire
<p><img width="1917" height="986" alt="image" src="https://github.com/user-attachments/assets/c37fb59d-fdc6-4be1-8e3e-1042aef00af8" /></p>
<p><img width="1919" height="666" alt="image" src="https://github.com/user-attachments/assets/25fba30d-38ca-4927-83d8-a8b5a87312fc" />
</p>
<p><img width="924" height="412" alt="image" src="https://github.com/user-attachments/assets/6b1c0ad3-07fa-40fc-91a9-9ba00fb6a6e0" />
</p>

daftar record cloudfiere

<p><img width="954" height="597" alt="image" src="https://github.com/user-attachments/assets/0858283c-0aef-4e80-bd1d-cddd25a81203" />
</p>

<p><img width="960" height="508" alt="image" src="https://github.com/user-attachments/assets/955193bf-eb3f-4c16-b3b5-2ddfa2d67680" />
</p>

<p><img width="960" height="547" alt="image" src="https://github.com/user-attachments/assets/8794a636-13df-44ce-ae1e-16056e43184a" />
</p>

## Create File setup-gateway-ssl.yaml


<p><img width="890" height="388" alt="image" src="https://github.com/user-attachments/assets/0c8db74b-a273-4266-99ea-b4af18db13f6" />
</p>
<p><img width="909" height="61" alt="image" src="https://github.com/user-attachments/assets/48f6528b-8f4b-4fa2-af89-7bb7a5fc8dd3" />
</p>
<p><img width="904" height="274" alt="image" src="https://github.com/user-attachments/assets/373d0ad5-f9e4-4a6e-86e7-3ca7a62aa9cb" />
</p>

<p>![Uploading image.png…]()
</p>
```yaml
---
- hosts: gateway
  become: true
  
  tasks:
    - name: Install Certbot and Nginx plugin
      ansible.builtin.apt:
        name:
          - certbot
          - python3-certbot-nginx
        state: present
        update_cache: yes

    - name: Verify service nginx is running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes

    # --- 1. GENERATE WILDCARD SSL CERTIFICATE MENGGUNAKAN CERTBOT ---
    - name: Generate Wildcard SSL certificate using Certbot (Manual DNS-01 mode)
      ansible.builtin.command:
        cmd: "certbot certonly --manual --preferred-challenges dns -d {{ domain_name }} -d {{ base_domain }} --agree-tos --email {{ admin_email }} --manual-public-ip-logging-ok"
      args:
        creates: "/etc/letsencrypt/live/{{ domain_name }}/fullchain.pem"

    # --- 2. KONFIGURASI NGINX REVERSE PROXY UNTUK KETIGA DOMAIN ---
    - name: Create configuration Nginx using templates
      ansible.builtin.template:
        src: "files/{{ item.src_file }}"
        dest: "/etc/nginx/sites-available/{{ item.dest_name }}"
        mode: "0644"
      loop:
        - { src_file: "registry.conf.j2", dest_name: "registry" }
        - { src_file: "staging-app.conf.j2", dest_name: "staging-app" }
        - { src_file: "api-staging.conf.j2", dest_name: "api-staging" }
      notify: Restart Nginx

    - name: Enable all Nginx sites by creating symlinks
      ansible.builtin.file:
        src: "/etc/nginx/sites-available/{{ item }}"
        dest: "/etc/nginx/sites-enabled/{{ item }}"
        state: link
      loop:
        - registry
        - staging-app
        - api-staging
      notify: Restart Nginx

    - name: Remove default Nginx site
      ansible.builtin.file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify: Restart Nginx

    # --- 4. OTOMATISASI PERPANJANGAN SSL (AUTO-RENEWAL CRONJOB) ---
    - name: Add Cronjob for automatic Certbot renewal
      ansible.builtin.cron:
        name: "Certbot Automatic Renewal"
        minute: "0"
        hour: "3"
        job: "certbot renew --quiet --post-hook 'systemctl reload nginx'"

  handlers:
    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted

```
