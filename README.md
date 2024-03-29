<h3 align="center">Inception of things</h3>

<!-- ABOUT THE PROJECT -->
#### Contributrices : Hina Razanamasy and Alice Line
#### Pre-requis : vagrant 2.3.4 et virtualBox 6.1

Ce projet nous a permis de découvrir l’orchestrateur de container Kubernetes, à travers K3S, la version light de K8S, ainsi que K3d, permettant d’installer nos clusters dans Docker.

Ce projet est composé de 3 parties :
- Dans la première partie, il s’agit d’un simple set up de noeud master ainsi qu’un worker avec deux VMs (nous utilisons un Vagrantfile)
- Dans la deuxième partie, nous déployons 3 services dans un même namespace, exposé sur le port 80 du cluster. L'intérêt est l’utilisation d’un ingress Traeffik qui permet de rediriger le trafic selon le nom de l’hôte indiqué dans la requête (configuration dans ingress.yaml). Si aucun nom n’est indiqué alors l’ingress nous redirige par défaut sur la troisième application.
- La troisième partie nous a permis de découvrir le CICD via ArgoCD. Cette fois-ci, nous avons utilisé K3d. Nous indiquons à ArgoCD le repo distant auquel se connecter afin qu’il puisse comparer l’état de notre application courante avec son état sur gitHub, et ainsi, les synchroniser afin de redéployer l’application. (continuous deployment) 

- Bonus : CICD avec gitlab installé en local : sur une VM (voir vagrant file). Il est egalement possible d'installer gitLab dans le cluster avec Helm 

<!-- GETTING STARTED -->

## Part 1
### Installation
vagrant up

## Part 2

### Installation
vagrant up

### config map
Des fichiers séparés html pour chaque application sont rangés dans notre dossier git. Nous créons un ConfigMap (qui permet de mettre à disposition à un namespace précis, des fichiers) par application. Chaque ConfigMap nous servira de volume pour chaque container.

### app.yaml
chaque app aura : 
son pod, définit dans le deployment, ainsi que son service rattaché.
C’est ici que le bon config-map sera appelé. 
Par exemple le ConfigMap "app-one" est monté en tant que volume avec le nom "app-one-volume" et est monté dans le répertoire "/usr/share/nginx/html" du conteneur Nginx. Nginx va donc utiliser le bon fichier html selon l’app.

rappel : Le service est la couche au-dessus du pod composé d’une ip ET un port fixe afin de nous éviter de communiquer directement avec nos containers (les ip pouvant changer).

### Ingress.yaml
Traefik agit comme un point d'entrée et se charge de distribuer le trafic aux services appropriés en fonction des règles définies dans l'Ingress.

Les services sont dans le même namespace et sont exposés sur le même port (80), mais l'Ingress redirige le trafic en fonction de l'hôte, cela est possible grâce au fonctionnement de l'Ingress Controller et à la configuration de dans ingress.yaml.

### Accéder à une app : 
#### Depuis le browser de l'hôte
La VM est lancée avec une adresse static privée (192.168.56.110/24) en host-only. La machine host (généralement attribuée à 192.168.56.1/24) peut alors s’y connecter (bien verifier que le firewall de la guest accepte la connexion depuis la machine hote)

#### Avec curl 
curl -H "Host:app2.com" 192.168.56.110 (si host) ou localhost (si guest)



## Part 3
CICD avec ArgoCD et repos gitHub et K3D
K3D permet de créer des cluster kubernetes dans Docker. 

### installation : 
##### bash tools-installation.sh (installation docker engine, k3d, kubectl, argocd)
##### bash create_infrastructure.sh (cr2ation cluster k3d avec mapping du loadbalancer : 80 dans le cluster a 8081 de la machine, creation namespace argocd avec les ressources dont le fichier est en remote https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
##### Creation du mdp argo cd dans le fichier mdp_init :  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d > mdp_init
##### Port forward the argocd API to port 7894 :  kubectl port-forward service/argocd-server -n argocd 7894:443
##### bash start_infrastructure.sh
##### pull l’image will-playground : docker pull wil42/playground:v2
##### bash start_infrastructure.sh (deploiement de l’app, indication du github a argocd)

#### networking : 
accès à API argo cd : localhost:7894 ; (user = admin)
accès à l’app : localhost:8081

## Documentation
### general kubernetes
#### https://www.free-work.com/fr/tech-it/blog/actualites-informatiques/comprendre-kubernetes-conseils-et-outils-indispensables
#### https://www.youtube.com/watch?v=VnvRFRk_51k
#### https://kubernetes.io/docs/concepts/overview/
#### https://geekflare.com/fr/kubernetes-architecture/
#### https://www.youtube.com/watch?v=vFfngcRPj9M

### K3S and K3D 
#### https://www.youtube.com/watch?v=f10VP3pXbsI&list=PLn6POgpklwWqfzaosSgX2XEKpse5VY2v5&index=55
#### https://thechief.io/c/editorial/k3s-vs-k3d/#:~:text=k3d%20is%20a%20wrapper%20of,prompt%20support%20for%20multiple%20clusters

### ARGO CD
#### https://codefresh.io/learn/argo-cd/#:~:text=Argo%20CD%20is%20a%20Kubernetes%20controller%2C%20responsible%20for%20continuously%20monitoring,specified%20in%20the%20Git%20repository
#### https://argo-cd.readthedocs.io/en/stable/cli_installation/ 
#### https://yashguptaa.medium.com/application-deploy-to-kubernetes-with-argo-cd-and-k3d-8e29cf4f83ee
#### https://argo-cd.readthedocs.io/en/stable/getting_started/?_gl=1*xqdimc*_ga*MTU0MDA0NjYxLjE2NzUxNjQzNzM.*_ga_5Z1VTPDL73*MTY3NTE2NDM3My4xLjAuMTY3NTE2NDM3NS4wLjAuMA
