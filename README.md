<!-- ABOUT THE PROJECT -->
Contributrices : Hina Razanamasy and Alice Line
Pre-requis : vagrant 2.3.4 et virtualBox 6.1

Ce projet m’a permis de découvrir l’orchestrateur de container Kubernetes, à travers K3S, la version light de K8S, ainsi que K3d, permettant d’installer nos clusters dans Docker.

Ce projet est composé de 3 parties :
- Dans la première partie, il s’agit d’un simple set up de noeud master ainsi qu’un worker avec deux VMs (nous utilisons un Vagrantfile)
- Dans la deuxième partie, nous déployons 3 services dans un même namespace, exposé sur le port 80 du cluster. L'intérêt est l’utilisation d’un ingress Traeffik qui permet de rediriger le trafic selon le nom de l’hôte indiqué dans la requête (configuration dans ingress.yaml). Si aucun nom n’est indiqué alors l’ingress nous redirige par défaut sur la troisième application.
- La troisième partie nous a permis de découvrir le CICD via ArgoCD. Cette fois-ci, nous avons utilisé K3d. Nous indiquons à ArgoCD le repo distant auquel se connecter afin qu’il puisse comparer l’état de notre application courante avec son état sur gitHub, et ainsi, les synchroniser afin de redéployer l’application. (continuous deployment) 

- Bonus : CICD avec gitlab installé en local : sur une VM (voir vagrant file). Il est egalement possible d'installer gitLab dans le cluster avec Helm 

<!-- GETTING STARTED -->
Création de 3 app/3 services dans le même namespace exposé sur le même port 80, mais distinguées par son hôte, grâce à l’ingress Traeffik (point d’entrée du cluster)

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

Le service étant la couche au-dessus du pod composé d’une ip ET un port fixe afin de nous éviter de communiquer directement avec nos containers (les ip pouvant changer).

### Ingress.yaml
Traefik agit comme un point d'entrée et se charge de distribuer le trafic aux services appropriés en fonction des règles définies dans l'Ingress.

Les services sont dans le même namespace et sont exposés sur le même port (80), mais l'Ingress redirige le trafic en fonction de l'hôte, cela est possible grâce au fonctionnement de l'Ingress Controller et à la configuration de l'Ingress. Si l’on veux acceder au port 80 par default sans indiquer d’hôte, 

### Accéder à une app : 
Les VM sont lancés avec une adresse static privée (192.168.56.110/24) en host-only. La machine host (généralement attribuée à 192.168.56.1/24) peut alors s’y connecter. 



## Part 3
CICD avec ArgoCD et repos gitHub et K3D
K3D permet de créer des cluster kubernetes dans Docker. 

### installation : 
bash tools-installation.sh (installation docker engine, k3d, kubectl, argocd)
bash create_infrastructure.sh (cr2ation cluster k3d avec mapping du loadbalancer : 80 dans le cluster a 8081 de la machine, creation namespace argocd avec les ressources dont le fichier est en remote https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
Creation du mdp argo cd dans le fichier mdp_init :  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d > mdp_init
Port forward the argocd API to port 7894 :  kubectl port-forward service/argocd-server -n argocd 7894:443
bash start_infrastructure.sh
pull l’image will-playground : docker pull wil42/playground:v2
bash start_infrastructure.sh (deploiement de l’app, indication du github a argocd)
networking : 
accès à API argo cd : localhost:7894 ; (user = admin)
accès à l’app : localhost:8081
