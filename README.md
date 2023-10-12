# Gen3 Local Setup
## Introduction
We will be using Helm to deploy a working instance of the Gen3 data commons. This POC is simply created for purposes of "playing around" with Gen3 and getting comfortable with its features. We will be deploying a Gen3 instance on a Kubernetes cluster. This cluster will be running on a laptop, desktop, or some virtual/physical machine in a cluster.

## Kubernetes
Kubernetes is a container orchestration framework. We need to have a functional Kubernetes cluster that is accessible on a physical or virtual machine.   

There are several lightweight tools that can be used to setup a local k8s cluster ('k8s' is the abbreviation for Kubernetes). A non-exhaustive list of tools include:
- [k3s](https://k3s.io/)
- [kind](https://kind.sigs.k8s.io/)
- [minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Rancher Desktop](https://rancherdesktop.io/)   
- [Colima](https://github.com/abiosoft/colima)   

It really doesn't matter which tool to use, but [Rancher Desktop is recommended](https://github.com/uc-cdis/gen3-helm/blob/master/docs/gen3_developer_environments.md#running-gen3-on-a-laptop-for-devs) because it has several useful features already installed and configured (like Docker, `kubectl`, and Helm). Rancher Desktop also has an Ingress already setup and configured.

## Helm
[Helm](https://helm.sh/) is a package manager for k8s that allows for the installation or deployment of applications onto a k8s cluster. If used properly, it can greatly simplify the management of k8s deployments. Helm templates can be used for k8s resources, like deployments and services. Sometimes we'd like some of the values inside these templates to change. This is where Helm is very convenient, since these values can all be stored inside a `values.yaml` file. The values inside this file then get injected into the Helm templates. A Helm Chart is a collection of Helm templates.  

We need to install Helm:
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
To see if Helm has been installed, we can run a simple `helm` command like:
```bash
helm list
```
and we should get an empty table as our output, i.e.,
| NAME          | NAMESPACE | REVISION  | UPDATED | STATUS  | CHART | APP VERSION |
| ------------- | --------- | --------- | ------- | ------- | ----- | ----------- |
|               |           |           |         |         |       |             |

## Using Minikube (on Ubuntu Server 20.04) with Helm
### Setting up Minikube Cluster
Let us go through the steps to install and use Minikube on an Ubuntu machine (virtual or physical). We'll begin by doing a general software dependency update:
```bash
apt-get update
```
Add the official Docker GPG key:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Add the Docker repository:
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list &gt; /dev/null
```
Install the necessary Docker dependencies:
```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release -y
```
The following two commands will install the latest version of the Docker engine:
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```
Add the user to the Docker group  with the following command:
```bash
sudo usermod -aG docker $USER
```
End the terminal session and then restart it. Docker should be installed.   
With Docker installed, we can go ahead and install the latest binary of Minikube:
```bash
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
Copy the Minikube executable to the `/usr/local/bin/` directory and give it the proper permissions with:
```bash
sudo cp minikube-linux-amd64 /usr/local/bin/minikube
sudo chmod +x /usr/local/bin/minikube
```
To use Kubernetes, we need to install `kubectl`, the command line tool for Kubernetes:
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s \
https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```
We need to give the `kubectl` executable the proper permissions:
```bash
chmod +x kubectl
```
The `kubectl` executable needs to be moved to the `/usr/local/bin/`:
```bash
sudo mv kubectl /usr/local/bin/
```
Minikube can now be started with:
```bash
minikube start --cpus 4 --memory 16g --driver=docker
```
and its status can be seen with:
```bash
minikube status
```
We should be seeing something like this:
```bash
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
Minikube has a list of addons (many of which are disabled by default) that can be seen with:
```bash
minikube addons list
```
To enable a particular addon, simply run:
```bash
minikube addons enable <addon>
```
We will need the ingress addon to be enabled:
```bash
minikube addons enable ingress
```
Minikube doesn't have the Traefik ingress controller installed (like Rancher Desktop), instead it uses the NGINX ingress controller. 

### Installing Gen3 Services with Helm
The Helm charts for the Gen3 services can be found in the [uc-cdis/gen3-helm](https://github.com/uc-cdis/gen3-helm.git) repository. We'd like to add the Gen3 Helm chart repository. To do this, we run:  

```bash
helm repo add gen3 http://helm.gen3.org
helm repo update
```
Before performing a `helm install`, we need to create a `values.yaml` file. This file should be inside the root and contain the contents of the `values.yaml` file that can be found in the root of this repository. For the `elastic-search-deployment` to run, we need to increase the max virtual memory areas by running:
```bash
sudo sysctl -w vm.max_map_count=262144
``` 
Now the Helm installation can begin by running:
```bash
helm upgrade --install local-gen3-poc gen3/gen3  -f values.yaml
```
In the above command, `local-gen3-poc` is the name of the release of the helm deployment. If the installation is successful, then a message similar to the following should be displayed in the terminal:
```bash
NAME: local-gen3-poc
LAST DEPLOYED: Wed Oct 11 08:12:04 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
```
If all went well, we should see the `revproxy-dev` deployment up and running with the following command:
```bash
kubectl get ingress
```
The output should look similar to this:
| NAME          | CLASS     | HOSTS     | ADDRESS       | PORTS   | AGE |
| ------------- | --------- | --------- | ------------- | ------- | --- |
| revproxy-dev  | nginx     | localhost | 192.168.49.2  | 80, 443 | 23s |

The list of deployments can be seen by running:
```bash
kubectl get deployments
```    

![alt text](/public/assets/images/local-gen3-deployments.png "Gen3 services deployed")  

We need to make
The nodePort of the `revproxy-service` is required in order to reach the `portal-service` web page. The following command exposes the `nodePort` of a service:
```bash
minikube service revproxy-service --url
```
This command creates a tunnel into the cluster. To access the `portal-service` web page url, we need to open another terminal window within the current ssh session. This can be done with:
```bash
tmux
```
In the new terminal window, we can use `kubectl` to obtain the nodePort of the `revproxy-service`:
```bash
kubectl get service revproxy-service --output='jsonpath="{.spec.ports[0].nodePort}"'
```
The output will be the `nodePort`. The Gen3 portal can now be accessed with:
```bash
curl http://$(minikube ip):nodePort
```
![alt text](/public/assets/images/html-of-gen3-portal.png "HTML of Gen3 Portal")  

