# MT-Ansible-Project

## Step 1: Launch EC2 Instances
Launch one EC2 instance as the MT-Maste-node and two EC2 instances as slave nodes named MT-s1-node and MT-s2-node.
OS: Ubuntu

## Step 2: Install Ansible on the Master Node

### Update the package list and install Ansible:
_sudo apt update_
_sudo apt install ansible_

## Step 3: Configure SSH Key-Based Authentication
### Generate an SSH key pair on the master node
_ssh-keygen_

![SSH](https://github.com/user-attachments/assets/b83fa984-9981-4930-a3df-d1e7051b2b17)

### Access and copy the public key to both slave nodes:
_vim id_ed25519.pub_
![Pub key](https://github.com/user-attachments/assets/d9b15e1b-8969-4773-bc35-03d8f2ccb7f7)
![Pub key2](https://github.com/user-attachments/assets/7cb1f0dd-858d-44d5-9ced-c769d598cf64)

## Step 4: Configure the Inventory file
### On the master node, create a directory for your Ansible project.
_mkdir ansible-apache-project_

### Create an inventory.ini file
_vim inventory.ini_

### Add the slave nodes' private ip to the inventory.ini file
```yaml
[webservers]
172.31.22.217
172.31.17.131
```

![Mkdir](https://github.com/user-attachments/assets/38ed8924-ade8-4680-b2d6-e4a62d1968d8)


## Step 5: Create a Simple Playbook
### Create a simple HTML file
_vim index.html_

#### Add the following content:
```
<html>
<head>
  <title>Welcome to Apache World</title>
</head>
<body>
  <h1>This is my first Ansible playbook deployment!</h1>
</body>
</html>
```

![HTML](https://github.com/user-attachments/assets/39c52e33-97a4-4de0-8779-26272422e0ab)


### Create the deploy-apache.yml playbook:
vim deploy-apache.yml
![Apache deploy](https://github.com/user-attachments/assets/fc13b6cc-2ed5-46f2-a25f-9cd4e79ad873)


### Add the following content to install Apache and deploy a sample HTML file on both slave nodes:

```yaml
---
- name: Install and start Apache HTTPD server
  hosts: webservers
  become: yes

  tasks:
    - name: Ensure Apache is installed on Debian-based systems
      apt:
        name: apache2
        state: present
      when: ansible_os_family == 'Debian'

    - name: Ensure Apache is installed on RHEL-based systems
      yum:
        name: httpd
        state: present
      when: ansible_os_family == 'RedHat'

    - name: Start and enable Apache on Debian systems
      systemd:
        name: apache2
        state: started
        enabled: yes
      when: ansible_os_family == 'Debian'

    - name: Start and enable Apache on RHEL-based systems
      systemd:
        name: httpd
        state: started
        enabled: yes
      when: ansible_os_family == 'RedHat'

    - name: Copy the dummy index.html file to /var/www/html
      copy:
        src: /home/ubuntu/ansible-apache-project/index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'""
                       
    - name: Ensure firewall is allowing HTTP traffic on Debian systems
      ufw:
        rule: allow
        name: "Apache Full"
      when: ansible_os_family == 'Debian'

    - name: Ensure firewall is allowing HTTP traffic on RHEL-based systems
      firewalld:
        service: http
        permanent: yes
        state: enabled
        immediate: yes
      when: ansible_os_family == 'RedHat'
```

![Deploy2](https://github.com/user-attachments/assets/cd68191c-27b5-4467-ad24-15f5e17c912f)

![deploy1](https://github.com/user-attachments/assets/4f93f355-6bb8-4c7b-a708-5e88e63d065e)

## Step 6: Run the Ansible Playbook
### Run the playbook from the master node to deploy Apache and the HTML file to both slave nodes:
_ansible-playbook -i inventory.ini deploy-apache.yml_

![Screenshot (234)](https://github.com/user-attachments/assets/e517bc39-e020-4291-b491-31bf21dff86e)
![Screenshot (233)](https://github.com/user-attachments/assets/0ef269a1-e148-4d82-810c-657019f9b377)



## Step 7: Verify the Deployment
Navigate to web browser and type the public ip of each of the slaves.

![Screenshot (240)](https://github.com/user-attachments/assets/2c5bfb93-4486-417f-83bd-066ad6a94bfa)

![Screenshot (239)](https://github.com/user-attachments/assets/53a47636-2442-4a90-802a-831f841bc7d9)



