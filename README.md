# kubernetes-minikube

Minikube is a tool that lets you run Kubernetes locally. 
minikube runs a single-node Kubernetes cluster on your personal computer (including Windows, macOS and Linux PCs) so that you can try out Kubernetes, or for daily development work.

## Docker installation

### installation for Mac, Windows 10 Pro, Enterprise, or Education

https://www.docker.com/get-started

Choose Docker Desktop

### installation for Windows home

https://docs.docker.com/docker-for-windows/install-windows-home/

## Kuberntes Minikube installation

https://minikube.sigs.k8s.io/docs/start/

Minikube provides a dashboard (web portal). Access the dashboard using the following command:

```
minikube dashboard
```

## Download this project

This project contains a web service coded in Java, but the language doesn't matter. This project has already been built and the binary version is there:

First of all, download and uncompress the project: https://github.com/charroux/kubernetes-minikube

You can also use git: `git clone https://github.com/charroux/kubernetes-minikube`

Then move to the sud directory with `cd kubernetes-minikube/myservice` where a DockerFile is.

## Test this project using Docker

Build the docker image:
```
docker build -t myservice .
```

Check the image:
```
docker images
```

Start the container:
```
docker run -p 4000:8080 -t myservice
```

8080 is the port of the web service, while 4000 is the port for accessing the container. Test the web service using a web browser: http://localhost:4000 It displays hello.

Ctrl-C to stop the Web Service.

Check the containerID:
```
docker ps
```

Stop the container:
```
docker stop containerID
```

## Publish the image to the Docker Hub

Retreive the image ID:
```
docker images
```

Tag the docker image: 
```
docker tag imageID yourDockerHubName/imageName:version
```

Example: `docker tag 1dsd512s0d myDockerID/myservice:1`

Login to docker hub: 
```
docker login
```
or
```
docker login http://hub.docker.com
```
or 
```
docker login -u username -p password
```

Push the image to the docker hub:
```
docker push yourDockerHubName/imageName:version
```

Example: `docker push myDockerID/myservice:1`

## Create a kubernetes deployment from a Docker image

```
kubectl get nodes
```
```
kubectl create deployment myservice --image=efrei/myservice:1
```

The image used comes from the Docker hub: https://hub.docker.com/r/efrei/myservice/tags

But you can use your own image instead.

Check the pod:
```
kubectl get pods
```

Check if the state is running.

Get complete logs for a pods: 
```
kubectl describe pods
```

Retreive the IP address but notice that this IP address is ephemeral since a pods can be deleted and replaced by a new one.

Then retrieve the deployment in the minikube dashboard. 
Actually the Docker container is runnung inside a Kubernetes pods (look at the pod in the dashboard).
  
You can also enter inside the container in a interactive mode with:
```
kubectl exec -it podname -- /bin/bash
```

where podname is the name of the pods obtained with:
```
kubectl get pods
```

List the containt of the container with:
```
ls
```

Don't forget to exit the container with:
```
exit
```

## Expose the Deployment through a service

A Kubernetes Service is an abstraction which defines a logical set of Pods running somewhere in the cluster, 
that all provide the same functionality. 
When created, each Service is assigned a unique IP address (also called clusterIP). 
This address is tied to the lifespan of the Service, and will not change while the Service is alive.

## Expose HTTP and HTTPS routes from outside the cluster to services within the cluster

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that’s outside of your cluster.

Kubernetes ServiceTypes allow you to specify what kind of Service you want. The default is ClusterIP.

Type values and their behaviors are:

* ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
* NodePort: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting NodeIP:NodePort.
* LoadBalancer: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
* ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record

## Expose HTTP and HTTPS route using NodePort

```
kubectl expose deployment myservice --type=NodePort --port=8080
```

Retrieve the service address:
```
minikube service myservice --url
```

This format of this address is `NodeIP:NodePort`.

Test this address inside your browser. It should display hello again.

Look from the NodeIP and the NodePort in the minikube dashboard.

## Scaling and load balancing

Check if the myservice deployment is running:

```
kubectl get deployments
```

How many instance are actually running:

```
kubectl get pods
```

Start a second instance:

```
kubectl scale --replicas=2 deployment/myservice
```
```
kubectl get deployments
```

and 

```
kubectl get pods
```

again

## Creating a Service of type LoadBalancer

Check if the myservice deployment is running:

```
kubectl get deployments
```

If a service is running in front of the deployment you must delete this service first in ordre to create a new one of kind LoadBalancer. So retreive the service using:

```
kubectl get services
```
And delete it:
```
kubectl delete service serviceName
```
```
kubectl expose deployment myservice --type=LoadBalancer --port=8080
```
```
minikube service myservice --url
```
Test in your web browser

## Rolling updates

Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones.

To update the image of the application to version 2, use the set image subcommand, followed by the deployment name and the new image version:
```
kubectl set image deployments/my-deployment my-deployment=dockerHudId/my-image:v2
```

You can also confirm the update by running the rollout status subcommand:
```
kubectl rollout status deployments/my-deployment
```

To roll back the deployment to your last working version, use the rollout undo subcommand:
```
kubectl rollout undo deployments/my-deployment
```

## Create a deployment and a service using a yaml file

Yaml files can be used instead of using the command `kubectl create deployment` and `kubectl expose deployment`

The yaml file for the deployment: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-deployment.yml

The yaml file for the node port service: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-service.yml

The yaml file for the node port service: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-loadbalancing-service.yml

Apply the deployment:
```
kubectl apply -f myservice-deployment.yml
```

Apply the node port service: 
```
kubectl apply -f myservice-service.yml
```

or 

Apply the service of type loadbalancer:
```
kubectl apply -f myservice-loadbalancing-service.yml
```
Then test if it works as expected.

# Routing rule to a service using Ingress

You can use Ingress to expose your Service. 
Ingress is not a Service type, but it acts as the entry point for your cluster. 
It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address.
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. 
An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting.

## Set up Ingress on Minikube with the NGINX Ingress Controller

Enable the NGINX Ingress controller: 

```
minikube addons enable ingress
```
Verify that the NGINX Ingress controller is running:
```
kubectl get pods -n ingress-nginx
```

Create a Deployment and expose it as a NodePort (not a loadbalancer).

Check if it works.

A yaml file for ingress: https://github.com/charroux/kubernetes-minikube/blob/main/ingress.yml

```
kubectl apply -f ingress.yml
```

Retrieve the IP address of Ingress: 

```
kubectl get ingress
```

```
NAME                 CLASS    HOSTS                  ADDRESS        PORTS   AGE

example-ingress      nginx   myservice.info         192.168.64.2   80      18m
```

On Linux: edit the `/etc/hosts` file and add at the bottom values for: 

IngressAddress myservice.info

Where address is given by:
```
minikube ip
```

On Mac: edit the `/etc/hosts` file and add at the bottom values for: 

127.0.0.1 myservice.info


Then check in your Web browser: 

http://myservice.info/

On Windows : edit the `c:\windows\system32\drivers\etc\hosts` file, add 

`127.0.0.1 myservice.info`	

Enable a tunnel for Minikube:

```
minikube addons enable ingress-dns
```
```
minikube tunnel
```

Then check in your Web browser: 

http://myservice.info/


Create a second deployment and its service, then add a new route to the ingress.yml file.

## Delete resources

```
kubectl delete services myservice
```
```
kubectl delete deployment myservice
```



---

# Annexe - Notes TP (ajout perso)

# kubernetes-minikube (TP prog distribue)

README de suivi du TP Kubernetes/Minikube, complete avec les commandes executees, les fichiers utilises et les problemes rencontres.

Ce depot contient une application Spring Boot (`MyService`) + des manifests Kubernetes pour la deployer sur Minikube.

## Objectif du TP

Valider les etapes suivantes:

- lancer un cluster local avec Minikube
- creer un Deployment Kubernetes a partir d'une image Docker
- exposer le service en `NodePort`
- tester le scaling (plusieurs replicas)
- exposer en `LoadBalancer`
- deployer via fichiers YAML (`kubectl apply -f`)
- configurer un `Ingress` et tester un hostname local (`myservice.info`)

## Arborescence utile

- `MyService/` : application Spring Boot + `Dockerfile`
- `myservice-deployment.yml` : Deployment Kubernetes
- `myservice-service.yml` : Service Kubernetes `NodePort`
- `myservice-loadbalancing-service.yml` : Service Kubernetes `LoadBalancer`
- `ingress.yml` : ressource Ingress (NGINX)

Important:

- le dossier de l'application est `MyService` (M majuscule), pas `myservice`

## Prerequis

- Docker Desktop installe et demarre
- `kubectl` installe
- `minikube` installe
- macOS (cas teste pendant le TP)

Verification rapide:

```bash
docker --version
kubectl version --client
minikube version
```

## Problemes rencontres (et solutions)

### 1) Minikube ne demarre pas (Docker daemon non lance)

Erreur rencontree:

```text
failed to connect to the docker API ... ~/.docker/run/docker.sock ... no such file or directory
```

Cause:

- Docker Desktop n'etait pas lance

Solution:

1. Ouvrir Docker Desktop
2. Attendre que le moteur Docker soit "running"
3. Relancer `minikube start --driver=docker`

### 2) Commandes lancees dans le mauvais dossier

Erreur rencontree:

```text
error: the path "myservice-deployment.yml" does not exist
```

Cause:

- commandes lancees depuis `.../TP` au lieu du depot `.../TP/kubernetes-minikube`

Solution:

```bash
cd "/Users/thomas/Documents/Université/Master/IAD/S2/Prog Distrib/TP/kubernetes-minikube"
```

### 3) `Ingress` applique trop tot (webhook refuse)

Erreur rencontree:

```text
failed calling webhook "validate.nginx.ingress.kubernetes.io" ... connection refused
```

Cause:

- le controleur NGINX Ingress n'etait pas encore pret juste apres `minikube addons enable ingress`

Solution:

Attendre que le deployment soit pret avant de reappliquer `ingress.yml`:

```bash
kubectl -n ingress-nginx rollout status deployment/ingress-nginx-controller --timeout=180s
kubectl apply -f ingress.yml
```

### 4) Notes macOS + driver Docker

- `minikube service myservice --url` peut demander de garder le terminal ouvert
- `minikube tunnel` doit rester lance dans un terminal separe pour certains acces (`LoadBalancer`, Ingress local)

## Etape 1 - Demarrer Minikube

```bash
minikube start --driver=docker
minikube status
kubectl get nodes
```

Resultat attendu:

- noeud `minikube` en `Ready`

## Etape 2 - Creer un Deployment depuis une image Docker (CLI)

Commande utilisee:

```bash
kubectl create deployment myservice --image=efrei/myservice:1
kubectl get deployments
kubectl get pods
kubectl get pods -w
```

Debug utile:

```bash
kubectl describe pods
kubectl logs deployment/myservice
```

Note:

- sur Mac Apple Silicon, l'image `efrei/myservice:1` peut prendre du temps a pull
- si probleme de compatibilite, utiliser l'image locale construite depuis `MyService/` (voir plus bas)

## Etape 3 - Exposer en NodePort (CLI)

```bash
kubectl expose deployment myservice --type=NodePort --port=8080
kubectl get services
minikube service myservice --url
```

Test:

- ouvrir l'URL retournee par `minikube service myservice --url`
- la reponse doit afficher `Hello from TP` (ou `hello` selon la version de l'image)

## Etape 4 - Scaling (2 replicas)

```bash
kubectl get deployments
kubectl get pods
kubectl scale --replicas=2 deployment/myservice
kubectl get deployments
kubectl get pods -w
```

Verification des endpoints du service:

```bash
kubectl get endpoints myservice
```

On doit voir 2 endpoints (`IP:8080`) quand les 2 pods sont `Running`.

## Etape 5 - Service de type LoadBalancer

Supprimer d'abord le service `NodePort` existant:

```bash
kubectl delete service myservice
```

Creer un `LoadBalancer`:

```bash
kubectl expose deployment myservice --type=LoadBalancer --port=8080
kubectl get services -w
```

Dans un second terminal, lancer le tunnel:

```bash
minikube tunnel
```

Puis tester:

```bash
minikube service myservice --url
```

## Etape 6 - Construire l'image locale depuis `MyService`

Le manifest `myservice-deployment.yml` a ete adapte pour utiliser une image locale:

```yaml
image: myservice:1
```

Construction de l'image:

```bash
cd "/Users/thomas/Documents/Université/Master/IAD/S2/Prog Distrib/TP/kubernetes-minikube/MyService"
docker build -t myservice:1 .
minikube image load myservice:1
cd ..
```

Verification optionnelle:

```bash
minikube image ls | grep myservice
```

## Etape 7 - Deployment + Service via YAML

Nettoyage (pour repartir proprement):

```bash
kubectl delete ingress example-ingress --ignore-not-found
kubectl delete service myservice --ignore-not-found
kubectl delete deployment myservice --ignore-not-found
```

Application des manifests:

```bash
kubectl apply -f myservice-deployment.yml
kubectl apply -f myservice-service.yml
kubectl get all
kubectl get pods -w
```

Quand le pod est `Running`, tester:

```bash
minikube service myservice --url
```

## Etape 8 - Ingress (NGINX) sur Minikube

### 8.1 Corrections appliquees dans les manifests

- `ingress.yml`: suppression d'un espace en debut de la ligne `apiVersion`
- `myservice-deployment.yml`: image remplacee par `myservice:1`

### 8.2 Activer l'addon Ingress

```bash
minikube addons enable ingress
kubectl -n ingress-nginx rollout status deployment/ingress-nginx-controller --timeout=180s
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

Le pod `ingress-nginx-controller` doit etre `1/1 Running`.

### 8.3 Appliquer l'Ingress

```bash
kubectl apply -f ingress.yml
kubectl get ingress
```

Exemple de resultat attendu:

```text
NAME              CLASS   HOSTS            ADDRESS        PORTS   AGE
example-ingress   nginx   myservice.info   192.168.49.2   80      ...
```

### 8.4 Ajouter l'entree hosts (macOS)

Ajouter dans `/etc/hosts`:

```text
127.0.0.1 myservice.info
```

Commande pratique (idempotente):

```bash
grep -qF "127.0.0.1 myservice.info" /etc/hosts || echo "127.0.0.1 myservice.info" | sudo tee -a /etc/hosts
```

### 8.5 Tester l'Ingress

Test direct via header `Host`:

```bash
curl -H "Host: myservice.info" http://127.0.0.1/
```

Test normal:

```bash
curl http://myservice.info
```

Resultat obtenu pendant le TP:

```text
Hello from TP
```

## Commandes utiles (debug / verification)

```bash
kubectl get all
kubectl get pods
kubectl get services
kubectl get ingress
kubectl logs deployment/myservice
kubectl describe pod <pod-name>
kubectl get events --sort-by=.lastTimestamp
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
minikube service myservice --url
minikube status
```

## Nettoyage des ressources

```bash
kubectl delete ingress example-ingress --ignore-not-found
kubectl delete service myservice --ignore-not-found
kubectl delete deployment myservice --ignore-not-found
```

## Bonus possibles (si demandes ensuite)

- Rolling update (`kubectl set image ...`, `kubectl rollout status`, `kubectl rollout undo`)
- Deuxieme deployment + deuxieme service
- Ajout d'une seconde route dans `ingress.yml`

