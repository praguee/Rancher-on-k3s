
# Rancher and Cert-Manager Installation on K3s

This repository contains the steps to install Rancher and Cert-Manager on a K3s cluster, using Helm and MetalLB for load balancing.

## Prerequisites

- A running K3s cluster
- Helm installed
- kubectl installed and configured to interact with your K3s cluster

## Installation Steps

1. **Download and Install Helm:**

   ```bash
   curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   chmod 700 get_helm.sh
   ./get_helm.sh
   
Add the Rancher Helm repository:

helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

Create a namespace for Rancher:

kubectl create namespace cattle-system

Install Cert-Manager:

a. Add the Jetstack Helm repository:

helm repo add jetstack https://charts.jetstack.io
helm repo update

b. Apply the Custom Resource Definitions (CRDs):

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.2/cert-manager.crds.yaml

c. Install Cert-Manager:

helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.13.2

Verify Cert-Manager installation:

kubectl get pods --namespace cert-manager

Install Rancher:

helm install rancher rancher-latest/rancher --devel --namespace cattle-system --set hostname=rancher.praguee.co --set bootstrapPassword=admin

Check the rollout status of Rancher:

kubectl -n cattle-system rollout status deploy/rancher

Expose Rancher via LoadBalancer:

kubectl expose deployment rancher --name=rancher-lb --port=443 --type=LoadBalancer -n cattle-system

Verify services:

kubectl get svc -n cattle-system

Install MetalLB for Load Balancing:

a. Create the MetalLB namespace:

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml

b. Install MetalLB:

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
c. Apply the IP address pool configuration:

Create a file named ipAddressPool.yaml with the following content:

apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.0.100-192.168.0.200  # Replace with your desired range

Then apply the configuration:

kubectl apply -f ipAddressPool.yaml

Check the MetalLB pods:

kubectl get pods -n metallb-system

Restart Rancher deployments if needed:

kubectl rollout restart deployment/rancher -n cattle-system
kubectl rollout restart deployment/rancher-webhook -n cattle-system

Accessing Rancher

You can access Rancher at the following URL:

https://192.168.0.101/
Default Credentials
Username: admin
Password: admin (change this after logging in for the first time)

Troubleshooting

Common Issues and Solutions

Rancher Pods are Not Running:

Check the status of Rancher pods:

kubectl get pods -n cattle-system

If any pods are in CrashLoopBackOff or Error, check the logs for more information:

kubectl logs <pod-name> -n cattle-system

MetalLB is Not Assigning an External IP:

Ensure that MetalLB is installed correctly and the pods are running:

kubectl get pods -n metallb-system

Check the configuration in ipAddressPool.yaml for any errors in the IP range.
Confirm that the IP range specified is available in your local network.

Unable to Access Rancher Web UI:

Verify that the LoadBalancer service is running and has an external IP assigned:

kubectl get svc -n cattle-system

If the external IP is not assigned, check MetalLB logs for errors:

kubectl logs -n metallb-system <metallb-pod-name>

TLS/SSL Errors:

If you encounter TLS/SSL errors when accessing Rancher, ensure that your hostname is correctly configured and that you have valid certificates for the domain.

Helm Chart Version Issues:

If the installation fails due to Helm chart version incompatibilities, ensure you are using compatible versions of the charts for your K3s cluster.

Network Connectivity Issues:

Verify that there are no network policies or firewall rules preventing access to the Rancher service. Check your network configuration.

Notes

Ensure you replace any hardcoded IP address ranges with those that suit your network configuration.
Review and update the Helm chart versions as needed for the latest features and security updates.
