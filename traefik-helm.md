## Install Traefik service from helm

```bash
# Create Traefik namespace
kubectl create ns traefik

# add helm repo 
helm repo add traefik https://traefik.github.io/charts

# update local helm repo cache
helm repo update
```
Output:
> ...Successfully got an update from the "traefik" chart repository. Update Complete. ⎈Happy Helming!⎈

```bash
# search for latest traefik chart version

helm search repo traefik
```

Output:  

| NAME                 	| CHART VERSION 	| APP VERSION 	| DESCRIPTION                                        	|
|----------------------	|---------------	|-------------	|----------------------------------------------------	|
| traefik/traefik      	| 20.8.0        	| v2.9.6      	| A Traefik based Kubernetes ingress controller      	|
| traefik/traefik-mesh 	| 4.1.1         	| v1.4.8      	| Traefik Mesh - Simpler Service Mesh                	|
| traefik/traefikee    	| 1.7.0         	| v2.9.1      	| Traefik Enterprise is a unified cloud-native ne... 	|
| traefik/hub-agent    	| 1.2.2         	| v1.1.0      	| Traefik Hub is an all-in-one global networking ... 	|
| traefik/maesh        	| 2.1.2         	| v1.3.2      	| Maesh - Simpler Service Mesh                       	|              

```bash
# install the traefik helm chart

helm install --namespace=traefik \
    traefik traefik/traefik
```

If u deal all actions successfully, u can see next message:

```plane
NAME: traefik
LAST DEPLOYED: Thu Feb  2 12:17:59 2023
NAMESPACE: traefik
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Traefik Proxy v2.9.6 has been deployed successfully
on traefik namespace !
```

If u see errors, u can remove helm chart and re-install them.