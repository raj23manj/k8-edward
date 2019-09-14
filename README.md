K8’s

brew install kubectl => to create a virtual node
install virtual box
brew cask install qinikube => to add containers

minikube start
minikube status
minikube ip

kubectl cluster-info

pods => have multiple containers, run one container unless there is tight dependency with each other

kubectl apply -f <filename> => pushes the services, pods into VM and starts automatically

kubectl get pods
kubectl get services
kubectl get  deployments

kubectl describe <object type> <object name>
				pod/service    

kubectl pods -o wide  -> gives more info


To update the image to latest by forcing ctl
kubectl set image <object_type> / <object_name> <container_name> = <new image to use>


To remove a object

kubectl delete -f <config file>

container -> pod -> node -> cluster

command to to see containers running inside the VM (193)
eval $(minikube docker-env)

Service:
   ClusterIp  -> expose a set of pods to other objects in the cluster
   NodePort -> Expose a set of pods to outside world(only dev purpose)
   LoadBalancer -> legacy way of getting network traffic into a cluster
   Ingress -> securely stores a piece of information in the cluster, such as a DB password

pod, deployment, service,  Secrets  => types of objects

kubectl logs <name of type>

Volume:

   Volume in k8s is at pod level only not like docker. For that we need to use

   Persistent Volume Claim
   Persistent Volume

kubectl get storageclass
kubectl describe storageclass

awsblockstore default by amazon

kubectl get pv -> persistent volumes
kubectl get pvc -> claims(bill board like showing what all is there)

Secrets as imperative:
  kubectl create secret <secret-type> <secret_name> —from-literal key=value

secret-type:
  * generic
  * docker-registry 
  * tls -> http setup

Production:
   helm is used to manage 3rd party plugins in k8s
   Use Helm for ingress setup

Https:

kubectl get certificates


Edward:
  
Useful Commands
minikube service <name> —url
kubectl exec nodehelloworld.example.com -- ls /app
kubectl describe service <service-name>


Questions

How does the AWS LoadBalancer routes traffic to the correct pod
The LoadBalancer Service will also create a port that is open on every node, that can be used by the AWS LoadBalancer to connect. That port is owned by the Service, so that the traffic can be routed to the correct pod

The load balancer uses a NodePort that is exposed on all non-master nodes

How do containers within a pod communicate?
Containers within a pod can communicate using the local port numbers(localhost/127.0.0.1)

K8 - Architecture

kubelet -> responsible to launch the pods, it gets the information from the master node
Kube-proxy -> is responsible to feed the iptables about what pods are there on the current node, which is used for routing
iptable -> it is the fire wall that routes traffic in linux
load-balancer -> has details about the nodes and forwards incoming traffic to iptables
At times there might be scenario where a pod in one node needs to connect to a pod in another node, iptables handles it

Scaling pods:
  Replication controller does this
  to make sure pod is always running use replicas, even if it one pod
  k8s automatically replaces failing pods
  “kind: ReplicationController”

   kubectl scale —replicas=4 -f <ymlfile>
   kubectl get rc (/repicationcontroller)

Replication Set:
   Newer version of replication controller
   it supports a new selector that can do selection based on filtering according to a set of values
   eg “environment”  == “dev”
   This is used by Deployment Object

Deployments:
  Allows us to do app deployments and updates
  we define the state of application
  just using replication controller or replication sets is difficult
  used to do:
    create a deployments
    Update a deployments
    do rolling updates(no downtime)
    roll back to a previous apiVersion
    pause/resume a deployment     
		
	Commands:
	  kubectl get Deployments
		kubectl get rs
		kubectl get pods --show-labels
		kubectl rollout status deployment/helloworl-deployment -> get deployment status
		kubectl set image deployment/helloworl-deployment ks-8s-demo=k8s-demo:2 ->run k8s demo with the image label version2(stephen used this)	
		kubectl edit deployment/helloworld-deployment -> edit the deployment object
		kubectl rollout status deployment/helloworl-deployment -> get the status of the rollout
		kubectl rollout history -> get history
		kubectl rollout undo deployment/helloworl-deployment -> rollback to previous version
		kubectl rollout undo deployment/helloworl-deployment --to-revision=n -> rollback to specific version