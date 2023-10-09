# Gen3 Local Setup
## Introduction
We will be using Helm to deploy a working instance of the Gen3 data commons. This POC is simply created for purposes of "playing around" with Gen3 and getting comfortable with its features. We will be deploying a Gen3 instance on a Kubernetes cluster. This cluster will be running on a laptop, desktop, or some virtual/physical machine in a cluster. We will try to avoid using the Cloud for this POC.

## Kubernetes
Kubernetes is a container orchestration framework. We need to have a functional Kubernetes cluster that is accessible on a physical or virtual machine.   

There are several lightweight tools that can be used to setup a local k8s cluster ('k8s' is the abbreviation for Kubernetes). A non-exhaustive list of tools include:
- [k3s](https://k3s.io/)
- [kind](https://kind.sigs.k8s.io/)
- [minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Rancher Desktop](https://rancherdesktop.io/)   

It really doesn't matter which tool to use, but Rancher Desktop is recommended because it has several useful features already installed and configured (like Docker, `kubectl`, and Helm). Rancher Desktop also has an Ingress already setup and configured.

## Helm
[Helm](https://helm.sh/) is a package manager for k8s that allows for the installation or deployment of applications onto a k8s cluster. If used properly, it can greatly simplify the management of k8s deployments. Helm templates can be used for k8s resources, like deployments and services. Sometimes we'd like some of the values inside these templates to change. This is where Helm is very convenient, since these values can all be stored inside a `values.yaml` file. The values inside this file then get injected into the Helm templates. A Helm Chart is a collection of Helm templates.  

The Helm charts for the Gen3 services can be found in the [uc-cdis/gen3-helm](https://github.com/uc-cdis/gen3-helm.git) repository. We'd like to add the Gen3 Helm chart repository. To do this, we run:  

```bash
helm repo add gen3 http://helm.gen3.org
helm update repo
```
Before performing a `helm install`, we need to create a `values.yaml` file. This file should be inside the root and contain the following:
```yaml
fence:
  FENCE_CONFIG:
    OPENID_CONNECT:
      google:
        client_id: ""
        client_secret: ""
```
The Helm installation can begin by running:
```bash
helm upgrade --install local-gen3-poc gen3/gen3  -f values.yaml
```
In the above command, `local-gen3-poc` is the name of the release of the helm deployment. If the installation is successful, then a message similar to the following should be displayed in the terminal:
```bash
NAME: local-gen3-poc
LAST DEPLOYED: Mon Oct  9 10:19:06 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
```
If using Rancher Desktop, then the `traefik` ingress controller is setup automatically. To see if the ingress is up and running, run the command:
```bash
kubectl get ingress
```
The output should look similar to this:
| NAME          | CLASS     | HOSTS     | ADDRESS       | PORTS   | AGE |
| ------------- | --------- | --------- | ------------- | ------- | --- |
| revproxy-dev  | traefik   | localhost | 192.168.205.2 | 80, 443 | 33s |

To visit the Gen3 portal, the following command can be run:
```bash
curl https://localhost -k
```
