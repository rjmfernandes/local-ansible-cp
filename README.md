# Local Ansible CP Based on Docker Instances (MacOS)


- [Local Ansible CP Based on Docker Instances (MacOS)](#local-ansible-cp-based-on-docker-instances-macos)
  - [Disclaimer](#disclaimer)
  - [Build the base image](#build-the-base-image)
  - [CP Ansible](#cp-ansible)
  - [Start Containers](#start-containers)
  - [Deploy with Ansible](#deploy-with-ansible)
  - [Cleanup](#cleanup)

## Disclaimer

The code and/or instructions here available are **NOT** intended for production usage. 
It's only meant to serve as an example or reference and does not replace the need to follow actual and official documentation of referenced products.

## Build the base image

The image used for our instances comes from the work by Jeff Geerling https://github.com/geerlingguy/docker-ubuntu2204-ansible

We have added a couple of packages including java and changed the ubuntu version for compatibility with CP Ansible.

First you will need to create the image:

```bash
docker build . -t my-geerlingguy-docker-ubuntu-ansible
```

## CP Ansible

Clone locally the repository:

```shell
git clone --depth 1 --branch v7.5.4 https://github.com/confluentinc/cp-ansible
```

Inside the repository you need to copy the playbooks to root:

```bash
cd cp-ansible
cp -fr playbooks/* .
```

Copy hosts.yml to cp-ansible.

```bash
cp ../hosts.yml .
```

Edit the variables of hosts.yml as the example here. Pay attention to the following variables:

```yml
    ansible_connection: docker
    ansible_user: root
    ansible_become: true
    ssl_enabled: false
    confluent.platform.ssl_required: false
    ansible_python_interpreter: /usr/bin/python3
    custom_java_path: /usr/lib/jvm/java-1.17.0-openjdk-arm64
```

You will also want to make sure the server instances in hosts.yml match the ones defined in the docker-compose.yml file (just like the example here). Comment out either kafka_controller entry or the zookeeper one.

## Start Containers

Finally run the docker-compose from the root of the project:

```bash
cd ..
docker compose up -d
```

You will need to map the host names on your `/etc/hosts` file:

```
127.0.0.1 zk1
127.0.0.1 zk2
127.0.0.1 zk3
127.0.0.1 kafka1
127.0.0.1 kafka2
127.0.0.1 kafka3
127.0.0.1 sr
127.0.0.1 rest
127.0.0.1 ksql-server
127.0.0.1 connect
127.0.0.1 cc
```

## Deploy with Ansible

Finally back to the cp-ansible cloned repository run:

```bash
cd cp-ansible
ansible-galaxy collection install git+https://github.com/confluentinc/cp-ansible.git,v7.5.4
ansible-playbook ./all.yml -i hosts.yml
```

You should be able to access control center on usual port http://localhost:9021/clusters

## Cleanup

```bash
cd ..
rm -fr cp-ansible
docker compose down -v
```