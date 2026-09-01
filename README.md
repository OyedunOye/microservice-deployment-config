## Deploy Microservices with Helmfile

A script can be written to contain all the helm install commands for all microservices as configured in `install.sh`. Uninstall microservices using `Uninstall.sh`. The downside of this is that it doesn't provide a single command to install and uninstall plus it is difficult to override individual values this way. Helmfile is the elegant clean way of deploying helm charts. To use helmfile for microservices deployment follow the steps:

- Create a helmfile which lists all microservice releases to be deployed in a yaml file. It is a declarative way for deploying helm charts. The syntax as follows:
```bash
releases:
  - name: rediscart
    chart: charts/redis
    values: 
      - values/redis-cart-values.yaml
```
It is possible to override individual values provided within `values/<actual_values_file>` like this:
```bash
releases:
  - name: rediscart
    chart: charts/redis
    values: 
      - values/redis-cart-values.yaml
      - appRelicas: 4
      - containerMountPath: /my-redis-data
```
- Install helmfile:
```bash
curl -s https://api.github.com/repos/helmfile/helmfile/releases/latest \
  | grep "browser_download_url.*linux_amd64.tar.gz" \
  | cut -d '"' -f 4 \
  | xargs curl -fsSL -o helmfile.tar.gz

tar -xzf helmfile.tar.gz helmfile
sudo mv helmfile /usr/local/bin/
rm helmfile.tar.gz
```
Confirm that the installation was successful by checking the version of helmfile:
```bash
helmfile --version
```
- Deploy helmfile with command:
```bash
helmfile sync
```
To uninstall releases, run:
```bash 
helmfile destroy
```

- Final project structure
```
microservice-deployment-config/
├── .gitignore
├── helmfile.yaml
├── install.sh
├── uninstall.sh
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