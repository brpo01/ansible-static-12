# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

This project gives an indepth understanding of how to apply configuration management to a group of servers using static assignments, static assignments involves the use of `imports` & `roles` all in the bid to make our playbooks reuseable, and which allows you to organize your tasks and reuse them when needed. Let me put this in simpler terms, imagine you write a piece of code to implement a particular function, and some days later you need to implement that function in another piece of code, would you start writing it from scratch?. the best way would be to *import* that function into your code. That's basically what we'll be implementing in this project.

To better understand this project, take a look at [ansible-config-mgt](https://github.com/brpo01/ansible-conf-11). This is meant to be a continuation of that project.

# Jenkins Job Enhancement
- To begin, let's enhance how our builds are being run in Jenkins. By default 