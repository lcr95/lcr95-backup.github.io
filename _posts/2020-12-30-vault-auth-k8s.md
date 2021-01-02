---
layout:     post
title:      "Vault Authorization - Kubernetes"
subtitle:   "" 
date:       2021-01-02 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Vault
    - Kubernetes
---

Vault provide an auth method to authenticate a client using Kubernetes Service Account Token. 
The token of a pod's service automatically mounted at `/var/run/secrets/kubernetes.io/<serviceaccount>/token` and Vault is configured with service account 
that have permission to access and verify the token. 

Followed by [previous post]({% post_url 2020-11-29-setup-vault-k8s-helm %}), in this article we will be using kubernetes to
authenticate our sample application.

{% include image.html src="post-vault-k8s-auth.png" data="group" title="Vault" %}

1. Springboot client application use it JWT token that mounted automatically to authenticate Vault.
2. Vault send the JWT token from client to Kubernetes Master API for authentication.
3. Vault Response back to client application
4. Springboot client application retrieve the secret from Vault. 


**Software Version:**<br/>
Vault - v1.5.4 <br/>
Kubernetes - v1.18.8

# Vault Setup 
**Assumption**:<br/>
- We assumed the vault provisioned and initialized with the steps in [Part 1]({% post_url 2020-11-29-setup-vault-k8s-helm %}).
- A service account `helm-vault` is created as part of the Helm Chart deployment.
- Root Token: `s.RjctGXkV4mhVaHIT8XuCILe1`

1. Enable kubernetes auth method. <br/>
```bash
kubectl exec helm-vault-0 -- sh -c "
    export VAULT_TOKEN='s.ZvuMI7pzUu2bpdjxmCz6LG9h'; \
    vault auth enable kubernetes"
```

2. Get service account name and token
```bash
K8S_TOKEN_NAME=$(kubectl get serviceaccounts/helm-vault -o jsonpath='{.secrets[0].name}')
K8S_TOKEN=$(kubectl get secret $K8S_TOKEN_NAME -o jsonpath='{.data.token}'| base64 --decode)
```

3. Get the internal Kubernetes CA from the pod that running Vault.
```bash
kubectl exec helm-vault-0 -- cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt >> ca.crt
```

4. Upload the CA to the Vault's pod.
```bash
kubectl cp ca.crt helm-vault-0:/vault
``` 

5. Configure the vault kubernetes auth method to use `helm-vault`'s token to verify other service account token.
```bash
kubectl exec helm-vault-0 -- sh -c "
    export VAULT_TOKEN='s.ZvuMI7pzUu2bpdjxmCz6LG9h'; \
    vault write auth/kubernetes/config \
    token_reviewer_jwt=$K8S_TOKEN \
    kubernetes_host=https://kubernetes.docker.internal:6443 \
    kubernetes_ca_cert=@/vault/ca.crt"
```
Note: <br/>
As we are using Window Docker Desktop, so `kubernetes_host` is set to `https://kubernetes.docker.internal:6443`. <br/><br/>
Use the following command to get your kubernetes cluster hostname: <br/> `kubectl config view --minify | grep server | cut -f 2- -d ":" | tr -d " "` 


6. Create a simple policy called `policy.hcl`.
    ```yaml
    path "secret/application" {
    
      capabilities = ["read", "list"]
    
    }
    
    path "secret/spring-native-example" {
    
        capabilities = ["read", "list"]
    
    }
    
    ```

7. Copy the policy to vault pod.
```bash
kubectl cp policy.hcl helm-vault-0:/vault
```

8. Create a policy called `test-policy`.
```bash
kubectl exec helm-vault-0 -- sh -c "
    export VAULT_TOKEN='s.ZvuMI7pzUu2bpdjxmCz6LG9h'; \
    vault policy write test-policy /vault/policy.hcl"
```

9. Create a role called `test`
```bash
kubectl exec helm-vault-0 -- sh -c "
	export VAULT_TOKEN='s.ZvuMI7pzUu2bpdjxmCz6LG9h'; \
	vault write auth/kubernetes/role/test \
	bound_service_account_names=default \
	bound_service_account_namespaces='*' \
	policies=test-policy ttl=24h"
```
This authorizes all the pod that running with `default` service account in all namespace under `test-policy` for 24 hours.

10. Verify the role creation.
```bash
    K8S_DEFAULT_TOKEN_NAME=$(kubectl get serviceaccounts/default -o jsonpath='{.secrets[0].name}')
    K8S_DEFAULT_TOKEN=$(kubectl get secret $K8S_TOKEN_NAME -o jsonpath='{.data.token}'| base64 --decode)
   
    kubectl exec helm-vault-0 -- sh -c "
        vault write auth/kubernetes/login \
        role=test \
        jwt=$K8S_DEFAULT_TOKEN"
```
<br/>Sample output as below:
    ```text
        Key                                       Value
        ---                                       -----
        token                                     s.ZSiZHKirpZtwqEfFPE0NWc4A
        token_accessor                            cwtXWr8CzfVRr51ZJOIpdiZv
        token_duration                            24h
        token_renewable                           true
        token_policies                            ["default" "test-policy"]
        identity_policies                         []
        policies                                  ["default" "test-policy"]
        token_meta_service_account_name           default
        token_meta_service_account_namespace      default
        token_meta_service_account_secret_name    default-token-gzpwk
        token_meta_service_account_uid            44d97ec5-27aa-41c7-8b02-07e9a87d8ba4
        token_meta_role                           test
    ```
11. Write a secert KV into Vault for testing purpose with `root` user.
```bash
kubectl exec helm-vault-0 -- sh -c "
	export VAULT_TOKEN='s.ZvuMI7pzUu2bpdjxmCz6LG9h';
	vault write secret/spring-native-example \
	password=pwd"
```

12. Validate it by reading the KV secret we wrote using the token(`s.ZSiZHKirpZtwqEfFPE0NWc4A`) we obtained from step 10.
```bash
kubectl exec helm-vault-0 -- sh -c "
	export VAULT_TOKEN='s.ZSiZHKirpZtwqEfFPE0NWc4A';
	vault read secret/spring-native-example"
```
You should be getting similar response as below:
```text
    Key                 Value
    ---                 -----
    refresh_interval    768h
    password            pwd
```

# Client Application
In this example we will be using a simple Java Springboot application to test the authorization.

## Truststore Setup
As we enabled tls for [our vault]({% post_url 2020-11-29-setup-vault-k8s-helm %}#configure-helm). 
Thus, we need to create a truststore for our application to communicate with Vault.

1. Create a truststore with a dummy entry inside.
```bash
keytool -genkey -keyalg RSA -alias dummy -keystore vault-truststore.jks
```

2. Remove the dummy entry
```bash
keytool -delete -alias dummy -keystore vault-truststore.jks
```

3. Import the certificate pem file `helm-vault.pem` that we [created]({% post_url 2020-11-29-setup-vault-k8s-helm %}#ssl-certificate-and-key) into `vault-truststore.jks`
```bash
keytool -import -v -trustcacerts -alias vault \
    -file helm-vault.pem \
    -keystore vault-truststore.jks
```

4. Create a Kubernetes secret for the truststore.
```bash
kubectl create secret generic spring-truststore \
    --from-file vault-truststore.jks
```

## Application Setup
Source code: [Github](https://github.com/lcr95/vault-k8s-test)

1. A class `VaultTestApplication` is created as below :
```java
    @SpringBootApplication
    @EnableConfigurationProperties
    public class VaultTestApplication implements CommandLineRunner {
    
        @Autowired
        VaultTemplate vaultTemplate;
    
        public static void main(String[] args) {
            SpringApplication.run(VaultTestApplication.class, args);
        }
    
        @Override
        public void run(String... strings) throws Exception {
            VaultResponse response = vaultTemplate.read("secret/spring-native-example");
            System.out.println(response.getData().get("password"));
        }
    }
```

2. Configure `bootstrap.yaml` as below:
```yaml
    spring.cloud.vault:
          generic:
              enabled: false
          enabled: true
          host: helm-vault
          port: 8200
          scheme: https
          authentication: KUBERNETES
          kubernetes:
            role: test
            service-account-token-file: /var/run/secrets/kubernetes.io/serviceaccount/token
          ssl:
            trust-store: vault-truststore.jks
            trust-store-password: secret
```

Note: This properties file was configured in [`spring-app.yaml`](https://github.com/lcr95/vault-k8s-test/blob/master/spring-app.yaml).

3. Apply the kubernetes deploymant file ([`spring-app.yaml`](https://github.com/lcr95/vault-k8s-test/blob/master/spring-app.yaml)).
```bash
kubectl apply -f spring-app.yaml 
```

4. Observe the `spring-app` pod's log. You will noticed the pod is authorized and able to print out the KV secret(`pwd`) we wrote earlier on.
    ```text
        2021-01-02 02:14:44.612  INFO 1 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService
          .   ____          _            __ _ _
         /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
        ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
         \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
          '  |____| .__|_| |_|_| |_\__, | / / / /
         =========|_|==============|___/=/_/_/_/
         :: Spring Boot ::        (v2.3.0.RELEASE)
        2021-01-02 02:14:44.748  INFO 1 --- [           main] com.vault.test.VaultTestApplication      : No active profile set, falling back to default profiles: default
        2021-01-02 02:14:45.166  INFO 1 --- [           main] o.s.cloud.context.scope.GenericScope     : BeanFactory id=fe77e05d-a150-3bc1-82d1-68049983f203
        2021-01-02 02:14:45.400  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
        2021-01-02 02:14:45.409  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
        2021-01-02 02:14:45.410  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.35]
        2021-01-02 02:14:45.477  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
        2021-01-02 02:14:45.477  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 712 ms
        2021-01-02 02:14:45.646  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
        2021-01-02 02:14:45.824  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
        2021-01-02 02:14:45.831  INFO 1 --- [           main] com.vault.test.VaultTestApplication      : Started VaultTestApplication in 2.132 seconds (JVM running for 2.385)
        2021-01-02 02:14:46.281  INFO 1 --- [           main] o.s.v.a.LifecycleAwareSessionManager     : Scheduling Token renewal
        pwd
    
    ```