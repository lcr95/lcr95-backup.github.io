---
layout:     post
title:      "Kubernetes Dashboard"
subtitle:   "" 
date:       2021-02-22 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Kubernetes
---

Kubernetes has a dashboard web UI that provide an overview of applications running on your cluster, as well as for 
creating or modifying individual Kubernetes resources.

# Deploy Dashboard
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.5/aio/deploy/recommended.yaml
```


# Accessing Dashboard
Run the following command to create proxy for us to access the dashboard.

```bash
kubectl proxy

# Use this command to run proxy in background
kubectl proxy </dev/null &>/dev/null &
```

Access dashboard via the link below:
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

You should be getting similar interface as below:
{% include image.html src="post-dashboard-login-page.png" data="group" title="Login page" %}

Use the following command get token:
```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
```

# Configure Dashboard
We can configure the dashboard to have:
- token won't get expired
- option to skip login
- metric data update every 1 second 

```bash
kubectl patch deployments kubernetes-dashboard -n kubernetes-dashboard -p '
spec:
  template:
    spec:
      containers:
        - name: kubernetes-dashboard
          args:
            - '--auto-generate-certificates'
            - '--namespace=kubernetes-dashboard'
            - '--token-ttl=0'
            - '--enable-skip-login'
            - '--metric-client-check-period=1'
'
```

# Setup Metric Server
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.2/components.yaml

# Patch setting to disable TLS 
kubectl patch deployments metrics-server -n kube-system -p '
spec:
  template:
    spec:
      containers:
        - name: metrics-server
          image: 'k8s.gcr.io/metrics-server/metrics-server:v0.4.2'
          args:
            - '--cert-dir=/tmp'
            - '--secure-port=4443'
            - '--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname'
            - '--kubelet-use-node-status-port'
            - '--kubelet-insecure-tls'
'
```

# Uninstall Dashboard and Metric Sever
```bash
# uninstall dashboard
kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.5/aio/deploy/recommended.yaml

# uninstall metric server
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.2/components.yaml

```
