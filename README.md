# gke-service-catalog-sample-gcs
This is a sample application that demonstrates how to use k8s Service Catalog, GCP Service Broker &amp; GCS from a GKE cluster with Istio enabled.

It extends the [quotes](https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/tree/master/service-catalog/cloud-storage) sample application to make it work from an Istio cluster.

## Prerequisites

Successful use of the sample requires:

*   A Kubernetes cluster, minimum version 1.8.x.
*   Kubernetes Service Catalog and the Google Cloud Platform Service Broker [installed](
    https://cloud.google.com/kubernetes-engine/docs/how-to/add-on/service-broker/install-service-catalog).
*   The [Service Catalog
    CLI](https://github.com/kubernetes-incubator/service-catalog/blob/master/docs/install.md#installing-the-service-catalog-cli)
    (`svcat`) installed.
*   A running [installation](https://istio.io/docs/setup/kubernetes/) of Istio

## Step 1: Create a Kubernetes namespace
```shell
kubectl apply -f manifests/storage-quotes-namespace.yaml
```

## Step 2: Provisioning Cloud IAM Service Account instance

This sample application is using a single service account for authentication with all Google Cloud services.

* Create the cloud-iam-service-account istance
```shell
kubectl apply -f manifests/gcp-iam-sa/instance.yaml
```
The instance is provisioned when status is `Ready` : ```svcat get instances --namespace storage-quotes```
* Create the binding to the cloud-iam-service-account istance
```shell
kubectl apply -f manifests/gcp-iam-sa/binding.yaml
```
Wait for the binding to be `Ready` : ```svcat get bindings --namespace storage-quotes```
* Once the instance and the binding are `Ready`, make sure to have the following secret:
```
kubectl get secrets -n storage-quotes
NAME                               TYPE                                  DATA      AGE
gcp-iam-sa-credentials             Opaque                                2         1h
```
## Step 3: Provisioning Cloud Storage instance

* Create the cloud-storage istance
```shell
kubectl apply -f manifests/gcp-gcs-quotes/instance.yaml
```
The instance is provisioned when status is `Ready` : ```svcat get instances --namespace storage-quotes```
* Create the binding to the cloud-storage istance
```shell
kubectl apply -f manifests/gcp-gcs-quotes/binding.yaml
```
Wait for the binding to be `Ready` : ```svcat get bindings --namespace storage-quotes```
* Once the instance and the binding are `Ready`, make sure to have the following secrets:
```
kubectl get secrets -n storage-quotes
NAME                               TYPE                                  DATA      AGE
gcp-iam-sa-credentials             Opaque                                2         1h
quotes-gcp-gcs-credentials         Opaque                                3         1h
```

## Step 4: Deploy the application
```shell
kubectl apply -f manifests/quotes-deployment.yaml
```
## Step 5: Access the application
Now that the application is up and running, we need to make the application accessible from outside of your Kubernetes cluster. An [Istio Gateway](https://istio.io/docs/concepts/traffic-management/#gateways) is used for this purpose.
```shell
kubectl apply -f manifests/quotes-gateway.yaml
```
### Determining the ingress IP and ports
Execute the following command to determine if your Kubernetes cluster is running in an environment that supports external load balancers:
```shell
kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   172.21.109.129   130.211.10.121  80:31380/TCP,443:31390/TCP,31400:31400/TCP   17h
```
If the EXTERNAL-IP value is set, your environment has an external load balancer that you can use for the ingress gateway.

```$ export GATEWAY_URL=$EXTERNAL-IP```
```shell
# Query the quotes:
curl -s http://${GATEWAY_URL}/quotes
{"quotes":[]}
```
```shell
# Create a new quote:
curl http://${GATEWAY_URL}/quotes -d '{"person": "Dalai Lama", "quote": "Be kind whenever possible. It is always possible."}'

# Query the quotes again:
curl http://${GATEWAY_URL}/quotes
{"quotes":[{"person":"Dalai Lama","quote":"Be kind whenever possible. It is always possible."}]}
```

Et voilà !

## Credits

- [GoogleCloudPlatform/kubernetes-engine-samples](https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/tree/master/service-catalog#application-service-account)
- [istio.io](https://istio.io/docs/reference/config/networking/v1alpha3)
- [istio.io/bookinfo](https://istio.io/docs/examples/bookinfo/)
