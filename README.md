# kubernetes-devops-project

If you come from the Kubernetes training, you're in the right place ! :) 

Run the application :

docker-compose up -d


mettre a jour paquet apt-get update #installer gnome sudo apt install gnome-terminal #installer Docker sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

#Ajoutez la clé GPG de Docker : curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - #Ajoutez le dépôt Docker : sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" #Mettez à jour à nouveau la liste des paquets : sudo apt-get update #Installez Docker: Vous le pouvez maintenant installer Docker avec la commande suivante : sudo apt-get install docker-ce #Installer kubernetes curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
creer le projet dans service (srv)

cd /srv/projet-kubernetes
Créer les dossiers principaux

mkdir -p manifests/deployments mkdir -p manifests/services mkdir -p manifests/ingress mkdir -p configs/nginx mkdir -p configs/db mkdir scripts mkdir volumes
Créer un fichier README.md

touch README.md
Créer un fichier .gitignore

touch .gitignore

docker image prune #pour supprimer images non utilisees par un conteneur docker image prune

kubectl delete deployments --all -n default 
kubectl apply -f "./*.yaml"
 kubectl delete ingress --all -n default 
 kubectl delete svc --all -n default 
kubectl delete pvc --all -n default 
kubectl delete -f "./*.yaml"




#.1 Appliquer le Namespace kubectl apply -f YAML-STANDARD/namespace.yaml

#.2 Appliquer les Secrets kubectl apply -f YAML-STANDARD/secret-postgres.yaml #.3 Appliquer les Déploiements kubectl apply -f YAML-STANDARD/deployments/deployment-postgres.yaml kubectl apply -f YAML-STANDARD/deployments/deployment-pgadmin.yaml kubectl apply -f YAML-STANDARD/deployments/deployment-fastapi.yaml

#.4 Appliquer les Services kubectl apply -f YAML-STANDARD/services/service-postgres.yaml kubectl apply -f YAML-STANDARD/services/service-pgadmin.yaml kubectl apply -f YAML-STANDARD/services/service-fastapi.yaml# #.5 Appliquer l'Horizontal Pod Autoscaler (HPA) (si nécessaire) kubectl apply -f YAML-STANDARD/deployments/hpa-fastapi.yaml

#.6 Appliquer l'Ingress (si nécessaire) kubectl apply -f YAML-STANDARD/deployments/ingress-fastapi.yaml

#. Vérifier l'État des Ressources Après avoir appliqué toutes les configurations, vérifiez l'état des pods et des services pour vous assurer qu'ils fonctionnent correctement : kubectl get pods --namespace=standard kubectl get services --namespace=standard

    PostgreSQL Deployment

Rôle : Ce fichier définit le déploiement de la base de données PostgreSQL.

apiVersion: Définit la version de l'API Kubernetes utilisée pour ce déploiement (ici, apps/v1).
kind: Indique le type de ressource à créer (ici, Deployment).
metadata: Contient des informations sur le déploiement, comme son nom (postgres).
spec: Définit les spécifications du déploiement, notamment :
    replicas: Nombre d'instances de PostgreSQL à exécuter (ici, 1).
    selector: Labels utilisés pour sélectionner les pods gérés par ce déploiement.
    template: Modèle pour créer les pods, qui inclut :
        containers: Définit le conteneur PostgreSQL, y compris son image, les variables d'environnement (utilisateur, mot de passe, base de données, méthode d'authentification) et les ports exposés.
        volumeMounts: Monte un volume pour stocker les données de manière persistante.
volumes: Définit un volume nommé qui utilise un PersistentVolumeClaim (PVC) pour la persistance des données.

    PostgreSQL Service

Rôle : Ce fichier définit le service qui exposera PostgreSQL à d'autres applications dans le cluster.

apiVersion: Version de l'API Kubernetes pour le service (v1).
kind: Type de ressource (ici, Service).
metadata: Informations sur le service, comme son nom (postgres).
spec: Spécifications du service, y compris :
    ports: Définit le port sur lequel le service écoutera (5432).
    selector: Correspond à des pods ayant un label spécifique (ici, app: postgres) pour diriger le trafic vers le bon déploiement.

    FastAPI Deployment

Rôle : Ce fichier définit le déploiement de l'application FastAPI.

apiVersion: Version de l'API Kubernetes utilisée pour ce déploiement (ici, apps/v1).
kind: Type de ressource (ici, Deployment).
metadata: Contient des informations sur le déploiement, comme son nom (fastapi).
spec: Définit les spécifications du déploiement, notamment :
    replicas: Nombre d'instances de FastAPI à exécuter (ici, 1).
    selector: Labels utilisés pour sélectionner les pods gérés par ce déploiement.
    template: Modèle pour créer les pods, qui inclut :
        containers: Définit le conteneur FastAPI, y compris son image, les ports exposés, et la variable d'environnement DATABASE_URL pour se connecter à PostgreSQL.

    FastAPI Service

Rôle : Ce fichier définit le service qui exposera l'application FastAPI à l'extérieur du cluster ou à d'autres services internes.

apiVersion: Version de l'API Kubernetes pour le service (v1).
kind: Type de ressource (ici, Service).
metadata: Informations sur le service, comme son nom (fastapi).
spec: Spécifications du service, y compris :
    type: Type de service (LoadBalancer ou NodePort), qui détermine comment le service sera exposé.
    ports: Définit le port sur lequel le service écoutera (5000) et le port cible sur le pod (également 5000).
    selector: Correspond à des pods ayant un label spécifique (ici, app: fastapi) pour diriger le trafic vers le bon déploiement.

    PersistentVolumeClaim pour PostgreSQL

Rôle : Ce fichier définit une demande de volume persistant qui sera utilisé par PostgreSQL pour stocker ses données de manière durable.

apiVersion: Version de l'API Kubernetes utilisée pour le PVC (v1).
kind: Type de ressource (ici, PersistentVolumeClaim).
metadata: Informations sur le PVC, comme son nom (postgres-pvc).
spec: Spécifications du PVC, y compris :
    accessModes: Définit comment le volume peut être utilisé (ici, ReadWriteOnce, ce qui signifie que le volume peut être monté en lecture-écriture par un seul nœud).
    resources: Indique les ressources demandées, ici, une capacité de stockage de 1 GiB.

Pour déployer vos configurations Kubernetes, voici les étapes à suivre :

Vérifiez votre configuration de cluster : Assurez-vous que votre cluster Kubernetes est prêt et fonctionne. Vous pouvez vérifier en listant les nœuds du cluster :

kubectl get nodes

Créer les objets Kubernetes : Utilisez kubectl apply -f <chemin_du_fichier> pour appliquer vos fichiers de configuration. Vous pouvez appliquer chaque fichier individuellement ou bien appliquer tout un répertoire.

Par exemple, pour le rou d'eau, tous les fichiers sous le répertoiremanifests et le volume PostgreSQL :

kubectl apply -f /srv/projet-kubernetes/volumes/persistentVolumeClaim-postgreSQL.yaml kubectl apply -f /srv/projet-kubernetes/manifests/deployments/fastapi-deployment.yaml kubectl apply -f services/postgreSQL-service.yaml kubectl apply -f services/fastapi-service.yaml kubectl apply -f services/PGAdmin-service.yaml

érifier le statut des déploiements et services : Une fois les fichiers appliqués, vérifiez que les déploiements et services sont correctement créés.

kubectl get deployments kubectl get services

Vous pouvez aussi inspecter les pods pour vous assurer qu’ils sont bien créés et qu’ils n’ont pas d’erreurs :

kubectl get pods

#se placer dans app pour acceder au Dockerfile docker login -u julh62

#«Construire l'image docker build -t julh62/exam-kubernetes_fastapi:latest . #verifier image docker images

#Pousse docker push julh62/exam-kubernetes_fastapi:latest
si l'image existe deja

docker pull julh62/exam-kubernetes_fastapi:latest

#changer le nom de l'imge dans le fichier de deployment de fastapi containers:

    name: fastapi image: docker.io/library/fastapi-image:latest #appliquer les changeemnts et redeployer kubectl apply -f /srv/projet-kubernetes//manifests/deployments/fastapi-deployment.yaml

verifier connexion à la base de données

kubectl exec -it <nom_du_pod_postgresql> -- /bin/bash

#Tester la Connexion : Une fois à l'intérieur,
vous pouvez essayer de vous connecter à PostgreSQL avec le client psql :
psql -h localhost -U admin storedb

#recuper l'image pour pgadmin   (image officielle sur dockerhub)  
docker pull dpage/pgadmin4:latest




#verifier si storageclass existe
kubectl get storageclass

kubectl get pvc
# apply le  pvc-postgres.yaml si pas encore fait

# passer à 3 replicas 
nano deployment-fastapi.yaml
kubectl apply -f deployment-fastapi.yaml 

#creer hpa pour surveiller l'utilisation CPU
#augmenter ou diminuer le nbre de réplicas
nano hpa-fastapi.yaml
kubectl apply -f hpa-fastapi.yaml
kubectl get all

# verifier installation cert-manager*
kubectl get crds | grep cert-manager
#telecharger cert-manager
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.6.1/cert-manager.yaml

kubectl get all

nano clusterissuer-letsencrypt.yaml
kubectl apply -f clusterissuer-letsencrypt.yaml 
kubectl get clusterissuer letsencrypt-prod

#créer fichier 
nano ingress-fastapi.yaml
kubectl apply -f ingress-fastapi.yaml
#L'ingress est une ressource Kubernetes native comme les Pods, 
#les déploiements, etc. À l'aide de l'ingress, nous pouvons gérer les #configurations de routage DNS. L‘Ingress Controller effectue le 
#routage réel en lisant les règles de routage à partir des objets 
#d'entrée stockés dans etcd.
kubectl get all
kubectl get secret

kubectl logs -n cert-manager deploy/cert-manager
kubectl describe certificate fastapi-tls

kubectl get pods -n cert-manager



#sauvegarde base donnée maitre kubernetes
# a faire plus tard
k3s etcd-snapshot save 
Désinstaller K3s

Exécutez la commande suivante pour désinstaller K3s proprement :

sudo /usr/local/bin/k3s-killall.sh
sudo /usr/local/bin/k3s-uninstall.sh

Cela arrêtera tous les services K3s et supprimera les fichiers liés à K3s.
2. Vérifier que K3s a été complètement désinstallé

Assurez-vous que le service K3s a été supprimé :

sudo systemctl status k3s

Cela doit renvoyer que le service n'existe plus ou est inactif.
3. Supprimer les fichiers de configuration et de données K3s

Vous pouvez également supprimer les fichiers restants dans /etc/rancher/k3s/ et /var/lib/rancher/k3s/ :

sudo rm -rf /etc/rancher/k3s
sudo rm -rf /var/lib/rancher/k3s

Cela supprimera toute configuration et tout état laissé par K3s.
4. Supprimer les containers Docker/Kubernetes (si nécessaire)

Si vous souhaitez également nettoyer les conteneurs Docker (en cas de réutilisation de Docker), vous pouvez supprimer tous les conteneurs et images Docker :

sudo docker system prune -a
    Vérifie si des contrôleurs (Deployments, DaemonSets) gèrent les pods avec kubectl get all --all-namespaces.
    Supprime ces contrôleurs pour éviter la régénération automatique des pods.
   
Les pods que tu vois sont gérés par des Deployments (comme coredns, local-path-provisioner, metrics-server). Pour éviter qu'ils ne soient recréés à chaque suppression de pod, il faut supprimer ces Deployments.

kubectl delete deployment coredns -n kube-system
kubectl delete deployment local-path-provisioner -n kube-system
kubectl delete deployment metrics-server -n kube-system

2. Supprimer les Jobs en cours

Les Jobs comme helm-delete-traefik et helm-install-traefik sont en cours d'exécution. S'ils ne sont plus nécessaires, tu peux les supprimer pour libérer des ressources :

kubectl delete job helm-delete-traefik -n kube-system
kubectl delete job helm-delete-traefik-crd -n kube-system
kubectl delete job helm-install-traefik -n kube-system
kubectl delete job helm-install-traefik-crd -n kube-system

