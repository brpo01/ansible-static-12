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


