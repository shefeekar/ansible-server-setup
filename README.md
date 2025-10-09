 # ansible-project

The goal of this project is to introduce you to the basics of configuration management using Ansible. You will write an Ansible playbook to configure a Linux server.

https://roadmap.sh/projects/configuration-management

## **Requirements**

If you have been doing the previous projects, you should already have a Linux server running. If not, setup a Linux server on [**DigitalOcean**](https://m.do.co/c/b29aa8845df8), AWS or another cloud provider.

You are required to write an Ansible playbook called **`setup.yml`** and create the following roles:

- **`base`** — basic server setup (installs utilities, updates the server, installs **`fail2ban`**, etc.)
- **`nginx`** — installs and configures **`nginx`**
- **`app`** — uploads the given tarball of a static HTML website to the server and unzips it.
- **`ssh`** - adds the given public key to the server

Set up the inventory file **`inventory.ini`** to include the server you are going to configure When you run the playbook, it should run the roles above in sequence. You should also assign proper tags to the roles so that you can run only specific roles.

Example:

bash

`# Run all the rolesansible-playbook setup.yml# Run only the app roleansible-playbook setup.yml --tags "app"`

### **Stretch goal**

Modify the **`app`** role to pull the repository from GitHub and deploy it.

---

Once you are done with the project, you should have a basic understanding of how Ansible works and how it can be used to manage server configuration.

# Solution

# ⚙️ Ansible Configuration Management Project

This project demonstrates how to use **Ansible** to automate the configuration and deployment of Linux web servers. It introduces the fundamentals of configuration management by deploying and managing a functional web server environment with automation, security, and modular roles.

The goal of this project is to **deploy and configure a complete web server stack**, including:

- Basic system setup and utilities
- Security tools installation (e.g., fail2ban)
- Nginx web server configuration
- Static website deployment
- SSH key management

All automation is handled through a **main playbook (`setup.yml`)** and **four modular roles**.

---

## 📂 Project Structure

| File/Role | Description |
| --- | --- |
| **setup.yml** | The main playbook that orchestrates all defined roles. |
| **inventory.ini** | Defines host groups and target servers for playbook execution. |
| **roles/ssh** | Adds the specified public SSH key to the target server. |
| **roles/base** | Performs system updates and installs general utilities and security tools like `fail2ban`. |
| **roles/nginx** | Installs and configures the Nginx web server. |
| **roles/app** | Deploys a static HTML website by uploading and extracting a tarball. |

**create an inventory file for the host called webservers**

`[webservers]
ansible-1 ansible_host=13.232.168.65`

`ansible-2 ansible_host=13.203.155.229`

**group_var** is used to store the variables  and saved 
ansible_ssh_private_key_file and ansible user

**create setup.yml for** 

```yaml
- name: Setup web servers
  hosts: webservers
  become: yes
  roles:
  - role: ssh 
    tags: ["ssh"]
  - role: base
    tags: ["base"]
  - role: nginx
    tags: ["nginx"]
  - role: app
    tags: ["app"]
  
    
```

**create roles for base utility install roles/tasks/base/main.yml**

 

```
- name: Update yum cache
  ansible.builtin.dnf:
    update_cache: true
- name: Install common packages
  ansible.builtin.dnf:
    name:
      - git
      - vim
      - wget
    state: present 
- name: Install fail2ban
  ansible.builtin.dnf:
    name: fail2ban
    state: present    
```

create app role for serving index page index it will copy the index page local system to the destination  aws linux server

```yaml
- name: Copy file with owner and permissions
  ansible.builtin.copy:
    src: /home/shefeek/Desktop/index.html
    dest: /usr/share/nginx/html/index.html
    owner: nginx
    group: nginx
    mode: '0644'
```

finallay create nginx role first i create an nginx **templatefile**

**roles/tasks/template/nginx.conf.j2**

server {
listen 80;
server_name 13.232.168.65 ;

```
root /usr/share/nginx/html;
index index.html;

location / {
    try_files $uri $uri/ =404;
}

```

}

**Then create nginx role**

```yaml
- name: Update yum cache
   yum:
        update_cache: yes

 - name: Install Nginx
   yum:
        name: nginx
        state: latest
 - name: Ensure Nginx service is running and enabled
   service:
        name: nginx
        state: started
        enabled: yes  
 - name: Disable default Nginx site
   file:
    path: /etc/nginx/conf.d/default.conf
    state: absent
         
 - name: Copy Nginx site configuration
   template:
            src: templates/nginx.conf.j2
            dest: /etc/nginx/conf.d/nginx.conf
            owner: nginx
            group: nginx
            mode: '0644'
   notify: restart nginx
   

   
 
```

and create a **handler function** for any configuration  change in the application restarting nginx 

 

```

 - name: restart nginx
   service:
        name: nginx
        state: restarted
        
```

finally apply the playbook 

***ansible-playbook  setup.yml  -i inventory.ini***