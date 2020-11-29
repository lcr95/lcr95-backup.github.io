---
layout:     post
title:      "Setup Vault with Helm"
subtitle:   ""
date:       2020-11-29 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Vault
    - Helm
    - Kubernetes
---

Secret management is always a serious issue in any kind of application. Although, Kubernetes has its own Secret feature
could help to manage secret, but after some reading I realized that it basically encode the secret with Base64.

So, I looked into **HashiCorp Vault**.

#  HashiCorp Vault

{% include image.html src="post-vault.png" data="group" title="Vault" %}

It is an open source secret management tool that used to manage access to sensitive credential. 

Refer [here](https://www.hashicorp.com/products/vault) for more information.


# Vault Setup
HashiCorp have provided a ready [Helm Chart](https://github.com/hashicorp/vault-helm) that we can use.

 1. Create the following files in our demo helm chart directory :
    - Chart.yaml
      
        ```yaml
       apiVersion: v1
       name: helm-vault
       version: 0.1.0
       appVersion: 1.0
       description: Helm Vault
       ```
      
    - requirements.yaml
         
      ```yaml     
         dependencies:
         - name: vault
           version: 0.8.0
           repository: "https://helm.releases.hashicorp.com"
           condition: vault.enabled
           alias: vault
      ```
    - values.yaml <br/>
      ** *We will come back to the value later.*

# Dev Mode
Vault provide an option to start Vault server in `dev` mode which does not require any further setup.  <br/>

**Note: Never, ever, ever run a "dev" mode server in production**
 
 In this section we will setup Vault in dev mode and validate it by running a simple KV store and read action via API.
 
 1. Edit `values.yaml` with the following value.
     ```yaml
     vault:
       server:
         dev:
           enabled: true
         readinessProbe:
           enabled: false
     ```
    - Set the dev.enabled to `true`.
    - Set the readinessProbe.enabled to `false` as the probe check will query the status with https which the Vault
     server currently does not support HTTPS scheme
  <br/><br/> 
     
 2. Helm install the Vault.<br/> `helm install helm-vault`
 <br/><br/> 
 
 3. Once the pod is ready, we will need to port forward the vault service to localhost to play around with Vault.<br/> 
    `kubectl port-forward helm-vault 8200:8200`
 <br/><br/> 
 
 4. Execute following command to store a KV into vault.
    
    ```shell
    curl --location --request POST 'http://localhost:8200/v1/secret/data/my-secret' \
    --header 'X-Vault-Token:root' \
    --header 'Content-Type: application/json' \
    --data '{
      "options": {
        "cas": 0
      },
      "data": {
        "secretKey": "secretValue"
      }
    }'
    ```
    - We will store the KV at `my-secret`.
    - A token `root` *(Dev Mode default token)* is set at the header. 
 <br/><br/>    
    
 5. Execute the following command to read the KV from the path we set in step4.
    
    ```shell
    curl --location --request GET http://localhost:8200/v1/secret/data/my-secret?version=1 \
    --header "X-Vault-Token: root" 
    ```
    
    Sample KV Read response:
    ```json
    {
        "request_id": "c7305d4f-29b3-e097-a837-1149cf95e3c5",
        "lease_id": "",
        "renewable": false,
        "lease_duration": 0,
        "data": {
            "data": {
                "foo": "bar",
                "zip": "zap"
            },
            "metadata": {
                "created_time": "2020-11-29T04:19:17.5690113Z",
                "deletion_time": "",
                "destroyed": false,
                "version": 1
            }
        },
        "wrap_info": null,
        "warnings": null,
        "auth": null
    }
    ```
    
# Standard mode
In this section, we will setup the vault with TSL. 
## SSL Certificate and Key
We will use [*cfssl*](https://github.com/cloudflare/cfssl) to generate SSL certificate and key for our vault to use. 

1. Create a directory named `ssl`, and download the required software.

    ```shell
    mkdir ssl
    cd ssl
   
    curl -L https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssl_1.4.1_linux_amd64 -o cfssl
    chmod +x cfssl
    curl -L https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssljson_1.4.1_linux_amd64 -o cfssljson
    chmod +x cfssljson
    curl -L https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssl-certinfo_1.4.1_linux_amd64 -o cfssl-certinfo
    chmod +x cfssl-certinfo
    ```

2. Create CA CSR, Configuration, Vualt Certificate CSR.

    ```shell 
    echo '''
    {
      "hosts": [
        "cluster.local"
      ],
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "MY",
          "L": "KualaLumpur",
          "O": "Example",
          "OU": "CA",
          "ST": "Example"
        }
      ]
    }
    '''> ca-csr.json
    
    echo '''
    
    {
      "signing": {
        "default": {
          "expiry": "8760h"
        },
        "profiles": {
          "default": {
            "usages": ["signing", "key encipherment", "server auth", "client auth"],
            "expiry": "8760h"
          }
        }
      }
    }
    '''> ca-config.json
    
    
    echo '''
    {
      "CN": "helm-vault",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "MY",
          "L": "KualaLumpur",
          "O": "Kubernetes",
          "OU": "Vault",
          "ST": ""
        }
      ]
    }
    '''> vault-csr.json
    ```

 3. Create CA cert and key.
    
    ```shell 
    cfssl gencert -initca ca-csr.json | cfssljson -bare ca
    ```

 4. Generate cert and private key for Vault.
 
     ```shell 
    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -hostname="helm-vault,helm-vault.playground.svc.cluster.local,helm-vault.playground.svc,localhost,127.0.0.1" \
      -profile=default \
      vault-csr.json | cfssljson -bare helm-vault
    ```
    Note: `-hostname` need to follow according the kubernetes service name and namespace. <br/>
    In my case, 
     - kubernetes service name = `helm-vault`
     - kubernetes namespace = `playground`
 
 5. At this point, you should have the below 6 files created :
    - `ca.pem`
    - `ca-key.pem`
    - `ca.csr`
    - `helm-vault.pem`
    - `helm-vault-key.pem`
    - `helm-vault.csr`
 
 6. Remove cfssl software.
     
    ```shell
    rm cfssl cfssl-certinfo cfssljson
    ``` 

## Configure Helm 
 1. Create a directory named `templates`.
 
 2. In directory `templates`, create a kubernetes secret, `vault-secret.yaml` :
    
    ```yaml
    {% raw %}
    apiVersion: v1
    kind: Secret
    metadata:
      name: vault-secret
    data:
      helm-vault.pem: |-
        {{ .Files.Get "ssl/helm-vault.pem" | b64enc }}
      helm-vault-key.pem: |-
        {{ .Files.Get "ssl/helm-vault-key.pem" | b64enc }}
      ca.pem: |-
        {{ .Files.Get "ssl/ca.pem" | b64enc }}
    {% endraw %}
    ``` 
    
 3. Configure `values.yaml` as below:
    
    ```shell
    vault:
      global:
        tlsDisable: false
      server:
        dev:
          enabled: false
        standalone:
          config: |
            ui = true
    
            listener "tcp" {
              tls_disable = 0
              address = "[::]:8200"
              cluster_address = "[::]:8201"
              tls_cert_file = "/vault/userconfig/vault-secret/helm-vault.pem"
              tls_key_file  = "/vault/userconfig/vault-secret/helm-vault-key.pem"
            }
            storage "file" {
              path = "/vault/data"
            }
        extraEnvironmentVars:
          VAULT_CACERT: /vault/userconfig/vault-secret/ca.pem
        extraVolumes:
        - type: secret
          name: vault-secret
    ```
    
    
## Run Vault
 1. Helm install the Vault.<br/> `helm install helm-vault`
  <br/><br/> 
 
 2. Unseal vault with following command <br> `kubectl exec pod/helm-vault-0 -- vault operator init -key-shares=1 -key-threshold=1`
 <br/>
 You will be getting an unseal key and token by executing the command. 
    ```text
    Unseal Key 1: 1+Pwsc6ZonA3nK51UVrB4rDqkMtTqtxyPbHT4StGkPA=
    
    Initial Root Token: s.0WGR21db8KpPuBL69zowkLjl
    
    Vault initialized with 1 key shares and a key threshold of 1. Please securely
    distribute the key shares printed above. When the Vault is re-sealed,
    restarted, or stopped, you must supply at least 1 of these keys to unseal it
    before it can start servicing requests.
    
    Vault does not store the generated master key. Without at least 1 key to
    reconstruct the master key, Vault will remain permanently sealed!
    
    It is possible to generate new unseal keys, provided you have a quorum of
    existing unseal keys shares. See "vault operator rekey" for more information.
 
    ```
    
    - Unseal Key : `1+Pwsc6ZonA3nK51UVrB4rDqkMtTqtxyPbHT4StGkPA=`
    - Root Token : `s.0WGR21db8KpPuBL69zowkLjl`
  <br/><br/> 
  
 3. Unseal Vault with the key obtained.
    ```shell 
    kubectl exec pod/helm-vault-0 -- vault operator unseal 1+Pwsc6ZonA3nK51UVrB4rDqkMtTqtxyPbHT4StGkPA=
    ```
  <br/>
  
 4. Execute the command below to enable KV store feature.<br/>
 
    ``` shell
    kubectl exec pod/helm-vault-0 -- sh -c 'export VAULT_TOKEN="s.0WGR21db8KpPuBL69zowkLjl"  && vault secrets enable -version=1 -path secret kv'
    ```
  <br/> 
     
 5. Vault is ready now.
  <br/><br/> 
   
 6. Execute following command to store a KV into vault.
    
    ```shell
    curl --location --request POST 'https://localhost:8200/v1/secret/data/my-secret' \
    --cacert helm-vault.pem \
    --header 'X-Vault-Token:s.0WGR21db8KpPuBL69zowkLjl' \
    --header 'Content-Type: application/json' \
    --data '{
      "options": {
        "cas": 0
      },
      "data": {
        "secretKey": "secretValue"
      }
    }'

    ```
    - We will store the KV at `my-secret`.
    - A token `s.0WGR21db8KpPuBL69zowkLjl` is set at the header.
    - `helm-vault.pem` is used for TSL.
 <br/><br/>    
    
 7. Execute the following command to read the KV from the path we set in step6.
    
    ```shell
    curl --location --request GET https://localhost:8200/v1/secret/data/my-secret?version=1 \
    --cacert helm-vault.pem \
    --header "X-Vault-Token: s.0WGR21db8KpPuBL69zowkLjl" 
    ```
    
    Sample KV Read response:
    ```json
    {
        "request_id": "c9af2608-ced3-492d-f1a8-e41b2d121709",
        "lease_id": "",
        "renewable": false,
        "lease_duration": 2764800,
        "data": {
            "data": {
                "secretKey": "secretValue"
            },
            "options": {
                "cas": 0
            }
        },
        "wrap_info": null,
        "warnings": null,
        "auth": null
    }
    ```
 
 
# Reference 
Vault KV API : 
[Documentation](https://www.vaultproject.io/api/secret/kv/kv-v2)

SSL Certificate Generation: 
[Ref1](https://github.com/marcel-dempers/docker-development-youtube-series/blob/master/hashicorp/vault/tls/ssl_generate_self_signed.txt) |
[Ref2](https://kubernetes.io/docs/concepts/cluster-administration/certificates/)