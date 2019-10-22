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
kubectl create -f <yml> --record 

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
		
		kubectl create -f <yml> --record -> to store rollouts
		
	Service Command Mauually:
	  kubectl expose deployment helloworld-deployment --type=NodePort -> creates a type nodeport, which we create as service file
		
	Services: (kubectl svc ....)
	  it is the logical bridge between the pods and other services or end users, because while deployent or update 
		new pods are created, so it is bad to access them directly
		
		services can run only between 30000 - 32767
		
		creating a service will create a end point for the pods
		  types:
			  ClusterIp -> a virtual IP addr only reachable from within the cluster(this is default)
				NodePort -> a port that is the same on each node that is also reachable externally
				LoadBalancer -> created by cloud provider to route external traffic to every node on the node port
			 	
	Labels: (under deployment nodeSelector)
	  kubectl label nodes node1 hardware=high-spec
		kubectl label nodes node2 hardware=low-spec
	  used to tags for pods
		once nodes are tagged you can use label selectors to let pods only run on specific nodes
		There are 2 steps:
		  1) First tag the node
			2) Then you add a nodeSelector to your pod configuration
			  nodeSelector:
				  hardware: spec
	
	Health Checks: (under deployment healthcheck)
	  https://www.baeldung.com/spring-boot-kubernetes-self-healing-apps
	  two ways:
		  1) running a command in a container periodically
			2) periodic checks on a URL -> used most	
				livenessProbe:
					httpGet:
						path: /
						port: nodejs-port
					initialDelaySeconds: 15
					timeoutSeconds: 30
					
			kubectl edit <pod-name>	 -> to see the settings
		
		livenessProbe:
		  indicates whether a container is running, if check fails the container will be restarted
		Readiness Probe:
		  indicates whether the container is ready to serve requests.
			if check fails the container will not be restarted, but the pods ip address will be removed from the service, so that it'll not serve any requests anymore	
		
		In general we configure both	
		kubectl create -f <yml> && watch -n1 kubectl get pods
		
	Pod	State:
	  Pod Status -> high level status 
		Pod Condition -> the condition of the pod
		Container State -> state of the containers itself
		Pending:
		  happens when the container image is still downloading
			if the pod cannot be scheduled because of resource constraints, it'll be in this status
		Succeeded -> all containers terminated and will not be restarted
		Failed -> all containers have been terminated and atleast one container returned a failure code
		  failure code is the exit code of the process when a container terminates
		Unknown -> the state of the pod could'nt be determined(a network error)		
		
		kubectl get pods -n kube-system -> pods from specific node name system
		kubectl describe pod <pod-name> -n <node-name>
		
		There are 5 different types of PodConditions
		  PodScheduled -> pod has been scheduled to a node
			Ready -> Pod can serve requests and is going to be added to matching Services
			Initialized -> initialization of the containers have been started successfully
			Unschedulable -> pod can't be Scheduled
			ContainersReady -> all containers in the pod are ready
			
		Get Container State:
		  kubectl get pod <pod-name> -n <node-name> -o yaml
			
	Pod Lifecyle: (pod-lifecycles)
	  kubectl exec -it <pod-name> -- tail <path-to-file> -f
	
	Secrets: (deployment)
	  provides a way to distribute secret data to pods
		provides secrets to app 
		use as env variables
		as a file in a pod using volumes
		used with dotenv
		
		To generate using files:
		  echo -n "root" > ./username.txt
			echo -n "password" > ./password.txt
			kubectl create secret generic db-user-pass --from-file=./usermane.txt --from-file=./password.txt
			
		Using Yaml:
		  encode with base 64:
			  $ echo -n "root"|base64 -> asd
				$ echo -n "password"|base64 -> xsx
		  data:
			  password: xsx
				username: asd
				
		To Get a Shell like in docker: (43)
		  kubectl exec <pod-name> -i -t -- /bin/bash	
			
		ENV wordpress secrets: (word-press)
		
		Questions:
		  kube-apiserver and kube-scheduler runs on the master nodes
			
Advanced Topics:

  Service Discovery(DNS): (service-discovery)					
		
		Using DNS	built in service this is launched automatically
		can be found on master node on /etc/kubernetes/addons
		DNS is used to find other services running in the same Cluster(namespace)
		Multiple containers running in the same pod don't need this DNS, they can contact with local host directly Using
		localhost:port
		To make DNS work, a pod will need a Service definition 
		There is a default namespace, we can have pods and services launched in logical namespace
		
		Check if connection was made:
		  kubectl logs <pod-name>
		
	Config Maps: (configmap)
		Configurations that are not secret can be put in ConfigMap
		As key-value pairs stored
		From app can be read a ENV variables, as container commandline arguments in the pod Configurations and 
		using volumes
		
	Ingress: (ingress)
	  https://akomljen.com/aws-alb-ingress-controller-for-kubernetes/
	  it is used to allow inbound connections to the cluster
		it is an alternative to external LoadBalancer and NodePorts
		with ingress you can run your own ingressController(loadbalancer) within the Cluster
		there are default images available	
		we can use ingressController to reduce LoadBalancer costs 
		
	External DNS: (external-dns)
	  we can use one loadbalancer to capture all the traffic and pass it to the ingressController
		works only for http(s)
		this tool will automatically setup necessary DNS records in your external DNS server(like route53)
		for every hostname that you use in ingress, it'll create a new record to send traffic to your loadbalancer
		
	Volumes: (volumes)
	  stateful application
	
	Volumes Provision Storage: (word press volumes)
	  storageclass	
		
	Pod Presets: (pod-preset)
	  it is used to inject information into pods at runtime
		it is used to inject kubernetes resources like secrets, configmaps, volumes and ENV variables, volumemounts
		if we have 20 apps, we can have one preset object which will inject variables to matching pods	
		
	Stateful Sets: (stateful)
	  when a pod needs a stable pod hostname instead of a podname-random-hashstring
		Pod name identity will be using Index, like podname-0, podname-1 etc
		statefulsets allow stateful apps stable storage with volumes on their number	
		deleting and/or scaling a statefulset down will not delete the volumes associated with it
		A statefulSet will allow your stateful app to use DNS to find other peers
		cassandra, elastic search
		
	Daemon Sets:
	  it ensures that every single node in the k8s cluster runs the same pod resources
		it is useful if we want to ensure that a certain pod is running on every single node
		when a node is added to the cluster a new pod will be started automatically
		same when a node is removed the pod will not be rescheduled on another node
		usecases:
		  Logging aggregators
			Monitoring
			LoadBalancer / reverse Proxies / API Gateways 
			Running a daemon that only needs one instance per physical instance
			
	Resource Usage Monitoring: (metric server) - profiling
	  * https://medium.com/@cagri.ersen/kubernetes-metrics-server-installation-d93380de008
	  Heapster(deprecated) enables Container Cluster Monitoring and Performance Analysis
		it is a prerequisite if you want to do a pod auto scaling
		Heapster exports cluster metrics via REST endpoints
		Visualizations can be shown using Grafana
		CAdvisor is used to accumulate the metrics	
		
		Heapster is deprecated so use Metric Server(prometeus)		
		
		kubectl top node/pod -> to see metrics
		
	Auto Scaling:
	  Horizontal auto scaling is possible
	
	Affinity and anti-affinity:
	 	similar to nodeSelector for node affinity, can do more complex scheduling	
		pod affinity/anti-affinity allows to create rules how pods should be scheduled taking into account other running pods
		This mechanism is only relevant during scheduling, once a pod is running it'll need to be recreated to apply rules again
		
	Taints and Tolerations:
	  opposite of node affinity
		used to say pods cannot run on same nodes
		taints are applied to nodes and tolerations are applied to pods 
		
	Custom Resource Definitions:
	  how we define the pods
	
	Operators:
	  it is a method of packaging, deploying and managing a k8s app	
		
		
Kubernetes Administration:
  
	Master Services:
	
	Resource Quotas:
	  Resource capacity: each container can specify request capacity and capacity limits 
		  the scheduler can use the request capacity to make decisions on where to put the pod on
			You can see it as a minimum amount of resources the pod needs 
			 		
		Resource Limits: is a limit imposed to the Container 
		
	NameSpace:
	  allow to create virtual clusters within the same physical cluster
		namespaces logically seperate our clusters
		the standard namespace is default 
		there is also namespace for kubernetes specific resources called kube-system(monitoring, dns addon, dashboard )
		namespaces are intended when you have multiple teams/projects using the k8s cluster
		name of the resource needs to be unique in a namespace
		we can divide resources of a k8's cluster using namespaces 
		we can limit resources on a per namespace basis
		
	request & limits:
	  https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-resource-requests-and-limits	
		
		https://medium.com/expedia-group-tech/kubernetes-container-resource-requirements-part-1-memory-a9fbe02c8a5f
		
		https://medium.com/expedia-group-tech/kubernetes-container-resource-requirements-part-2-cpu-83ca227a18b1
		
		https://medium.com/faun/java-application-optimization-on-kubernetes-on-the-example-of-a-spring-boot-microservice-cf3737a2219c
		
		https://medium.com/@betz.mark/understanding-resource-limits-in-kubernetes-cpu-time-9eff74d3161b
		
	User Management:
	  A normal User, which is used to access the user externally. (through kubectl), this 
		user is not managed using objects
		A service User which is managed by an object in K8s
		  this user needs authentication within the cluster(from inside a pod or from a kubelet)
			
	Networking:
	
	High Availability
	
	TLS on ELB:
	  Assigning cert
	
	Helm:
	  Package manager for K8s	
		
	Istio: (service Mesh)
	  provides a side car(envoy proxy) to each containers, 
		a common config place (pilot)
		Citadel -> to issue TLS crts to each envoy proxy	
		
		Istio-ingress:
		  to define a service for istio-ingress we need to define a virtual service
			
			
# Practial 

  Create a label for different node
	  kubectl label nodes minikube env_running=dev			
		kubectl get nodes --show-labels
		
		
# Minikube Build Image 
  * https://stackoverflow.com/questions/42564058/how-to-use-local-docker-images-with-minikube
	# Start minikube
	minikube start

	# Set docker env
	eval $(minikube docker-env)

	# Build image
	docker build -t foo:0.0.1 .

	# Run in minikube
	kubectl run hello-foo --image=foo:0.0.1 --image-pull-policy=Never

	# Check that it's running
	kubectl get pods		
	  		

# External end
* https://unofficial-kubernetes.readthedocs.io/en/latest/concepts/services-networking/service/ 
* https://akomljen.com/kubernetes-tips-part-1/
* https://v1-13.docs.kubernetes.io/docs/concepts/services-networking/service/
* https://docs.okd.io/latest/dev_guide/integrating_external_services.html
* https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-mapping-external-services
  * https://github.com/kubernetes/minikube/issues/2735
  * https://stackoverflow.com/questions/49289009/during-local-development-with-kubernetes-minikube-how-should-i-connect-to-postg
	kind: Service
	apiVersion: v1
	metadata:
	name: postgres
	namespace: default
	spec:
	type: ExternalName
	# https://docs.docker.com/docker-for-mac/networking/#use-cases-and-workarounds
	externalName: host.docker.internal
	ports:
		- name: port
			port: 5432			
			
# Check Ingress:
  * https://medium.com/@ManagedKube/kubernetes-troubleshooting-ingress-and-services-traffic-flows-547ea867b120
	
# labels, selectors
  * https://blog.mayadata.io/openebs/kubernetes-label-selector-and-field-selector
	
# Minikube Config
  * https://darkowlzz.github.io/post/minikube-config/			
	
	
# Prod EKS setup	
  * http://bit.ly/2nZVsNR
  * https://docs.bitnami.com/aws/get-started-eks/
	* https://eksworkshop.com/helm_root/helm_nginx/installnginx/
	* https://medium.com/@tanzwud.work/k8s-aws-eks-helm-tiller-setup-with-rbac-ae73ea4dc312		
	
	
# Securing Ngnix
  * https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/	
	
# Setup Https on k8s
  * https://www.youtube.com/watch?v=gEzCKNA-nCg&feature=youtu.be
	
# Output Mode
  * kubectl get <command> <name> -o yaml			