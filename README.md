# ECK

This is a collection of showcase example to help you to use **Elastic Cloud on Kubernetes** on **GCP** as a real production environment.

### Before start
If you wanna run this example without make any change on the manifest, please make sure you're using the following configuration:
- GKE node pool with a Kubernetes label called **pool** : **elasticsearch**
- Kubernetes namespace called **infra**
- GKE Cluster running on **europe-west3** region

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

#### HTTP Settings
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

#### Node Configuration
We can relate this section with elasticsearch.yml file, which means we're defining one Elasticsearch node name called **data-zone-a** as a data node. You can see that the other roles are disabled.

We're also defining routing allocation called **zone** and attributing the routing for a specific zone **europe-west3-a**.

By default, Elasticsearch uses memory mapping (mmap) to efficiently access indices. Usually, default values for virtual address space on Linux distributions are too low for Elasticsearch to work properly, which may result in out-of-memory exceptions, that's why we're using **node.store.allow_mmap : false**


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

#### Volume Claim
For production workloads is highly recommended to configure your own volume claim template with the desired storage capacity. Here we're using a StorageClass called **fast-europe-west3** with 30Gi.


```
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 30Gi
        storageClassName: fast-europe-west3
```

#### Node Affinity & Pod Affinity
The affinity feature restrict scheduling pods in a group of Kubernetes nodes based on labels.Node affinity is conceptually similar to **nodeSelector** which defined in qhich nodes your pod will be scheduled on based on the label. nodeAffinity greatly extends the types of constrains you can express using enhancements labels.

`podAntiAffinity` will prevent scheduling Elasticsearch nodes on the same host.

In this example `podAffinity` & `nodeAffinity` are using **requiredDuringSchedulingIgnoredDuringExecution** affinity type. That means, a **hard** limit which rules must be met for a pod to be scheduled in a node. In this example, we're defining something like: "only run the pod on nodes in a zone **europe-west3-a** AND with a label called **pool** : **elasticseasrch**.


```
    podTemplate:
      spec:
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: failure-domain.beta.kubernetes.io/zone
                  operator: In
                  values:
                  - europe-west3-a
              - matchExpressions:
                - key: pool
                  operator: In
                  values:
                  - elasticsearch
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  elasticsearch.k8s.elastic.co/cluster-name: elastic-prod
              topologyKey: kubernetes.io/hostname
        nodeSelector:
          pool: elasticsearch
```

### To be implemented

- Setup my own certificate
- Cronjob with snapshot config


