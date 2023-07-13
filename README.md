
##ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project we will continue working with 'ansible-config-mgt' repository and make some improvements of our code. Now you need to refactor our Ansible code, create assignments, and learn how to use the imports functionality. Imports allows to effectively re-use previously created playbooks in a new playbook – it helps to organize tasks and reuse them when needed.

Firstly, let's explain code refactoring.

[_ **Refactoring** _](https://en.wikipedia.org/wiki/Code_refactoring) is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.It involves making changes to the code's internal structure, organization, and implementation while preserving its functionality.

###**STEP 1- JENKINS JOB ENHANCEMENT**

Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require [_ **Copy Artifact** _](https://plugins.jenkins.io/copyartifact/) plugin.

1. Go to your Jenkins-Ansible server and create a new directory called 'ansible-config-artifact' – which will be utilized to store all artifacts after each build.

**$ sudo mkdir /home/ubuntu/ansible-config-artifact**

1. Change permissions to this directory, so Jenkins could save files there –

**$ chmod -R 0777 /home/ubuntu/ansible-config-artifact**.

![](RackMultipart20230712-1-gghol5_html_d0f35ecd37c0745a.png)

1. Go to Jenkins web console -\> **Manage Jenkins** -\> **Manage Plugins** -\> on **Available plugins** tab search for **Copy Artifact** and install this plugin without restarting Jenkins.

![](RackMultipart20230712-1-gghol5_html_b1e83cdc72cb4ef.png)

1. We will create a new Freestyle project and name it **'save\_artifacts'**.

1. This project will be triggered by completion of your existing ansible project. Hence, we will configure it accordingly:

![](RackMultipart20230712-1-gghol5_html_f3df3534bcfcadb0.png)

![](RackMultipart20230712-1-gghol5_html_e2eed18d20b02c75.png)

**Please note:** Depending on the number of builds you might want to keep, say only last 2 or 5 build results, you can configure number of builds to keep in order to save space on the server.

1. The main idea of 'save\_artifacts project' is to save artifacts into **/home/ubuntu/ansible-config-artifact** directory. To achieve this, create a Build step and choose **Copy artifacts from other project** , specify **ansible** as a source project and **/home/ubuntu/ansible-config-artifact** as a target directory.

![](RackMultipart20230712-1-gghol5_html_ce30dde1e1d282d6.png)

1. Test your set up by making some changes in README.MD file inside your **ansible-config-mgt repository** (right inside main/master branch).

If both Jenkins jobs have completed one after another – you shall see your files inside **/home/ubuntu/ansible-config-artifact directory** and it will be updated with every commit to your master branch.

Now your Jenkins pipeline is neater and cleaner!.

![](RackMultipart20230712-1-gghol5_html_f1ded4cd3740f07f.png)_ **Image above shows build triggered in 'save-artifacts' project.** _

![](RackMultipart20230712-1-gghol5_html_b6de29fd7e29a01.png)

_ **Image above also shows build triggered simultaneously in 'ansible' project on Jenkins.** _

![](RackMultipart20230712-1-gghol5_html_5595290a84e5b683.png)

**STEP 2. REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO site.yml**

Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it _refactor._

$ git pull

$ git branch refactor

![](RackMultipart20230712-1-gghol5_html_e6ffb9ef3310100a.png)

1. Within _ **playbooks** _ folder, create a new file and name it _ **site.yml** _ – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, _ **site.yml** _ will become a parent to all other playbooks that will be developed. Including _ **common.yml** _ that you created previously. Don't worry, you will understand more what this means shortly.??

1. Create a new folder in root of the repository and name it **static-assignments**. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.??

![](RackMultipart20230712-1-gghol5_html_96af36c08e763f2e.png)

1. Move _ **common.yml** _ file into the newly created **static-assignments** folder.

1. Inside _ **site.yml** _ file, import _ **common.yml** _ playbook.

![](RackMultipart20230712-1-gghol5_html_e9b7809dd508111e.png)

update _site.yml_ with - import\_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against _dev_ servers:

![](RackMultipart20230712-1-gghol5_html_c5054928efaec0c.png)

**$ cd /home/ubuntu/ansible-config-mgt/**

**$ ansible-playbook -i inventory/dev.yml playbooks/site.yaml**

Wait!!! Before running the ansible-playbook command we need to push our code changes unto github.

**Commit your code into GitHub:**

1. Use git commands to **add,**** commit **and** push** your branch to GitHub.

**$ git status -this shows us what branch we are currently working on.**

In this case, we are currently on our feature/project-45 branch.

**$ git add \<selected files\>**

**$ git add .** (to add every change in the ansible-config-mgt repository)

**$ git push**

![](RackMultipart20230712-1-gghol5_html_164b66f9e303abdb.png)

**$ ansible-playbook -i inventory/dev.yml playbooks/site.yaml**

![](RackMultipart20230712-1-gghol5_html_da4b3f22ed623c21.png)

Make sure that wireshark is deleted on all the servers by running

$ **wireshark --version**

![](RackMultipart20230712-1-gghol5_html_1ed17479c5df0250.png)

Now you have learned how to use _import\_playbooks_ module and you have a ready solution to install/delete packages on multiple servers with just one command.

**STEP 3. CONFIGURE UAT WEBSERVERS WITH A ROLE 'WEBSERVER'**

We have our nice and clean _ **dev** _ environment, so let us put it aside and configure 2 new Web Servers as _ **uat** _. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated [_ **role** _](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) to make our configuration reusable.

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our _ **uat** _ servers, so give them names accordingly –  **Web1-UAT**  and  **Web2-UAT**.

1. To create a role, you must create a directory called  **roles/**** , **relative to the playbook file or in ** /etc/ansible/** directory.

1. Use an Ansible utility called **ansible-galaxy** inside **ansible-config-mgt/roles** directory (you need to create roles directory upfront)

**$ mkdir roles**

**$ cd roles**

**$ ansible-galaxy init webserver**

You can also do this directly on VSCode editor or via the terminal.

Under the directory _ **roles/webserver** _, remove unnecessary directories and files so that the _ **roles** _ structure looks exactly like this below.

![](RackMultipart20230712-1-gghol5_html_51f06fd5486aabb1.png)

└── **webserver**

├── README.md

├── defaults

│ └── main.yml

├── handlers

│ └── main.yml

├── meta

│ └── main.yml

├── tasks

│ └── main.yml

└── templates

1. Now we will Update our inventory **ansible-config-mgt/inventory/uat.yml** file with IP addresses of our 2 UAT Web servers.

[**uat-webservers]**

\<Web1-UAT-Server-Private-IP-Address\> ansible\_ssh\_user='ec2-user'

\<Web2-UAT-Server-Private-IP-Address\> ansible\_ssh\_user='ec2-user'

![](RackMultipart20230712-1-sr7fh8_html_11522b3a2aedee3b.png)

1. In _ **/etc/ansible/ansible.cfg** _ file uncomment  **roles\_path**  string and provide a full path to your roles directory _ **roles\_path = /home/ubuntu/ansible-config-mgt/roles** __ **,** _ so Ansible can know where to find configured roles

**$ sudo vi /etc/ansible/ansible.cfg**

![](RackMultipart20230712-1-sr7fh8_html_ca62039449e82109.png)

1. We will now start adding some logic to the webserver role. Go into _ **tasks** _ directory, and within the _ **main.yml** _ file, start writing configuration tasks to do the following:

- Install and configure Apache ( **httpd**  service)
- Clone  **Tooling website**  from GitHub [https://github.com/\<your-name\>/tooling.git](https://github.com/%3Cyour-name%3E/tooling.git). (from Project 7) or any repository of your choice
- Ensure the tooling website code is deployed to  **/var/www/html****  **on each of 2 UAT Web servers.
- Make sure  **httpd**  service is started.

Your _ **main.yml** _ may consist of following tasks:

---

- name: install apache

become: true

ansible.builtin.yum:

name: "httpd"

state: present

- name: install git

become: true

ansible.builtin.yum:

name: "git"

state: present

- name: clone a repo

become: true

ansible.builtin.git:

repo: https://github.com/\<your-name\>/tooling.git

dest: /var/www/html

force: yes

- name: copy html content to one level up

become: true

command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started

become: true

ansible.builtin.service:

name: httpd

state: started

- name: recursively remove /var/www/html/html/ directory

become: true

ansible.builtin.file:

path: /var/www/html/html

state: absent

**STEP 4. REFERENCE 'WEBSERVER' ROLE**

Now we will create a new assignment for  **uat-webservers**  called _ **uat-webservers.yml** __ **.** _This is where you will reference the role. This has to be created in the _ **static-assignments** _ folder.

---

- hosts: uat-webservers

roles:

- webserver

![](RackMultipart20230712-1-sr7fh8_html_d7c7336ff28f4e36.png)

Remember that the entry point to our ansible configuration is the _ **site.yml** _ file. Therefore, you need to refer your _ **uat-webservers.yml** _ role inside _ **site.yml** _.

So, we should have this in _ **site.yml** _ file.

---

- hosts: all

- import\_playbook: ../static-assignments/common.yml

- hosts: uat-webservers

- import\_playbook: ../static-assignments/uat-webservers.yml

**STEP 5. COMMIT & TEST**

Now we will commit our changes, create a Pull Request and merge them to master branch, and make sure webhook triggers two consequent Jenkins jobs. We will also ensure they ran successfully and have copied all the files to our Jenkins-Ansible server into **/home/ubuntu/ansible-config-mgt/** directory.

**Commit your code into GitHub:**

1. Use git commands to **add,**** commit **and** push** your branch to GitHub.

**$ git status -this shows us what branch we are currently working on.**

In this case, we are currently on our refactor branch.

**$ git add .** # _to add all the files in the current directory_ **'ansible.config.mgt'**

**$ git commit -m "commit message"**

Commit message can be a short phrase. For example, to indicate what has been added.

**$ git push**

1. **Create a Pull request (PR)**

Merge the code to the master branch. Go to your github and create pull request.

**$ git push origin**

![](RackMultipart20230712-1-sr7fh8_html_45665c642ad33249.png)

![](RackMultipart20230712-1-sr7fh8_html_88e5a60f68363de5.png)

![](RackMultipart20230712-1-sr7fh8_html_1ec0f422c963ac6c.png)

# Git merge

**$ git checkout main**

**$ git merge refactor**

**$ git add .**

**$ git merge –continue**

![](RackMultipart20230712-1-sr7fh8_html_9daea53e7b6e6d24.png)

![](RackMultipart20230712-1-sr7fh8_html_d43cf46bd21b906a.png)

![](RackMultipart20230712-1-sr7fh8_html_e6fad4b53b7ab2b0.png)_ **Build triggered above. See commit message.** _

Next, we will ssh in to the uat-webservers from out Ansible host

**$ eval `ssh-agent -s`**

**$ ssh-add \<path/to/private-key\>**

Confirm the key has been added with the command below. You should see a fingerprint of the private key.

**$ ssh-add -l**

**$ ssh -A ec2-user@public-ip**

![](RackMultipart20230712-1-sr7fh8_html_9b1d6ec6ed41a187.png)

_ **Now run the playbook against your uat inventory and see what happens:** _

_ **$ ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml -v** _

_Upon running this ansible command, I encountered an error as seen below._

**[35.176.77.126]: UNREACHABLE! =\> {"changed": false, "msg": "Failed to connect to the host via ssh: ec2-user@35.176.77.126: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).", "unreachable": true} fatal: [13.40.105.188]: UNREACHABLE! =\> {"changed": false, "msg": "Failed to connect to the host via ssh: ec2-user@13.40.105.188: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).", "unreachable": true}**

![](RackMultipart20230712-1-sr7fh8_html_49407f6449ce11e.png)

_To resolve this, I modified_ _the_ _ **ansible.cfg** _ _file in my working (playbook) directory._

$ sudo vi /etc/ansible/ansible.cfg

**[defaults]**

inventory = /home/ubuntu/ansible-config-mgt/inventory

private\_key\_file = /ubuntu/.ssh/id\_rsa or path/to/private\_key.pem

remote\_user = \<SSH\_USERNAME\>

_and saved the changes and run._

You should be able to see that the play is being run successfully.

![](RackMultipart20230712-1-sr7fh8_html_c34ad631d55ba4f6.png)

![](RackMultipart20230712-1-sr7fh8_html_f8759f30f6f0de6f.png)

Both of our UAT Web servers are now configured, and we can try to reach them from your browser:

http://\<Web1-UAT-Server-Public-IP-or-Public-DNS-Name\>/index.php

or

http://\<Web2-UAT-Server-Public-IP-or-Public-DNS-Name\>/index.php

![](RackMultipart20230712-1-sr7fh8_html_dfa7fc6c5d3e7c3f.png)

Now our Ansible architecture looks like the diagram below:

![](RackMultipart20230712-1-sr7fh8_html_cf14ed19324cd962.png)

7 **|** Page
