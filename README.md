
# Deploy Microservices Application in Akamai's Managed Linode Kubernetes Engine (LKE)

This project was focused on configuration of configurations for deploying a fictitious online shopping application which is composed of 11 microservices in LKE cluster. The microservices application deployed here is hosted on this [github repo](https://github.com/techworld-with-nana/microservices-demo.git).

## Steps
- Gather the required info to deploy the application:
    1. What microservices need to be deployed?
    2. Which microservices talk to other microservices, and how do they communicate?
    3. Which database is used in this application?
    4. On which port does each microservice run?
    5. What are the environment variables required in each microservice for them to function properly?
- Create a Deployment and Service configuration for each of the microservices. **Note: This has been set up in the config.yaml file in this repo.**
All the config for each of the microservices are consolidated into a single config file.
- Prepare a LKE cluster where the microservices will be deployed as follows:
    1. Go to Akamai Cloud Services, from the left sidebar, choose Kubernetes.
    2. Fill in cluster label (as desired), Region, k8s version.
    3. Choose the type of Linode, in my case, 3 2GB, 1 CPU shared CPU Linode and create cluster.
- When the nodes are provisioned and running, download the kubeconfig file from Akamai's cloud for the cluster.
- Move the file into ~/.kube directory.
```bash
cp /mnt/c/Users/<username>/Download/kubeconfig.yaml ~/.kube/akamai-lke.yaml
```
- Backup the current config file
```bash
cp ~/.kube/config ~/.kube/config.bak
```
- Combine the new cluster's config with the existing cluster configurations in ~/.kube/config
```bash
KUBECONFIG=~/.kube/config:~/.kube/akamai-lke.yaml kubectl config view --flatten > ~/.kube/config_merged
```
- Override the ~/.kube/config with updated config consisting of all k8s cluster configurations and restrict the permissions on the file.
```bash 
mv ~/.kube/config_merged ~/.kube/config
chmod 600 ~/.kube/config
```
- Now it is possible to switch between contexts and execute kubectl commands in the desired cluster by choosing the context. Set current context to akamai's cluster.
```bash 
kubectl config use-context lke-context
```
- Create a microservices namespace
```bash 
kubectl create namespace microservices
```
- Create the deployments and services for the microservice application by applying the config file in this project in `microservices` namepace:
```bash 
kubectl apply -f microservice-deployment-config/config.yaml -n microservices
```
- Inspect all resources created using the command:
```bash
kubectl get all -n microservices
```
- Access frontend (entry point into this application) at `any_node_ip:30007`

**Note:** A config template which I used to set up configuration for each of the microservice's Deployment and Service is saved in config-template.yaml

