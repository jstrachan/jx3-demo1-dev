# Jenkins X Environment Repository

Install the [jx-git-operator](https://github.com/jenkins-x/jx-git-operator#installing)

Now [setup](https://github.com/jenkins-x/jx-git-operator#create-the-git-url-secret) the `Secret` to enable the [jx-git-operator](https://github.com/jenkins-x/jx-git-operator) to start triggering the boot jobs:
  
```bash 
kubectl create secret generic jx-boot --from-literal=url=https://$GIT_USERNAME:$GIT_TOKEN@github.com/jstrachan/env-git-operator-demo1.git
kubectl label secret jx-boot git-operator.jenkins.io/kind=git-operator
```



## Building locally

To populate the local source run

```bash 
make fetch build
```

To install it into a kubernetes cluster

```bash 
make apply
```    

## Rerun Kustomize and apply

Run this command to regenerate the resources using `kustomize`

```bash 
make build apply
```

Note that the first apply may not work so you may need to try again:

```bash 
make apply
```

## Enabling / disabling Kustomize

The use of [kustomize](https://kustomize.io/) is totally optional depending on your requirements.

To enable `kustomize` make sure that the line in the [Makefile](Makefile) is uncommented to enable it:

```make 
.PHONY: build
# uncomment this line to enable kustomize
build: build-kustomise
#build: build-nokustomise
```

to disable kustomize make sure `build` depends on `build-nokustomize` instead:
                                                                     
```make 
.PHONY: build
# uncomment this line to enable kustomize
#build: build-kustomise
build: build-nokustomise
```

## Setting up Vault

In one terminal start port forwarding:

```bash                                
jx ns vault-infra  
``` 

now you need to wait for the vault operator to startup along with the vault:

```bash 
kubectl wait --for=condition=Ready pod/vault-0
``` 

Once that pod is running you can [port forward vault](bin/vault-port-forward.sh):

```bash
./bin/vault-port-forward.sh                                
```

Now in another terminal you can access vault:

First setup your secrets as env vars...

```bash 
export PIPELINE_GIT_TOKEN="...."
export HMAC_TOKEN="....."
```

Now run the [bin/vault-populate.sh](bin/vault-populate.sh)  command...

```bash           
./bin/vault-populate.sh 
```    


## Local customizations

To get automate GitOps CI/CD you'll need to switch the git repository in the kustomize overlays:

* [resources/production/namespaces/jx/jxboot-helmfile-resources/environments.yaml](https://github.com/jstrachan/env-configsync-bootv3-scratch3/blob/master/resources/production/namespaces/jx/jxboot-helmfile-resources/environments.yaml#L9)
* [resources/production/namespaces/jx/jxboot-helmfile-resources/repositories.yaml](https://github.com/jstrachan/env-configsync-bootv3-scratch3/blob/master/resources/production/namespaces/jx/jxboot-helmfile-resources/repositories.yaml#L8-L9)


## Setting Ingress

The ingresses are all setup to `foo.cluster.local` by default. 

To set a different ingress use [jx-gitops ingress](https://github.com/jenkins-x/jx-gitops/blob/master/docs/cmd/jx-gitops_ingress.md) command in the `config-root/namespaces` directory which by default takes the value from the `jx-requirements.yml` in the `ingress.domain` settings.


