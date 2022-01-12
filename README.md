## Description

In this project I am going to build a Docker image and push that image to Docker hub using ansible-playbook. The Docker image that I have used to build is Nodejs.

## Pre-requirements

1. We need to install Ansible on our local machine.

2. Docker Hub account.

## Installation

You can install ansible by using the below command. Also we need to install git on the server

```
amazon-linux-extras install -y ansbile2

yum install git -y

```

## Ansible Modules used

- [yum](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html) 
- [pip](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/pip_module.html)
- [service](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html)
- [git](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/git_module.html)
- [docker_image](https://docs.ansible.com/ansible/2.8/modules/docker_image_module.html)
- [docker_login](https://docs.ansible.com/ansible/2.9/modules/docker_login_module.html)


## Variables Used

```
    git_url: https://github.com/Ismailpb/Docker_nodejs.git                    # Git repo URL
    docker_user: "ismailpb"                                                   # DockerHub username
    docker_password: "****************"                                       # DockerHub password
    image_name: "ismailpb/nodejs-app"                                         # Image name

```

## Behind the code

1.At first we need to install the packages and dependencies to create the image in playbook.

```

- name: "Building Docker Image"
  hosts: build
  become: true
  vars:
    git_url: "https://github.com/Ismailpb/Docker_nodejs.git"
    docker_user: "abab"
    docker_password: "xyxyxyxy"
    image: "ismailpb/nodejs-app"
  tasks:

    - name: "Package Installation"
      yum:
        name:
          - docker
          - git
          - python2-pip
        state: present


    - name: "Enabling/Restarting Service"
      service:
        name: docker
        state: restarted
        enabled: true


    - name: "Installing Docker-client for python"
      pip:
        name: docker-py
        state: present

```

Here the hosts is set as "build" as I have created a seperate ec2-instance to create the image. We can use localhost if we are using our own local machine to build the image.

2. Once the dependencies were installed, we need to clone the repository from Github. For that we are using the ansible module "git".

```
    - name: "Cloning  github repository"
      git:
        repo: "{{ git_url }}"
        dest: "/var/nodejs/"

      register: git_status


[ec2-user@ip-172-31-0-157 ~]$ ls -la /var/nodejs/
total 16
drwxr-xr-x  3 root root   90 Jan 11 13:28 .
drwxr-xr-x 20 root root  283 Jan 11 13:28 ..
-rw-r--r--  1 root root  189 Jan 11 13:28 Dockerfile
drwxr-xr-x  8 root root  180 Jan 11 13:28 .git
-rw-r--r--  1 root root  265 Jan 11 13:28 package.json
-rw-r--r--  1 root root 3473 Jan 11 13:28 README.md
-rw-r--r--  1 root root  277 Jan 11 13:28 server.js


```
3. In the next step, we need to login to Dockerhub with the username and password mentioned in "vars" to push the image that we have setup in the build server. For that we user the ansible module docker_login and docker_image.

```
    - name: "Login to Docker Hub"
      when: git_status.changed == true
      docker_login:
        username: "{{ docker_user }}"
        password: "{{ docker_password }}"


    - name: "Building Docker Image"
      when: git_status.changed == true
      docker_image:
        build:
          path: "/var/nodejs"
        name: "{{ image }}"
        tag: "{{ item }}"
        push: yes
        source: build
      with_items:
        - "latest"
        - "{{ git_status.after }}"

```

## Final Output

Once the above process completed, we can run the playbook using the command

```
ansible-playbook -i hosts dockerimage-build.yml
```

![image](https://github.com/Ismailpb/Docker-Image-Build-and-Push-using-Ansible/blob/d0c261fafe7000891c47b34261d17db289e304f0/Screenshot%20from%202022-01-12%2008-03-23.png)

## Conclusion

The Image that we build have been updated in the Dockerhub.

![image](https://github.com/Ismailpb/Docker-Image-Build-and-Push-using-Ansible/blob/d0c261fafe7000891c47b34261d17db289e304f0/Screenshot%20from%202022-01-12%2008-00-37.png)



