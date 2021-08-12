# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

This project gives an indepth understanding of how to apply configuration management to a group of servers using static assignments, static assignments involves the use of `imports` & `roles` all in the bid to make our playbooks reuseable, and which allows you to organize your tasks and reuse them when needed. Let me put this in simpler terms, imagine you write a piece of code to implement a particular function, and some days later you need to implement that function in another piece of code, would you start writing it from scratch?. the best way would be to *import* that function into your code. That's basically what we'll be implementing in this project.

To better understand this project, take a look at [ansible-config-mgt](https://github.com/brpo01/ansible-conf-11). This is meant to be a continuation of that project.

# Jenkins Job Enhancement
- To begin, let's improve how our builds are being run in Jenkins. By default when the builds are run and the artifacts are archived, a new folder always get's created in the /var/lib/jenkins/jobs/ansible-config-mgt/builds/<build_number>/archive directory and this takes up space on the jenkins server with every subsequent change.

- Create a new folder in the root directory and call it `ansible-config-artifacts`, this is where we will store all the build artifacts.

```
$ sudo mkdir ansible-config-mgt
```

- Change the permissions on the file, to allow the jenkins server save files in there

```
$ sudo chmod 777 ansible-config-artifacts
```

- Install the **Copy Artifact** plugin from the Jenkins Dashboard, this will enable the jenkins server to copy over the build artifacts to the newly created folder.

- Create a freestyle project & call it **save-artifacts**, this new job will be configured to send the build artifacts to the new folder we just created

- Configure the project to look like the below images:

![1](https://user-images.githubusercontent.com/47898882/129137570-908055c2-366d-4ff3-8301-1f2db39f3248.JPG)

![2](https://user-images.githubusercontent.com/47898882/129137574-4afc2f8f-51d0-46ad-814f-387f8b68f0b7.JPG)

![3](https://user-images.githubusercontent.com/47898882/129137575-6cc043b7-3195-4f54-b6fd-a94444fd6774.JPG)

- From the image above, set the general section of the freestyle project to discard old builds using a strategy of log rotation and specify the number of builds you want to keep at a given specfic time. In the build trigger section, set the build trigger to build after other projects have been built, and in this case that'll be the `ansible-config-mgt` project, it will act as an upstream project for the `save-artifacts` projects. Finally, in the build section, let this project copy the latest build artifacts from the `ansible-config-mgt` project, you also see that we set the target directory to the newly created folder in /home/ubuntu/ansible-config-artifacts directory.

- Verify your setup works by making a change in the master branch of your repository and commit the changes. Make sure you have configured your webhook. If everything was configured accurately, the two jobs will run subsequently and you should find the latest build artifact within the `ansible-config-artifacts` folder.

![4](https://user-images.githubusercontent.com/47898882/129138463-dfe563d3-5bb5-4109-83a1-7aa1325c6887.JPG)

![5](https://user-images.githubusercontent.com/47898882/129138712-948fd2a7-bcae-49db-9f3a-7db4e2334e8b.JPG)

# REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS
- In this section, we'll refactor the playbook we created in the last project to become reusable. DevOps philosophy implies constant iterative improvement for better efficiency – refactoring is one of the techniques that can be used. Imagine you want to confiugure several tasks on multiple target servers, it could get all messed up & hard to understand. Breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

- Within the Playbooks folder, create a new file called site.yml, this file will be the the parent playbooks & entry point to our infrastructure configuration, i.e this is the file that will be run

```
$ sudo touch site.yml
```

- Create a new folder called `static-assignments`, this folder will store all the children playbook, as I have said, this will aid the organisation of your work.

```
$ sudo mkdir static-assignments
```

- Move the common.yml file within the `playbooks` folder into `static-assignments`

```
$ sudo mv playbooks/common.yml static-assignments/
```

- The folder structure should look like the below image:

![6](https://user-images.githubusercontent.com/47898882/129141442-2fdb1602-ff8e-40e8-9ad5-85d73cceb847.JPG)

- Copy this snippet of code below and add it to the site.yml file. What this playbooks enables is reusability, we can import playbooks into other playbooks and implement the tasks within. 

![7](https://user-images.githubusercontent.com/47898882/129141436-ef8710c3-50d1-4494-af7d-62de57d8dfee.JPG)

- This imported playbook will run the same tasks as in the last project, so lets create a set of tasks to delete the wireshark utility form the target servers.

- Create a new file called `common.yml` in the `static-assignments` folder and add the code below for deleting wireshark:

![8](https://user-images.githubusercontent.com/47898882/129141998-828d9bd4-eadd-4346-8841-52adbd8311cb.JPG)

- Import the the common-del.yml playbook into the site.yml parent playbook.

![9](https://user-images.githubusercontent.com/47898882/129142246-fd9436b9-cdd8-4166-b282-f2b80cc86780.JPG)

- Run the playbook using this `ansible-playbook` command. You should get following output

```
$ ansible-playbook -i inventory/dev.txt playbooks/site.yml
```

![10](https://user-images.githubusercontent.com/47898882/129228385-14e1d88f-1170-4fa2-916e-7a1a699aec9a.JPG)


# CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER'

- We have considered imports can be used to make our playbooks reuasble. Another way we can promote reusability is through *roles*. To show the concept of how roles work, spin up two new ec2 instances/servers. We will be deploying an application to these servers using our playbook configuration.

- Create a folder called `roles` within the ansible-config-mgt directory.

```
$ sudo mkdir roles
$ cd roles
```
- The Ansible utility known an ansible-galaxy will be used to create the file structure we need to use roles within our playbook configuration. Run the command below within the roles directory

```
$ ansible-galaxy init webserver
```

- A file structure will have been created, you can delete the files, vars & test files as we do not need them within our configuration. Your file structure should look like below:

![11](https://user-images.githubusercontent.com/47898882/129230398-817282ab-1998-4027-80d1-7064b1c698f8.JPG)

- Update your inventory file with the deatils/information about the webservers. We'll be using the uat.txt file

![13](https://user-images.githubusercontent.com/47898882/129230827-abeaf6f1-ba86-4d6e-971e-a6d3b1f3fac7.JPG)

- There has to be an update of the `roles_path` inthe ansible configuration file within the /etc/ansible/ansible.cfg directory, we have to specify where our roles are being stored within the file system. Open the configuration file and make changes below:

```
$ sudo vi /etc/ansible/ansible.cfg
```
![14](https://user-images.githubusercontent.com/47898882/129231460-67487019-cc48-4314-8117-53b04eafee52.JPG)

- We will be creating our roles in the main.yml file within the tasks directory, this role we are creating is meant to deploy a website solution to both target servers. The tasks we'll be configuring on the target servers will be:

1. Install and configure Apache (httpd service)
2. Clone Tooling website from GitHub https://github.com/brpo01/tooling.git.
3. Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
4. Make sure httpd service is started

- So let us write our configuration as code. Use the snippet of code below to perform the tasks.

```
---
# tasks file for webserver
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
    repo: https://github.com/brpo01/tooling.git
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
```

- The role we just created can be actually be run now, but it'll defeat the concept of reusability we have been talking about, to use this role we have to reference it within our `static-assignments folder`, remember the static-assigment folder stores all of the children playbook which will be later be imported to be used by the parent playbook. Our way of doing things is in a bid to make the codebase organized and easy to understand.

- Create a file called `uat-servers` within the static-assignment folder.

```
$ cd static-assignments
$ sudo touch uat-servers.yml
```

- Reference the main.yml role within the uat-servers.yml file

![15](https://user-images.githubusercontent.com/47898882/129233820-73f0f529-88f9-446e-9c48-76fcf62a6405.JPG)

- Now we can import this file into the parent/entrypoint playbook (site.yml). This is the playbook file that'll run the configuration of the target servers

![16](https://user-images.githubusercontent.com/47898882/129234132-15a6c033-fcc5-45a3-967a-10a646ccca3e.JPG)

- Run your playbook using the `ansible-playbook` command. You should check that none of the tasks failed.

```
$ ansible-playbook -i inventory/uat.txt playbook/site.yml
```

```
PLAY [all] *******************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [172.31.1.76]
ok: [172.31.1.220]
[WARNING]: Could not match supplied host pattern, ignoring: webservers
[WARNING]: Could not match supplied host pattern, ignoring: nfs
[WARNING]: Could not match supplied host pattern, ignoring: db

PLAY [update web, nfs and db servers] ****************************************************************************************************
skipping: no hosts matched
[WARNING]: Could not match supplied host pattern, ignoring: lb

PLAY [update LB server] ******************************************************************************************************************
skipping: no hosts matched

PLAY [uat-servers] ***********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [172.31.1.76]
ok: [172.31.1.220]

PLAY [uat-servers] ***********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [172.31.1.76]
ok: [172.31.1.220]

TASK [webserver : install apache] ********************************************************************************************************
ok: [172.31.1.220]
ok: [172.31.1.76]

TASK [webserver : install git] ***********************************************************************************************************
ok: [172.31.1.220]
ok: [172.31.1.76]

TASK [webserver : clone a repo] **********************************************************************************************************
changed: [172.31.1.76]
changed: [172.31.1.220]

TASK [webserver : copy html content to one level up] *************************************************************************************
changed: [172.31.1.76]
changed: [172.31.1.220]

TASK [webserver : Start service httpd, if not started] ***********************************************************************************
changed: [172.31.1.76]
changed: [172.31.1.220]

TASK [webserver : recursively remove /var/www/html/html/ directory] **********************************************************************
changed: [172.31.1.76]
changed: [172.31.1.220]

PLAY RECAP *******************************************************************************************************************************
172.31.1.220               : ok=9    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.31.1.76                : ok=9    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

- Test your setup by running the website on your browser. Use the pubic ip-address of our server in this format `http://public-ip-address/index.php`

![17](https://user-images.githubusercontent.com/47898882/129235261-4532495d-088a-4b91-9458-22601a5ad2fc.JPG)

- After configuration, your Ansible architecture should look like this:

![18](https://user-images.githubusercontent.com/47898882/129235595-43e4cf5b-bc1d-4f1a-9681-b3c23b3c2480.JPG)


### Congratulations, You have learnt how to refactor your ansible code, how to import your playbooks, and how to use roles to make your playbooks reuasble.