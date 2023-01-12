# HIGH AVAILABILITY SETUP IN DOCKER SWARN 

- Manager does all the things which is concern forthe management of cluster.

- But Client can directly access the worker (for using purpose)

- But if manager nnode goes down then there is no one to manage the workers/clusters.As it becomes the single point of failure.

- Hence there will be more managers for the worker node.As if one goes down others will be there to avoid SPOF.And Providing High Availablity.

## Setting Up high Availability at Manager node.
- For HA more nodes are required to be made manager node.
- To differentiate between the master and worker a token is there for workers and master.
  token will be generated by the first manager node(leader):
 docker:
- docker swarm join-token manager

- In cluster anytime a new node can be joined be it worker or manager.
- If th leader master goes down then the next master we created will become the leader.
- (This means at a time we can have multiple masters but at a time leader can only be one).

- But this will give warning that that it wont provide the fault tolerance,As we run the docker node ls in the new leader it shows that the leader is offline this is  becz swarm uses a prootocol in the HA setup for manager ,between the master there is constant share of info among them, This is done using the raft protocol(raft  consensus algo duty to share info of cluster among multi setup master).Hence due to which being a manager any node can take decision this result in the problem of  split brain(confilicted decision),hence there is only one leader who can take decision,Rest other managers are used when the main leader goes down then they become  the leader.RAFT takes the decision of who become the leader,but how?

- In HA world This is done here by election based on multiple parameters,nd this will work better if we have odd no. of nodes as the votes given can varry,Qouram have one formula or some ways to decide which should be next leader i.e. minimum there should be 3 manager nodes for making it a fault tolerance cluster.

- This results in active passive HA cluter.
-Mostly odd no. of nodes are important to have fault tolerance as qouram have a formula of (n-1)/2

eg: N=2
n-1/2 = 1/2 =0.5 (no fault tolerance)

N=3
n-1/2=3-1/2 = 1 ( 1 fault tolerance)


- Hence that's why they were giving the warning as a/c to fault tlerance we were having 2 node as manager hence there can be no fault tolerance. Hence we have to have   3 atleast.

- Hence connect more atleast 3 manager to the cluster.

- docker service scale=5 myweb
- Update command gives a lot of options
- docker service update --replicas=5.

- If now leader node goes down,the fault tolerance will be done out of 3 , 1 goes down then fault tolerance can be done automatcally 1 of  the 2 will become the master.

# How to remove a manager from the manager postion

- using the promotion and demotion commands:
docker node demote <mangernode-address>

# making worker to manager:
- docker node promote <node hostname>.

- Manager have to work work as a manager as well as the worker.( But in worker resources usage will become if it wrk both as worker as well as manager,hence can slow   down).

- Avoid making leader a worker power f we have limited resources.
## For this we have to make our cluster a master only cluster.
- by draining the leader(stop any present and avoid future container launch in that node)

- docker node update --availability drain <NODE-HOSTNAME>

- this will stop any task in that manager making it a manager only node.It will not work with as a worker ie. no task can be launched inside the master(leader).

- Drain command is mostly used as maintenance command such as with one drain command we can avoid downtime by making it drain,and the prev container can be launched in   the other nodes,and maintenance can be done on the node.