# Local Ansible CP Based on Docker Instances (MacOS)

## Get it

```bash
    git clone git@github.com:tomasalmeida/local-ansible-cp.git --recursive 
    cd local-ansible-cp
```

## Build the base image

The image used for our instances comes from the work by Jeff Geerling https://github.com/geerlingguy/docker-ubuntu2204-ansible

We have added a couple of packages including java and changed the ubuntu version for compatibility with CP Ansible.

First you will need to create the image:

```bash
docker build . -t my-geerlingguy-docker-ubuntu-ansible
```

The folder cp-ansible is a submodule from Confluent CP Ansible git repository (https://github.com/confluentinc/cp-ansible) using the tag for CP 7.5.4. If the folder is empty, it is because you cloned it without the flag `--recursive`. So in that case do:

```bash
    git submodule update --init
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
    custom_java_path: /usr/lib/jvm/java-17-openjdk-arm64
```

You will also want to make sure the server instances in hosts.yml match the ones defined in the docker-compose.yml file (just like the example here). Comment out either kafka_controller entry or the zookeeper one.

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

Finally back to the cp-ansible cloned repository run:

```bash
cd cp-ansible
ansible-galaxy collection install git+https://github.com/confluentinc/cp-ansible.git,v7.5.4
ansible-playbook ./all.yml -i hosts.yml
```

You should be able to access control center on usual port http://localhost:9021/clusters
