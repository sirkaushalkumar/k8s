- [Kubernetes](#kubernetes)
  * [Kubernetes - Launch Single Node Kubernetes Cluster](#kubernetes---launch-single-node-kubernetes-cluster)
    + [Step 1 - Start Minikube](#step-1---start-minikube)
    + [Step 2 - Cluster Info](#step-2---cluster-info)
    + [Step 3 - Deploy Containers](#step-3---deploy-containers)
    + [Step 4 - Dashboard](#step-4---dashboard)
  * [Kubernetes - Getting Started With Kubeadm](#kubernetes---getting-started-with-kubeadm)
    + [Step 1 - Initialise Master](#step-1---initialise-master)
    + [Step 2 - Join Cluster](#step-2---join-cluster)
    + [Step 3 - View Nodes](#step-3---view-nodes)
    + [Step 4 - Deploy Container Networking Interface (CNI)](#step-4---deploy-container-networking-interface--cni-)
    + [Step 5 - Deploy Pod](#step-5---deploy-pod)
    + [Step 6 - Deploy Dashboard](#step-6---deploy-dashboard)
  * [Kubernetes - Start containers using Kubectl](#kubernetes---start-containers-using-kubectl)
    + [Step 1 - Launch Cluster](#step-1---launch-cluster)
    + [Step 2 - Kubectl Run](#step-2---kubectl-run)
    + [Step 3 - Kubectl Expose](#step-3---kubectl-expose)
    + [Step 4 - Kubectl Run and Expose](#step-4---kubectl-run-and-expose)
    + [Step 5 - Scale Containers](#step-5---scale-containers)
  * [Kubernetes - Deploy Containers Using YAML](#kubernetes---deploy-containers-using-yaml)
    + [Step 1 - Create Deployment](#step-1---create-deployment)
    + [Step 2 - Create Service](#step-2---create-service)
    + [Step 3 - Scale Deployment](#step-3---scale-deployment)


# Kubernetes

## Kubernetes - Launch Single Node Kubernetes Cluster

Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a VM on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

More details can be found at https://github.com/kubernetes/minikube

### Step 1 - Start Minikube

Minikube has been installed and configured in the environment. Check that it is properly installed, by running the minikube version command:

`minikube version`

Start the cluster, by running the minikube start command:

`minikube start`

Great! You now have a running Kubernetes cluster in your online terminal. Minikube started a virtual machine for you, and a Kubernetes cluster is now running in that VM.

### Step 2 - Cluster Info

The cluster can be interacted with using the kubectl CLI. This is the main approach used for managing Kubernetes and the applications running on top of the cluster.

Details of the cluster and its health status can be discovered via `kubectl cluster-info`

To further debug and diagnose cluster problems, use `kubectl cluster-info dump`.

To view the nodes in the cluster using `kubectl get nodes`

If the node is marked as NotReady then it is still starting the components.

This command shows all nodes that can be used to host our applications. Now we have only one node, and we can see that it’s status is ready (it is ready to accept applications for deployment).

### Step 3 - Deploy Containers

With a running Kubernetes cluster, containers can now be deployed.

Using `kubectl run`, it allows containers to be deployed onto the cluster - `kubectl run first-deployment --image=katacoda/docker-http-server --port=80`

The status of the deployment can be discovered via the running Pods - `kubectl get pods`

Once the container is running it can be exposed via different networking options, depending on requirements. One possible solution is NodePort, that provides a dynamic port to a container.

`kubectl expose deployment first-deployment --port=80 --type=NodePort`

The command below finds the allocated port and executes a HTTP request.

```
export PORT=$(kubectl get svc first-deployment -o go-template='{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}')
echo "Accessing host01:$PORT"
curl host01:$PORT
```

The results is the container that processed the request.

### Step 4 - Dashboard


The Kubernetes dashboard allows you to view your applications in a UI. In this deployment, the dashboard has been made available on port 30000.

The URL to the dashboard is https://2886795363-30000-frugo04.environments.katacoda.com/ 

## Kubernetes - Getting Started With Kubeadm

In this scenario you'll learn how to bootstrap a Kubernetes cluster using Kubeadm.

Kubeadm solves the problem of handling TLS encryption configuration, deploying the core Kubernetes components and ensuring that additional nodes can easily join the cluster. The resulting cluster is secured out of the box via mechanisms such as RBAC.

More details on Kubeadm can be found at https://github.com/kubernetes/kubeadm

### Step 1 - Initialise Master
Kubeadm has been installed on the nodes. Packages are available for Ubuntu 16.04+, CentOS 7 or HypriotOS v1.0.1+.

The first stage of initialising the cluster is to launch the master node. The master is responsible for running the control plane components, etcd and the API server. Clients will communicate to the API to schedule workloads and manage the state of the cluster.

Task
The command below will initialise the cluster with a known token to simplify the following steps.

`kubeadm init --token=102952.1a7dd4cc8d1f4cc5 --kubernetes-version $(kubeadm version -o short)`

In production, it's recommend to exclude the token causing kubeadm to generate one on your behalf.

To manage the Kubernetes cluster, the client configuration and certificates are required. This configuration is created when kubeadm initialises the cluster. The command copies the configuration to the users home directory and sets the environment variable for use with the CLI.

```
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

### Step 2 - Join Cluster

Once the Master has initialised, additional nodes can join the cluster as long as they have the correct token. The tokens can be managed via kubeadm token, for example `kubeadm token list`.

Task
On the second node, run the command to join the cluster providing the IP address of the Master node.

`kubeadm join --discovery-token-unsafe-skip-ca-verification --token=102952.1a7dd4cc8d1f4cc5 172.17.0.18:6443`

This is the same command provided after the Master has been initialised.

The `--discovery-token-unsafe-skip-ca-verification tag` is used to bypass the Discovery Token verification. As this token is generated dynamically, we couldn't include it within the steps. When in production, use the token provided by `kubeadm init`.

### Step 3 - View Nodes


The cluster has now been initialised. The Master node will manage the cluster, while our one worker node will run our container workloads.

Task
The Kubernetes CLI, known as kubectl, can now use the configuration to access the cluster. For example, the command below will return the two nodes in our cluster.

`kubectl get nodes`

At this point, the Nodes will not be ready.

This is because the Container Network Interface has not been deployed. This will be fixed within the next step.

### Step 4 - Deploy Container Networking Interface (CNI)

The Container Network Interface (CNI) defines how the different nodes and their workloads should communicate. There are multiple network providers available.

Task
In this scenario we'll use WeaveWorks. The deployment definition can be viewed at `cat /opt/weave-kube`

This can be deployed using kubectl apply.

`kubectl apply -f /opt/weave-kube`

Weave will now deploy as a series of Pods on the cluster. The status of this can be viewed using the command `kubectl get pod -n kube-system`

When installing Weave on your cluster, visit https://www.weave.works/docs/net/latest/kube-addon/ for details.

### Step 5 - Deploy Pod


The state of the two nodes in the cluster should now be Ready. This means that our deployments can be scheduled and launched.

Using Kubectl, it's possible to deploy pods. Commands are always issued for the Master with each node only responsible for executing the workloads.

The command below create a Pod based on the Docker Image katacoda/docker-http-server.

`kubectl run http --image=katacoda/docker-http-server:latest --replicas=1`

The status of the Pod creation can be viewed using `kubectl get pods`

Once running, you can see the Docker Container running on the node.

`docker ps | grep docker-http-server`


### Step 6 - Deploy Dashboard
Kubernetes has a web-based dashboard UI giving visibility into the Kubernetes cluster.

Task
Deploy the dashboard yaml with the command kubectl apply -f dashboard.yaml

The dashboard is deployed into the kube-system namespace. View the status of the deployment with kubectl get pods -n kube-system

A ServiceAccount is required to login. A ClusterRoleBinding is used to assign the new ServiceAccount (admin-user) the role of cluster-admin on the cluster.
```
cat <<EOF | kubectl create -f - 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF
```
This means they can control all aspects of Kubernetes. With ClusterRoleBinding and RBAC, different level of permissions can be defined based on security requirements. More information on creating a user for the Dashboard can be found in the Dashboard documentation.

Once the ServiceAccount has been created, the token to login can be found with:

kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

When the dashboard was deployed, it used externalIPs to bind the service to port 8443. This makes the dashboard available to outside of the cluster and viewable at https://2886795294-8443-cykoria01.environments.katacoda.com/

Use the admin-user token to access the dashboard.

For production, instead of externalIPs, it's recommended to use kubectl proxy to access the dashboard. See more details at https://github.com/kubernetes/dashboard.

## Kubernetes - Start containers using Kubectl

In this scenario, you'll learn how to use Kubectl to create and launch Deployments, Replication Controllers and expose them via Services without writing yaml definitions. This allows you to quickly launch containers onto the cluster.

### Step 1 - Launch Cluster

To start we need to launch a Kubernetes cluster.

Execute the command below to start the cluster components and download the Kubectl CLI.

`minikube start`

Wait for the Node to become Ready by checking `kubectl get nodes`


### Step 2 - Kubectl Run
The run command creates a deployment based on the parameters specified, such as the image or replicas. This deployment is issued to the Kubernetes master which launches the Pods and containers required. Kubectl run_ is similar to docker run but at a cluster level.
`
The format of the command is kubectl run `<name of deployment> <properties>`

Task
The following command will launch a deployment called http which will start a container based on the Docker Image katacoda/docker-http-server:latest.

`kubectl run http --image=katacoda/docker-http-server:latest --replicas=1`

You can then use kubectl to view the status of the deployments

`kubectl get deployments`

To find out what Kubernetes created you can describe the deployment process.

`kubectl describe deployment http`

The description includes how many replicas are available, labels specified and the events associated with the deployment. These events will highlight any problems and errors that might have occurred.

In the next step we'll expose the running service.

### Step 3 - Kubectl Expose

With the deployment created, we can use kubectl to create a service which exposes the Pods on a particular port.

Expose the newly deployed http deployment via kubectl expose. The command allows you to define the different parameters of the service and how to expose the deployment.

Task
Use the following command to expose the container port 80 on the host 8000 binding to the external-ip of the host.

`kubectl expose deployment http --external-ip="172.17.0.77" --port=8000 --target-port=80`

You will then be able to ping the host and see the result from the HTTP service.

`curl http://172.17.0.77:8000`


### Step 4 - Kubectl Run and Expose

With kubectl run it's possible to create the deployment and expose it as a single command.

Task
Use the command command to create a second http service exposed on port 8001.

`kubectl run httpexposed --image=katacoda/docker-http-server:latest --replicas=1 --port=80 --hostport=8001`

You should be able to access it using `curl http://172.17.0.77:8001`

Under the covers, this exposes the Pod via Docker Port Mapping. As a result, you will not see the service listed using `kubectl get svc`

To find the details you can use `docker ps | grep httpexposed`

Pause Containers
Running the above command you'll notice the ports are exposed on the Pod, not the http container itself. The Pause container is responsible for defining the network for the Pod. Other containers in the pod share the same network namespace. This improves network performance and allow multiple containers to communicate over the same network interface..


### Step 5 - Scale Containers
With our deployment running we can now use kubectl to scale the number of replicas.

Scaling the deployment will request Kubernetes to launch additional Pods. These Pods will then automatically be load balanced using the exposed Service.

Task
The command kubectl scale allows us to adjust the number of Pods running for a particular deployment or replication controller.

`kubectl scale --replicas=3 deployment http`

Listing all the pods, you should see three running for the http deployment `kubectl get pods`

Once each Pod starts it will be added to the load balancer service. By describing the service you can view the endpoint and the associated Pods which are included.

`kubectl describe svc http`

Making requests to the service will request in different nodes processing the request.

`curl http://172.17.0.77:8000`

## Kubernetes - Deploy Containers Using YAML

In this scenario, you'll learn how to use Kubectl to create and launch Deployments, Replication Controllers and expose them via Services by writing yaml definitions.

YAML definitions define the Kubernetes Objects that become scheduled for deployment. The objects can be updated and redeployed to the cluster to change the configuration.

### Step 1 - Create Deployment
One of the most common Kubernetes object is the deployment object. The deployment object defines the container spec required, along with the name and labels used by other parts of Kubernetes to discover and connect to the application.

Task
Copy the following definition to the editor. The definition defines how to launch an application called webapp1 using the Docker Image katacoda/docker-http-server that runs on Port 80.

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
```

This is deployed to the cluster with the command `kubectl create -f deployment.yaml`

As it's a Deployment object, a list of all the deployed objects can be obtained via `kubectl get deployment`

Details of individual deployments can be outputted with `kubectl describe deployment webapp1`

### Step 2 - Create Service

Kubernetes has powerful networking capabilities that control how applications communicate. These networking configurations can also be controlled via YAML.

Task
Copy the Service definition to the editor. The Service selects all applications with the label webapp1. As multiple replicas, or instances, are deployed, they will be automatically load balanced based on this common label. The Service makes the application available via a NodePort.

```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1
```

All Kubernetes objects are deployed in a consistent way using kubectl.

Deploy the Service with `kubectl create -f service.yaml`

As before, details of all the Service objects deployed with `kubectl get svc`. By describing the object it's possible to discover more details about the configuration `kubectl describe svc webapp1-svc`.

`curl host01:30080`

### Step 3 - Scale Deployment

Details of the YAML can be changed as different configurations are required for deployment. This follows an infrastructure as code mindset. The manifests should be kept under source control and used to ensure that the configuration in production matches the configuration in source control.

Task
Update the deployment.yaml file to increase the number of instances running. For example, the file should look like this:
```
replicas: 4
```
Updates to existing definitions are applied using kubectl apply. To scale the number of replicas, deploy the updated YAML file using `kubectl apply -f deployment.yaml`

Instantly, the desired state of our cluster has been updated, viewable with `kubectl get deployment`

Additional Pods will be scheduled to match the request. `kubectl get pods`

As all the Pods have the same label selector, they'll be load balanced behind the Service NodePort deployed.

Issuing requests to the port will result in different containers processing the request `curl host01:30080`

Additional Kubernetes Networking details and Object Definitions will will be covered in future scenarios.

