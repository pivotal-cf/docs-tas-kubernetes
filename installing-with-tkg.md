---
title: Installing Tanzu Application Service for Kubernetes on Tanzu Kubernetes
Grid
owner: Tanzu Application Service Release Engineering
---
## Installing TAS for K8s on AWS TKG

### Preparing the Kubernetes Cluster

#### Tanzu Kubernetes Grid
Ensure you have a Kubernetes cluster created with the following characteristics:
- The TKG version should be v1.0+
- The Kubernetes cluster should have at least 5 worker nodes
- Each worker node should have at least 2 CPU and 7.5 GB of memory available for allocation.
- The worker nodes in the Kubernetes cluster should trust the container image
  registry that Tanzu Application Service for Kubernetes will use to store
  buildpack-based app images. If the registry has a certificate issued from a
  private certificate authority (CA), this typically entails configuring the
  worker nodes in the Kubernetes cluster to trust the certificate of that CA.

  See the [TKG docs](some link goes here) for information about setting up TKG
  and creating a cluster.

#### Configuring Storage

To check whether your Kubernetes cluster already has a default storage class installed:

1. Retrieve the list of StorageClass resources in the cluster:
  <pre><code>$ kubectl get storageclass</code></pre>
  For example:
  <pre><code>$ kubectl get storageclass
NAME             PROVISIONER                    AGE
gp2 (default)    kubernetes.io/aws-ebs          34h
</code></pre>

1. If the list includes an entry for which `(default)` follows the storage class name, then the Kubernetes cluster has a default storage class installed already, and you are ready to provision volumes dynamically.

If the list does not include a default, you must install one before deploying Tanzu Application Service for Kubernetes.
For your convenience, the Tanzu Application Service for Kubernetes installation bundle provides an AWS provisioner storage class for you to install:

1. Ensure that you have the Tanzu Application Service for Kubernetes installation bundle downloaded and extracted to your local workstation.

1. Ensure that you have installed the `kapp` utility.

1. Change to the `tanzu-application-service` directory in your terminal.

1. Run the following command to install the storage class:
  <pre><code>$ kapp deploy -a default-storage-class -f config-optional/aws-default-storage-class.yml</code></pre>

Before proceeding, review the [Configuring Installation Values](configuring-installation-values.html) topic to ensure that you have configured all of the required or recommended installation resources.


## <a id='install-tas-for-k8s'></a> Installing Tanzu Application Service for Kubernetes

<p class="note">
	Installing Tanzu Application Service for Kubernetes takes approximately 15 minutes, depending on cluster resources and bandwidth.
</p>

1. Run `kubectl cluster-info` and inspect the output to ensure that your Kubernetes client configuration targets the intended cluster for installation.

1. Change into the `tanzu-application-service` directory in your terminal.

1. Run the installation script, configured to use the deployment values you generated previously:

```bash
$ ./bin/install-tas.sh ../configuration-values
```


## <a id='post-installation-networking-configuration'></a> Post-Installation Networking Configuration

This section discusses how to configure DNS entries and load balancers for the Tanzu Application Service for Kubernetes ingress gateway. Follow one of the configuration cases below, depending on your installation.


### DNS Configuration with No Load Balancer for Ingress Gateway

By default, Tanzu Application Service for Kubernetes is configured not to create a Kubernetes LoadBalancer service for the ingress gateway.
If you have deployed Tanzu Application Service for Kubernetes in this configuration, and do not have an external load balancer to use for ingress to the installation, set up your DNS records to establish ingress connectivity directly to the worker nodes:

1. Use the `kubectl` CLI to retrieve the list of worker nodes with their external IP addresses:
  <pre><code>$ kubectl get nodes --output='wide'</code></pre>
  For example:
  <pre><code>$ kubectl get nodes --output='wide'
NAME                                   STATUS   ROLES    AGE     VERSION   INTERNAL-IP    EXTERNAL-IP
5e329c31-f1d7-4548-936b-3a58d4b166d3   Ready    \<none>   5h49m   v1.15.5   10.85.87.133   10.85.87.133
a6ad3f07-787c-4d90-b8e1-032be34e9d7f   Ready    \<none>   5h43m   v1.15.5   10.85.87.134   10.85.87.134
a8eb78a2-e3b4-4d8a-8c32-67bf0e13c0bf   Ready    \<none>   5h43m   v1.15.5   10.85.87.135   10.85.87.135
af7dc8da-a7b0-4cf2-a940-c9248168e609   Ready    \<none>   5h43m   v1.15.5   10.85.87.136   10.85.87.136
cc6ef11f-e253-4553-9cb0-bebc7d958f64   Ready    \<none>   5h42m   v1.15.5   10.85.87.137   10.85.87.137
</code></pre>

1. In your DNS zone, create a wildcard `A` record for the system domain, `*.PLACEHOLDER-SYSTEM-DOMAIN`, resolving to the set of external IP addresses for the worker nodes.
Make sure you include the `*.` wildcard prefix so that all subdomains of the system domain also resolve to these IP addresses.






### Post-Installation Networking Configuration

#### DNS Configuration with a Kubernetes Load Balancer for Ingress Gateway

If you have instead optionally configured Tanzu Application Service for Kubernetes to [use a Kubernetes LoadBalancer Service for the ingress gateway](#adjust-installation-resources-networking), set up DNS for your system domain to resolve to the external IP of the load balancer:

1. Use `kubectl` to retrieve the value of the external IP assigned to the Istio ingress gateway service:
  <pre><code>$ kubectl -n istio-system get service istio-ingressgateway -ojsonpath='{.status.loadBalancer.ingress[0].ip}'</code></pre>
  For example:
  <pre><code>$ kubectl -n istio-system get service istio-ingressgateway -ojsonpath='{.status.loadBalancer.ingress[0].ip}'
10.193.105.162
</code></pre>

1. In your DNS zone, create a wildcard `A` record for the system domain, `*.PLACEHOLDER-SYSTEM-DOMAIN`, resolving to this external IP address.
Make sure you include the `*.` wildcard prefix so that all subdomains of the system domain also resolve to this IP address.

## <a id='post-installation-configuration'></a> Post-Installation Validation


<p class="note">
  <strong>Note:</strong> The route for the test application defaults to a subdomain of the system domain.
</p>

1. Ensure that you are still logged into the Tanzu Application Service for Kubernetes installation as the admin user.

1. Create and target an organization and space for the verification application:
  <pre><code>$ cf create-org test-org
  $ cf create-space -o test-org test-space
  $ cf target -o test-org -s test-space
  </code></pre>

1. Clone the [Cloud Foundry test application](https://github.com/cloudfoundry-samples/test-app) from GitHub to your workstation and change into the resulting `test-app` directory.

1. Push the test app to the installation:
  <pre><code>$ cf push test-app --hostname test-app</code></pre>

1. While the `cf push` command is running, open another terminal pane and monitor the build logs:
  <pre><code>$ cf logs test-app</code></pre>

1. After the `cf push` command succeeds, make a request to the app:
  <pre><code>$ curl test-app.PLACEHOLDER-SYSTEM-DOMAIN</code></pre>
