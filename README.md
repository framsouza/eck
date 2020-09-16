# ECK

This is a collection of showcase example to help you to use **Elastic Cloud on Kubernetes** on **GCP** as a real production environment.

These examples supose you're using GKE node pool with a Kubernetes Label (In this example **pool**:**Elasticsearch**)

### Content
- StorageClass manifest
- Elasticsearch manifest
- Elasticsearch Master service manifest (Internal)

### To do

- Setup my own certificate
- Cronjob with snapshot config

### Features implemented

- Dedicated nodes
- Zone awareness
- GCS repository plugin
- Node & Pod Affinity
- ReadinessProbe
