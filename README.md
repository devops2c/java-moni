# java-monitoring [Prometheus + Grafana]
tester prometheus et grafana avec application JAVA

Steps:
depot app:
https://github.com/spring-projects/spring-petclinic.git

note : repo is safe
✅ Source officielle : Maintenu par Spring Projects, l’équipe qui développe le framework Spring, un des plus populaires pour les applications Java.
✅ Utilisé à des fins pédagogiques : C’est une application de démonstration pour montrer les bonnes pratiques de Spring Boot (architecture, test, persistance, etc.).
✅ Code open source : Il est ouvert à la communauté, très surveillé et régulièrement mis à jour.
✅ Pas de dépendances malicieuses : Toutes les dépendances sont standards (Spring, H2, JPA, etc.) et gérées via Maven avec des versions bien connues.

conseils secu:
Conseils de vérification (par bonne habitude) :
Même si ce projet est sûr, voici quelques réflexes à garder pour n’importe quel repo :
🔎 Vérifie les auteurs/mainteneurs (ici : spring-projects)
📅 Regarde l’activité récente du dépôt (commits, issues)
🧪 Jette un œil au fichier pom.xml pour voir les dépendances
📁 Lancer d’abord en local dans un environnement isolé (ex: Docker ou VM)

builder image docker :
probleme avec cmd : ./mvnw spring-boot:build-image
debug 
docker version
docker info
dockerd ou Laisse cette commande tourner dans un terminal, ou exécute-la en arrière-plan avec nohup dockerd &
docker ps

docker login a docker.hub
$docker login
root@mBELHADI1-PC:~# docker tag spring-petclinic:3.4.0-SNAPSHOT mohamedbelhedi/spring-petclinic:latest
root@mBELHADI1-PC:~# docker push mohamedbelhedi//spring-petclinic:latest

Run l'app depuis maven:
![image](https://github.com/user-attachments/assets/24ae9d28-6061-43c0-ba37-6d7ba215685e)

DB H2:
![image](https://github.com/user-attachments/assets/a77c3277-1272-4706-af52-21a8f3d4dd11)
conf db :

instlataion prometheus
conf dans java : dependency

instllation grafana
contournement instlataion et lancement grafana

Lancer Grafana directement (sans systemctl) : manuellement
sudo /usr/sbin/grafana-server --homepath=/usr/share/grafana
http://localhost:3000 (admin/admin)

probleme restart prometheus , sln restart manuellement:
pkill prometheus
./prometheus --config.file=prometheus.yml &
stresser l'app:
sudo apt install wrk
avant stresse
![image](https://github.com/user-attachments/assets/74af8c03-7f61-4037-8b3d-affb43e3aae3)
stresse:
wrk -t5 -c40000 -d400s http://localhost:8080/
-t = threads
-c = connexions simultanées
-d = durée
![image](https://github.com/user-attachments/assets/d0b29eb5-bc99-4cff-bafb-63d65d9b7060)
![image](https://github.com/user-attachments/assets/77a3864c-e4cf-4da4-998d-a3c2e998b265)

Tester l'app sur K8s:
Démarrer Minikube (dans WSL2) :
minikube start --driver=docker
![image](https://github.com/user-attachments/assets/a4a8c95f-71c6-4367-aa0f-67516e206ad1)

🛠️ Construire ton image Docker
Creer Dockerfile pour ton app Java
Vérifier que ton app tourne en local : docker run -p 8080:8080 mon-app-java:v1

Micro_K8s:

3 facons d acces à lapp
1-Utiliser un Service de type NodePort avec creation tunnel entre ton pc et le service kub
ajouter/modifer service.yml comme suite:
type: NodePort
nodePort: 30080
===================
Tu rediriges directement le port du service vers ton machine locale sans passer par Minikube ou Docker. Cela contourne tout problème potentiel de réseau ou de configuration de port.
Donc, même si Minikube expose un port (comme 30080), kubectl port-forward te permet de tester plus rapidement et de vérifier que l'application fonctionne correctement.

lancer la cmd suivante en cas de probleme:
kubectl port-forward svc/mon-app-java-service 8080:8080

2 loadbalancer
il faut modifier service.yml comme suite:
spec:
  type: LoadBalancer
  lancer la cmd : minikube tunnel

Minikube va simuler un vrai LoadBalancer Cloud.
Tu vas recevoir une IP externe directe.
![image](https://github.com/user-attachments/assets/cefb3dc3-a570-4547-86ee-6bd48ed885cb)
![image](https://github.com/user-attachments/assets/d61c56bf-eed1-4992-8bb8-858dca4d0672)

3-Utiliser un Ingress Controller ( non testé)

************* Commandes et notes utiles **********
trouver un fichier avec extension spec : 
find /home/mohamed/projets/app_java/spring-petclinic -name '*.jar'

Dockerfile j'ai eu des soucis avec le chemin absolu , donc j'ai utilisé le chemin relatif.
Alternative Microk8s : https://microk8s.io/compare

kubectl delete deployment mon-app-java
kubectl apply -f deployment.yaml
