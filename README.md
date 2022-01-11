# docker-build-and-Push-via-Ansible

This is a playbook demo for installing the Docker and building a python application image from a Docker file and then it pushed to your Docker Hub account via Ansible.

## Requirements

- Install Ansible on your Master Machine
- Basic Knowledge of ansible
- Basic Knowledge of docker
- Need a docker hub account

## Ansible Modules used

- [yum](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html) 
- [pip](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/pip_module.html)
- [service](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html)
- [git](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/git_module.html)
- [docker_image](https://docs.ansible.com/ansible/2.8/modules/docker_image_module.html)
- [docker_login](https://docs.ansible.com/ansible/2.9/modules/docker_login_module.html)

## Provisioning

```
---
- name: "Building the Docker Image"
  hosts: build
  become: true
  vars:
    git_url: "https://github.com/AryaMathew/Python-Docker-Flask.git"
    docker_user: "aryamathew"
    docker_password: "jmjjmj99**"
    image_name: "aryamathew/flaskapp"
  tasks:

    - name: "Build - Package Installation"
      yum:
        name:
          - docker
          - git
          - python2-pip
        state: present

    - name: "Build - Sarting the Docker Service"
      service:
        name: docker
        state: restarted
        enabled: true

    - name: "Build - Installing Docker Client For Python"
      pip:
        name: docker-py
        state: present

    - name: "Build - Cloning Github Repository {{ git_url }}"
      git:
        repo: "{{ git_url }}"
        dest: "/var/flaskapp/"
      register: git_status

    - name: "Build - Login to github Repository"
      when: git_status.changed == true
      docker_login:
        username: "{{ docker_user }}"
        password: "{{ docker_password }}"

    - name: "Build - Docker image from Dockerfile"
      when: git_status.changed == true
      docker_image:
        build:
          path: "/var/flaskapp/"
        name: "{{image_name}}"
        tag: "{{ item }}"
        push: yes
        source: build
      with_items:
        - "latest"
        - "{{ git_status.after }}"

    - name: "Build - Removing Images From the Build Server"
      when: git_status.changed == true
      docker_image:
        name: "{{image_name}}"
        tag: "{{ item }}"
        state: absent
      with_items:
        - "latest"
        - "{{ git_status.after }}"
```
## Usage

```
git clone https://github.com/AryaMathew/docker-build-and-Push-via-Ansible.git
cd docker-build-and-Push-via-Ansible.git
ansible-playbook dockerimage-build.yml
```

## Result

Now we have our docker image build and pushed that into our docker Hub account with the tags.

[![output.png](https://i.postimg.cc/qRLYgmNJ/output.png)](https://postimg.cc/N9Kbzk8S)

## Conclusion

It's just creating a docker image via ansible and then push that same to Docker hub.

