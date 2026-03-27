### Sujet ECF : Amélioration et extension d'une architecture microservices e-commerce
#### **Contexte**

Vous avez accès au code source d'une application e-commerce partielle, structurée en microservices Spring Boot avec une interface React. L'architecture actuelle est fonctionnelle mais incomplète et perfectible. En tant qu'architecte logiciel, votre mission est triple : **analyser et critiquer** l'architecture existante, **l'améliorer** en appliquant les bonnes pratiques, et **la compléter** par de nouveaux microservices que vous jugerez pertinents.

#### **Description de l'application existante**

L'application repose sur plusieurs microservices Spring Boot et une interface utilisateur React.

- **Microservice manquant à compléter :**
  - **Service de suggestions de recherche :** Ce service doit être implémenté pour proposer des suggestions de recherche en temps réel.

- **Interface utilisateur (UI) React :** Consomme les API fournies par les microservices.

## Obligatoire
4. **Dockerisation et orchestration :**
   - Créez un `Dockerfile` optimisé pour chaque microservice (multi-stage build recommandé).
   - Dockerisez l'interface utilisateur React.
   - Rédigez les manifests kubernetes.

5. **Documentation :**
   - Documentez l'architecture cible avec un diagramme (C4, UML ou équivalent).
   - Fournissez les instructions de build et de démarrage de l'ensemble de l'application.
   - Décrivez les choix techniques et architecturaux effectués, et les compromis associés.

#### **Livrables**

- Document d'architecture (état actuel + cible) avec justification des choix.
- `Dockerfile` pour chaque microservice et pour l'interface utilisateur.
- manifests Kubernetes pour l'orchestration complète.
- Documentation de déploiement et guide de test.
