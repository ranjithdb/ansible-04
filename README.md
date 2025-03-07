# ansible-04

## Ansible Playbooks for Installing Docker and Deploying Nginx container

This repository contains Ansible playbooks for setting up Docker on Ubuntu and deploying an Nginx container.

## Prerequisites

- Ansible installed on the control node
- Target Ubuntu hosts accessible via SSH
- `sudo` privileges on target machines

## Inventory File (`inventory.ini`)

Define the target hosts in an inventory file:

```ini
[ubuntu]
ubuntu-server1 ansible_host=<your-server-ip> ansible_user=<your-ssh-user>

ubuntu-server2 ansible_host=<your-server-ip> ansible_user=<your-ssh-user>
```

## Install Docker on Ubuntu

This playbook installs Docker on Ubuntu machines.

### Playbook

 `install_docker_ubuntu.yml`

```yaml
- name: Install Docker on Ubuntu
  hosts: ubuntu
  become: true
  tasks:
    - name: Install dependencies
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
        state: present
        update_cache: true

    - name: Add Docker GPG key
      shell: |
        install -m 0755 -d /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | tee /etc/apt/keyrings/docker.asc > /dev/null
        chmod a+r /etc/apt/keyrings/docker.asc

    - name: Add Docker repository
      shell: |
        echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

    - name: Install Docker packages
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        state: present
        update_cache: true

    - name: Start and enable Docker service
      systemd:
        name: docker
        state: started
        enabled: true
```

### Explanation of `install_docker_ubuntu.yml`

This Ansible playbook automates the installation of Docker on Ubuntu machines. Here’s a breakdown of what each task does:

1. **Define the playbook**

   ```yaml
   - name: Install Docker on Ubuntu
     hosts: ubuntu
     become: true
   ```

   - This playbook targets hosts in the `ubuntu` group.
   - `become: true` ensures the tasks run with elevated privileges (`sudo`).

2. **Install dependencies**

   ```yaml
   - name: Install dependencies
     apt:
       name:
         - ca-certificates
         - curl
         - gnupg
       state: present
       update_cache: true
   ```

   - Installs required packages:
     - `ca-certificates`: Ensures secure HTTPS communication.
     - `curl`: Used to download files from the internet.
     - `gnupg`: Required for managing GPG keys.
   - `update_cache: true` ensures the package index is updated before installation.

3. **Add the Docker GPG key**

   ```yaml
   - name: Add Docker GPG key
     shell: |
       install -m 0755 -d /etc/apt/keyrings
       curl -fsSL https://download.docker.com/linux/ubuntu/gpg | tee /etc/apt/keyrings/docker.asc > /dev/null
       chmod a+r /etc/apt/keyrings/docker.asc
   ```

   - Creates `/etc/apt/keyrings` directory (if it doesn’t exist).
   - Downloads Docker’s official GPG key and stores it in `/etc/apt/keyrings/docker.asc`.
   - Ensures the key file is readable.

4. **Add the Docker repository**

   ```yaml
   - name: Add Docker repository
     shell: |
       echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

   - Adds the official Docker repository for the detected Ubuntu version (`$(lsb_release -cs)` dynamically fetches the codename).

5. **Install Docker packages**

   ```yaml
   - name: Install Docker packages
     apt:
       name:
         - docker-ce
         - docker-ce-cli
         - containerd.io
       state: present
       update_cache: true
   ```

   - Installs Docker components:
     - `docker-ce`: The Docker engine.
     - `docker-ce-cli`: The Docker command-line tool.
     - `containerd.io`: The container runtime.
   - `update_cache: true` ensures the package index is refreshed before installation.

6. **Start and enable Docker service**

   ```yaml
   - name: Start and enable Docker service
     systemd:
       name: docker
       state: started
       enabled: true
   ```

   - Starts the Docker service.
   - Ensures Docker starts automatically on system boot.

### Run the Playbook

```bash
ansible-playbook -i inventory.ini install_docker_ubuntu.yml --ask-become-pass
```

## Deploy Nginx using Docker

This playbook deploys an Nginx container using Docker.

### Playbook: `deploy_nginx.yml`

```yaml
- name: Deploy Nginx with Docker
  hosts: ubuntu
  become: true
  tasks:
    - name: Pull Nginx image
      docker_image:
        name: nginx
        source: pull

    - name: Run Nginx container
      docker_container:
        name: nginx
        image: nginx
        state: started
        restart_policy: always
        ports:
          - "80:80"
```

### Run the Container Deployment Playbook

```bash
ansible-playbook -i inventory.ini deploy_nginx.yml --ask-become-pass
```

## Verify Deployment

After running the playbooks, verify the setup:

### Check Docker Installation

```bash
docker --version
```

### Check Running Containers

```bash
docker ps
```

### Test Nginx Deployment

Open a browser and visit `http://<your-server-ip>` to see the Nginx welcome page.

## Cleanup

To remove the Nginx container:

```bash
docker rm -f nginx
```

To uninstall Docker:

```bash
sudo apt remove -y docker-ce docker-ce-cli containerd.io
```
