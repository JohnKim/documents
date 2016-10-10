```
# create Docker Host (Service Discovery : "m-service-discovery")
docker-machine create \
--driver [SPECIFIED] \
m-service-discovery

# Run consul container
docker $(docker-machine config m-service-discovery) run -d \
-p "8400:8400" \
-p "8500:8500" \
-h "consul" \
consul agent -data-dir /data --server -bootstrap-expect 1 -client=0.0.0.0

# create Swarm Master Host (Swarm Master Node : "m-master")
docker-machine create \
--driver [SPECIFIED] \
--swarm --swarm-master \
--swarm-discovery="consul://$(docker-machine ip m-service-discovery):8500" \
--engine-opt="cluster-store=consul://$(docker-machine ip m-service-discovery):8500" \
--engine-opt="cluster-adverise=eth1:2376" \
m-master

# create Swarm Slave Host (Swarm Slave Node : "m-slave-001")
docker-machine create \
--driver [SPECIFIED] \
--swarm
--swarm-discovery="consul://$(docker-machine ip m-service-discovery):8500" \
--engine-opt="cluster-store=consul://$(docker-machine ip m-service-discovery):8500" \
--engine-opt="cluster-adverise=eth1:2376" \
m-slave-001

# create Swarm Slave Host (Swarm Slave Node : "m-slave-002")
docker-machine create \
--driver [SPECIFIED] \
--swarm
--swarm-discovery="consul://$(docker-machine ip m-service-discovery):8500" \
--engine-opt="cluster-store=consul://$(docker-machine ip m-service-discovery):8500" \
--engine-opt="cluster-adverise=eth1:2376" \
m-slave-002

# CHECK IT OUT
eval $(docker-machine env --swarm m-master)
docker info

# Install weave on master node
# deploy containers - weaveworks/weaveexec, weaveworks/weave and weaveworks/plugin
# https://www.weave.works/docs/net/latest/installing-weave/
docker-machine ssh m-master \
'curl -L git.io/weave -o /usr/local/bin/weave; chmod a+x /usr/local/bin/weave'
docker-machine ssh m-master weave launch --init-peer-count 3

docker-machine ssh m-slave-001 \
'curl -L git.io/weave -o /usr/local/bin/weave; chmod a+x /usr/local/bin/weave'
docker-machine ssh m-slave-001 weave launch --init-peer-count 3
docker-machine ssh m-slave-001 weave connect "$(docker-machine ip m-master)"

docker-machine ssh m-slave-002 \
'curl -L git.io/weave -o /usr/local/bin/weave; chmod a+x /usr/local/bin/weave'
docker-machine ssh m-slave-002 weave launch --init-peer-count 3
docker-machine ssh m-slave-002 weave connect "$(docker-machine ip m-master)"

# CHECK IT OUT
docker-machine ssh m-master weave status

# Search internal dns and dns-search
docker-machine ssh m-master weave dns-args

# run docker-compose.yml
docker-compose up -d
docker-compose ps
docker ps

# CHECK IT OUT
docker-machine ssh m-master weave status dns


#####  weave scope #####
# https://www.weave.works/install-weave-scope/
docker-machine ssh m-master 'curl -L git.io/scope -o /usr/local/bin/scope; sudo chmod a+x /usr/local/bin/scope'
docker-machine ssh m-master scope launch

docker-machine ssh m-slave-001 'curl -L git.io/scope -o /usr/local/bin/scope; sudo chmod a+x /usr/local/bin/scope'
docker-machine ssh m-slave-001 scope launch

docker-machine ssh m-slave-002 'curl -L git.io/scope -o /usr/local/bin/scope; sudo chmod a+x /usr/local/bin/scope'
docker-machine ssh m-slave-002 scope launch  






```
