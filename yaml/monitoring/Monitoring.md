# Create Monitoring cluster on Kubernetes
In this example, we are going to setup an Elasticsearch cluster along with Kibana and APM server. The version deployed here is 8.6.0 for all the components. If you wish to use a newer version, change the tag 'version' in all YAML files. 

## Checkout Git repository

```
git clone https://github.com/adityasinghal26/elastic-on-k8s.git
cd yaml/monitoring
```

## Deploy Elasticsearch and Kibana

Since we are using elastic operator, we will make use of the Elastic CRDs in our YAML files. Below steps assume that Elastic Operator is successfully setup in Kubernetes cluster. 

To deploy the entire monitoring cluster, following steps need to be executed. 
1. Install elasticsearch using elasticsearch.yaml file. This file will create an elasticsearch cluster of 3 master and 2 data nodes in default namespace. 
   
   ```
   kubectl apply -f elasticsearch.yaml
   ```
   The configurations for the elasticsearch nodes are as below.
    - **Master nodes**: 2 vCPUs. 18Gi memory and 50Gi persistent volume for each node
    - **Data nodes**: 2 vCPUs, 18Gi memory and 2 Ti persistent volume for each node
    - **Storage class**: oci-bv
    - **Node Selector**: name 'ESMon' and kubelet version '1.24.0'
    - **Service Type**: Internal Flexible Load Balancer in Oracle Cloud, with a min '10 Mbps' and max '100 Mbps' bandwidth

2. Install Kibana by applying kibana.yaml file in default namespace. This will apply below configurations.

   ```
   kubectl apply -f kibana.yaml
   ```
   The configurations for the kibana nodes are as below.
   - **Kibana nodes**: 2Gi memory request
   - **Node Selector**: name 'ESMon' and kubelet version '1.24.0'
   - **Service Type**: Internal Flexible Load Balancer in Oracle Cloud, with a min '10 Mbps' and max '100 Mbps' bandwidth

## Deploy APM server 

In this example, we will be using Legacy APM setup with uses CRD **ApmServer**. This is deprecated and the new APM setup is done using Elastic Agent CRD.

To install APM server, apply apm-server.yaml file as below with following configurations. This YAML directly refers elasticsearch and kibana using **elasticsearchRef** and **kibanaRef** tags. In addition, **spec.config.apm-server.kibana** and **spec.config.output.elasticsearch** override certain values such as connection string and SSL properties to take _mycompany TLS certificate_ in account.

   ```
   kubectl apply -f apm-server.yaml
   ```
   The configurations for the elasticsearch nodes are as below.
   - **APM nodes**: 512Mi memory
   - **Node Selector**: name 'ESMon' and kubelet version '1.24.0'
   - **Service Type**: Internal Flexible Load Balancer in Oracle Cloud, with a min '10 Mbps' and max '100 Mbps' bandwidth
   - **TLS**: Disabled any TLS encryption
   - **Real User Monitoring**: Enabled Real User Monitoring with POST and OPTIONS HTTP headers (as per requirement)

## DNS configurations

Once the above deployments are successful, below hostnames can be created in any DNS nameservers

| Component     | DNS                         | Connection Strings                      |
|---------------|-----------------------------|-----------------------------------------|
| Elasticsearch | elasticsearch.mycompany.com | http://elasticsearch.mycompany.com:9200 |
| Kibana        | kibana.mycompany.com        | http://kibana.mycompany.com:443         |
| APM           | apm-server.mycompany.com    | http://apm-server.mycompany.com:443     |

## NOTE
- The above steps and YAMLs referred are as per usage in Oracle Cloud. For deploying the same in AWS/Azure/GCP, service annotations and storage class will need to be modified.
- The services will be exposed as internal load-balancer and uses a company TLS certificate **mycompany-tls** with domain **\*.mycompany.com**.

