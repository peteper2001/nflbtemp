# Influbbit Service

Influbbit is a Python service to pull data from the InfluxDB database used by Heapster (on a Kubernetes cluster)
and send that information to a remote RabbitMQ server outside of Kubernetes

# ==============================================================================================================

# Influbbit Production Installation

## Prerrequisites:

- Kubernetes cluster with Heapster addon installed (using InfluxDB as backend)

TO-DO

# ==============================================================================================================

# Influbbit Development Installation

## Prerrequisites:

- Docker
    curl -sSL https://get.docker.com/ | sh
    
- RabbitMQ
    Sample Docker based installation for test/debug:
    - sudo mkdir /var/log/rabbitmq
    - sudo mkdir -p /var/lib/rabbitmq/mnesia
    - sudo docker run -d --name=rabbitmq -p 0.0.0.0:5672:5672 -p 0.0.0.0:15672:15672 -v /var/log/rabbitmq:/data/log -v /var/lib/rabbitmq/mnesia:/data/mnesia rabbitmq:3-management
    - Visit http://localhost:15672 
    - Login as user "guest" with password "guest"
    
- InfluxDB backend for Kubernetes Heapster
    Sample Docker based installation for test/debug:
    - sudo mkdir /var/lib/influxdb
    - sudo docker run -d --name influxdb -p 0.0.0.0:8083:8083 -p 0.0.0.0:8086:8086 -v /var/lib/influxdb:/data kubernetes/heapster_influxdb
    - Visit http://localhost:8083
    - Login as user "root" with password "root"
    - On database k8s ejecute "list series" command
    - Make a select like "select * from /log.events/"

## Install requirements

- sudo apt-get install python-pip

  NOTE: Ubuntu 14.04 and Linux Mint 17 ships python-daemon 1.5.5 package.
        So, we use python-pip to install a 2.0.5 version (and provide
        the same version across different distros)

- sudo pip install python-daemon

- pip install pika

  NOTE: See https://pika.readthedocs.org/en/latest

- pip install influxdb

  NOTE: See http://influxdb-python.readthedocs.org/en/latest/

## Install and use influbbit

- mkdir /var/run/influbbit

- mkdir /var/log/influbbit

- python influbbit.py start

- python influbbit.py restart

- python influbbit.py stop

  NOTE: Support for "status" should be provided on the LSB script

## Backup and restore Influxdb database (for test/development)

- Connect to Kubernetes master

- kubectl get nodes

- kubectl describe nodes <node-name>  (search node running system pod monitoring-influx-grafana*)

- Connect via SSH with that node/minion

- sudo docker exec -it <container-id> /bin/bash

- apt-get install wget jq

- wget https://raw.githubusercontent.com/eckardt/influxdb-backup.sh/master/influxdb-backup.sh -O /usr/bin/influxdb-backup.sh 

- chmod +x /usr/bin/influxdb-backup.sh 

- influxdb-backup.sh dump k8s > influx-backup.json

- exit

- sudo docker cp <id>:/influx-backup.json .

- Copy influx-backup.json to the dev server

- apt-get install wget jq

- wget https://raw.githubusercontent.com/eckardt/influxdb-backup.sh/master/influxdb-backup.sh -O /usr/bin/influxdb-backup.sh 

- chmod +x /usr/bin/influxdb-backup.sh 

- cat influx-backup.json | influxdb-backup.sh restore k8s
 
## Export Rabbit messages to file

- http://hg.rabbitmq.com/rabbitmq-management/raw-file/c1c6c64ac1ab/bin/rabbitmqadmin -O rabbitmqadmin

- chmod +x rabbitmqadmin

- ./rabbitmqadmin get queue=test_q requeue=true count=100 > rabbit.txt 

## Push of image to Docker Hub

- sudo docker login
	Username: blancsys
	Password: <password>
	Email: alex@blancsys.com

  NOTE: File $HOME/.dockercfg file will be created

- sudo docker build -t blancsys/influbbit .

- sudo docker push blancsys/influbbit

## Running as local Docker:

- sudo docker run --name=influbbit -v /var/lib/influbbit:/var/lib/influbbit -e RABBIT_HOST=172.17.42.1 -e INFLUX_HOST=172.17.42.1 blancsys/influbbit

## Creation of influbbitdockerhubkey

- cat $HOME/.dockercfg | base64 --wrap=0

- Edit and save the serialized value into dockersecret.yml

## Creation of influbbit.db sqlite database (without tables)

- sqlite3 influbbit.db

- create table test (id int);

- .quit

- cat influbbit.db | base64 --wrap=0

- Copy content into sqlitesecret.yml

## Running on Kubernetes

- curl -s https://s3.amazonaws.com/influbbit/influbbit-launch.sh | bash -s [IP-ADDRESS-RABBIT]

  NOTE: IP address of server is optionally. If not configured, the default IP address included on that script will be used

- kubectl get pod  (get influbbit-xxxx pod name)

- kubectl logs influbbit-xxxxx

# To enter to InfluxDB container:

- kubectl get pod --namespace=kube-system (see pod name)

- kubectl exec --namespace=kube-system monitoring-influx-grafana-v1-5xw6v -c influxdb -i -t /bin/bash
