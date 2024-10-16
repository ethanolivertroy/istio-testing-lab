# istio-testing-lab

**Setting Up an Istio Environment in AWS with EKS: A Step-by-Step Guide**

This guide will walk you through the process of setting up an Istio test environment on Amazon Web Services (AWS) using Amazon Elastic Kubernetes Service (EKS). The instructions include commands and explanations to help you replicate the environment seamlessly.

### Prerequisites
- **AWS CLI** installed and configured.
- **kubectl** installed and configured.
- **eksctl** installed to simplify EKS cluster management.
- **Istio CLI** (istioctl) installed.
- An AWS account with sufficient permissions to create EKS resources.

### Step 1: Create an EKS Cluster
First, we need to create an EKS cluster that will host our Istio deployment. Execute the following command:

```bash
# Create an EKS cluster named "istio-test-cluster" with a node group of 3 nodes.
eksctl create cluster \
  --name istio-test-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

This command will create a Kubernetes cluster with three nodes in the `us-east-1` region. The process might take around 10-15 minutes.

### Step 2: Verify the Cluster Setup
After the EKS cluster is created, verify that the nodes are running:

```bash
# Verify if nodes are ready.
kubectl get nodes
```

You should see three nodes in the `Ready` state.

### Step 3: Install Istio CLI
Download and install Istio CLI if you haven't already. For this guide, we'll be using Istio version 1.23.2:

```bash
# Download Istio version 1.23.2
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.23.2 sh -

# Change directory to the Istio package
cd istio-1.23.2

# Add istioctl to your PATH
export PATH=$PWD/bin:$PATH
```

### Step 4: Install Istio on the EKS Cluster
Install Istio with default settings for a demo profile. The demo profile is suitable for a testing environment.

```bash
# Install Istio in the Kubernetes cluster
istioctl install --set profile=demo -y
```

This command installs Istio with default configurations suitable for learning and testing purposes.

### Step 5: Label the Namespace for Automatic Sidecar Injection
Label the default namespace (or any other namespace of your choice) for automatic Istio sidecar injection:

```bash
# Label the default namespace for automatic sidecar injection
kubectl label namespace default istio-injection=enabled
```

### Step 6: Deploy a Sample Application
To verify that Istio is working correctly, deploy a sample application. We'll use the Bookinfo app provided by Istio:

```bash
# Deploy the Bookinfo application
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

After deploying, confirm that the pods are running:

```bash
# Check the status of the Bookinfo application pods
kubectl get pods
```

### Step 7: Create an Istio Gateway
Create an Istio Gateway to expose the Bookinfo application:

```bash
# Apply the Gateway configuration for Bookinfo
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

# Verify that the gateway has been created
kubectl get gateway
```

### Step 8: Access the Application
To access the Bookinfo application, determine the external IP address of the Istio ingress gateway:

```bash
# Get the external IP address of the Istio ingress gateway
kubectl get svc istio-ingressgateway -n istio-system
```

Visit the external IP in your browser to see the Bookinfo application running.

### Step 9: Verify Istio Components
To ensure Istio is functioning as expected, run the following command to check the status of all Istio components:

```bash
# Get the status of Istio components
istioctl proxy-status
```

This command will display the synchronization status of all proxies in the mesh.

### Step 10: Clean Up
If you're done testing and want to clean up the resources, run:

```bash
# Delete the Bookinfo application
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml

# Delete the Istio installation
istioctl uninstall --purge y

# Delete the EKS cluster
eksctl delete cluster --name istio-test-cluster
```
