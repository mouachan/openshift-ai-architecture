# Architecture Globale OpenShift AI 2.22

## 🏗️ Vue d'ensemble

OpenShift AI 2.22 est construit sur une architecture modulaire utilisant un meta-operator qui déploie et gère tous les composants via un objet DataScienceCluster sur OpenShift Container Platform 4.19.

## 📊 Diagramme interactif

👉 **[Voir le diagramme interactif complet](./index.html)**

## 🎛️ Meta-Operator : Red Hat OpenShift AI Operator

**Définition officielle** : *Un méta-opérateur qui déploie et maintient tous les composants et sous-opérateurs qui font partie d'OpenShift AI.*

### DataScienceCluster (DSC)
L'objet principal qui configure tous les composants OpenShift AI :

```yaml
apiVersion: datasciencecluster.opendatahub.io/v1
kind: DataScienceCluster
metadata:
  name: default-dsc
spec:
  components:
    dashboard:
      managementState: Managed
    workbenches:
      managementState: Managed
    kserve:
      managementState: Managed
    modelmeshserving:
      managementState: Managed
    datasciencepipelines:
      managementState: Managed
    # ... autres composants
```

### Configuration managementState
- **Managed** : L'opérateur gère activement le composant, l'installe, et essaie de le maintenir actif
- **Removed** : L'opérateur gère activement le composant mais ne l'installe pas. Si déjà installé, l'opérateur essaiera de le supprimer

## 📁 Namespaces créés automatiquement

Lors de l'installation de l'opérateur, les namespaces suivants sont créés :

- **`redhat-ods-operator`** : Contient l'opérateur Red Hat OpenShift AI
- **`redhat-ods-applications`** : Installe le dashboard et autres composants requis
- **`redhat-ods-monitoring`** : Contient les services de monitoring et facturation
- **`rhods-notebooks`** : Où les environnements de notebooks sont déployés par défaut

## 🧩 Composants Core

### Dashboard
*Un dashboard orienté client qui montre les applications disponibles et installées pour l'environnement OpenShift AI ainsi que les ressources d'apprentissage comme les tutoriels, guides de démarrage rapide et documentation.*

**Fonctionnalités** :
- Interface web utilisateur
- Gestion des projets et utilisateurs
- Configuration des images de notebooks
- Gestion des profils d'accélérateurs GPU
- Administration des runtimes de serving

### Workbenches
*Une application auto-gérée qui permet aux data scientists de configurer leur propre environnement de serveur de notebooks et développer des modèles de machine learning dans JupyterLab.*

**Caractéristiques** :
- Environnements JupyterLab personnalisables
- Images de base configurables
- Intégration avec stockage persistant
- Support GPU optionnel

### Model Registry (🆕 Nouveauté 2.22)
*Les opérateurs Red Hat Authorino, Red Hat OpenShift Serverless, et Red Hat OpenShift Service Mesh ne sont plus requis pour utiliser le composant model registry dans OpenShift AI.*

**Améliorations 2.22** :
- OAuth Proxy intégré remplace Authorino
- Migration automatique des instances existantes
- Simplification de l'installation
- Nouvelles instances utilisent OAuth proxy par défaut

## 🚀 Model Serving Components

### KServe (Single-Model Serving)
*Pour les modèles volumineux, comme les grands modèles de langage, la plateforme Single-Model Serving utilise KServe, déployant chaque modèle sur son propre serveur.*

**Modes de déploiement** :
- **Serverless** : Utilise Knative pour autoscaling
- **RawDeployment** : Déploiement standard Kubernetes

**Dépendances** :
- Red Hat OpenShift Serverless (pour mode Serverless)
- Red Hat OpenShift Service Mesh (pour mode Advanced)
- Authorino (optionnel, pour authentification)

### ModelMesh (Multi-Model Serving)
*Pour les modèles petits et moyens, la plateforme Multi-Model Serving utilise ModelMesh pour déployer plusieurs modèles sur un serveur unique, partageant les ressources efficacement.*

**Avantages** :
- Optimisation des ressources
- Partage efficace des coûts
- Gestion centralisée
- Surveillance unifiée

## ⚙️ MLOps & Pipeline Components

### Data Science Pipelines
*Les data scientists peuvent construire des workflows de machine learning (ML) portables avec data science pipelines 2.0, utilisant des conteneurs Docker.*

**Mise à jour 2.22** : Kubeflow Pipelines version 2.5.0

**Fonctionnalités** :
- Workflows automatisés
- Conteneurs Docker
- Orchestration des étapes ML
- Intégration avec stockage S3

### TrustyAI
**Fonctionnalités** :
- Monitoring des modèles en production
- Explicabilité de l'IA
- Détection de biais
- Métriques de performance

## 🖥️ Distributed Workloads

*Les data scientists peuvent utiliser plusieurs nœuds en parallèle pour entraîner des modèles de machine learning ou traiter des données plus rapidement.*

### CodeFlare
- SDK Python pour orchestration distribuée
- Interface simplifiée pour Ray
- Gestion des ressources automatisée

### Ray
- Calcul distribué et parallel processing
- Mise à l'échelle automatique
- Support pour machine learning distribué

### Kueue
- Gestion des queues de travail
- Planification intelligente des ressources
- Partage équitable des ressources

### Training Operator
- Entraînement ML distribué
- Support pour jobs parallèles
- Optimisation multi-GPU

## 🔗 Dépendances Externes

### Opérateurs requis pour KServe
- **Red Hat OpenShift Serverless** : Knative Serving/Eventing
- **Red Hat OpenShift Service Mesh** : Istio Control Plane
- **Red Hat - Authorino** (optionnel) : Authentification des modèles

### Stockage
**Requis** : Stockage compatible S3 (AWS S3, MinIO, Ceph, IBM Cloud Storage)
**Utilisation** :
- Stockage des modèles ML
- Artifacts des pipelines
- Données d'entraînement
- Logs et métriques

### Support GPU (optionnel)
- **NVIDIA GPU Operator**
- **Node Feature Discovery Operator**

## 🆕 Nouveautés principales OpenShift AI 2.22

### Simplification du Model Registry
- **Avant** : Nécessitait Authorino + Serverless + Service Mesh
- **Maintenant** : OAuth Proxy intégré uniquement
- **Migration** : Automatique pour les instances existantes

### Amélioration de la résilience
- **Opérateur** : Maintenant avec 3 répliques
- **Haute disponibilité** : Amélioration de la résilience pour les workloads de production
- **Distribution des webhooks** : Opérations réparties sur plusieurs instances

### Mise à niveau des pipelines
- **Kubeflow Pipelines 2.5.0** : Nouvelles fonctionnalités et corrections
- **Compatibilité améliorée** : Meilleure intégration avec l'écosystème

## 📚 Documentation officielle

### Sources principales
- [Architecture d'OpenShift AI Cloud Service](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_cloud_service/1/html/installing_and_uninstalling_openshift_ai_cloud_service/architecture-of-openshift-ai_install)
- [Architecture OpenShift AI Self-Managed](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2-latest/html/installing_and_uninstalling_openshift_ai_self-managed/architecture-of-openshift-ai-self-managed_install)
- [Installation et déploiement](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_cloud_service/1/html/installing_and_uninstalling_openshift_ai_cloud_service/installing-and-deploying-openshift-ai_install)
- [Serving Models](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_cloud_service/1/html-single/serving_models/index)

### Nouveautés et releases
- [Nouvelles fonctionnalités 2.22](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_cloud_service/1/html/release_notes/new-features-and-enhancements_relnotes)
- [Issues résolues](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_cloud_service/1/html/release_notes/resolved-issues_relnotes)

## 🔧 Installation et configuration

### Prérequis
1. **OpenShift Container Platform 4.19+**
2. **Privilèges cluster administrator**
3. **Stockage par défaut** configuré et provisionnement dynamique
4. **Pour KServe** : Installation des opérateurs Serverless et Service Mesh

### Étapes d'installation
1. **Installer l'opérateur** depuis OperatorHub
2. **Créer DataScienceCluster** avec composants souhaités
3. **Configurer le stockage** S3-compatible
4. **Activer GPU** si nécessaire
5. **Configurer l'authentification** selon les besoins

---

**Basé sur** : Documentation officielle Red Hat OpenShift AI 2.22  
**Compatible** : OpenShift Container Platform 4.19+
