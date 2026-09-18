<!-- # Ansible-nginx-Deployment
 🚀A collection of Ansible playbooks and roles for infrastructure automation, server configuration, and application deployment.

Bilkul. Passwordless authentication ke liye SSH key-based authentication use karte hain. Password GitHub Secrets mein store karne ki zarurat nahi hogi.

Aapki repo:

Ugrasen-G/Ansible-nginx-Deployment

ke liye main ye production-style structure rakhunga.

1. Repository structure
Ansible-nginx-Deployment/
│
├── .github/
│   └── workflows/
│       └── deploy-nginx.yml
│
├── ansible/
│   ├── inventory/
│   │   └── hosts.ini
│   │
│   ├── playbooks/
│   │   └── nginx.yml
│   │
│   ├── roles/
│   │   └── nginx/
│   │       ├── tasks/
│   │       │   └── main.yml
│   │       ├── handlers/
│   │       │   └── main.yml
│   │       ├── templates/
│   │       │   └── nginx.conf.j2
│   │       └── defaults/
│   │           └── main.yml
│   │
│   └── group_vars/
│       └── nginx_servers.yml
│
└── README.md
2. Azure VM setup
Maan lete hain aapke paas 3 Ubuntu VMs hain:

VM1 → 10.0.1.4
VM2 → 10.0.1.5
VM3 → 10.0.1.6
Azure VM create karte waqt SSH public key use karein.

Aapke local machine par key generate:

ssh-keygen -t ed25519 -C "ansible-nginx-deployment"
Isse generally ye files milengi:

~/.ssh/ansible-nginx-deployment
~/.ssh/ansible-nginx-deployment.pub
.pub = public key, Azure VMs par jayegi.

बिना .pub wali file = private key, ise GitHub Secret mein rakhenge.

Azure VM ke azureuser account ke ~/.ssh/authorized_keys mein public key honi chahiye.

Test:

ssh -i ~/.ssh/ansible-nginx-deployment azureuser@VM_IP
Agar password nahi poochta aur login ho jata hai, SSH passwordless authentication ready hai.

3. Ansible inventory
ansible/inventory/hosts.ini

[nginx_servers]
web01 ansible_host=10.0.1.4
web02 ansible_host=10.0.1.5
web03 ansible_host=10.0.1.6

[nginx_servers:vars]
ansible_user=azureuser
ansible_python_interpreter=/usr/bin/python3
Agar GitHub Actions runner Azure VNet ke andar hai, to private IPs use kar sakte hain.

Agar GitHub-hosted runner use kar rahe hain aur VMs ke public IPs hain:

[nginx_servers]
web01 ansible_host=20.xx.xx.10
web02 ansible_host=20.xx.xx.11
web03 ansible_host=20.xx.xx.12

[nginx_servers:vars]
ansible_user=azureuser
ansible_python_interpreter=/usr/bin/python3
4. Ansible playbook
ansible/playbooks/nginx.yml

---
- name: Deploy Nginx on Azure VMs
  hosts: nginx_servers
  become: true

  roles:
    - nginx
5. Nginx task
ansible/roles/nginx/tasks/main.yml

---
- name: Update apt cache
  ansible.builtin.apt:
    update_cache: true
    cache_valid_time: 3600

- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Deploy Nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: "0644"
    validate: "nginx -t"
  notify: Restart Nginx

- name: Enable and start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
6. Handler
ansible/roles/nginx/handlers/main.yml

---
- name: Restart Nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
7. Variables
ansible/group_vars/nginx_servers.yml

---
nginx_port: 80
nginx_server_name: "_"
8. Nginx template
ansible/roles/nginx/templates/nginx.conf.j2

user www-data;

worker_processes auto;

pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {

    include /etc/nginx/mime.types;

    default_type application/octet-stream;

    sendfile on;

    keepalive_timeout 65;

    server {

        listen {{ nginx_port }};

        server_name {{ nginx_server_name }};

        location / {
            return 200 "Nginx deployed successfully using Ansible\n";
            add_header Content-Type text/plain;
        }
    }
}
9. GitHub Actions
Sabse important file:

.github/workflows/deploy-nginx.yml

name: Deploy Nginx to Azure VMs

on:
  push:
    branches:
      - main

  workflow_dispatch:

jobs:

  deploy:

    name: Ansible Nginx Deployment

    runs-on: ubuntu-latest

    steps:

      # --------------------------------
      # Checkout Repository
      # --------------------------------

      - name: Checkout repository
        uses: actions/checkout@v4


      # --------------------------------
      # Install Ansible
      # --------------------------------

      - name: Install Ansible
        run: |
          python3 -m pip install --upgrade pip
          pip install ansible


      # --------------------------------
      # Setup SSH
      # --------------------------------

      - name: Setup SSH private key
        run: |

          mkdir -p ~/.ssh

          echo "${{ secrets.AZURE_SSH_PRIVATE_KEY }}" > ~/.ssh/ansible-nginx-deployment

          chmod 600 ~/.ssh/ansible-nginx-deployment


      # --------------------------------
      # Start SSH Agent
      # --------------------------------

      - name: Start SSH Agent
        run: |

          eval "$(ssh-agent -s)"

          ssh-add ~/.ssh/ansible-nginx-deployment


      # --------------------------------
      # Add SSH known hosts
      # --------------------------------

      - name: Add Azure VMs to known_hosts
        run: |

          ssh-keyscan -H ${{ secrets.VM1_IP }} >> ~/.ssh/known_hosts

          ssh-keyscan -H ${{ secrets.VM2_IP }} >> ~/.ssh/known_hosts

          ssh-keyscan -H ${{ secrets.VM3_IP }} >> ~/.ssh/known_hosts


      # --------------------------------
      # Test Ansible connectivity
      # --------------------------------

      - name: Ansible Ping
        run: |

          ansible nginx_servers \
            -i ansible/inventory/hosts.ini \
            -m ping \
            --private-key ~/.ssh/ansible-nginx-deployment


      # --------------------------------
      # Deploy Nginx
      # --------------------------------

      - name: Deploy Nginx
        run: |

          ansible-playbook \
            -i ansible/inventory/hosts.ini \
            ansible/playbooks/nginx.yml \
            --private-key ~/.ssh/ansible-nginx-deployment
Lekin ek improvement
hosts.ini mein IPs hard-code karne ke bajay GitHub Secrets se inventory generate karna better hai.

Isliye workflow ko is version mein use karna recommended hai:

name: Deploy Nginx to Azure VMs

on:
  push:
    branches:
      - main

  workflow_dispatch:

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout repository
        uses: actions/checkout@v4


      - name: Install Ansible
        run: |
          python3 -m pip install --upgrade pip
          pip install ansible


      - name: Setup SSH key
        run: |

          mkdir -p ~/.ssh

          printf '%s\n' "${{ secrets.AZURE_SSH_PRIVATE_KEY }}" \
            > ~/.ssh/ansible-nginx-deployment

          chmod 600 ~/.ssh/ansible-nginx-deployment


      - name: Generate Ansible inventory
        run: |

          cat > ansible/inventory/hosts.ini <<EOF

          [nginx_servers]

          web01 ansible_host=${{ secrets.VM1_IP }}

          web02 ansible_host=${{ secrets.VM2_IP }}

          web03 ansible_host=${{ secrets.VM3_IP }}


          [nginx_servers:vars]

          ansible_user=${{ secrets.ANSIBLE_USER }}

          ansible_python_interpreter=/usr/bin/python3

          EOF


      - name: Add hosts to known_hosts
        run: |

          ssh-keyscan -H "${{ secrets.VM1_IP }}" \
            >> ~/.ssh/known_hosts

          ssh-keyscan -H "${{ secrets.VM2_IP }}" \
            >> ~/.ssh/known_hosts

          ssh-keyscan -H "${{ secrets.VM3_IP }}" \
            >> ~/.ssh/known_hosts


      - name: Test connectivity
        run: |

          ansible nginx_servers \
            -i ansible/inventory/hosts.ini \
            -m ping \
            --private-key ~/.ssh/ansible-nginx-deployment


      - name: Deploy Nginx
        run: |

          ansible-playbook \
            -i ansible/inventory/hosts.ini \
            ansible/playbooks/nginx.yml \
            --private-key ~/.ssh/ansible-nginx-deployment
10. GitHub Secrets
Repository mein:

Settings → Secrets and variables → Actions → New repository secret

Create karein:

AZURE_SSH_PRIVATE_KEY
ANSIBLE_USER
VM1_IP
VM2_IP
VM3_IP
Example:

ANSIBLE_USER = azureuser

VM1_IP = 20.xx.xx.10
VM2_IP = 20.xx.xx.11
VM3_IP = 20.xx.xx.12
AZURE_SSH_PRIVATE_KEY:

-----BEGIN OPENSSH PRIVATE KEY-----
...
...
-----END OPENSSH PRIVATE KEY-----
Private key ko kabhi GitHub repo ke andar file ke रूप में commit nahi karna hai.

11. Passwordless authentication ka flow
                    GitHub
                       │
                       │ git push
                       ▼
              GitHub Actions Runner
                       │
                       │ SSH Private Key
                       ▼
                ┌───────────────┐
                │ Azure Network │
                └───────┬───────┘
                        │
             ┌──────────┼──────────┐
             │          │          │
             ▼          ▼          ▼
           VM-01      VM-02      VM-03
             │          │          │
          Nginx       Nginx      Nginx
Authentication:

GitHub Secret
    │
    │ Private SSH Key
    ▼
GitHub Runner
    │
    │ SSH
    ▼
Azure VM
    │
    │ Public Key verification
    ▼
Passwordless Login
12. Ek important point: sudo password
SSH login passwordless hone ke baad bhi Ansible mein humne:

become: true
use kiya hai.

Iska matlab Nginx install karne ke liye sudo chahiye. Azure Ubuntu VMs mein azureuser ko normally sudo access milta hai, lekin agar sudo password maangta hai to become ke liye password issue aa sakta hai.

Production setup mein azureuser ko passwordless sudo dena possible hai:

azureuser ALL=(ALL) NOPASSWD: ALL
Lekin ise blindly enable karne ke bajay, ideally sirf required commands/permissions tak restrict karna safer hai.

Recommended Azure architecture
Agar aapki VMs private IPs par hain, to:

GitHub
   │
   ▼
GitHub Actions
   │
   ▼
Self-hosted Runner
   │
   │ Azure VNet
   ├──────────────┐
   ▼              ▼
 Azure VM 1     Azure VM 2
   │              │
 Nginx          Nginx
Agar VMs public IP par SSH expose karni hain, to GitHub-hosted runner bhi

ChatGPT said:
Unable to connect
•

Retry -->