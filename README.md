### Sujet ECF : Amélioration et extension d'une architecture microservices e-commerce
#### **Contexte**

Vous avez accès au code source d'une application e-commerce partielle, structurée en microservices Spring Boot avec une interface React. L'architecture actuelle est fonctionnelle mais incomplète et perfectible. En tant qu'architecte logiciel, votre mission est triple : **analyser et critiquer** l'architecture existante, **l'améliorer** en appliquant les bonnes pratiques, et **la compléter** par de nouveaux microservices que vous jugerez pertinents.

#### **Description de l'application existante**

L'application repose sur plusieurs microservices Spring Boot et une interface utilisateur React.

- **Microservice manquant à compléter :**
  - **Service de suggestions de recherche :** Ce service doit être implémenté pour proposer des suggestions de recherche en temps réel.

- **Interface utilisateur (UI) React :** Consomme les API fournies par les microservices.

## Pré-requis

- [Docker](https://www.docker.com/) installé et configuré (`docker login`).  
- [Kind](https://kind.sigs.k8s.io/) pour créer un cluster Kubernetes local.  
- [kubectl](https://kubernetes.io/docs/tasks/tools/) installé et configuré pour votre cluster.  
- Code source des microservices et du front React.  
- Accès à Docker Hub pour pousser les images.  

---

## 1️⃣ Création des images Docker

### Auth Service
```powershell
docker build -t ecf-auth-service ./server/authentication-service
docker tag ecf-auth-service:latest <user-name>/ecf-auth-service:latest
docker push <user-name>/ecf-auth-service:latest
```

### Common Data Service
```powershell
docker build -t ecf-common-data-service ./server/common-data-service
docker tag ecf-common-data-service:latest <user-name>/ecf-common-data-service:latest
docker push <user-name>/ecf-common-data-service:latest
```

### Payment Service
```powershell
docker build -t ecf-payment-service ./server/payment-service
docker tag ecf-payment-service:latest <user-name>/ecf-payment-service:latest
docker push <user-name>/ecf-payment-service:latest
```

### Front React App
```powershell
docker build -t ecf-front-app ./client
docker tag ecf-front-app:latest <user-name>/ecf-front-app:latest
docker push <user-name>/ecf-front-app:latest
```
---

## 2️⃣ Déploiement Kubernetes

### Création du cluster (Kind)
```powershell
kind create cluster --config .\manifests\kind\basic-cluster.yaml
kubectl cluster-info
```

### Création du namespace
```powershell
kubectl apply -f .\manifests\namespace.yaml
```

---

## 3️⃣ Déploiement de MySQL
```powershell
kubectl apply -f .\manifests\mysql\mysql-secret.yaml
kubectl apply -f .\manifests\mysql\mysql-pv.yaml
kubectl apply -f .\manifests\mysql\mysql-pvc.yaml
kubectl apply -f .\manifests\mysql\mysql-deployment.yaml
kubectl apply -f .\manifests\mysql\mysql-service.yaml
```
### Vérification et port-forward
```powershell
kubectl get pods -n ecf-namespace -o wide
kubectl port-forward -n ecf-namespace svc/mysql 3307:3306
```

---

## 4️⃣ Déploiement de Redis
```powershell
kubectl apply -f .\manifests\redis\redis-secret.yaml
kubectl apply -f .\manifests\redis\redis-pv.yaml
kubectl apply -f .\manifests\redis\redis-pvc.yaml
kubectl apply -f .\manifests\redis\redis-deployment.yaml
kubectl apply -f .\manifests\redis\redis-service.yaml
```
### Vérification et port-forward
```powershell
kubectl get pods -n ecf-namespace -o wide
kubectl port-forward -n ecf-namespace svc/redis 6379:6379
```

---

## 5️⃣ Déploiement des microservices Spring Boot

### Auth Service
```powershell
kubectl apply -f .\manifests\auth\auth-deployment.yaml
kubectl apply -f .\manifests\auth\auth-service.yaml
kubectl port-forward -n ecf-namespace svc/auth-service 7000:8080
```

### Payment Service
```powershell
kubectl apply -f .\manifests\payment\payment-deployment.yaml
kubectl apply -f .\manifests\payment\payment-service.yaml
kubectl port-forward -n ecf-namespace svc/payment-service 9050:8080
```

### Common Data Service
```powershell
kubectl apply -f .\manifests\common-data\common-data-deployment.yaml
kubectl apply -f .\manifests\common-data\common-data-service.yaml
kubectl port-forward -n ecf-namespace svc/common-data-service 9000:9000
```

---

## 6️⃣ Déploiement du Front React
```powershell
kubectl apply -f .\manifests\front\front-deployment.yaml
kubectl apply -f .\manifests\front\front-service.yaml
kubectl port-forward -n ecf-namespace svc/front-app-service 3000:3000
```

---

## 7️⃣ Commandes utiles Kubernetes

| Action                        | Commande                                                                                 |
| ----------------------------- | ---------------------------------------------------------------------------------------- |
| Lister tous les pods          | `kubectl get pods -n ecf-namespace`                                                      |
| Lister tous les services      | `kubectl get svc -n ecf-namespace`                                                       |
| Supprimer un déploiement      | `kubectl delete deployment <nom> -n ecf-namespace`                                       |
| Redéployer un déploiement     | `kubectl apply -f <deployment.yaml>`                                                     |
| Vérifier les logs             | `kubectl logs -n ecf-namespace <pod-name>`                                               |
| Vérifier la configuration PVC | `kubectl get pvc -n ecf-namespace`                                                       |
| Décrire un pod                | `kubectl describe pod <pod-name> -n ecf-namespace`                                       |
| Forward un port               | `kubectl port-forward -n ecf-namespace svc/<service-name> <local-port>:<container-port>` |

---

# Liste des routes API

## Auth Service

| Méthode | Route           | Description |
|---------|----------------|-------------|
| GET     | `/test`        | Vérifie que le service est actif. Renvoie "success". |
| POST    | `/signup`      | Création d’un compte utilisateur. Reçoit un `AccountCreationRequest` JSON et retourne `AccountCreationResponse`. |
| POST    | `/authenticate` | Authentification. Reçoit un header `Authorization: Basic base64(username:password)` et retourne un JWT et le prénom de l’utilisateur dans `AuthenticationResponse`. |

## Common Data Service

| Méthode | Route                            | Paramètres     | Description |
|---------|---------------------------------|----------------|-------------|
| GET     | `/products`                     | `q`            | Retourne les produits filtrés par catégories. |
| GET     | `/products`                     | `product_id`   | Retourne un ou plusieurs produits par ID. |
| GET     | `/home`                          | -              | Donne les données pour l’écran principal (MainScreenResponse). |
| GET     | `/tabs`                          | -              | Donne les onglets pour le front (HomeTabsDataResponse). |
| GET     | `/filter`                        | `q`            | Retourne les attributs de filtre pour les produits (FilterAttributesResponse). |
| GET     | `/search-suggestion-list`        | -              | Retourne la liste des suggestions de recherche (SearchSuggestionResponse). |

## Payment Service

| Méthode | Route      | Description |
|---------|------------|-------------|
| GET     | `/test`    | Vérifie que le service est actif. Renvoie "success". |
| POST    | `/payment` | Effectue le paiement via Stripe. Reçoit un `CardToken` JSON et retourne un `PaymentStatus`. |






