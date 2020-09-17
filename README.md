# ECK

This is a collection of showcase example to help you to use **Elastic Cloud on Kubernetes** on **GCP** as a real production environment.

These examples supose you're using GKE node pool with a Kubernetes Label (In this example **pool** : **Elasticsearch**) and also a Kubernetes manifest called **infra**.

### Content
- StorageClass manifest
- Elasticsearch manifest

### Features implemented

- Dedicated nodes
- Zone awareness
- GCS repository plugin
- Node & Pod Affinity
- ReadinessProbe
- Resource management 

### Architecture 

![ECK Architecture](img/architecture.png)


### Manifest explained
This section will guide you to understand each piece of Elasticsearch manifest.

##### HTTP Settings
Here we're creating a Internal loadbalancer on GCP attaching **only** data nodes on it. The default behavior will create a service with all Elasticsearch nodes attached which works perfect, but here I'm removing master node from the LoadBalancer so they can focus on maintaining global cluster state.

```apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elastic-prod
  namespace: infra
spec:
  http:
    service:
      metadata:
        annotations:
          cloud.google.com/load-balancer-type: Internal
          elasticsearch.k8s.elastic.co/node-master: "false"
      spec:
        type: LoadBalancer
```

##### Node Configuration
We can relate this section with elasticsearch.yml file, which means we're defining one Elasticsearch node name called **data-zone-a** as a data node. You can see that the other roles are disabled.

We're also defining routing allocation called **zone** and attributing the routing for a specific zone **europe-west3-a**.

 By default, Elasticsearch uses memory mapping (mmap) to efficiently access indices. Usually, default values for virtual address space on Linux distributions are too low for Elasticsearch to work properly, which may result in out-of-memory exceptions, that's why we're using *node.store.allow_mmap : false*


```
  nodeSets:
  - name: data-zone-a
    count: 1
    config:
      node.attr.zone: europe-west3-a
      cluster.routing.allocation.awareness.attributes: zone
      http.max_content_length: 200mb
      node.data: true
      node.ml: false
      node.ingest: false
      node.master: false
      node.store.allow_mmap: false
```

### To be implemented

- Setup my own certificate
- Cronjob with snapshot config


