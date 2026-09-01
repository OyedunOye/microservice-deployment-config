
# Create Helm Chart for Microservices
Improving on the deployed microservice application where individual deployment and services was configured for each microservice, a blueprint could be created for Deployment and another for Service with placeholders for value variables. Helm charts serve this purpose. A default values.yaml file can hold the default variable values and these can be overwritten by providing new values for the variables when reusing the same helm chart for different microservices.

For this application, 10 of 11 microservices have similar configuration and will share an helm chart and a second helm chart will be created for redis because its configurations are very different.

## Steps to Create Helm Chart
- Create helm chart directory with `helm create <chart_name>` command in charts folder:
```bash
helm create microservice
helm create redis
```
The created microservice directory contain many auto generted files and folders, clean them up leaving the below folder structure. The contents of the template files and values.yaml was cleared to start from a clean slate. 
```
charts/
├── microservice/
│   ├── .helmignore
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── charts/               (empty)
│   └── templates/
│       ├── deployment.yaml
│       └── service.yaml
└── redis/
    ├── .helmignore
    ├── Chart.yaml
    ├── values.yaml
    ├── charts/               (empty)
    └── templates/
        ├── deployment.yaml
        └── service.yaml
```
- Create a basic template file for a Deployment and Service in their respective template files, setting variables with placeholder syntax where needed. Sample variable syntax(**Note that the variableName must follow the camelcase naming convention**):
```bash
{{ .Values.variableName}}
```
The deployment templates implements best practices to improve on the previous service configuration including:
    1. liveness and readiness probe configuration
    2. resources request and request limit for each container
    3. the entrypoint to the cluster switched from NodePort to LoadBalancer to protect the cluster and minimize attack surface
- Configure default values in values.yaml within each helm chart directory, these would be overwritten by values.yaml value supplied when creating each microservice using this chart. The third way of suppling values to variables is with parameters passed with `--set` flag.
Dynamic environment variables be set as follows for single env variable:
```bash
- name : {{ .Values.containerEnvVar.name }}
  value: {{ .Values.containerEnvVar.value }}
```
And for an array, the syntax is:
```bash
{{- range Values.containerEnvVar}}
- name : {{ .name }}
  value: {{ .value }}
{{- end}}
```
- Create a values directory outside the helm repo. This is where a values file for each microservices is created with values for the variable which Override the default values in helm repo's values.yaml.
- Validate  that the values.yaml to override the default is correct using the command to view the yaml output of final template file:
```bash
pwd  ==> microservice-deployment-config/
helm template -f vlues/<actual_values.yaml> charts/<chart_name>
```
- Another way to verify the yaml syntax is with `--dry-run` flag:
```bash
# syntax
helm install --dry-run -f values/<actual_values.yaml> <release_name> charts/chart_name

# actual example
helm install --dry-run -f values/adservice-values.yaml adservice charts/microservice
```
- Verify no linting issue with:
```bash 
# syntax
helm lint -f values/<actual_values.yaml> charts/chart_name

# actual example
helm lint -f values/adservice-values.yaml charts/microservice
```
- Create a namespace for the resources (optional):
```bash
kubectl create ns microservices
```
- Deploy a service into a namespace other than default with:
```bash
helm install -f values/<actual_values.yaml> <release_name> charts/chart_name -n microservices

# actual example
helm install -f values/adservice-values.yaml adservice charts/microservice -n microservices
```
- Check the installed helm charts with:
```bash
helm ls
```
- Final project structure
```
microservice-deployment-config/
├── .gitignore
├── README.md
├── config.yaml             # config without helm chart
├── charts/
│   ├── microservice/
│   │   ├── .helmignore
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   ├── charts/               (empty)
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       └── service.yaml
│   └── redis/
│       ├── .helmignore
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── charts/               (empty)
│       └── templates/
│           ├── deployment.yaml
│           └── service.yaml
└── values/
    ├── redis-cart-values.yaml
    ├── adservice-values.yaml
    ├── cart-service-values.yaml
    ├── checkout-service-values.yaml
    ├── currency-service-values.yaml
    ├── email-service-values.yaml
    ├── frontend-service-values.yaml
    ├── payment-service-values.yaml
    ├── product-catalog-service-values.yaml
    ├── recommendation-service-values.yaml
    └── shipping-service-values.yaml

```