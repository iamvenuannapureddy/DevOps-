<h1>Project-1</h1>

### **Basic Server Setup Automation with Ansible**

#### **Goal:**  
Automate basic server configurations like setting up users, installing packages, and configuring services (e.g., installing Nginx or Apache, configuring SSH).

---

### **Project Directory Structure**

```
basic-server-setup/
├── ansible.cfg               # Ansible configuration file
├── inventory.ini             # Inventory file (list of remote hosts)
├── playbooks/
│   └── setup-server.yml      # Main Ansible playbook for server setup
└── roles/
    └── common/
        ├── tasks/
        │   └── main.yml      # Task to install packages and configure services
        └── vars/
            └── main.yml      # Variables used in the playbook
```

---

### **1. Ansible Configuration File (`ansible.cfg`)**

This file tells Ansible where to find the inventory file and defines other important settings.

```ini
[defaults]
inventory = inventory.ini
host_key_checking = False
remote_user = ubuntu
private_key_file = ~/.ssh/id_rsa
```

- `inventory`: Specifies the file that lists the servers Ansible will manage.
- `remote_user`: The default user to SSH into the servers.
- `private_key_file`: Path to the SSH private key used for authentication.

---

### **2. Inventory File (`inventory.ini`)**

This file contains the IP addresses or hostnames of the remote servers.

```ini
[webservers]
server1 ansible_host=192.168.1.10 ansible_user=ubuntu

[dbservers]
server2 ansible_host=192.168.1.11 ansible_user=ubuntu
```

---

### **3. Ansible Playbook (`playbooks/setup-server.yml`)**

This is the main playbook that orchestrates the tasks for setting up the server.

```yaml
---
- hosts: webservers
  become: yes
  roles:
    - common
```

- **hosts**: Specifies the group of servers defined in the inventory file (`webservers`).
- **become**: Escalates privileges (i.e., uses `sudo`).

---

### **4. Role Tasks (`roles/common/tasks/main.yml`)**

Defines the tasks to be executed on the remote hosts. In this case, installing Nginx, creating users, and configuring SSH.

```yaml
---
# Install required packages (Nginx, UFW)
- name: Install required packages
  apt:
    name:
      - nginx
      - ufw
    state: present
    update_cache: yes

# Start and enable Nginx service
- name: Ensure Nginx is running
  service:
    name: nginx
    state: started
    enabled: yes

# Configure firewall rules (allowing HTTP and SSH traffic)
- name: Allow 'Nginx Full' profile in UFW
  ufw:
    rule: allow
    name: 'Nginx Full'

- name: Allow SSH traffic in UFW
  ufw:
    rule: allow
    port: 22

# Set up a new user with sudo privileges
- name: Create a new user
  user:
    name: deployuser
    groups: sudo
    password: "{{ 'my_password' | password_hash('sha512') }}"
    state: present

# Configure SSH to disable root login
- name: Disable root login over SSH
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'
    backup: yes

- name: Restart SSH service to apply changes
  service:
    name: ssh
    state: restarted
```

- **`apt`**: Installs Nginx and UFW (Uncomplicated Firewall).
- **`service`**: Ensures Nginx is running and enabled to start on boot.
- **`ufw`**: Configures firewall rules to allow HTTP (Nginx Full profile) and SSH traffic.
- **`user`**: Creates a new user (`deployuser`) and adds them to the `sudo` group.
- **`lineinfile`**: Modifies the SSH configuration to disable root login.

---

### **5. Role Variables (`roles/common/vars/main.yml`)**

Stores variables for the playbook. You can customize these variables as needed.

```yaml
nginx_package: nginx
firewall_rules:
  - { name: 'Nginx Full', port: '80,443' }
  - { name: 'SSH', port: '22' }
```

---

### **How to Run the Playbook**

1. **Ensure SSH access**: Make sure you have SSH access to the servers listed in the inventory (`inventory.ini`) with the appropriate SSH keys.

2. **Run the Playbook**:

   ```bash
   ansible-playbook playbooks/setup-server.yml
   ```

This command will connect to the `webservers` group defined in the inventory and run the playbook to install and configure Nginx, firewall rules, SSH settings, and user management.

---

### **Explanation of Key Tasks:**

- **Installing Nginx**: The playbook ensures Nginx is installed and running. This is useful for automating the setup of a basic web server.
- **Firewall Configuration**: UFW is used to restrict incoming traffic, allowing only HTTP and SSH connections.
- **User Management**: A new user (`deployuser`) is created with sudo privileges, ensuring secure access to the server.
- **SSH Configuration**: Disabling root login enhances the security of the server.

---

### **Use Case Examples**:

- Automating basic server configuration for development or production environments.
- Quickly setting up new cloud instances (e.g., AWS EC2, DigitalOcean droplets) with standard configurations.
- Ensuring consistency across multiple servers by applying the same setup using Ansible.

This setup is scalable and reusable for managing multiple servers by adding more hosts to the inventory and extending the role to handle more configurations.
