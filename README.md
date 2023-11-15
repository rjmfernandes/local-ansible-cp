# Local Ansible CP Based on Docker Instances (MacOS)

The image used for our instances comes from the work by Jeff Geerling https://github.com/geerlingguy/docker-ubuntu2204-ansible

We have added a couple of packages including java and changed the ubuntu version for compatibility with CP Ansible.

First you will need to create the image:

```bash
docker build . -t my-geerlingguy-docker-ubuntu-ansible
```

You will also need to clone the CP Ansible git repository.

```bash
git clone https://github.com/confluentinc/cp-ansible.git
```

Inside the repository you need to copy the playbooks to root:

```bash
cp -fr playbooks/* .
```

Edit the variables of hosts.yml as the example here. Pay attention yo the following variables:

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

Finally run the docker-compose:

```bash
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

Finally run:

```bash
ansible-galaxy collection install git+https://github.com/confluentinc/cp-ansible.git,7.5.x
```

And deploy:

```bash
ansible-playbook ./all.yml -i hosts.yml
```

You should be able to access control center on usual port http://localhost:9021/clusters