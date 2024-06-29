# hng-devops-stage-0

## Task: Static Website Deployment
In this stage, you'll step into the shoes of a DevOps engineer and deploy a static website onto a cloud platform.

## Static Website Deployment with Ansible and NGINX

This repository contains an Ansible playbook to deploy a static website on an AWS EC2 instance using NGINX. The playbook will clone your project from GitHub, configure NGINX to serve the website, and handle static files using a regex pattern.

## Prerequisites

- Ansible installed on your local machine
- AWS EC2 instance running Ubuntu
- SSH access to the EC2 instance

## Directory Structure

```
hng-devops-stage-0/
├── hosts.yaml
├── site-setup.yaml
├── index.html
├── styles/
│   └── index.css
├── README.md
├── .gitignore
└── templates/
    └── hng-nginx.conf.j2

```

## Inventory File

Create an hosts file (`hosts.yaml`) with the details of your EC2 instance:

```yaml
[web]
your-ec2-public-dns or ip 

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/your-key-pair.pem
```

Note: Replace `your-ec2-public-dns or ip address` with your EC2 instance's public DNS, and `your-key-pair.pem` with the path to your SSH private key.

## Playbook File

Create the playbook file (`site-setup.yaml`) to define the tasks for deployment:

```yaml
- hosts: web
  become: yes
  tasks:
    - name: Install required packages
      apt:
        name:
          - nginx
          - git
        state: present
        update_cache: yes

    - name: Clone the project from GitHub
      git:
        repo: 'https://github.com/Onyekachukwu-Nweke/hng-devops-stage-0.git'
        dest: /var/www/html/hng-devops-stage-0
        version: main

    - name: Configure Nginx to serve the application
      template:
        src: templates/hng-nginx.conf.j2
        dest: /etc/nginx/sites-available/hng-stage-0

    - name: Enable Nginx configuration
      file:
        src: /etc/nginx/sites-available/hng-stage-0
        dest: /etc/nginx/sites-enabled/hng-stage-0
        state: link

    - name: Remove default Nginx configuration
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify:
        - restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted

```

## NGINX Configuration Template

Create the NGINX configuration template (`hng-nginx.conf.j2`):

```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg|webp|ttf|woff|woff2|eot)$ {
        try_files $uri $uri/ =404;
        expires 30d;
        access_log off;
    }
}
```

## Running the Playbook

1. Run the Ansible playbook:
I didn't include hosts.yaml file because of security but it contains the hostname of the target server

   ```sh
   ansible-playbook -i hosts.yaml site-setup.yaml
   ```

## Site Preview
![Website Screenshot](/img/screenshot.png)

This playbook will install NGINX, clone your project from GitHub, configure NGINX to serve your static site, and restart NGINX to apply the configuration.
