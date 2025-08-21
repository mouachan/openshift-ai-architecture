# Architecture Globale OpenShift AI 2.22

## 🏗️ Vue d'ensemble

OpenShift AI 2.22 est construit sur une architecture modulaire utilisant un meta-operator qui déploie et gère tous les composants via un objet DataScienceCluster sur OpenShift Container Platform 4.19.

## 📊 Diagramme d'architecture

👉 **[Voir le diagramme interactif complet](./index.html)**

```mermaid
graph TB
    subgraph "🏗️ OpenShift Container Platform 4.19"
        K8S[Kubernetes API Server<br/>etcd • CRI-O • Kubelet]
        OCP[OpenShift Core<br/>OAuth • RBAC • Routes]
        OLM[Operator Lifecycle Manager<br/>OperatorHub]
    end
    
    subgraph "🎛️ Red Hat OpenShift AI Operator Meta-Operator"
        DSC[DataScienceCluster DSC<br/>Gestion composants<br/>managementState: Managed/Removed]
        DSCI[DSCInitialization DSCI<br/>Configuration globale<br/>Namespaces]
    end
    
    subgraph "📁 Namespaces OpenShift AI"
        NS1[redhat-ods-operator<br/>Opérateur principal]
        NS2[redhat-ods-applications<br/>Composants applicatifs]
        NS3[redhat-ods-monitoring<br/>Monitoring et métriques]
        NS4[rhods-notebooks<br/>Workbenches utilisateur]
    end
    
    subgraph "🧩 Composants Core"
        DASH[Dashboard<br/>Interface web<br/>Gestion projets/utilisateurs<br/>managementState: Managed]
        WB[Workbenches<br/>JupyterLab<br/>Notebooks personnalisés<br/>managementState: Managed]
        MR[Model Registry<br/>Gestion modèles<br/>🆕 OAuth Proxy 2.22<br/>managementState: Managed]
    end
    
    subgraph "🚀 Model Serving"
        KS[KServe<br/>Single-Model Serving<br/>LLMs • GPU autoscaling<br/>Serverless/RawDeployment<br/>managementState: Managed]
        MM[ModelMesh<br/>Multi-Model Serving<br/>Ressources partagées<br/>managementState: Managed]
    end
    
    subgraph "⚙️ MLOps & Pipelines"
        DSP[Data Science Pipelines<br/>Kubeflow Pipelines 2.5.0<br/>Workflows Docker<br/>managementState: Managed]
        TA[TrustyAI<br/>Monitoring modèles<br/>Explicabilité IA<br/>managementState: Managed]
    end
    
    subgraph "🖥️ Distributed Workloads"
        CF[CodeFlare<br/>SDK Python<br/>Orchestration<br/>managementState: Managed]
        RAY[Ray<br/>Calcul distribué<br/>Parallel processing<br/>managementState: Managed]
        KUEUE[Kueue<br/>Gestion queues<br/>Ressources partagées<br/>managementState: Managed]
        TO[Training Operator<br/>Entraînement ML<br/>Jobs distribués<br/>managementState: Managed]
    end
    
    subgraph "🔗 Opérateurs Dépendants Externes"
        SM[OpenShift Service Mesh<br/>Istio Control Plane<br/>Requis pour KServe Advanced]
        SL[OpenShift Serverless<br/>Knative Serving/Eventing<br/>Requis pour KServe Serverless]
        AUTH[Authorino Optionnel<br/>Authentification modèles<br/>Token validation]
        GPU[NVIDIA GPU Operator<br/>Node Feature Discovery<br/>Support GPU]
    end
    
    subgraph "💾 Stockage & Dépendances"
        S3[Stockage S3-Compatible<br/>AWS S3 • MinIO • Ceph<br/>IBM Cloud Storage<br/>Requis pour modèles]
    end
    
    %% Relations principales
    K8S --> DSC
    OCP --> DSC
    OLM --> DSC
    
    DSC --> NS1
    DSC --> NS2
    DSC --> NS3
    DSC --> NS4
    
    DSC --> DASH
    DSC --> WB
    DSC --> MR
    DSC --> KS
    DSC --> MM
    DSC --> DSP
    DSC --> TA
    DSC --> CF
    DSC --> RAY
    DSC --> KUEUE
    DSC --> TO
    
    %% Dépendances externes
    KS --> SM
    KS --> SL
    KS --> AUTH
    WB --> GPU
    TO --> GPU
    
    %% Stockage
    KS --> S3
    MM --> S3
    DSP --> S3
    MR --> S3
    
    %% Styles
    classDef platform fill:#2c3e50,stroke:#34495e,color:#fff
    classDef operator fill:#e74c3c,stroke:#c0392b,color:#fff
    classDef core fill:#3498db,stroke:#2980b9,color:#fff
    classDef serving fill:#00b894,stroke:#00a085,color:#fff
    classDef mlops fill:#9b59b6,stroke:#8e44ad,color:#fff
    classDef workload fill:#f39c12,stroke:#e67e22,color:#fff
    classDef dependency fill:#95a5a6,stroke:#7f8c8d,color:#fff
    classDef storage fill:#27ae60,stroke:#229954,color:#fff
    classDef namespace fill:#e8f4f8,stroke:#3498db,color:#2c3e50
    
    class K8S,OCP,OLM platform
    class DSC,DSCI operator
    class NS1,NS2,NS3,NS4 namespace
    class DASH,WB,MR core
    class KS,MM serving
    class DSP,TA mlops
    class CF,RAY,KUEUE,TO workload
    class SM,SL,AUTH,GPU dependency
    class S3 storage
```

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
