---
layout: post
title: "Seamless Integration: Writing Container Service Data Directly to Azure Blob Storage"
subtitle: ""
date: 2024-04-10
author: ChenRiang
header-style: text
tags:
  - Azure
  - Kubernetes
  - Blob
---
In today's cloud-driven landscape, optimizing data storage and management is paramount for developers seeking efficient and cost-effective solutions. **Azure Blob Storage** stands out as a reliable and scalable storage service, while container services offer agility and flexibility for deploying applications. Leveraging Container Storage Interface (CSI) drivers from Azure, developers can now seamlessly integrate Azure Blob Storage directly into container services like **Azure Kubernetes Service (AKS)**. This integration streamlines data management, eliminates complexity, and unlocks new possibilities for application development. In this blog post, we'll delve into the process of integrating Azure Blob Storage with container services using **Azure CSI Driver**.

### Azure Kubernetes Service (AKS)
[Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/) is a managed Kubernetes service provided by Microsoft Azure, designed to simplify the deployment, management, and scaling of containerized applications using Kubernetes.

### Azure Blob Storage 
[Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) is a cloud-based storage service provided by Microsoft Azure, designed to store vast amounts of unstructured data. It treats data as blobs, making it ideal for storing text or binary data, such as images, videos, documents, logs, and backups.

### Azure CSI Driver
[Azure CSI Driver](https://learn.microsoft.com/en-us/azure/aks/azure-blob-csi?tabs=NFS) is a Container Storage Interface (CSI) compliant driver provided by Azure, designed to facilitate seamless integration between AKS and Azure storage services.


### Integration Steps

#### 1. Enabling CSI Driver
To enable the driver on a new cluster:
```bash
az aks create --enable-blob-driver -n myAKSCluster -g myResourceGroup
```

To enable the driver on an existing cluster:
```bash
az aks update --enable-blob-driver -n myAKSCluster -g myResourceGroup
```

#### 2. Create Storage Class
Define a StorageClass resource (***my-azureblob-fuse-class***) in Kubernetes referencing Azure CSI Driver and specifying configuration parameters for provisioning Azure Blob Storage volumes.

a. Create the following yaml to a file (storageClass.yaml).

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-azureblob-fuse-class
provisioner: blob.csi.azure.com
parameters:
  skuName: Premium_LRS  # available values: Standard_LRS, Premium_LRS, Standard_GRS, Standard_RAGRS
  storageAccount: <storage account>
  resourceGroup: <resource group>
  isHnsEnabled: 'true'
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - -o allow_other
  - --file-cache-timeout-in-seconds=120
  - --use-attr-cache=true
  - --cancel-list-on-mount-seconds=10  # prevent billing charges on mounting
  - -o attr_timeout=120
  - -o entry_timeout=120
  - -o negative_timeout=120
  - --log-level=LOG_WARNING  # LOG_WARNING, LOG_INFO, LOG_DEBUG
  - --cache-size-mb=1000  # Default will be 80% of available memory, eviction will happen beyond that.
  - --use-adls=true
```

- `storageAccount` optional, remove this and the driver finds a suitable storage account that matches `skuName` in the same resource group.

- `isHnsEnabled: 'true'` and `--use-adls=true` is required to use Azure DataLake Gen2 storage account. 

-  see [here](https://learn.microsoft.com/en-us/azure/aks/azure-csi-blob-storage-provision?tabs=mount-nfs%2Csecret#storage-class-parameters-for-dynamic-persistentvolumes) for more configuration parameters.



b. Create the storage class with the following command:

```bash
kubectl apply -f storageClass.yaml
```
#### 3. Create PVC
Create PersistentVolumeClaim (PVC) resources (***task-pv-claim***) in Kubernetes to request persistent storage for your applications. Kubernetes will dynamically provision Azure Blob Storage volumes based on the StorageClass configuration (***my-azureblob-fuse-class***).

a. Create the following yaml to a file (pvc.yaml).
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: my-azureblob-fuse-class
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

b. Create the PVC with the following command:

```bash
kubectl apply -f pvc.yaml
```


#### 4. Mount Volume in pod
In this post, we will be using nginx as our application and mount the volume into the pod.

a. Create the following yaml into a file (pod.yaml).
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

b. Create the pod with following command:
```bash
kubectl apply -f pod.yaml
```

#### 5. Validation
Now that the pod is up and running, let's SSH into the pod and create a file (_**someSpecialFile123**_) into it.

{% include image.html src="ssh-validate.png" data="group" title="" %}

Next, you'll notice that the file will automatically be created in blob storage as well.

{% include image.html src="portal-validate.png" data="group" title="" %}


### Conclusion 
In this post, we've integrated AKS with Azure Blob Storage using Azure CSI Driver, streamlining storage management, simplifying provisioning, boosting scalability, and enhancing efficiency for developers in cloud-native environments. This solution enables efficient data management, driving innovation and efficiency in cloud-native environments.


**Reference**

1. Create and use a volume with Azure Blob storage in Azure Kubernetes Service (AKS) - [here](https://learn.microsoft.com/en-us/azure/aks/azure-csi-blob-storage-provision?tabs=mount-nfs%2Csecret)
2. Upgrading Azure Blob Storage with Azure Data Lake Storage Gen2 capabilities - [here](https://learn.microsoft.com/en-us/azure/storage/blobs/upgrade-to-data-lake-storage-gen2)