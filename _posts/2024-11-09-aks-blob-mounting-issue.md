---
layout: post
title: Azure AKS and Blob Storage Volume Mounting Issue
subtitle: failed to initialize new pipeline [failed to authenticate credentials for azstorage]
date: 2024-11-09
author: ChenRiang
header-style: text
catalog: false
tags:
  - Azure
  - Blob
  - Kubernetes
---

A few months ago, I encountered a problem while setting up an Azure Kubernetes Service (AKS) pod with Azure Blob Storage. It started off as a straightforward setup, but one of the pods kept restarting with the following error message:

> “Unknown desc = exit status 1 Error: failed to initialize new pipeline [failed to authenticate credentials for azstorage]”

I checked the usual culprits: role settings, managed identity setup, firewall configurations, and the node agent pod’s logs. No luck—everything seemed to be configured correctly.

After exhausting the usual troubleshooting steps without success, I turned to blob-fuse's GitHub issues for additional insights. I came across a [suggestion](https://github.com/Azure/azure-storage-fuse/issues/1376#issuecomment-2029211757) to examine the blobfuse2.log file on the AKS node (`/var/log/blobfuse2.log`). 


In blobfuse2.log, I found this error message:

> LOG_ERR [azstorage.go (161)]: AzStorage::configureAndTest : Failed to validate credentials [GET
> 
> https://<blob>.dfs.core.windows.net/ttb#012--------------------------------------------------------------------------------#012RESPONSE
> 
>  409: 409 This endpoint does not support BlobStorageEvents or SoftDelete. Please disable these account features if you would like to use this endpoint.#012ERROR CODE: EndpointUnsupportedAccountFeatures#012--------------------------------------------------------------------------------#012{#012  "error": {#012    "code": "EndpointUnsupportedAccountFeatures",#012    "message": "This endpoint does not support BlobStorageEvents or SoftDelete. Please disable these account features if you would like to use this endpoint.\nRequestId:xxx\nTime:xxx"#012  }#012}#012--------------------------------------------------------------------------------#012]
> 
> Sep 27 09:34:50 aks-runtime-xxx-vmss000001 blobfuse2[205703]: [/var/lib/kubelet/plugins/kubernetes.io/csi/blob.csi.azure.com/xxx/globalmount] LOG_ERR [azstorage.go (101)]: AzStorage::Configure : Failed to validate storage account [failed to authenticate credentials for azstorage]
>  

So, what was the real issue? As it turns out, **the SoftDelete feature on my storage account was enabled**, which this specific setup didn’t support. After disabling it, my pod finally started up successfully.

If you’re facing similar Blob Storage mounting issues in AKS, try checking the logs directly on the AKS node. Node-level logs (like blobfuse2.log) reveal specific details that may not show up in pod or agent logs, offering critical insights to help pinpoint the root cause.


