# Ansible-Basic-Playbook

## Objective

The objective of this lab is to create a Basic Ansible Playbook, ensure Services are Started in my Ansible Playbook and adding variables to my Ansible Playbook.

### Skills Learned

Upon completion of this intermediate-level lab, you will be able to:

Use the command line to create a new Ansible playbook
Install and configure the Apache webserver using an Ansible playbook
Make our playbook more reusable by adding variables

### Tools Used

- Amazon AWS - EC2 Instance
- Ansible
- Ubuntu Linux server

## What is Ansible

Ansible is an open-source automation tool designed for IT tasks such as configuration management, application deployment, orchestration, and provisioning. It enables IT administrators and DevOps teams to automate repetitive tasks, ensuring consistency and efficiency across systems.

## Step 1 - Check Ansible version

To check if ansible is installed and what version is installed you can use the command

```
ansible --version
```

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/fd26ad18-a4de-4a08-9213-8f3e43f50383)

## Step 2 - Creating a .Yaml file and running it

I am creating a basic playbook that consists of 1 task that installs 4 packages.
Be aware that YAML syntax is sensitive to whitespace. In YAML, the level of indentation matters as does whether you use spaces in tabs. To avoid errors, use spaces instead of tabs to indent YAML lines.

```
nano lamp.yml
---
- hosts: localhost
  gather_facts: false
  connection: local
  become: yes

  tasks:
    - name: Install our packages
      apt:
        name: ['apache2', 'mysql-server', 'mysql-common', 'mysql-client']
        state: present                
        update_cache: true
```

Next we will run the Playbook, but before we do we can verify that the required packages are not currently installed by issuing the following command

```
dpkg -l apache2 mysql-server
```

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/dd951692-3e8d-4a6e-a6a4-bed1cb46c789)

Now lets run the playbook

```
ansible-playbook lamp.yml
```

Because the inventory file isn't configured you'll see a warning. However, it doesn't matter when using Ansible to configure localhost.

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/3a198507-774a-41c8-b3fe-09dcff12b126)

To confirm that the packages have been installed, re-run the dpkg command

```
dpkg -l apache2 mysql-server
```

This time we can see output ending with the following, confirming that the packages have been installed

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/43b71f44-0b21-4b23-8ec1-bd05fe75133d)

## Step 3 - Ensuring Services are Started in your Ansible Playbook

In this lab step, we will modify our Ansible playbook to start the Apache webserver and Mysql server that we installed

First we will open our playbook file in nano

```
nano lamp.yml
```

now adding on the following to the end of the existing code

```
    - name: Confirm services are running
      service:
        name: "{{ item }}"
        state: started
      with_items:
        - apache2
        - mysql
```

The lamp.yml playbook should look like this now

```
---
- hosts: localhost
  gather_facts: false
  connection: local
  become: yes

  tasks:
    - name: Install our packages
      apt:
        name: ['apache2', 'mysql-server', 'mysql-common', 'mysql-client']
        state: present                
        update_cache: true
    - name: Confirm services are running
          service:
            name: "{{ item }}"
            state: started
          with_items:
            - apache2
            - mysql
```

To start the Apache and Mysql services, re-run your playbook

```
ansible-playbook lamp.yml
```

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/1206bcf5-eb8e-4f1d-94f1-09541d96eab8)

To check if the apache server started we can get the ip

```
curl http://checkip.amazonaws.com
```

Then copy this IP into a web browser

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/8a6efbdf-b222-44b9-9b50-9c42586e3301)

## Step 4 - Adding Variables to your Ansible Playbook

Ansible allows you to use variables to make your playbooks more reusable because you can edit the variables at runtime. 

Lets edit our lamp.yml file again

```
nano lamp.yml
```

The services and packages are currently both static, so we will create variables for these. we will delete all the contents of the existing file (ctrl+k) and add in the new code

```
---
- hosts: localhost
  gather_facts: false
  connection: local
  become: yes

  vars:
    packages:
      - apache2
      - mysql-server
      - mysql-common
      - mysql-client
    services:
      - apache2
      - mysql

  tasks:
    - name: Install our packages
      apt:
        name: "{{ packages }}"
        state: present
    - name: Confirm services are running
      service:
        name: "{{ item }}"
        state: started
      with_items: "{{ services }}"
```

In this new version we have defined a vars YAML mapping with sub mappings for packages and services. The value of the sub mappings are arrays. The arrays contain the values that we had hard-coded in the task definitions previously.

Lets re-run the playbook

```
ansible-playbook lamp.yml
```

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/ce819d28-3a9c-4716-a750-1cceb3c0be89)

We will see the same output as before. The playbook is functionally the same, except now we are using variables to define the services and packages.

By default, Apache only has HTTP enabled. To enable an HTTPS site in Apache, we need to run the following commands

```
sudo a2enmod ssl
sudo a2ensite default-ssl
```

We don't need to run these commands manually, we will make ansible do that for us

We will open our lamp.yml file again and add the following to the bottom

```
- name: Enable Apache2 modssl
  shell: a2enmod ssl
- name: Enable Apache2 Default HTTPS site
  shell: a2ensite default-ssl
```

These tasks use Ansible's shell service to run shell commands on the server

After re-running the playbook if we return to our site with the Apache webserver's default site open and prefix the IP address with https:// you will see a message that the site can't be reached. This is because after enabling the SSL module and the HTTPS site, Apache needs to be restarted for the changes to take effect.

We need to open our lamp.yml file again and add in at the bottom

```
- name: Restart Apache 
      service:
        name: apache2
        state: restarted
```

After running we can see that Ansible returned changed for the tasks to enable the SSL module and the HTTPS site. This is because even though they are already enabled, Ansible does not know this and cannot detect it.

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/6a240d13-5076-403a-93a4-c587b63aa38d)

Testing the browser again this time with https:// we can confirm that we have successfully enabled the default HTTPS Apache site

![image](https://github.com/Matt4llan/Ansible-Basic-Playbook/assets/156334555/0706bec2-65e3-492e-9f49-4dce3ebc16d9)
