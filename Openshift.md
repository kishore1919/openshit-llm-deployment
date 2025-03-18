###

# **Installation:**

To Install the latest version of Openshit :  

1. go to <https://mirror.openshift.com/pub/> and select AWS to get the latest version of the open shift installer and client
2. Extract the zip files and once the installer is extracted the user will get the install-config.yaml
3. Change the install configuration with the below yaml
4. It will take one hour to install the open shift and create the resources
5. Once the resources are created.

**Install The latest version using the link:-**  
<https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/4.16.9/>

**VM Creation Yaml:**

```
additionalTrustBundlePolicy: Proxyonly
apiVersion: v1
baseDomain: aimintopenshift.com
compute:
  - name: worker  # GPU worker nodes
    hyperthreading: Enabled
    platform:
      aws:
        type: g5.2xlarge
        rootVolume:
          size: 400
        zones:
          - us-east-1a  # Adjust based on availability
    replicas: 1
controlPlane:
  name: master
  hyperthreading: Enabled
  platform:
    aws:
      type: m6i.2xlarge  # Single master (x86_64 for compatibility)
      rootVolume:
        size: 400
      zones:
        - us-east-1a  # Keep it in the same zone as workers
  replicas: 1  # Only 1 master node (Not HA but cost-efficient)
metadata:
  creationTimestamp: null
  name: ai-mint-ocp
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  aws:
    region: us-east-1
publish: External
pullSecret: <Pull Secrete>

```

## Sources

<https://faun.pub/basic-openshift-4-cluster-setup-on-aws-7b8ea694fd41>

<https://catalog.ngc.nvidia.com/orgs/nvidia/teams/aiworkflows/helm-charts/rag-app-multiturn-chatbot#deploying-nvidia-nim-microservices>

<https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/1.8/openshift/install-gpu-ocp.html>

<https://github.com/NVIDIA/nim-deploy/tree/main/helm>

# **Cluster Installation**

## **Pre-requisites**

**RedHat Developer Account:**

Go to <https://developers.redhat.com/> & create one account

**Personal Domain:**

We need one personal domain because we will be deploying OpenShift in the public cloud & it works on the concept of DNS Load Balancing. Plus, OpenShift will have its own managed certificates which need the personal domain to be self-signed.

**AWS Account:**

We need an AWS account, if you have an AWS IAM account, then that needs the Administrator Access

Note your **Access Key** and **Secret Key** details below.

**Setting up AWS Hosted Zone**

Go to Route 53 and create one Hosted Zone to allow AWS to use our Domain Name to point to the Backend OpenShift servers.

**Setup Bastion Host**

From AWS, launch one instance to drive the OpenShift setup. Let's choose one Amazon Linux 2 Instance with x86-64 architecture with at least 2 CPU Core & 2 GiB Memory.

## Setting up an OpenShift trial account

RedHat offers a 60-day trial account.

Go to — <https://console.redhat.com/> & select “OpenShift” —

- OpenShift Installer — It’s a command line tool that we will be using to deploy our OpenShift Cluster.
- Pull Secret: This file contains the credentials to pull the OpenShift Container Images from the Red Hat Private Registry.
- Command-line Interface: This will simply give you the “oc” command to interact with the OpenShift Cluster.

## Install the software in Bastion Host

1. sudo su -
2. mkdir ocp_software
3. cd ocp_software
4. Install The latest version using the link:-  
    [https://mirror.openshift.com/pub/openshift-v4/x86_64/](https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/4.16.9/)
5. wget <https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/4.17.9/openshift-client-linux-4.17.9.tar.gz>
6. wget <https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/4.17.9/openshift-install-linux.tar.gz>

Now simply extract the tar files & put them in “/usr/bin” folder so that we can use the commands from any directory…

1. tar xvf openshift-client-linux-4.17.9.tar.gz -C /usr/bin
2. tar xvf openshift-install-linux.tar.gz -C /usr/bin
3. openshift-install version
4. oc version

## OpenShift Cluster Installation

In your Bastion Host writer below command

openshift-install create install-config --dir=ocp_install

This will create the directory “ocp_install” first & then it will save the default configuration file in that folder. Now you will see it’s asking for some basic inputs which you need to provide like

- Platform: AWS
- Access Key & Secret Key
- Region: ap-south-1 (in my case)
- Base Domain: aimintopenshift.com (it will fetch the domain automatically from Route53 Hosted Zones once you give your access key, secret key & region)
- Cluster Name: (give whatever you like)
- Pull Secret: provide the pull secret copied from RedHat Cloud Console.

Change the install configuration with the below yaml
```
additionalTrustBundlePolicy: Proxyonly
apiVersion: v1
baseDomain: aimintopenshift.com
compute:
  - name: worker  # GPU worker nodes
    hyperthreading: Enabled
    platform:
      aws:
        type: g5.2xlarge
        rootVolume:
          size: 400
        zones:
          - us-east-1a  # Adjust based on availability
    replicas: 1
controlPlane:
  name: master
  hyperthreading: Enabled
  platform:
    aws:
      type: m6i.2xlarge  # Single master (x86_64 for compatibility)
      rootVolume:
        size: 400
      zones:
        - us-east-1a  # Keep it in the same zone as workers
  replicas: 1  # Only 1 master node (Not HA but cost-efficient)
metadata:
  creationTimestamp: null
  name: ai-mint-ocp
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  aws:
    region: us-east-1
publish: External
pullSecret: <Pull Secrete>

```
It will take one hour to install the openshift and create the resources

Go to your root directory & from there run the below command

**nohup openshift-install create cluster --dir=ocp_install --log-level=debug &**

You can use “watch” command to check if the command is going on or not…

**watch ‘ps -aux | grep openshift’**

If you go to your AWS account, you will see it launched four instances


##

# Post Installation setup

Login to your Bastion Host & check the “nohup.out” file & at the end of the log, you will find your credentials to log in to your Cluster using “kubeadmin” user.

- Your “kubeadmin” password is stored in “ocp_install/auth/kubeadmin-password” file
- The output shows that you can “export” the kubeconfig file in an environmental variable to connect to your cluster using the “oc” command. But this will work only for that console wherever you will do the export. A better approach can be just copying the content of “kubeconfig” file to the “.kube/config” file which is the default config file for the “oc” command to read the credentials.
- Now oc command is working fine. Let's log in to the OpenShift Console from the browser using the “kubeadmin” credentials.

# Installing the Node Feature Discovery (NFD) Operator

The Node Feature Discovery (NFD) Operator is a prerequisite for the NVIDIA GPU Operator. It can be installed from the Red Hat OperatorHub catalog in the OpenShift Container Platform web console.

1. Follow the Red Hat documentation guidance in [The Node Feature Discovery Operator](https://docs.openshift.com/container-platform/latest/hardware_enablement/psap-node-feature-discovery-operator.html) to install the Node Feature Discovery Operator.

## Verify the Node Feature Discovery Operator is running

**oc get pods -n openshift-nfd**

1. Once Node Feature Discovery is installed, create an instance of Node Feature Discovery through the **NodeFeatureDiscovery** tab.
2. Click **Operators** > **Installed Operators** from the side menu.
3. Find the **Node Feature Discovery** entry.
4. Click **NodeFeatureDiscovery** under the **Provided APIs** field.
5. Click **Create NodeFeatureDiscovery**.

## In the subsequent screen, click **Create**. This starts the Node Feature Discovery Operator that proceeds to label the nodes in the cluster that have GPUs

## **Verifying NFD Installation:**

The Node Feature Discovery Operator uses vendor PCI IDs to identify hardware in a node. NVIDIA uses the PCI ID 10de. Use the OpenShift Container Platform web console or the CLI to verify that the Node Feature Discovery Operator is functioning correctly.

1. In the OpenShift Container Platform web console, click **Compute** > **Nodes** from the side menu.
2. Select a worker node that you know contains a GPU.
3. Click the **Details** tab.
4. Under **Node labels** verify that the following label is present:  
    **feature.node.kubernetes.io/pci-10de.present=true**
5. Verify the GPU device (pci-10de) is discovered on the GPU node:  
    **oc describe node | egrep 'Roles|pci' | grep -v master**  
    **No**

# **Installing the NVIDIA GPU Operator**

- In the OpenShift Container Platform web console from the side menu, select **Operators** > **OperatorHub**, then search for the **NVIDIA GPU Operator**
- Select the **NVIDIA GPU Operator**, and click **Install**. In the subsequent screen click **Install**.
- Select the **ClusterPolicy** tab, then click **Create ClusterPolicy**. The platform assigns the default name _gpu-cluster-policy_.
- Click **Create**.

At this point, the GPU Operator proceeds and installs all the required components to set up the NVIDIA GPUs in the OpenShift 4 cluster. This may take a while so be patient and wait at least 10-20 minutes before digging deeper into any form of troubleshooting.

- The status of the newly deployed ClusterPolicy _gpu-cluster-policy_ for the NVIDIA GPU Operator changes to State:ready once the installation succeeded.

## **Verify the successful installation of the NVIDIA GPU Operator**

Run the following command to view these new pods and daemonsets:

- $ oc get pods,daemonset -n nvidia-gpu-operator-resources

## **Running a sample GPU Application**

Run the following:

```
cat << EOF | oc create -f -

apiVersion: v1
kind: Pod
metadata:
  name: cuda-vectoradd
spec:
 restartPolicy: OnFailure
 containers:
 - name: cuda-vectoradd
   image: "nvidia/samples:vectoradd-cuda11.2.1"
   resources:
     limits:
       nvidia.com/gpu: 1
EOF

```
- pod/cuda-vectoradd created

Check the logs of the container:

- oc logs cuda-vectoradd

## **Getting information on the GPU**

Change to the gpu-operator-resources project:

- oc project nvidia-gpu-operator-resources

Run the following command to view these new pods:

- oc get pod -owide -lapp=nvidia-driver-daemonset

Run the nvidia-smi command within the pod:

- oc exec -it nvidia-driver-daemonset-pbplc -- nvidia-smi  

# **RAG Application: Multiturn Chatbot**

## Deployment

Fetch the helm chart from NGC

helm fetch <https://helm.ngc.nvidia.com/nvidia/aiworkflows/charts/rag-app-multiturn-chatbot-24.08.tgz> --username='$oauthtoken' --password=&lt;YOUR API KEY&gt;

## Deploying NVIDIA NIM for LLMs

Setting up the environment

Set the NGC_API_KEY environment variable to your NGC API key, as shown in the following example

- export NGC_API_KEY="key from ngc"

Clone this repository and change directories into nim-deploy/helm. The following commands must be run from that directory

- <https://github.com/NVIDIA/nim-deploy.git>
- cd nim-deploy/helm

Setting up your helm values

- helm show values nim-llm/

Create a namespace

- kubectl create namespace nim

Launching a NIM with a minimal configuration

- helm --namespace nim install my-nim nim-llm/ --set model.ngcAPIKey=$NGC_API_KEY --set persistence.enabled=true

Using a custom values file

- kubectl -n nim create secret docker-registry registry-secret --docker-server=nvcr.io --docker-username='$oauthtoken' --docker-password=$NGC_API_KEY
- kubectl -n nim create secret generic ngc-api --from-literal=NGC_API_KEY=$NGC_API_KEY

NOTE: If you created these secrets in the past, the key inside the ngc-api secret has changed to NGC_API_KEY to be consistent with the variables used by NIMs. Please update your secret accordingly.
```
image:
  # Adjust to the actual location of the image and version you want
  repository: nvcr.io/nim/meta/llama3-8b-instruct
  tag: 1.0.0
imagePullSecrets:
  - name: registry-secret
model:
  name: meta/llama3-8b-instruct # not strictly necessary, but enables running "helm test" below
  ngcAPISecret: ngc-api
persistence:
  enabled: true
  annotations:
    helm.sh/resource-policy: keep
statefulSet:
    enabled: false
resources:
  limits:
    nvidia.com/gpu: 1

  resources:
  limits:
    nvidia.com/mig-4g.24gb: 1
```
Launching NIM in Kubernetes with a values file

- helm --namespace nim install my-nim nim-llm/ -f ./custom-values.yaml

Running inference

- kubectl get pods -n nim
- helm -n nim test my-nim --logs
- kubectl -n nim port-forward service/my-nim-nim-llm 8000:8000
```
curl -X 'POST' \\
'<http://localhost:8000/v1/chat/completions>' \\
\-H 'accept: application/json' \\
\-H 'Content-Type: application/json' \\
\-d '{
"messages": \[
{
"content": "You are a polite and respectful chatbot helping people plan a vacation.",
"role": "system"
},
{
"content": "What should I do for a 4 day vacation in Spain?",
"role": "user"
}
\],
"model": "meta/llama3-8b-instruct",
"max_tokens": 16,
"top_p": 1,
"n": 1,
"stream": false,
"stop": "\\n",
"frequency_penalty": 0.0
}'
```
## Deploying NVIDIA Nemo Retriever Embedding Microservice

Setup Environment

First, create your namespace and your secrets
```
NAMESPACE=nvidia-nims
DOCKER_CONFIG='{"auths":{"nvcr.io":{"username":"$oauthtoken", "password":"'${NGC_API_KEY}'" }}}'
echo -n $DOCKER_CONFIG | base64 -w0
NGC_REGISTRY_PASSWORD=$(echo -n $DOCKER_CONFIG | base64 -w0 )


kubectl create namespace ${NAMESPACE}
kubectl apply -n ${NAMESPACE} -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: nvcrimagepullsecret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ${NGC_REGISTRY_PASSWORD}
EOF
kubectl create -n ${NAMESPACE} secret generic ngc-api --from-literal=NGC_API_KEY=${NGC_API_KEY}

```
```
kubectl create -n ${NAMESPACE} secret generic ngc-api --from-literal=NGC_API_KEY=${NGC_API_KEY}
```
Install the chart
```
helm upgrade \\
\--install \\
\--username '$oauthtoken' \\
\--password "${NGC_API_KEY}" \\
\-n ${NAMESPACE} \\
\--set persistence.class="local-nfs" \\
text-embedding-nim \\
<https://helm.ngc.nvidia.com/nim/nvidia/charts/text-embedding-nim-1.2.0.tgz>
Set the Container Image
- \--set image.repository="nvcr.io/nim/nvidia/nv-embedqa-e5-v5" \\
- \--set image.tag="1.2.0" \\
```

## Deploying Milvus Vectorstore Helm Chart

Create a new nanespace for vectorstore

- kubectl create namespace vectorstore

Add the milvus repository

- helm repo add milvus <https://zilliztech.github.io/milvus-helm/>

Update the helm repository

- helm repo update

Create a file named custom_value.yaml with below content to utilize GPU's
```
standalone:
resources:
requests:
nvidia.com/gpu: "1"
limits:
nvidia.com/gpu: "1"
```
# Install the helm chart and point to the above created file using -f argument as shown below.

- helm install milvus milvus/milvus --set cluster.enabled=false --set etcd.replicaCount=1 --set minio.mode=standalone --set pulsar.enabled=false -f custom_value.yaml -n vectorstore

###Check the status of the pods

- kubectl get pods -n vectorstore

#### Configuring Prompts
\---
depth: 2
local: true
backlinks: none
\---