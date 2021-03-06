---
title: "Charger une image personnalisée pour Azure RemoteApp | Microsoft Docs"
description: "Découvrez comment télécharger une image personnalisée pour Azure RemoteApp."
services: remoteapp
documentationcenter: 
author: ericorman
manager: mbaldwin
ms.assetid: 299e0510-1a6b-4fdf-914a-3631b061a360
ms.service: remoteapp
ms.workload: compute
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/23/2016
ms.author: ericor
translationtype: Human Translation
ms.sourcegitcommit: aaf97d26c982c1592230096588e0b0c3ee516a73
ms.openlocfilehash: b93861bc6277aaca5954bf9f4c338fed22a0c3fc
ms.lasthandoff: 04/27/2017


---
# <a name="upload-a-custom-image-for-azure-remoteapp"></a>Téléchargement d'une image personnalisée pour Azure RemoteApp
> [!IMPORTANT]
> Azure RemoteApp ne sera plus disponible à partir du 31 août 2017. Pour plus d’informations, lisez [l’annonce](https://go.microsoft.com/fwlink/?linkid=821148) .
> 
> 

Maintenant que vous avez créé votre image de modèle personnalisée ou l’avez mise à jour avec des modifications, vous devez télécharger cette image dans votre bibliothèque d’images Azure RemoteApp. Suivez ces étapes.

## <a name="before-you-start"></a>Avant de commencer
1. Vérifiez que votre image personnalisée est conforme aux [exigences relatives aux images](remoteapp-imagereqs.md) et aux [exigences relatives aux applications](remoteapp-appreqs.md).
2. Installez le [module Azure PowerShell](/powershell/azure/overview).

## <a name="step-by-step-on-how-to-upload-custom-image"></a>Guide étape par étape pour le téléchargement d’une image personnalisée
1. Ouvrez le portail de gestion Azure et accédez à la page RemoteApp.
2. Dans l’onglet **Images de modèle**, cliquez sur **Télécharger** en bas de la page.
3. Entrez un nom convivial pour votre image et spécifiez l’emplacement du compte de stockage. Vérifiez que l’emplacement est le même que votre collection RemoteApp ou est un emplacement où vous souhaitez en créer une.
4. Lorsque vous y êtes invité, téléchargez le script sur votre ordinateur local.
5. Copiez les paramètres de commande de la zone de texte dans votre Presse-papiers.
6. Ouvrez une fenêtre Windows PowerShell avec élévation de privilèges.
7. Dans la fenêtre Windows PowerShell avec élévation de privilèges, accédez au répertoire où vous avez téléchargé le script.
8. Collez la commande copiée et appuyez sur **Entrée**.
   
   Le téléchargement commence. Sa durée dépend de nombreux facteurs, notamment la vitesse de votre réseau et la taille de l’image
9. Si votre téléchargement échoue en raison d’une interruption du réseau ou d’un problème similaire, vous pouvez toujours reprendre le processus de téléchargement commencé. Pour reprendre un téléchargement, exécutez le script en utilisant la même ligne de commande.

> [!WARNING]
> Ne modifiez jamais le script de téléchargement. Des  vérifications spécifiques ont été implémentées pour vous assurer que l’image répond aux exigences relatives aux images et aux applications.
> 
> 

## <a name="common-problems"></a>Problèmes courants
* Veillez à utiliser Windows PowerShell, et non Azure PowerShell. Vous devez installer le module Azure PowerShell, car certains modules sont nécessaires pour le processus de téléchargement.
* Ne modifiez jamais le script, les validations sont pensées pour vous faciliter la tâche.
* Si le fichier de disque dur virtuel est verrouillé pendant le téléchargement, copiez le fichier ou déplacez-le vers un nouvel emplacement et essayez à nouveau. Un processus Windows peut empêcher le téléchargement.  


