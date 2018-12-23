----
Setting Up Kafka on Kubernetes
----

1.  Reference for setup is taken from https://github.com/Yolean/kubernetes-kafka/ [single-node, scale-2 branch contains 1, 2 node Kafka cluster respectively] repository. Some files are added and some are modified for bare-metal cluster setup

2.  In `./configure` directory local storage-classes are defined for storing Kafka and Zookeeper data. For local storage classes persistent volumes also needs to be defined for each base-metal server as mentioned in document https://kubernetes.io/blog/2018/04/13/local-persistent-volumes-beta/ . But in our setup we have already defined these storage classes and persistent volumes in `./configure` folder.

3.  Also Kafka manager service in `./yahoo-kafka-manager` is modified from ClusterIP to NodePort. So that we can access kafka manager from outside and we can monitor topic.

4.  3 separate Folders are maintained for Kafka setup on 

    **More than 2 Nodes :** [More than 2 Nodes](https://github.com/TechDevCode/Kubernetes-Kafka-Setup/tree/master/kafka-kubernetes-testing-more-than-2-nodes)

    **For 2 Nodes :** [2 Nodes](https://github.com/TechDevCode/Kubernetes-Kafka-Setup/tree/master/kafka-kubernetes-testing-scale-2)

    Scales to 2 brokers + 3 zookeeper instances

    **For Single Node :** [Single Node](https://github.com/TechDevCode/Kubernetes-Kafka-Setup/tree/master/kafka-kubernetes-testing-single-node)

----
Steps For Setup
----
**Step 1:** 

First setup storage classes and persistent volumes. Move to `./configure` directory of setup you want to do(e.g. for 1, 2 or more than 2 nodes). Now run `kubectl apply -f local-storageclass-{}.yml` and `kubectl apply -f persistent-vol-kafka-{}.yml`, `kubectl apply -f persistent-vol-zookeeper-{}.yml`. Here for more than 2 nodes case, persistent volumes for each node needs to be defined.

**Step 2:**

Now move to parent folder. Create namespace by `kubectl apply -f 00-namespace.yml`

**Step 3:**

Run `kubectl apply -f ./zookeeper`

**Step 4:**

Once zookeepers are up then finally Run `kubectl apply -f ./kafka`

**Step 5:**
Now Kafka setup is up and running. For deploying Kafka Manager for monitoring topics run following command 

`kubectl apply -f ./yahoo-kafka-manager`

Now Kafka Manager is up and running. I will be available at at port `32400`. 

Kafka Manager can be accessed by specifying `<IP address of any node in Kubernetes cluster>:32400`. This is possible because we have defined Kafka manager service as `Nodeport` type.

To add Kafka Cluster in Kafka Manager, add `zookeeper.kafka:2181` in Cluster Zookeeper Hosts[As Kubernetes pods from one service can connect to pod in another service through `<service-name>.<namespace>`. In this case `service-name` is `zookeeper` and `namespace` is `kafka`]. 

----
Steps for Removing Kafka Cluster from Kubernetes
----
Run Following Commands Sequenctially for tearing down kafka cluster.

`kubectl delete -f ./yahoo-kafka-manager`

`kubectl delete -f ./kafka`

`kubectl delete -f ./zookeeper`

`kubectl delete -f 00-namespace.yml`

`kubectl delete  -f ./configure`

Finally remove persistent volume contents from each Kubernetes worker node.
