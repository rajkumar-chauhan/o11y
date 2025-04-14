## Documentation for Tempo Stack

### Table of content :-
* [Introduction](#Introduction)
* [ Prerequisites](#Prerequisites)
* [Resources](#Resources)
* [Architecture and flow](#architecture-and-flow)
* [Scaling and Versioning](#scaling-and-versioning)
* [Redeploying ](#Redeploying )
* [Important Considerations During Redeployment](#important-considerations-during-redeployment)

### Introduction
This document outlines the architecture, prerequisites, resource requirements, data flow, and redeployment process for the Tempo stack configured using the provided values.yaml.The aim of this  Tempo stack is to collect, aggregate, store, and provide querying capabilities for traces within the Kubernetes cluster

###  Prerequisites
Before deploying this tempo stack, ensure the following prerequisites are met:

**Kubernetes Cluster:**  A running Kubernetes cluster .

**Helm:** Helm package manager installed and configured to interact with your Kubernetes cluster.

**Storage Provisioner:**  A default StorageClass configured in your cluster like in our case  buildpiper-storage .

### Resources:

| **Component**        | **CPU Requests** | **CPU Limits** | **Memory Requests** | **Memory Limits** |
|----------------------|------------------|----------------|----------------------|-------------------|
| **Ingester**         | 0.3              | 0.5            | 300Mi                | 600Mi             |
| **Metrics Generator**| 0.2              | 0.4            | 200Mi                | 400Mi             |
| **Distributor**      | 0.2              | 0.4            | 200Mi                | 400Mi             |
| **Compactor**        | 0.3              | 0.5            | 400Mi                | 700Mi             |
| **Querier**          | 0.2              | 0.4            | 200Mi                | 400Mi             |
| **Query Frontend**   | 0.3              | 0.5            | 300Mi                | 500Mi             |
| **Memcached**        | 0.3              | 0.5            | 400Mi                | 700Mi             |

**Important Considerations:**

The resource requests and limits should be adjusted based on the actual log volume and query load in your environment.

Ensure your Kubernetes nodes have sufficient resources to accommodate these deployments.

The persistent volume sizes for Ingesters, Memcached, and Index Gateway should be determined based on your retention policies and expected data growth.

### Architecture and flow 

![Screenshot 2025-04-11 005237](https://github.com/user-attachments/assets/ea5d6b85-2794-49b8-8ef6-625b530f1bd6)

The above diagram shows the working architecture of Grafana Tempo.

Firstly, the distributor receives spans in different formats  OpenTelemetry and sends these spans to ingesters by hashing the trace ID. Ingester then creates batches of traces which are called blocks.

Then it sends those blocks to the backend storage (S3/GCS). When you have a trace ID that you want to troubleshoot, you will use Grafana UI and put the trace ID in the search bar. Now querier is responsible for getting the details from either ingester or object storage about the trace ID you entered.

Firstly, it checks if that trace ID is present in the ingester; if it doesn’t find it, it then checks the storage backend.  Meanwhile, the compactor takes the blocks from the storage, combines them, and sends them back to the storage to reduce the number of blocks in the storage.

### Scaling and Versioning 

To ensure performance and availability of the Tempo stack under varying workloads, the configuration supports horizontal scaling through Kubernetes replica counts. While autoscaling is currently disabled for most components, it can be enabled for the ingester, distributor, and queryFrontend based on CPU or memory utilization metrics.

You can modify the dependency details to suit your requirements. This includes updating the version, changing the alias, or using a different repository. This flexibility allows teams to stay updated with newer versions or use internal/custom Helm repositories. You can update the Charts.yaml based on your requirement 
```
dependencies:
  - name: tempo-distributed
    alias: tempo
    version: 1.9.3
    repository: https://grafana.github.io/helm-charts
```
Versioning is managed through the image.tag field configuration. For consistency and reliability, ensure all Tempo components use the same version tag. It is advised to avoid using latest in production and instead specify a stable 
You can change the image version by changing in value.yaml .In values.yaml you find a field which is below:-  
```
  tempo:
    image:
      registry: docker.io
      repository: grafana/tempo
      tag: <version>
```
By changing this your tempo version (all component version) will be changed.
                              OR 
To change image of specific component you can also do that there is a specfic image field below each component configuration  .   


### Redeploying 
To redeploy your tempo stack after making changes to the values.yaml file, you will typically use the Helm command-line interface. Assuming you initially deployed tempo using a Helm release named tracing .
Navigate to the directory containing your updated values.yaml file.

Run the Helm upgrade command:
```
helm upgrade tracing . -f values.yaml -n logging
```

The . in the command refers to the current directory where your Helm chart (if you have one locally) is located. If you deployed from a remote chart repository, you would specify the chart name and repository again 

Monitor the rollout: After running the upgrade command, Helm will apply the changes defined in your updated values.yaml to your Kubernetes cluster. You can monitor the status of the deployments and stateful sets using kubectl:

### Important Considerations During Redeployment

**Rolling Updates:** Helm typically performs rolling updates for Deployments and StatefulSets, minimizing downtime. However, depending on the changes you've made (e.g., significant resource changes, image updates), some pods might be temporarily unavailable.

**Persistent Volumes:** Changes to persistent volume claims (size, storage class) might not be applied automatically and could require manual intervention or recreation of the resources. Be cautious when modifying these sections.

**Syntax Errors:** Ensure your values.yaml file is syntactically correct (YAML format). Helm will likely fail to upgrade if there are errors in the file.

### Note:- 
 Configuring tempo to use S3/GCS storage in your production environment, you'll benefit from improved durability, scalability, and cost-effectiveness for your tracing infrastructure. Remember to secure your S3/GCS bucket with appropriate  policies

By following these steps, you can effectively manage and update your Tempo stack configuration using Helm and the values.yaml file. Remember to always review the changes you make to values.yaml and understand their potential impact on your running system.







