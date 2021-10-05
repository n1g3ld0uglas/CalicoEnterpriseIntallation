# Calico Enterprise Intallation
Step-by-Step process for installing Calico Enterprise in Air-Gapped AKS Clusters.

## Prerequisite Checks

Enable network mode using the aks-preview extension:
```
az extension add --name aks-preview
az feature register -n AKSNetworkModePreview --namespace Microsoft.ContainerService
az provider register -n Microsoft.ContainerService
```
Create cluster using ```az deployment group create``` with the values:
```
"networkProfile": {
 "networkPlugin": "azure",
 "networkMode": "transparent"
}
```
## AKS Azure Files storage

In the following example for an AKS cloud provider integration, the StorageClass tells Calico Enterprise to use LRS disks for log storage:<br/>
https://docs.tigera.io/getting-started/create-storage

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: tigera-elasticsearch
provisioner: kubernetes.io/azure-disk
parameters:
  cachingmode: ReadOnly
  kind: Managed
  storageaccounttype: StandardSSD_LRS
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

## Install Calico Enterprise

If pulling images directly from ```quay.io/tigera```, you will likely want to use the credentials provided to you by Nigel:

```
kubectl create secret generic tigera-pull-secret \
    --type=kubernetes.io/dockerconfigjson -n tigera-operator \
    --from-file=.dockerconfigjson=<path/to/pull/secret>
```

If using a ```PRIVATE REGISTRY```, use your private registry credentials instead: <br/>
https://docs.tigera.io/v3.10/getting-started/private-registry/private-registry-regular

## Push Calico Enterprise images to your private registry

Use the following commands to pull the required Calico Enterprise images:

```
docker pull quay.io/tigera/operator:v1.22.0
docker pull quay.io/tigera/cnx-manager:v3.10.0
docker pull quay.io/tigera/voltron:v3.10.0
docker pull quay.io/tigera/guardian:v3.10.0
docker pull quay.io/tigera/cnx-apiserver:v3.10.0
docker pull quay.io/tigera/cnx-queryserver:v3.10.0
docker pull quay.io/tigera/kube-controllers:v3.10.0
docker pull quay.io/tigera/calicoq:v3.10.0
docker pull quay.io/tigera/typha:v3.10.0
docker pull quay.io/tigera/calicoctl:v3.10.0
docker pull quay.io/tigera/cnx-node:v3.10.0
docker pull quay.io/tigera/dikastes:v3.10.0
docker pull quay.io/tigera/dex:v3.10.0
docker pull quay.io/tigera/fluentd:v3.10.0
docker pull quay.io/tigera/es-proxy:v3.10.0
docker pull quay.io/tigera/kibana:v3.10.0
docker pull quay.io/tigera/elasticsearch:v3.10.0
docker pull quay.io/tigera/cloud-controllers:v3.10.0
docker pull quay.io/tigera/intrusion-detection-job-installer:v3.10.0
docker pull quay.io/tigera/es-curator:v3.10.0
docker pull quay.io/tigera/intrusion-detection-controller:v3.10.0
docker pull quay.io/tigera/compliance-controller:v3.10.0
docker pull quay.io/tigera/compliance-reporter:v3.10.0
docker pull quay.io/tigera/compliance-snapshotter:v3.10.0
docker pull quay.io/tigera/compliance-server:v3.10.0
docker pull quay.io/tigera/compliance-benchmarker:v3.10.0
docker pull quay.io/tigera/ingress-collector:v3.10.0
docker pull quay.io/tigera/l7-collector:v3.10.0
docker pull quay.io/tigera/license-agent:v3.10.0
docker pull quay.io/tigera/cni:v3.10.0
docker pull quay.io/tigera/firewall-integration:v3.10.0
docker pull quay.io/tigera/egress-gateway:v3.10.0
docker pull quay.io/tigera/honeypod:v3.10.0
docker pull quay.io/tigera/honeypod-exp-service:v3.10.0
docker pull quay.io/tigera/honeypod-controller:v3.10.0
docker pull quay.io/tigera/key-cert-provisioner:v1.1.0
docker pull quay.io/tigera/anomaly_detection_jobs:v3.10.0
docker pull quay.io/tigera/elasticsearch-metrics:v3.10.0
docker pull quay.io/tigera/packetcapture-api:v3.10.0
docker pull quay.io/tigera/prometheus-service:v3.10.0
docker pull quay.io/tigera/es-gateway:v3.10.0
docker pull quay.io/tigera/performance_hotspots:v3.10.0
docker pull quay.io/tigera/deep-packet-inspection:v3.10.0
docker pull quay.io/tigera/eck-operator:1.7.1
docker pull quay.io/tigera/prometheus-operator:v0.40.0
docker pull quay.io/tigera/prometheus:v2.17.2
docker pull quay.io/tigera/alertmanager:v0.20.0
docker pull quay.io/tigera/prometheus-config-reloader:v0.40.0
docker pull quay.io/tigera/configmap-reload:v0.0.1
docker pull quay.io/calico/pod2daemon-flexvol:v3.20.0
docker pull quay.io/tigera/envoy:v3.10.0
docker pull quay.io/tigera/envoy-init:v3.10.0
```

Re-tag the images with the name of your private registry ```$PRIVATE_REGISTRY```:

```
docker tag quay.io/tigera/operator:v1.22.0 $PRIVATE_REGISTRY/tigera/operator:v1.22.0
docker tag quay.io/tigera/cnx-manager:v3.10.0 $PRIVATE_REGISTRY/tigera/cnx-manager:v3.10.0
docker tag quay.io/tigera/voltron:v3.10.0 $PRIVATE_REGISTRY/tigera/voltron:v3.10.0
docker tag quay.io/tigera/guardian:v3.10.0 $PRIVATE_REGISTRY/tigera/guardian:v3.10.0
docker tag quay.io/tigera/cnx-apiserver:v3.10.0 $PRIVATE_REGISTRY/tigera/cnx-apiserver:v3.10.0
docker tag quay.io/tigera/cnx-queryserver:v3.10.0 $PRIVATE_REGISTRY/tigera/cnx-queryserver:v3.10.0
docker tag quay.io/tigera/kube-controllers:v3.10.0 $PRIVATE_REGISTRY/tigera/kube-controllers:v3.10.0
docker tag quay.io/tigera/calicoq:v3.10.0 $PRIVATE_REGISTRY/tigera/calicoq:v3.10.0
docker tag quay.io/tigera/typha:v3.10.0 $PRIVATE_REGISTRY/tigera/typha:v3.10.0
docker tag quay.io/tigera/calicoctl:v3.10.0 $PRIVATE_REGISTRY/tigera/calicoctl:v3.10.0
docker tag quay.io/tigera/cnx-node:v3.10.0 $PRIVATE_REGISTRY/tigera/cnx-node:v3.10.0
docker tag quay.io/tigera/dikastes:v3.10.0 $PRIVATE_REGISTRY/tigera/dikastes:v3.10.0
docker tag quay.io/tigera/dex:v3.10.0 $PRIVATE_REGISTRY/tigera/dex:v3.10.0
docker tag quay.io/tigera/fluentd:v3.10.0 $PRIVATE_REGISTRY/tigera/fluentd:v3.10.0
docker tag quay.io/tigera/es-proxy:v3.10.0 $PRIVATE_REGISTRY/tigera/es-proxy:v3.10.0
docker tag quay.io/tigera/kibana:v3.10.0 $PRIVATE_REGISTRY/tigera/kibana:v3.10.0
docker tag quay.io/tigera/elasticsearch:v3.10.0 $PRIVATE_REGISTRY/tigera/elasticsearch:v3.10.0
docker tag quay.io/tigera/cloud-controllers:v3.10.0 $PRIVATE_REGISTRY/tigera/cloud-controllers:v3.10.0
docker tag quay.io/tigera/intrusion-detection-job-installer:v3.10.0 $PRIVATE_REGISTRY/tigera/intrusion-detection-job-installer:v3.10.0
docker tag quay.io/tigera/es-curator:v3.10.0 $PRIVATE_REGISTRY/tigera/es-curator:v3.10.0
docker tag quay.io/tigera/intrusion-detection-controller:v3.10.0 $PRIVATE_REGISTRY/tigera/intrusion-detection-controller:v3.10.0
docker tag quay.io/tigera/compliance-controller:v3.10.0 $PRIVATE_REGISTRY/tigera/compliance-controller:v3.10.0
docker tag quay.io/tigera/compliance-reporter:v3.10.0 $PRIVATE_REGISTRY/tigera/compliance-reporter:v3.10.0
docker tag quay.io/tigera/compliance-snapshotter:v3.10.0 $PRIVATE_REGISTRY/tigera/compliance-snapshotter:v3.10.0
docker tag quay.io/tigera/compliance-server:v3.10.0 $PRIVATE_REGISTRY/tigera/compliance-server:v3.10.0
docker tag quay.io/tigera/compliance-benchmarker:v3.10.0 $PRIVATE_REGISTRY/tigera/compliance-benchmarker:v3.10.0
docker tag quay.io/tigera/ingress-collector:v3.10.0 $PRIVATE_REGISTRY/tigera/ingress-collector:v3.10.0
docker tag quay.io/tigera/l7-collector:v3.10.0 $PRIVATE_REGISTRY/tigera/l7-collector:v3.10.0
docker tag quay.io/tigera/license-agent:v3.10.0 $PRIVATE_REGISTRY/tigera/license-agent:v3.10.0
docker tag quay.io/tigera/cni:v3.10.0 $PRIVATE_REGISTRY/tigera/cni:v3.10.0
docker tag quay.io/tigera/firewall-integration:v3.10.0 $PRIVATE_REGISTRY/tigera/firewall-integration:v3.10.0
docker tag quay.io/tigera/egress-gateway:v3.10.0 $PRIVATE_REGISTRY/tigera/egress-gateway:v3.10.0
docker tag quay.io/tigera/honeypod:v3.10.0 $PRIVATE_REGISTRY/tigera/honeypod:v3.10.0
docker tag quay.io/tigera/honeypod-exp-service:v3.10.0 $PRIVATE_REGISTRY/tigera/honeypod-exp-service:v3.10.0
docker tag quay.io/tigera/honeypod-controller:v3.10.0 $PRIVATE_REGISTRY/tigera/honeypod-controller:v3.10.0
docker tag quay.io/tigera/key-cert-provisioner:v1.1.0 $PRIVATE_REGISTRY/tigera/key-cert-provisioner:v1.1.0
docker tag quay.io/tigera/anomaly_detection_jobs:v3.10.0 $PRIVATE_REGISTRY/tigera/anomaly_detection_jobs:v3.10.0
docker tag quay.io/tigera/elasticsearch-metrics:v3.10.0 $PRIVATE_REGISTRY/tigera/elasticsearch-metrics:v3.10.0
docker tag quay.io/tigera/packetcapture-api:v3.10.0 $PRIVATE_REGISTRY/tigera/packetcapture-api:v3.10.0
docker tag quay.io/tigera/prometheus-service:v3.10.0 $PRIVATE_REGISTRY/tigera/prometheus-service:v3.10.0
docker tag quay.io/tigera/es-gateway:v3.10.0 $PRIVATE_REGISTRY/tigera/es-gateway:v3.10.0
docker tag quay.io/tigera/performance_hotspots:v3.10.0 $PRIVATE_REGISTRY/tigera/performance_hotspots:v3.10.0
docker tag quay.io/tigera/deep-packet-inspection:v3.10.0 $PRIVATE_REGISTRY/tigera/deep-packet-inspection:v3.10.0
docker tag quay.io/tigera/eck-operator:1.7.1 $PRIVATE_REGISTRY/tigera/eck-operator:1.7.1
docker tag quay.io/tigera/prometheus-operator:v0.40.0 $PRIVATE_REGISTRY/tigera/prometheus-operator:v0.40.0
docker tag quay.io/tigera/prometheus:v2.17.2 $PRIVATE_REGISTRY/tigera/prometheus:v2.17.2
docker tag quay.io/tigera/alertmanager:v0.20.0 $PRIVATE_REGISTRY/tigera/alertmanager:v0.20.0
docker tag quay.io/tigera/prometheus-config-reloader:v0.40.0 $PRIVATE_REGISTRY/tigera/prometheus-config-reloader:v0.40.0
docker tag quay.io/tigera/configmap-reload:v0.0.1 $PRIVATE_REGISTRY/tigera/configmap-reload:v0.0.1
docker tag quay.io/calico/pod2daemon-flexvol:v3.20.0 $PRIVATE_REGISTRY/calico/pod2daemon-flexvol:v3.20.0
docker tag quay.io/tigera/envoy:v3.10.0 $PRIVATE_REGISTRY/tigera/envoy:v3.10.0
docker tag quay.io/tigera/envoy-init:v3.10.0 $PRIVATE_REGISTRY/tigera/envoy-init:v3.10.0
```

Push the images to your private registry:

```
docker push $PRIVATE_REGISTRY/tigera/operator:v1.22.0
docker push $PRIVATE_REGISTRY/tigera/cnx-manager:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/voltron:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/guardian:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/cnx-apiserver:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/cnx-queryserver:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/kube-controllers:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/calicoq:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/typha:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/calicoctl:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/cnx-node:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/dikastes:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/dex:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/fluentd:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/es-proxy:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/kibana:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/elasticsearch:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/cloud-controllers:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/intrusion-detection-job-installer:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/es-curator:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/intrusion-detection-controller:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/compliance-controller:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/compliance-reporter:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/compliance-snapshotter:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/compliance-server:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/compliance-benchmarker:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/ingress-collector:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/l7-collector:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/license-agent:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/cni:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/firewall-integration:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/egress-gateway:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/honeypod:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/honeypod-exp-service:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/honeypod-controller:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/key-cert-provisioner:v1.1.0
docker push $PRIVATE_REGISTRY/tigera/anomaly_detection_jobs:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/elasticsearch-metrics:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/packetcapture-api:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/prometheus-service:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/es-gateway:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/performance_hotspots:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/deep-packet-inspection:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/eck-operator:1.7.1
docker push $PRIVATE_REGISTRY/tigera/prometheus-operator:v0.40.0
docker push $PRIVATE_REGISTRY/tigera/prometheus:v2.17.2
docker push $PRIVATE_REGISTRY/tigera/alertmanager:v0.20.0
docker push $PRIVATE_REGISTRY/tigera/prometheus-config-reloader:v0.40.0
docker push $PRIVATE_REGISTRY/tigera/configmap-reload:v0.0.1
docker push $PRIVATE_REGISTRY/calico/pod2daemon-flexvol:v3.20.0
docker push $PRIVATE_REGISTRY/tigera/envoy:v3.10.0
docker push $PRIVATE_REGISTRY/tigera/envoy-init:v3.10.0
```

## Run the operator using images from your private registry:

Download the Tigera operator and Custom Resource Definitions (CRD's):

```
wget https://docs.tigera.io/manifests/tigera-operator.yaml
```

Before applying ```tigera-operator.yaml```, modify registry references to use your custom registry:

```
sed -ie "s?quay.io?$PRIVATE_REGISTRY?g" tigera-operator.yaml
```

Next, ensure that an image pull secret has been configured for your custom registry. <br/>
Set the enviroment variable ```PRIVATE_REGISTRY_PULL_SECRET``` to the secret name. <br/>
Then add the image pull secret to the operator deployment spec:

```
sed -ie "/serviceAccountName: tigera-operator/a \      imagePullSecrets:\n\      - name: $PRIVATE_REGISTRY_PULL_SECRET"  tigera-operator.yaml
```

Simpilarly, we need to download the Prometheus operator and related Custom Resource Definitions (CRD's):

```
wget https://docs.tigera.io/manifests/tigera-prometheus-operator.yaml
```

If you are installing Prometheus operator as part of Calico Enterprise, then before applying ```tigera-prometheus-operator.yaml```, modify registry references to use your custom registry:

```
sed -ie "s?quay.io?$PRIVATE_REGISTRY?g" tigera-prometheus-operator.yaml
sed -ie "/serviceAccountName: calico-prometheus-operator/a \      imagePullSecrets:\n\      - name: $PRIVATE_REGISTRY_PULL_SECRET"  tigera-prometheus-operator.yaml
```

Before applying ```custom-resources.yaml```, modify registry references to use your custom registry:

```
sed -ie "s?quay.io?$PRIVATE_REGISTRY?g" custom-resources.yaml
```

## Configure the operator to use images from your private registry:

Set the ```spec.registry``` field of your Installation resource to the name of your custom registry. For example:

```
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  variant: TigeraSecureEnterprise
  imagePullSecrets:
    - name: tigera-pull-secret
  registry: myregistry.com
```

## Monitoring Install Progress

You can now monitor progress with the following command:

```
watch kubectl get tigerastatus
```

Wait until the ```apiserver``` shows a status of ```Available```, then proceed to the next section.

## Install the Calico Enterprise license:

In order to use Calico Enterprise, you must install the license provided to you by Nigel:

```
kubectl create -f </path/to/license.yaml>
```

Again, you can monitor progress with the following command:

```
watch kubectl get tigerastatus
```

When all components show a status of ```Available```, proceed to the next section.

## Secure Calico Enterprise with network policy:

To secure Calico Enterprise component communications, install the following set of network policies:

```
kubectl create -f https://docs.tigera.io/manifests/tigera-policies.yaml
```

## Configure access to Calico Enterprise Manager UI:

We agreed to configure your cluster with an ingress controller to implement the Ingress resource: <br/>
https://docs.tigera.io/getting-started/cnx/access-the-manager<br/>
<br/>


```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/backend-protocol: HTTPS
 creationTimestamp: "2021-09-30T13:08:38Z"
 generation: 9
 name: tigera-manager
 namespace: tigera-manager
 resourceVersion: "2341835"
 uid: f05ddf42-967f-4675-9fb5-66458d795148
spec:
 rules:
 - host: azuwevelcbaksd001.velux.org
   http:
     paths:
     - backend:
         service:
           name: tigera-manager
           port:
             number: 9443
       path: /
       pathType: Prefix
```

Our outstanding update for the Ingress manifest is to expose a specific URI: <br/>
```domain.com/Tigera/policies/tiered``` instead of ```domain.com/policies/tiered```

## Authentication Quickstart:

Get started quickly with our default token authentication to log in to Calico Enterprise Manager UI and Kibana: <br/>
https://docs.tigera.io/getting-started/cnx/authentication-quickstart

#### Log in to Calico Enterprise Manager:

First, create the service account ```Jinhong``` in the desired namespace:

```
kubectl create sa Jinhong -n default
```

Give the service account permissions to access the Calico Enterprise Manager UI, and a Calico Enterprise cluster role:

```
kubectl create clusterrolebinding <binding_name> --clusterrole <role_name> --serviceaccount <namespace>:<service_account>

```
