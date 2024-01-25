# Introduction to Kubernetes : K8S, K9S, K3S <br>

![](/images/04-image01.png)
  Figure 1: Most Popular Kubernetes Distribution Tools (K8S), Visualization (K9S) and Lightweight version (K3S)
<br><br>

Kubernetes is an open-source container orchestration platform that automates the<br>
deployment, scaling, and management of containerized applications. It provides a<br>
framework for managing and coordinating multiple containers across multiple hosts,<br>
making it easier to deploy and manage complex applications. <br><br>

Kubernetes is considered difficult to learn for several reasons:
```txt
1. Complexity: Kubernetes is a complex system with many components and concepts to
   understand. It has a steep learning curve, especially for those who are new to
   containerization and distributed systems.

2. Abstraction: Kubernetes abstracts away many underlying details, making it hard
   to understand how different components interact and work together. It requires 
   a deep understanding of networking, storage, and other infrastructure concepts.

3. Configuration: Kubernetes relies heavily on YAML configuration files for defining
   resources and their desired state. Understanding and writing these configuration 
   files correctly can be challenging, especially for beginners.

4. Scalability: Kubernetes is designed to handle large-scale deployments, which adds
   complexity to its architecture and concepts. Understanding how to scale applications,
   manage resources, and handle failures can be overwhelming for beginners.

5. Ecosystem: Kubernetes has a vast ecosystem with various tools, plugins, and
   extensions. Navigating this ecosystem and understanding which tools to use for
   specific use cases can be overwhelming.
```
Despite its complexity, Kubernetes is widely adopted in the industry due to its<br> 
ability to manage and scale containerized applications effectively. With time, practice,<br>
and resources like documentation, tutorials, and online communities, learning Kubernetes<br>
becomes more manageable.

Below are a few Kubernetes KxS jargons, but there are other more popular cloud or<br>
on-prem specific Kubernetes distribution tools such as Red Hat Open Shift, Rancher<br>
(mainly from K3S), Azure AKS, Amazon EKS, Google GKE. K9S is just a visualization<br>
tool depend on Docker Desktop and a running kubectl and could not run by itself.


|   | K0S | K1S | K3S | K8S | K9S |
|---|-----|-----|-----|-----|-----|
| Definition | A lightweight Kubernetes distribution | A lightweight Kubernetes distribution | A lightweight Kubernetes distribution | A full-fledged Kubernetes distribution | A command-line interface (CLI) for Kubernetes |
| Purpose | Designed for edge computing and IoT devices | Designed for edge computing and IoT devices | Designed for edge computing and IoT devices | Designed for large-scale production deployments | Designed for managing Kubernetes clusters |
| Size | Small footprint and minimal resource requirements | Small footprint and minimal resource requirements | Small footprint and minimal resource requirements | Large footprint and resource requirements | Small footprint and minimal resource requirements |
| Features | Simplified installation and management | Simplified installation and management | Simplified installation and management | Full set of Kubernetes features and capabilities | Enhanced CLI for managing Kubernetes resources |
| Scalability | Limited scalability due to its lightweight nature | Limited scalability due to its lightweight nature | Limited scalability due to its lightweight nature | Highly scalable and can handle large clusters | Limited scalability as it primarily focuses on CLI operations |
| Use Cases | Edge computing, IoT devices, and resource-constrained environments | Edge computing, IoT devices, and resource-constrained environments | Edge computing, IoT devices, and resource-constrained environments | Large-scale production deployments with high resource availability | Managing Kubernetes clusters efficiently through CLI |
| Community Support | Supported by the community and Rancher Labs | Supported by the community and Rancher Labs | Supported by the community and Rancher Labs | Widely supported by the Kubernetes community | Supported by the community and open-source contributors |
| Learning Curve | Relatively easy to learn and use | Relatively easy to learn and use | Relatively easy to learn and use | Steeper learning curve due to its complexity | Relatively easy to learn and use |
| Customization | Limited customization options | Limited customization options | Limited customization options | Highly customizable and extensible | Limited customization options |
| Popularity | Gaining popularity in edge computing and IoT domains | Gaining popularity in edge computing and IoT domains | Gaining popularity in edge computing and IoT domains | Widely adopted in production environments | Gaining popularity as a CLI tool for Kubernetes management |
Launch Year | 2020 | 2021 | 2021 | 2014 | 2019 |


<br>
<br>

## THE BASIC <br>
To learn Kubernetes we need to have it run, if possible the most cost effective way.<br>
Some says, it would need multiple instances (if running on AWS EC2) with minimum of<br>
2 CPU, 8GB memory, 15GB storage and up to Ubuntu 20.04 to connect one node to another.<br>
By early 2024 I use Ubuntu 23.10 and it was not ready even for 22.04 for example.<br><br>

If you are using EC2, t2.medium is the minimum for master node as it gives you<br> 
2 cpus by default, you still need to adjust the storage manually to 15GB for example.<br>
The ami below (ami-0c7217cdde317cfec) reflects Ubuntu 20.04. See main.tf for details.

```sh
resource "aws_instance" "kmaster" {
  ami           = "ami-0c7217cdde317cfec" 
  instance_type = "t2.medium"
  key_name      = "wcd-project"
  subnet_id     = aws_subnet.pub-subnet.id
  vpc_security_group_ids = [
    aws_security_group.sg_api_project6.id
  ]
  associate_public_ip_address = true
  tags = {
    Name = "kmaster"
  }
}
```

Also do not forget to open up port 6443, 2379, 2380, 10250, 10251, 10252 among<br>
other common ports (such as 22). If you plan to expose services externally using<br>
NodePort, you need to open 30000-32767 for the Kubernetes NodePort service type.<br><br>

You need to run sh commands shown in master.sh, worker.sh for master, worker nodes.<br>
Make sure each single line was run (not skipped), other the nodes won't properly.

Setting up Kubernetes instances with EC2 are very straight forward. <br>
By the end you should see something like below.<br>

![](/images/04-image02.png)
  Figure 2: An example of kubeadm join commands with token to worker node to join master node cluster
<br>

![](/images/04-image03.png)<br>
  Figure 3: An example of kubectl get nodes showing worker node join master node with above token.
<br><br>

Now, what is a bit tricky is running instances from your local computer (ideally free).<br>
We may need ubuntu and multipass which is virtualization tool that allows users to create<br>
and manage virtual machines on their computer. It is developed by Canonical, the company<br>
behind Ubuntu Linux. With Multipass, users can easily spin up and manage lightweight<br>
virtual machines, which can be used for various purposes such as software development,<br>
testing, and running different operating systems. It provides a command-line interface<br>
for creating, launching, and managing virtual machines, making it a convenient tool for<br>
developers and system administrators.<br><br>

```txt
multipass launch --name master -c 2 -d 4G -m 2G 20.04
multipass launch --name master -c 1 -d 4G -m 2G 20.04
```

When the instances were ready, you probably see something like below

```txt

devops@msi:~$ multipass list --format table
Name                    State             IPv4             Image
master                  Running           10.73.136.115    Ubuntu 20.04 LTS
                                          10.42.0.0
                                          10.42.0.1
worker                  Running           10.73.136.133    Ubuntu 20.04 LTS
                                          10.42.1.0
                                          10.42.1.1
devops@msi:~$ multipass info master
Name:           master
State:          Running
Snapshots:      0
IPv4:           10.73.136.115
                10.42.0.0
                10.42.0.1
Release:        Ubuntu 20.04.6 LTS
Image hash:     1fcdeb82d33a (Ubuntu 20.04 LTS)
CPU(s):         2
Load:           0.32 0.29 0.23
Disk usage:     3.1GiB out of 3.8GiB
Memory usage:   801.6MiB out of 1.9GiB
Mounts:         --
devops@msi:~$ multipass info worker
Name:           worker
State:          Running
Snapshots:      0
IPv4:           10.73.136.133
                10.42.1.0
                10.42.1.1
Release:        Ubuntu 20.04.6 LTS
Image hash:     1fcdeb82d33a (Ubuntu 20.04 LTS)
CPU(s):         1
Load:           0.00 0.05 0.01
Disk usage:     2.3GiB out of 3.8GiB
Memory usage:   281.4MiB out of 1.9GiB
Mounts:         --


```

We can either install K8S or K3S in Multipass. Let me just show the K3s here.<br>
Inside the master node, we will simply install the whole K3S like below

I found this link is very useful
https://gitlab.com/cloud-versity/rancher-k3s-first-steps

### Install Master
```txt
curl -sfL https://get.k3s.io | sh -s - --disable traefik --write-kubeconfig-mode 644 --node-name master
```

### Install Worker
Grab token from the master node to be able to add worked nodes to it:
```txt
cat /var/lib/rancher/k3s/server/node-token
```

Install k3s on the worker node and add it to our cluster:
```txt
curl -sfL https://get.k3s.io | K3S_NODE_NAME=k3s-worker-01 K3S_URL=https://<IP>:6443 K3S_TOKEN=<TOKEN> sh - 
```

The difference in K3S here, we do not join worker and master through kubeadm join.<br>
Kubeadm command is not even available in K3S. Instead we curl -sfL command to join.<br>

As we have the worker and master nodes ready, we can now do so many thing.<br><br>


## CLUSTER, NODES, PODS, CONTAINERS
![](/images/04-image04.png)<br>
  Figure 4: Venn Diagram of a Cluster, Nodes, Pods, Containers.
<br><br>

Let focus on one Node and playing around with multiple pods for now.<br>

We will use some basic kubectl commands which available in both K8S and<br>
K3S along with some visualization coming from K9S. Herewith some basic<br>

```sh
kubectl create -f xyz.yaml  # --> to deploy a brand new deployment (if done twice may cause error)
kubectl apply -f xyz.yaml  # --> to update an old deployment or a brand new pod 
kubectl delete -f xyz.yaml # --> to delete a deployment 
```

A deployment may consist of multiple pods, depends on how many replicas<br>
mentioned in yaml file<br>

### Exercise 1:
We will create a Pod.yaml with an Nginx image, with the following setting
 - the resources limits to 500m (50%)
 - for cpu and 128 Mi for memory.

In Kubernetes, the CPU resource is measured in "millicores" or "milliCPU".<br>
So, in this case, "500m" means 500 milliCPU or 0.5 CPU.

After we create a pod.yaml using below command for example
```txt
nano pod.yaml
```
with below content
```txt
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
    type: front-end
spec:
  containers:
  - name: mynginx
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
    - containerPort: 80
```
we can deploy it with 

```txt
kubectl apply -f pod.yaml
```

#### DRY RUN AND VERBOSITY

Let see some application of dry run and verbosity in this opportunity.<br>
- If we do not want to deploy and want to see if it does not have error<br>
- having a vast choice of details from 0 to 9 (normally from 6 and up)<br>
See below for an example

```txt
kubectl apply -f pod.yaml --dry-run=client -v 6
```

and herewith how the output looks like

```txt
devops@msi:~/DevOps/K9s-lab$ kubectl apply -f pod.yaml --dry-run=client -v 6
I0124 16:51:16.405711  209404 loader.go:395] Config loaded from file:  /home/devops/.kube/config
I0124 16:51:16.415197  209404 round_trippers.go:553] GET https://kubernetes.docker.internal:6443/openapi/v3?timeout=32s 200 OK in 8 milliseconds
I0124 16:51:16.423186  209404 round_trippers.go:553] GET https://kubernetes.docker.internal:6443/openapi/v3/api/v1?hash=51331B914D145C7C32E3CC656F6976A010DCA42158C75F039291F07058A18EE1B6D46FD578159F05658721386FD790CEF23C99B906854BE3486FBC03565BA97A&timeout=32s 200 OK in 7 milliseconds
I0124 16:51:16.466975  209404 round_trippers.go:553] GET https://kubernetes.docker.internal:6443/api/v1/namespaces/default/pods/myapp 404 Not Found in 3 milliseconds
pod/myapp created (dry run)
I0124 16:51:16.467250  209404 apply.go:546] Running apply post-processor function
```

#### INSTALLING DOCKER DESKTOP AND K9S
To be able to use K9S, we need to install Docker Desktop first.<br>
https://www.docker.com/products/docker-desktop/<br><br>

![](/images/04-image05.png)<br>
  Figure 5: Enabling Kubernetes in Docker.
<br>

![](/images/04-image06.png)<br>
  Figure 6: Docker Generated Images as Kubernetes is enabled.
<br><br>


To install K9s, it depends on your machine, see below:<br><br>
Windows (if you have Chocolatey installed):<br>
choco install k9s<br><br>
macOS (if you have Brew installed):<br>
brew install k9s<br><br>
Linux: see https://k9scli.io/topics/install/<br><br>

Installing Docker Desktop and K9S are optional, <br>
we can still deploy K8S without both of them.<br>

Herewith pod.yaml K9S visualization it was as deployed. <br>


![](/images/04-image07.png)<br>
  Figure 7: Overview of all pods in a node (in this case the node/context is docker-desktop)<br><br>
![](/images/04-image08.png)<br>
  Figure 8: All containers in myapp pod<br><br>
![](/images/04-image09.png)<br>
  Figure 9: All logs in mynginx container<br><br>

And herewith list of pod (including pod.yaml) without K9S.<br>
```txt
No resources found in default namespace.
devops@msi:~/DevOps/K9s-lab$ kubectl apply -f pod.yaml
pod/myapp created
devops@msi:~/DevOps/K9s-lab$ kubectl get pods -o wide
devops@msi:~/DevOps/K9s-lab$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
myapp   1/1     Running   0          16s   10.1.0.175   docker-desktop   <none>           <none>
devops@msi:~/DevOps/K9s-lab$ 
```
### Exercise 2:
Below is an example of another deployment from Replicaset.yaml.<br>

```txt
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-example
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      type: front-end
  template:
    metadata:
      labels:
        app: nginx
        type: front-end
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
```

Question is, how many CPU and memory limit in percentage?

```txt
CPU limit in %:
CPU limit is defined as the ratio of requested CPU to the CPU limit.
In this case, the requested CPU is 100m and the CPU limit is 250m. 
So, the CPU limit in % would be:

CPU limit in % = (requested CPU / CPU limit) * 100
              = (100m / 250m) * 100
              = 40%

Memory limit in %:
Memory limit is defined as the ratio of requested memory to the memory limit. 
In this case, the requested memory is 128Mi and the memory limit is 256Mi. 
So, the memory limit in % would be:

Memory limit in % = (requested memory / memory limit) * 100
                 = (128Mi / 256Mi) * 100
                 = 50%
```

To get above yaml file deployed we simply run
```txt
kubectl apply -f Replicaset.yaml
```
![](/images/04-image10.png)<br>
  Figure 10: Three pods are generated as in yaml it mentioned about 3 replicas<br><br>

### Exercise 3
We will introduce revisionHistoryLimit, for example we set into 3 below
```txt
revisionHistoryLimit: 3
```
Herewith is the complete example of Deployment.yaml with an Nginx image.<br> 
Set its strategy type to RollingUpdate and revisionHistoryLimit to 3.

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-example
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: httpd
      env: prod
  template:
    metadata:
      labels:
        app: httpd
        env: prod
    spec:
      containers:
      - name: httpd
        image: httpd
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
```
same command below
```txt
kubectl apply -f Deployment.yaml
```
and we will get

![](/images/04-image11.png)<br>
  Figure 11: Another three pods are generated as in yaml it mentioned about 3 replicas<br><br>

I make numerous changes and keep deploying, herewith example of revisions kept.<br>

```txt
devops@msi:~/DevOps/K9s-lab$ kubectl rollout history deployment/deploy-example
deployment.apps/deploy-example 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```
  
and herewith how to revert back to specific previous revision

```txt
devops@msi:~/DevOps/K9s-lab$ kubectl rollout undo deployment/deploy-example --to-revision 1
deployment.apps/deploy-example rolled back
```  

### Exercise 4
At this exercise we would not deploy any new pod from service.yaml,<br>
but instead adding access to web page from the last pods.

```txt
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  type: LoadBalancer
  selector:
    app: httpd
    env: prod
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
```

by running
```txt 
kubectl apply -f service.yaml
```
we can got the browser localhost:8080 to see a test page like below<br>

![](/images/04-image12.png)<br>
  Figure 12: Default message from the config file of httpd in /etc/httpd/conf/httpd.conf<br><br>

The other approach to send a quick message to browser would like this [one](https://gist.github.com/sdenel/1bd2c8b5975393ababbcff9b57784e82).

### Exercise 5
This section is just to demonstrate the reusability.<br>
Look at below ConfigMaps.yaml file for example.
```txt
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-example
data:
  Program: DevOps
  Instructor: Usman
```
Now we can use Instructor, Program variable like below.

```txt
apiVersion: v1
kind: Pod
metadata:
  name: mybox1
spec:
  restartPolicy: Always
  containers:
  - name: mybox1
    image: busybox
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    command:
    - sleep
    - "3600"
    env:
    - name: Instructor
      valueFrom:
        configMapKeyRef:
          name: cm-example
          key: Instructor
    - name: Program
      valueFrom:
        configMapKeyRef:
          name: cm-example
          key: Program
```

### Exercise 6
Similar to last exercise, let's look at secret.yaml file below.
```txt
apiVersion: v1
kind: Secret
metadata:
   name: secrets
type: Opaque
data:
   username: d2NkCg==
   password: dGhpc2lzbXlwYXNzd29yZAo=
```
Having above depolyment, we can use USERNAME, PASSWORD like below.
```txt
apiVersion: v1
kind: Pod
metadata:
  name: mybox2
spec:
  restartPolicy: Always
  containers:
  - name: mybox
    image: busybox
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    command:
      - sleep
      - "3600"
    env:
      - name: USERNAME
        valueFrom:
          secretKeyRef:
            name: secrets
            key: username
      - name: PASSWORD
        valueFrom:
          secretKeyRef:
            name: secrets
            key: password
``` 
