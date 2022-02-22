# IAC with Ansible

![My project (12)](https://user-images.githubusercontent.com/98178943/154736652-fa4a2f9e-2455-4b4e-8063-9337db10dfa7.png)


### Let's create Vagrantfile to create Three VMs for Ansible architecture
#### Ansible controller and Ansible agents 

```

# -*- mode: ruby -*-
 # vi: set ft=ruby :
 
 # All Vagrant configuration is done below. The "2" in Vagrant.configure
 # configures the configuration version (we support older styles for
 # backwards compatibility). Please don't change it unless you know what
 
 # MULTI SERVER/VMs environment 
 #
 Vagrant.configure("2") do |config|
 # creating are Ansible controller
   config.vm.define "controller" do |controller|
     
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    # config.hostsupdater.aliases = ["development.controller"] 
    
   end 
 # creating first VM called web  
   config.vm.define "web" do |web|
     
     web.vm.box = "bento/ubuntu-18.04"
    # downloading ubuntu 18.04 image
 
     web.vm.hostname = 'web'
     # assigning host name to the VM
     
     web.vm.network :private_network, ip: "192.168.33.10"
     #   assigning private IP
     
     #config.hostsupdater.aliases = ["development.web"]
     # creating a link called development.web so we can access web page with this link instread of an IP   
         
   end
   
 # creating second VM called db
   config.vm.define "db" do |db|
     
     db.vm.box = "bento/ubuntu-18.04"
     
     db.vm.hostname = 'db'
     
     db.vm.network :private_network, ip: "192.168.33.11"
     
     #config.hostsupdater.aliases = ["development.db"]     
   end
 
 
 end
```
# eng103a_IaC_ansible
---------------

What is infrastructure as code (IaC)? IaC means to codify everything from end to end in a language that machines understand. It is the management of IT infrastructure using configuration files.

 YAML/YML stands for Yet Another Markup Language. It is an extremely powerful language as it:
 - Easy to understand for 'humans'
 - Portable across most programming languages
 - Easy to implement

Ansible is an open source automation tool using scripting procedures. It is a configuration management tool using a language that machines will understand. Essentially, the end result should be a file that we can use to communicate with hundreds of servers.

Playbook is an advanced form of shell script. We want it to go between various instances and nodes using one command. 

Orchestration with terraform - we can use orchestration and scheduling tools strive to eliminate silos, streamline processes and automate repetitive tasks so that IT departments can move quickly and efficiently.

Ansible is agentless -  Agentless means you don't have to have an additional agent pre-installed on the box (e.g. db and web)

[Ansible is agentless, meaning that it does not install software on the nodes that it manages. This removes a potential point of failure and security vulnerability and simultaneously saves system resources.](https://searchitoperations.techtarget.com/definition/Ansible#:~:text=Ansible%20is%20agentless%2C%20meaning%20that,and%20simultaneously%20saves%20system%20resources.)

### Setting up the Ansible installer
- install required dependencies. e.g. python.
- instal ansible
- tree
- set up agent nodes
- default folder structure for ansible controller, once it is installed it is in /etc/ansible 
- - in that location we have a hosts file
- we have to let the host file what agent nodes to talk to and how.
- hosts file - agent node called web and the ip of the web. Ansible controller looks into host file for these and will ssh in.
- sudo nano /etc/ansible/hosts

## Setting up ansible on Vagrant on localhost
-----
Using the Vagrantfile:
- vagrant up to launch the 3 vms
- enter each vm and do `sudo apt-get update -y && sudo apt-get upgrade -y`
- Enter the controller by `ssh` and run the commands:
```
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible # add repo
sudo apt-get install ansible
ansible --version
cd /etc
cd ansible/
sudo apt install tree
tree
ssh vagrant@192.168.33.10 # web vm
# password is vagrant
ssh vagrant@192.168.33.11 # db vm
# password is vagrant
```
The purpose of entering the web and db vm is to copy keys from controller to web and db vm.

To ping our web and db vms, we can do `ping xx.xx.xxx.xxxx`

To access the web and db vms we have to edit the hosts file so the controller can ssh in
```
cd /etc/ansible
sudo nano hosts
[web] 192.168.33.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant 
[db] 192.168.33.11 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
```

Afterwards, we can do `sudo ansible all -m ping` to see what servers we have.

<img src="https://user-images.githubusercontent.com/98178943/154301577-215b26c5-21e2-4f7f-9642-4234dfce0778.png" width="400">

### Ad hoc commands for ansible
Ad hoc commands are a fast way to interact with servers from the controller.

Some useful commands are:
```
sudo ansible all/web/db -m ping
sudo ansible all/web/db -a "uname -a"
sudo ansible db -a "ls -a"
sudo ansible web -a "free" # view memory
ansible all -m copy -a "src=/etc/ansible/README.md dest=/home/vagrant/README.md" # copy files from one dest to another

### Configuration Management with playbooks
Playbooks are YAML files with scripts to implement config management
They save time and are **reusable** - let's say we want to run the playbook on the cloud, we don't need to make many changes apart from IP addresses and keys to access the cloud. The script doesn't change.

To create a playbook file:
- `sudo nano install_nginx.yml`
- it starts with three dashes (---) inside the yaml file. 

Running a playbook within a playbook:
```
import pytest
import file.yml

## Creating a playbook to install nginx
```
# configure and install nginx in web node
---
# Which host we install nginx in
- hosts: web
  gather_facts: yes

# collect logs or gather facts -


# we need admin access to install anything
  become: true
# add the instructions to install nginx in web server
  tasks:
  - name: Installing Nginx in web Agent Node
    apt: pkg=nginx state=present
# HINT: be mindful of indentation
# use 2 spaces - avoid using tab
```
## installing node and running app on port 3000

Edit Vagrantfile:
```
 
# -*- mode: ruby -*-
 # vi: set ft=ruby :
 
 # All Vagrant configuration is done below. The "2" in Vagrant.configure
 # configures the configuration version (we support older styles for
 # backwards compatibility). Please don't change it unless you know what
 
 # MULTI SERVER/VMs environment 
 #
 Vagrant.configure("2") do |config|
    # creating are Ansible controller
      config.vm.define "controller" do |controller|
        
       controller.vm.box = "bento/ubuntu-18.04"
       
       controller.vm.hostname = 'controller'
       
       controller.vm.network :private_network, ip: "192.168.33.12"
	   
	   controller.vm.synced_folder "./app", "/home/vagrant/app"
       
       # config.hostsupdater.aliases = ["development.controller"] 
       
      end 
    # creating first VM called web  
      config.vm.define "web" do |web|
        
        web.vm.box = "bento/ubuntu-18.04"
       # downloading ubuntu 18.04 image
    
        web.vm.hostname = 'web'
        # assigning host name to the VM
        
        web.vm.network :private_network, ip: "192.168.33.10"
		# web.vm.synced_folder "./app", "/home/vagrant/app"
        #   assigning private IP
        
        #config.hostsupdater.aliases = ["development.web"]
        # creating a link called development.web so we can access web page with this link instread of an IP   
            
      end
      
    # creating second VM called db
      config.vm.define "db" do |db|
        
        db.vm.box = "bento/ubuntu-18.04"
        
        db.vm.hostname = 'db'
        
        db.vm.network :private_network, ip: "192.168.33.11"
        
        #config.hostsupdater.aliases = ["development.db"]     
      end
    
    
    end
```

Copy app folder from controller to web vm:
```
# yml file to copy over file
---

- hosts: web

  gather_facts: yes

  become: true

  tasks:
  - name: copying file over to web VM
    copy:
      src: /home/vagrant/app
      dest: /home/vagrant/
```

Create a provision script to install dependencies:
```
#!/bin/bash

# Update
sudo apt-get update -y

#Upgrade packages
sudo apt-get upgrade -y

# install dependencies
sudo apt-get install nginx -y
sudo systemctl restart nginx
sudo systemctl enable nginx
sudo apt-get install python-software-properties
# curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install nodejs -y
sudo apt-get install npm -y
sudo npm install pm2 -g
```

Creating a playbook to run script:
```
---
- name: Install node.js and app on web
  hosts: web
  gather_facts: yes
  tasks:
  - name: load node v6
    shell: curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
  - name: Run script
    script: /etc/ansible/provision_web.sh
```

## Alternatively, we can do the entire copying and installing packages into one yml file instead of using a provision script:
```
---
# web vm
- hosts: web

# gather logs
  gather_facts: yes

# give admin access
  become: true

# install dependencies
  tasks:
  - name: Copy app folder from local to web node
    synchronize:
      src: /home/vagrant/app/
      dest: /home/vagrant/

  - name: Add nodejs apt key
    apt_key:
      url: https://deb.nodesource.com/gpgkey/nodesource.gpg.key
      state: present

  - name: add nodejs 6.x version
    apt_repository:
      repo: deb https://deb.nodesource.com/node_6.x bionic main
      update_cache: yes

  - name: Install nodejs
    apt: pkg=nodejs state=present

  - name: Install npm
    apt: pkg=npm state=present
```
The website should be running on port 3000.

## Installing mongodb in vm

`sudo nano mongo.yml` to create a new yml file:
```
# Install mongo in db VM
---
- hosts: db

# Gather facts
  gather_facts: yes
# admin access
  become: true

# install mongo
  tasks:
  - name: Installing mongodb in db vm
    apt: pkg=mongodb state=present
```

Check the status of mongod by doing `ansible db -a "systemctl status mongodb"`

## Change mongod ip to 0.0.0.0
```
- hosts: db
# gather logs
  gather_facts: yes
# admin access
  become: true

  tasks:
  - name: change mongod ip
#    shell: sudo sed -i 's/127.0.0.1/0.0.0.0/' /etc/mongod.conf
    lineinfile:
      path: /etc/mongodb.conf
      # specifies what string to replace
      regexp: '^bind_ip ='
      # replaces string
      line: bind_ip = 0.0.0.0
```
## install mongodb pre-requisites
```
---
-  hosts: db
   gather_facts: yes
   become: true
   tasks:
   - name: installing mongo pre-requisites
     shell: sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927; echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodbs

   - name: restart mongod
     service: name=mongodb state=restarted

   - name: mongod enable
     service: name=mongodb enabled=yes
```

## Optionally put it all into one for mongod
```
---
- hosts: db
  gather_facts: yes
  become: true
# install mongodb in db instance and ensure it's running
  tasks:
  - name: installing mongo pre-requisites
    apt: pkg=mongodb state=present
  - name: allow 0.0.0.0
    ansible.builtin.lineinfile:
      path: /etc/mongodb.conf
      regexp: '^bind_ip = '
      insertafter: '^#bind_ip = '
      line: bind_ip = 0.0.0.0
  - name: restart and enable mongod
    service: name=mongodb state=restarted enabled=yes
```

## Add reverse proxy to web app
```

---
# reverse proxy
- hosts: web

  gather_facts: yes

  become: yes

  tasks:
#  - name: load node v6
#   shell: curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
#  - name: Run script
#    script: /etc/ansible/provision_web.sh

   - name: reverse proxy
     synchronize:
       src: /home/vagrant/app/default
       dest: /etc/nginx/sites-available/default

   - name: restart nginx
     service: name=nginx state=restarted

   - name: enable nginx
     service: name=nginx enabled=yes
```
## set up ports to access db on web vm by creating environment var
```
---
-  hosts: web
   gather_facts: yes
   become: yes
   tasks:
     - name: Insert line at end of file
       lineinfile:
         path: /home/vagrant/.bashrc
         line: "export DB_HOST='mongodb://192.168.33.11:27017/posts'"
```

## source .bashrc
```
---
- hosts: web

  gather_facts: yes

  become: true
  tasks:
    - name: sourcing .bashrc
#      sudo: no
      shell: source /home/vagrant/.bashrc
      args:
        executable: /bin/bash
```

## Importing playbooks
```
# importing all the playbooks to set up web app
---

# for web
- name: update upgrade
  import_playbook: update_upgrade.yml
   
- name: Install nginx
  import_playbook: install_nginx.yml

- name: install web app with access to port 3000
  import_playbook: install_app_web.yml

- name: install reverse proxy
  import_playbook: reverse_proxy.yml

- name: change ports
  import_playbook: change_ports.yml

- name: source .bashrc
  import_playbook: source_bashrc.yml
```

Running it all at once for web^

```
# for db
- name: update upgrade
  import_playbook: update_upgrade.yml

- name: Install mongodb
  import_playbook: mongo.yml

- name: Import mongo keys
  import_playbook: mongodb_prerq.yml

- name: change mongodb ip
  import_playbook: change_mongod_conf.yml
```
Running it all for the db set up^

We can also automate the upgrade-update process:
```
---
- hosts: all
  become: yes
  tasks:
  - name: Update and upgrade apt packages
    apt:
      upgrade: yes
      update_cache: yes
```

## Ultimate Automation!
We can put all of these playbooks in playbooks into ONE playbook that will completely install app and db with one click:
Playbook name = `master_import_playbooks.yml`

```
# launch all playbooks for db and web from scratch
- name: web playbooks
  import_playbook: import_playbooks.yml

- name: db playbooks
  import_playbook: import_db_playbooks.yml
```
![The beauty of DevOps!](https://user-images.githubusercontent.com/98178943/154564715-215163e8-8657-41ff-bb78-8a8fe17dd3ba.png)
![ezgif com-gif-maker](https://user-images.githubusercontent.com/98178943/154567191-b3c8ead2-d0aa-4b81-9b6a-913ffa5ceb59.gif)


## Ansible hybrid deployment

Make fresh vm and update/upgrade
- `sudo apt-add-repository --yes --update ppa:ansible/ansible` adds ansible folder
- `sudo apt-get install ansible`
- `sudo apt-get install python3-pip` install python 3
- `pip3 install awscli`
- `pip3 install boto boto3`
- `aws --version`
- `cd /etc/ansible`
- Create a group_vars folder and then an `all` folder inside it. (directory /etc/ansible/group_vars/all$)
- `sudo ansible-vault create pass.yml`
- Enter `ii` to enter insert mode, enter your access and secret keys as:
```
aws_access_key:
aws_secret_key:
```
- Save the file by pressing `esc` and typing `:wq!`
- `sudo cat pass.yml` to view contents
![image](https://user-images.githubusercontent.com/98178943/154943905-65b2fce5-1ce5-4426-a2fe-59f4f7c9b60b.png)
- `sudo chmod 600 pass.yml` gives required permission to read
- `sudo ansible all -m ping --ask-vault-pass`, here we are checking if it is working and it is verifying the keys we have entered

We have created a secure authentication method using ansible for hybrid cloud.

We have provided aws credentials that were encrypted, so we did not need to expose the keys at any stage.

Enter into /home/vagrant/.ssh and enter `ssh-keygen -t rsa -b 4096` to generate the pub and private keys which we'll need later on.

Create a playbook called `ec2.yml`
```

---
- hosts: localhost
  connection: local
  gather_facts: yes
  vars_files:
  - /etc/ansible/group_vars/all/pass.yml
  vars:
    key_name: id_rsa
    region: eu-west-1
    image: ami-07d8796a2b0f8d29c
    id: "yacob-ansible5"
    sec_group: "{{ id }}-sec"
  tasks:
    - name: Provisioning EC2 instances
      block:
      - name: Upload public key to AWS
        ec2_key:
          name: "{{ key_name }}"
          key_material: "{{ lookup('file', '/home/vagrant/.ssh/id_rsa.pub') }}"
          region: "{{ region }}"
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
      - name: Create security group
        ec2_group:
          name: "{{ sec_group }}"
          description: "Sec group for app {{ id }}"
          region: "{{ region }}"
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
          rules:
            - proto: tcp
              ports:
                - 22
              cidr_ip: 0.0.0.0/0
              rule_desc: allow all on ssh port
            - proto: tcp
              ports:
                - 80
              cidr_ip: 0.0.0.0/0
              rule_desc: allow all on http port
        register: result_sec_group
      - name: Provision instance(s)
        ec2:
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
          key_name: "{{ key_name }}"
          id: "{{ id }}"
          group_id: "{{ result_sec_group.group_id }}"
          image: "{{ image }}"
          instance_type: t2.micro
          region: "{{ region }}"
          wait: true
          count: 1
          instance_tags:
            name: eng103a_yacob_ansible
      tags: ['never', 'create_ec2']

```
Enter `sudo ansible-playbook ec2.yml --ask-vault-pass --tags create_ec2 --tags=ec2-create -e "ansible_python_interpreter=/usr/bin/python3"`

or `sudo ansible-playbook start_ec2.yml --connection=local -e "ansible_python_interpreter=/usr/bin/python3" --ask-vault-pass`

Once that is done, we can edit the hosts file to ssh into the instance we made.
An example:
```
[aws]
54.74.xx.xx ssh_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa
```

To test the connection, enter:
`sudo ansible aws -m ping --ask-vault-pass`

This image should pop up:

![image](https://user-images.githubusercontent.com/98178943/154974150-83687620-054f-4bca-a28a-a69cb802809d.png)

To install nginx, construct a playbook, changing the hosts to the name you entered in the hosts file.

## Creating an Ansible controller on AWS

Follow the commands from setting up an Ansible controller.

Enter into /home/ubuntu/.ssh and enter `ssh-keygen -t rsa -b 4096` to generate the pub and private keys which we'll need later on.

To launch the app and db instances I entered the code below:

APP:
```yml
---
- hosts: localhost
  connection: local
  gather_facts: yes
  vars_files:
  - /etc/ansible/group_vars/all/pass.yml
  vars:
    ec2_instance_name: eng103a-yacob-ansible-app
    ec2_sg_name: eng103a_yacob_vpc_app
    ec2_pem_name: eng103a
  tasks:
  - ec2_key:
      name: "{{ec2_pem_name}}"
      key_material: "{{ lookup('file', '/home/ubuntu/.ssh/id_rsa.pub') }}"
      region: "eu-west-1"
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
  - ec2:
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
      key_name: "{{ec2_pem_name}}"
      instance_type: t2.micro
      image: ami-07d8796a2b0f8d29c
      wait: yes
      group: "{{ec2_sg_name}}"
      region: "eu-west-1"
      count: 1
      vpc_subnet_id: subnet-xxxxxxxxxx
      assign_public_ip: yes
      instance_tags:
        Name: "{{ec2_instance_name}}"
```
DB:
```yml
---
- hosts: localhost
  connection: local
  gather_facts: yes
  vars_files:
  - /etc/ansible/group_vars/all/pass.yml
  vars:
    ec2_instance_name: eng103a-yacob-ansible-db
    ec2_sg_name: eng103a_yacob_vpc_db_sg
    ec2_pem_name: eng103a
  tasks:
  - ec2_key:
      name: "{{ec2_pem_name}}"
      key_material: "{{ lookup('file', '/home/ubuntu/.ssh/id_rsa.pub') }}"
      region: "eu-west-1"
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
  - ec2:
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
      key_name: "{{ec2_pem_name}}"
      instance_type: t2.micro
      image: ami-07d8796a2b0f8d29c
      wait: yes
      group: "{{ec2_sg_name}}"
      region: "eu-west-1"
      count: 1
      vpc_subnet_id: subnet-xxxxxxxxxxxxxx
      assign_public_ip: yes
      instance_tags:
        Name: "{{ec2_instance_name}}"
```

To launch these instances enter `sudo ansible-playbook start_ec2.yml --connection=local -e "ansible_python_interpreter=/usr/bin/python3" --ask-vault-pass`

Note: when running the playbooks, we need to make sure we set user to ubuntu and to set the paths to ubuntu before executing the playbooks.