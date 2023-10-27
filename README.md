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
Let us go through the steps to install and use [Minikube](https://minikube.sigs.k8s.io/docs/start/) on an Ubuntu machine (virtual or physical). We'll begin by doing a general software dependency update:
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
Minikube doesn't have the Traefik ingress controller installed (like Rancher Desktop), instead it uses the NGINX ingress controller. To verify that the ingress is running, use the command:
```bash
kubectl get pods -n ingress-nginx
```    
The output should look something like this:   
| NAME                                      | READY | STATUS    | RESTARTS | AGE |
| ----------------------------------------- | ----- | --------- | -------- | --- |
| ingress-nginx-admission-create-rmr4p      | 0/1   | Completed | 0        | 15m |
| ingress-nginx-admission-patch-xf642       | 0/1   | Completed | 1        | 15m |
| ingress-nginx-controller-7799c6795f-c7ntq | 1/1   | Running   | 1        | 15m |


To access the Minikube dashboard, simply run:
```bash
minikube dashboard
```
The dashboard can be visited inside the browser by visiting the dashboard url, which can be obtained from running:
```bash
minikube dashboard --url
```
**NOTE:** If running the Ubuntu OS on an AWS EC2 instance, the process for visiting the dashboard in the browser is slightly different. Use an open terminal window (which is connected to the EC2 instance) to create a tunnel into the Minikube dashboard:
```bash
minikube dashboard --url
```
Let's suppose that the dashboard url is located at 
```bash
http://127.0.0.1:45255/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```
Then inside a separate terminal window, ssh into the EC2 instance using the following command (which is a slight modification from the original command used for generating an EC2 instance session):
```bash
ssh -i "/path/to/cert.pem" -L 8081:127.0.0.1:45255 rootuser@ec2-ip-address.compute-1.amazonaws.com
```
With these two sessions currently active, a browser window can be opened and the Minikube dashboard can be visited over here:
```bash
http://127.0.0.1:8081/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default
```
![Minikube Dashboard in Browser](/public/assets/images/minikube-dashboard.png "Minikube Dashboard in Browser")  

**k9s** is a useful tool that makes troubleshooting issues in a Kubernetes cluster easier. Installing it is highly recommended. It can be downloaded and installed as follows:
```bash
wget https://github.com/derailed/k9s/releases/download/v0.25.18/k9s_Linux_x86_64.tar.gz
tar -xzvf k9s_Linux_x86_64.tar.gz
chmod +x k9s
sudo mv k9s /usr/local/bin/
```
To open it, simply run:
```bash
k9s
```

### Setting up MinIO for Object Storage
[MinIO](https://min.io/docs/minio/kubernetes/upstream/index.html) is an open-source object storage solution which provides all the core Amazon S3 features and is compatible with the Amazon S3 API. It is built to be deployed anywhere - public cloud, private cloud, baremetal infrastructure, etc.   

We will be using MinIO to store uploaded files (like CSV, TSV, or JSON files) in a local volume. This volume will be a directory on the same host machine that the current Minikube cluster is provisioned on. The YAML files for the MinIO pod, service, and ingress are featured in this repository. To create the resources (we will be using the `default` namespace), run:
```bash
kubectl apply -f minio/minio-pod.yaml
kubectl apply -f minio/minio-service.yaml
```
The above commands created a `minio-service` of type **ClusterIP** (it is also possible to create the service as a load balancer). To see the list of services, run:
```bash
kubectl get services
```
and there should be a row in the table that looks like this:
| NAME           | TYPE      | CLUSTER-IP     | EXTERNAL-IP   | PORT(S)           | AGE  |
| -------------- | --------- | -------------- | ------------- | ----------------- | ---- |
| minio-service  | ClusterIP | 10.109.116.135 |    <none>     | 9000/TCP,9001/TCP | 86s  |

**ClusterIP** services are designed to be accessible only from within the same Kubernetes cluster. In order to access the `minio-service` from outside the cluster, we need to make use of our Ingress controller. The Ingress controller will allow us to access MinIO from the browser. Using the `local-gen3-ingress.yaml` file in this repo, we run:
```bash
kubectl apply -f local-gen3-ingress.yaml
```   

The MinIO dashboard can be accessed as follows (or it can be opened up in the browser if you are developing locally, and not on an AWS EC2 instance): 
```bash
curl http://$(minikube ip):9001/minio
```
![HTML of MinIO Console](/public/assets/images/minio-console-in-terminal.png "HTML of MinIO Console")   

Alternatively, the MinIO service can be exposed as a service of type **LoadBalancer**. This type of service does not require an ingress. This can be achieved by applying the `minio-service-loadbalancer.yaml` resource manifest as follows:
```bash
kubectl apply -f minio/minio-service-loadbalancer.yaml
```
This will deploy a load balancer MinIO service. A browser window can be opened by running:
```bash
minikube service minio-service-lb
```
If using an EC2 instance or some other vm, a simple way to access the MinIO console is by port-forwarding the service, e.g.,
```bash
kubectl port-forward --address 0.0.0.0 svc/minio-service 8088:9001
```
This will allow for the browser to access the `minio-service` on port `<public-ip-of-ec2-instance>:8088`:   

![MinIO in the Browser](/public/assets/images/minio-console-in-browser.png "MinIO in the Browser")  

Default login credentials are `minioadmin` and `minioadmin`.   

(Additional information about using persistent volumes, persistent volume claims, and storage classes will be added in the future).   

### Installing Gen3 Services with Helm
The Helm charts for the Gen3 services can be found in the [uc-cdis/gen3-helm](https://github.com/uc-cdis/gen3-helm.git) repository. We'd like to add the Gen3 Helm chart repository. To do this, we run:  

```bash
helm repo add gen3 http://helm.gen3.org
helm repo update
```
The Gen3 Helm chart repository contains the templates for all the microservices making up the Gen3 stack. For the `elastic-search-deployment` to run in a Linux host machine, we need to increase the max virtual memory areas by running:
```bash
sudo sysctl -w vm.max_map_count=262144
``` 
This setting will only last for the duration of the session. The host machine will be reset to the original value if it gets rebooted. For this change to be set permanently on the host machine, the `/etc/sysctl.conf` file needs to be edited with `vm.max_map_count=262144`. To see the current value, run `/sbin/sysctl vm.max_map_count`. More details can be found on the [official Elasticsearch website](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html).   

Before performing a `helm install`, we need to create a `values.yaml` file. This file should be inside the root and contain the contents of the `values.yaml` file that can be found in the root of this repository. Now the Helm installation can begin by running:
```bash
helm upgrade --install local-gen3 gen3/gen3  -f values.yaml
```
In the above command, `local-gen3` is the name of the release of the helm deployment. If the installation is successful, then a message similar to the following should be displayed in the terminal:
```bash
NAME: local-gen3
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
| NAME          | CLASS     | HOSTS           | ADDRESS       | PORTS   | AGE |
| ------------- | --------- | --------------- | ------------- | ------- | --- |
| revproxy-dev  | nginx     | gen3local.co.za | 192.168.49.2  | 80, 443 | 23s |

The list of deployments can be seen by running:
```bash
kubectl get deployments
```    

![Gen3 services deployed](/public/assets/images/local-gen3-deployments.png "Gen3 services deployed")  

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
![HTML of Gen3 Portal](/public/assets/images/html-of-gen3-portal.png "HTML of Gen3 Portal")   

We can use port-forwarding to access the Gen3 portal in the browser:
```bash
kubectl port-forward --address 0.0.0.0 svc/revproxy-service 8082:80
```
![Gen3 Portal Unauthorized](/public/assets/images/gen3-portal-unauthorized.png "Gen3 Portal Unauthorized")   

### Fence Authentication with Google
To see what's inside the `fence-config` secret, run:
```bash
kubectl get secret fence-config -o jsonpath='{.data}'
```
This value will be encoded. To decode it, the following command needs to be run:
```bash
echo 'encoded value from previous step' | base64 --decode
```
This decoded value can be scrutinised to see if the correct values specified in the `values.yaml` file has been applied to the `fence-deployment`.   

The `fence-service` requires at least one **OPEN ID CONNECT (OIDC)** client to be setup so that login can work. In our example, we'll use the Google Cloud Console to obtain a [Client ID and Client Secret](https://developers.google.com/identity/protocols/OpenIDConnect). The redirect URIs in Google need to be set to `{{BASE_URL}}/login/google/login`, where `BASE_URL` must correspond to the same value as set in the `values.yaml` file (for local development, it is usually http://localhost).   

More information about the `fence` config can be obtained from the [uc-cdis gen3 repository](https://github.com/uc-cdis/fence/blob/master/tests/test-fence-config.yaml#L72).   

If using an EC2 instance with Google as an auth provider, there might be a few challenges that one may face. For instance, Google requires an authorised redirect uri to be specified. However, ip addresses are not allowed. So the EC2 instance will need to have a domain name allocated to it. Registering a domain name is generally not free, so this might be a problem for testing. In this repository, the domain name `gen3local.co.za` has been registered with [GoDaddy](https://www.godaddy.com/en-za).  

Minikube has a `gcp-auth` addon that can be enabled. However, the **Google Cloud SDK** needs to be installed first with:
```bash
sudo apt install apt-transport-https ca-certificates gnupg
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - 
echo "deb https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list 
sudo apt update  
sudo apt install google-cloud-sdk 
```
Once the **Google Cloud SDK** is installed, the following command should be run to get a link where an authorisation code can be obtained:
```bash
 gcloud auth application-default login
```
The output will be a web browser link. Copying that link into the browser will lead the user to a Google login screen. After logging in, an authorisation code will be provided. This code is required in order for the `gcp-auth` addon to be enabled. 

Despite using a registered domain name (gen3local.co.za), authentication still fails. More investigation is required regarding `fence` authentication.  
![Registered Gen3 Portal Unauthorized](/public/assets/images/registered-gen3-portal-unauthorized.png "Registered Gen3 Portal Unauthorized")   

An easier option might be to install [Caddy](https://caddyserver.com/) on the EC2 instance, and then configure the EC2 DNS name in the Gen3 `values.yaml` file as the host name. When everything comes up, then run 
```bash
minikube service revproxy-service --url
``` 
to get the URL and then put it into `/etc/caddy/CaddyFile`. Caddy will need to be restarted in order for the changes to be reflected. (Disclaimer: I have not tried this yet, but it was recommended by Alan Walsh. He used such a [setup in Jetstream2 for Gen3 Helm deployments](https://github.com/alan-walsh/gen3-dev)).

If running on a local laptop or desktop computer, all this can be avoided by simply using `localhost`.
