# IAC with Ansible


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
sudo ansible db -a "ls-a"
sudo ansible web -a "free" # view memory
ansible all -m copy -a "src=/etc/ansible/README.md dest=/home/vagrant/README.md" # copy files from one dest to another