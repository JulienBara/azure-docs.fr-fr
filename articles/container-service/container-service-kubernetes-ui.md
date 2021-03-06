---
title: "Gérer le cluster Kubernetes Azure avec l’interface utilisateur Web | Microsoft Docs"
description: "Utilisation de l’interface web Kubernetes dans Azure Container Service"
services: container-service
documentationcenter: 
author: bburns
manager: timlt
editor: 
tags: acs, azure-container-service, kubernetes
keywords: 
ms.assetid: 
ms.service: container-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/21/2017
ms.author: bburns
translationtype: Human Translation
ms.sourcegitcommit: 2a381431acb6436ddd8e13c69b05423a33cd4fa6
ms.openlocfilehash: 5987b1034fc9c52b13606c469683adff06729984
ms.lasthandoff: 02/22/2017


---

# <a name="using-the-kubernetes-web-ui-with-azure-container-service"></a>Utilisation de l’interface utilisateur Web Kubernetes avec Azure Container Service

## <a name="prerequisites"></a>Composants requis
Cette procédure pas à pas suppose que vous avez [créé un cluster Kubernetes à l’aide d’Azure Container Service](container-service-kubernetes-walkthrough.md).


Elle suppose également que vous avez installé les outils `kubectl` et Azure CLI 2.0.

Vous pouvez tester si l’outil `az` est installé en exécutant :

```console
$ az --version
```

Si l’outil `az` n’est pas installé, suivez les instructions figurant [ici](https://github.com/azure/azure-cli#installation).

Vous pouvez tester si l’outil `kubectl` est installé en exécutant :

```console
$ kubectl version
```

Si `kubectl` n’est pas installé, vous pouvez exécuter :

```console
$ az acs kubernetes install-cli
```

## <a name="overview"></a>Vue d’ensemble

### <a name="connect-to-the-web-ui"></a>Connexion à l’interface web
Vous pouvez lancer l’interface web Kubernetes en exécutant :

```console
$ az acs kubernetes browse -g [Resource Group] -n [Container service instance name]
```

Cette commande ouvre un navigateur web configuré pour communiquer avec un proxy sécurisé en connectant votre machine locale à l’interface web Kubernetes.

### <a name="create-and-expose-a-service"></a>Création et exposition d’un service
1. Dans l’interface web Kubernetes, cliquez sur le bouton **Créer** dans la fenêtre supérieure droite.

    ![Interface de création Kubernetes](media/k8s/create.png)

    Une boîte de dialogue dans laquelle vous pouvez commencer à créer votre application s’ouvre.

2. Donnez-lui le nom `hello-nginx`. Utilisez le [ `nginx` conteneur de Docker](https://hub.docker.com/_/nginx/) et déployez trois réplicas de ce service web.

    ![Boîte de dialogue de création de pods Kubernetes](media/k8s/nginx.png)

3. Sous **Service**, sélectionnez **Externe** est saisissez le port 80.

    Ce paramètre permet d’équilibrer la charge du trafic pour les trois réplicas.

    ![Boîte de dialogue de création de service Kubernetes](media/k8s/service.png)

4. Cliquez sur **Déployer** pour déployer ces conteneurs et ces services.

    ![Déploiement de Kubernetes](media/k8s/deploy.png)

### <a name="view-your-containers"></a>Affichage de vos conteneurs
Une fois que vous cliquez sur **Déployer**, l’interface utilisateur affiche une vue de votre service pendant son déploiement :

![État de Kubernetes](media/k8s/status.png)

Vous pouvez voir l’état de chaque objet Kubernetes dans le cercle sur le côté gauche de l’interface utilisateur, sous **Pods**. Si le cercle est partiellement complet, l’objet est toujours en cours de déploiement. Lorsqu’un objet est entièrement déployé, une coche verte s’affiche :

![Kubernetes déployé](media/k8s/deployed.png)

Une fois que tout est en cours d’exécution, cliquez sur l’un de vos pods pour afficher les détails sur le service web exécuté.

![Pods kubernetes](media/k8s/pods.png)

Dans la vue **Pods**, vous pouvez voir des informations sur les conteneurs dans le pod, ainsi que les ressources de processeur et de mémoire utilisées par ces conteneurs :

![Ressources Kubernetes](media/k8s/resources.png)

Si vous ne voyez pas les ressources, vous devrez peut-être attendre quelques minutes que les données de surveillance se propagent.

Cliquez sur **Afficher les journaux** pour afficher les journaux de votre conteneur.

![Journaux Kubernetes](media/k8s/logs.png)

### <a name="viewing-your-service"></a>Affichage de votre service
Outre l’exécution de vos conteneurs, l’interface Kubernetes a créé un élément `Service` externe qui approvisionne un équilibrage de charge pour diriger le trafic vers les conteneurs de votre cluster.

Dans le volet de navigation de gauche, cliquez sur **Services** pour afficher tous les services (pour le moment, vous ne devriez en voir qu’un).

![Services Kubernetes](media/k8s/service-deployed.png)

Dans cette vue, vous devriez voir un point de terminaison externe (adresse IP) qui a été affecté à votre service.
Si vous cliquez sur cette adresse IP, vous devez voir votre conteneur Nginx en cours d’exécution derrière l’équilibrage de charge.

![Vue nginx](media/k8s/nginx-page.png)

### <a name="resizing-your-service"></a>Redimensionnement de votre service
Outre l’affichage de vos objets dans l’interface, vous pouvez modifier et mettre à jour les objets API Kubernetes.

Cliquez d’abord sur **Déploiements** dans le volet de navigation gauche pour afficher le déploiement pour votre service.

Une fois que vous êtes dans cette vue, cliquez sur ReplicaSet, puis sur **Modifier** dans la barre de navigation supérieure :

![Modification de Kubernetes](media/k8s/edit.png)

Modifiez le champ `spec.replicas` pour avoir la valeur `2`, et cliquez sur **Mettre à jour**.

Il n’y aura donc plus que deux réplicas, car l’un de vos pods sera supprimé.

 


