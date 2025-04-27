# Java-Deploiment-Monitoring [Prometheus + Grafana]
## Tester prometheus et grafana pour une application JAVA avec MicroK8s : minikube , puis Kubeadm
###  Date :      Avril 2025
### Stack technique et techno :
Cloud, Docker , Linux , Kubernetes , architecture des applications web , HA/HD , Git , Java , Github MK , Gestion projet , Experience utlisateurs , TTM , Web , Réseaux ,VScode, WSL , VMware.
### @Mohamed BELHEDI :+1: linkedin https://www.linkedin.com/in/mohamed-b-17986b94/  :earth_africa:
> [!NOTE]
> Ceci n'est pas un tuto mais plutot le deploiment d'une app Java et la mise en place de la chaine de supervision
> avec les problemes rencontrés

> [!IMPORTANT]
> Il s'agit d'une montée en competence la recolte des données et bonnes pratiques afin de centraliser toutes les etapes dans un seul doc et de le partager.
> Projet en cours ,la documentation et des elements (fonctionnalités,securités) sont des taches en cours et adaptés à fur et à mesure.
> Ce document peut avoir des fautes d'hortpghraphe,mise en page non respecté qui seront traités ultérieurement.

> [!TIP]
> Vous trouvez les tips en bas du de la page.

## Steps:
Depot de l'app Java:
https://github.com/spring-projects/spring-petclinic.git

### Note : Vérifier toujours qu'il s'agit d'un repo safe (pour ce repo)
- Source officielle : Maintenu par Spring Projects, l’équipe qui développe le framework Spring, un des plus populaires pour les applications Java.
+ Utilisé à des fins pédagogiques : C’est une application de démonstration pour montrer les bonnes pratiques de Spring Boot (architecture, test, persistance, etc.).
* Code open source : Il est ouvert à la communauté, très surveillé et régulièrement mis à jour.
+ Pas de dépendances malicieuses : Toutes les dépendances sont standards (Spring, H2, JPA, etc.) et gérées via Maven avec des versions bien connues.

conseils secu:
Conseils de vérification (par bonne habitude) :
Même si ce projet est sûr, voici quelques réflexes à garder pour n’importe quel repo :
+ Vérifie les auteurs/mainteneurs (ici : spring-projects)
+ Regarde l’activité récente du dépôt (commits, issues)
+ Jette un œil au fichier pom.xml pour voir les dépendances
+ Lancer d’abord en local dans un environnement isolé (ex: Docker ou VM)

Builder image docker :
```
probleme avec cmd : ./mvnw spring-boot:build-image
debug 
docker version
docker info
dockerd ou Laisse cette commande tourner dans un terminal, ou exécute-la en arrière-plan avec nohup dockerd &
docker ps
```
Se connecter a docker.hub
```
$docker login (sasis des identifiants compte docker hub)
Taguer ton image:
root@mBELHADI1-PC:~# docker tag spring-petclinic:3.4.0-SNAPSHOT mohamedbelhedi/spring-petclinic:latest
Publier votre image
root@mBELHADI1-PC:~# docker push mohamedbelhedi//spring-petclinic:latest
```
Execution de l'app depuis maven </br>
![image](https://github.com/user-attachments/assets/24ae9d28-6061-43c0-ba37-6d7ba215685e)

### DB H2:
H2 est un système de gestion de base de données relationnelles écrit en Java. Il peut être intégré à une application Java ou bien fonctionner en mode client-serveur. Son fichier jar est petit : environ 2.5 Mo </br>
![image](https://github.com/user-attachments/assets/a77c3277-1272-4706-af52-21a8f3d4dd11)
conf db :

### Instlataion prometheus
conf dans java : dependency

### Vérification Targer up: </br>
![image](https://github.com/user-attachments/assets/71d1ebcd-a44f-4e16-ba7b-9dae24cbcdc7) </br>
### Scraper les métriques : prometheus.yml
```
scrape_configs:
  - job_name: 'java-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['my-java-app-service:8080']
```
### Instllation grafana
contournement instlataion et lancement grafana
Lancer Grafana directement (sans systemctl) : manuellement
sudo /usr/sbin/grafana-server --homepath=/usr/share/grafana
http://localhost:3000 (admin/admin)

Si vous rencontrer un probleme Start/stop prometheus ,solution restart manuellement:
```
pkill prometheus
./prometheus --config.file=prometheus.yml &
```
### Création/import tableau de board Grafana
Depuis Grafana creer(+) / Import ou bien http://localhost:3000/dashboard/import) </br>
Pour démmarer j'ai utiliser le tableau id=4701 /
![image](https://github.com/user-attachments/assets/f2ec25ec-38b2-4d23-b4fc-e17cb2b0bc71)</br>
### stresser l'app:
J'ai strésser l'app pour voir le changement au niveau du dashboard Grafna :star_struck:	
```
sudo apt install wrk
```
### Avant stresse
![image](https://github.com/user-attachments/assets/74af8c03-7f61-4037-8b3d-affb43e3aae3)
Stresse commande & arguments :
```
wrk -t5 -c40000 -d400s http://localhost:8080/
```
+ -t = threads
+ -c = connexions simultanées
+ -d = durée 

![image](https://github.com/user-attachments/assets/d0b29eb5-bc99-4cff-bafb-63d65d9b7060)
![image](https://github.com/user-attachments/assets/77a3864c-e4cf-4da4-998d-a3c2e998b265)

### Tester l'app sur Micro K8s:</br>
Démarrer Minikube sur WSL2
```
minikube start --driver=docker
```
![image](https://github.com/user-attachments/assets/a4a8c95f-71c6-4367-aa0f-67516e206ad1)

🛠️ Construire ton image Docker
Creer Dockerfile pour ton app Java
Vérifier que ton app tourne en local : 
```
docker run -p 8080:8080 mon-app-java:v1
```
### Micro_K8s:

### Il existe 3 facons pour acceder/utliliser à lapp
### 1 Utiliser un Service de type NodePort avec creation tunnel entre ton pc et le service kub
Ajouter/modifer service.yml comme suite: </br> 
```
type: NodePort
nodePort: 30080
```
Tu rediriges directement le port du service vers ton machine locale sans passer par Minikube ou Docker. Cela contourne tout problème potentiel de réseau ou de configuration de port.
Donc, même si Minikube expose un port (comme 30080), kubectl port-forward te permet de tester plus rapidement et de vérifier que l'application fonctionne correctement.

Lancer la cmd suivante en cas de probleme:
```
kubectl port-forward svc/mon-app-java-service 8080:8080
```
### 2 Loadbalancer
Il faut modifier le type du fichier service.yml en type : LoadBalancer
```
spec:
  type: LoadBalancer
```
Lancer la cmd : 
  ```
minikube tunnel
```
Minikube va simuler un vrai LoadBalancer Cloud. </br>
Tu vas recevoir une IP externe directe , dans notre cas elle est 127.0.0.1 (localhost)
![image](https://github.com/user-attachments/assets/cefb3dc3-a570-4547-86ee-6bd48ed885cb)
![image](https://github.com/user-attachments/assets/d61c56bf-eed1-4992-8bb8-858dca4d0672)

### 3 Utiliser un Ingress Controller ( non testé)

## Autoscalling
### Installe metrics-server qui permet de superviser la charge appliquée </br>
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Ajouter hpa.yaml comme suite:
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: mon-app-java-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mon-app-java   # le nom EXACT de ton deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```
Ici : si la cpu dépasse 50% d'utilisation il va scaller
Déployer le hpa.yml </br>  
```
kubectl apply -f hpa.yaml
kubectl get hpa
```

### Commandes utiles
- Trouver tout les fichiers sous /home/mohamed/projets/app_java/spring-petclinic avec l'extension jar
  ```
  find /home/mohamed/projets/app_java/spring-petclinic -name '*.jar'
  ```
- Kubectl commande
```
kubectl delete deployment mon-app-java
kubectl apply -f deployment.yaml
```
- Voir ce qui utilise le port 3306 
  ```
sudo lsof -i :3306
  ```
  
### Notes utiles
+ Dockerfile j'ai eu des soucis avec le chemin absolu , donc j'ai utilisé le chemin relatif.

### Liens utiles
- Les bases de forme de base github : https://docs.github.com/fr/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax
- Complete list of github markdown emoji markup https://gist.github.com/rxaviers/7360908
- Alternative Microk8s : https://microk8s.io/compare

Reste à faire/implementer :warning:
- [ ] Développer les tableaux de board Grafana :tada:
- [ ] Rajouter notifications Grafana (adresse e mail test + conf smtp ...)
- [ ] Fixer la retention des données Grafana.
- [ ] Integrer d'autres services ( DB , API ... )
- [ ] Migration vers kubeadm ( minikube reste limité)
- [ ] Comprendre l'architecture K8s : Services / API , Ingress / Egress.

========================================
====      Mohamed BELHEDI           ====
====     *** ***** ***** ***        ====
====     Architect Techinique       ====
========================================
