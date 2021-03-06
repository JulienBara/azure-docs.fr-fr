---
title: "Script Azure CLI - Surveillance et mise à l’échelle d’une instance SQL Database unique | Microsoft Docs"
description: "Exemple de script Azure CLI - Surveillance et mise à l’échelle d’une instance SQL Database unique à l’aide de l’interface Azure CLI"
services: sql-database
documentationcenter: sql-database
author: janeng
manager: jstrauss
editor: carlrab
tags: azure-service-management
ms.assetid: 
ms.service: sql-database
ms.custom: sample
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: sql-database
ms.workload: database
ms.date: 04/24/2017
ms.author: janeng
translationtype: Human Translation
ms.sourcegitcommit: 432752c895fca3721e78fb6eb17b5a3e5c4ca495
ms.openlocfilehash: f29da889f90968a82dccaeb1fa7e3c20e6b44458
ms.lasthandoff: 03/30/2017

---

# <a name="monitor-and-scale-a-single-sql-database-using-the-azure-cli"></a>Surveiller et mettre à l’échelle une instance unique SQL Database à l’aide de l’interface Azure CLI

Cet exemple de script CLI permet la mise à l’échelle d’une instance unique Azure SQL Database vers un nouveau niveau de performance après l’analyse des données de taille de la base de données. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>Exemple de script

[!code-azurecli[main](../../../cli_scripts/sql-database/monitor-and-scale-database/monitor-and-scale-database.sh "Surveillance et mise à l’échelle d’une instance SQL Database unique")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Une fois l’exemple de script exécuté, la commande suivante permet de supprimer le groupe de ressources et toutes les ressources associées.

```azurecli
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Remarques |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az sql server create](https://docs.microsoft.com/cli/azure/sql/server#create) | Crée un serveur logique qui héberge une base de données. |
| [az sql db show-usage](https://docs.microsoft.com/cli/azure/sql/db#show-usage) | Affiche les données de taille d’une base de données. |
| [az sql db update](https://docs.microsoft.com/cli/azure/sql/db#update) | Met à jour les propriétés de la base de données (par exemple, le niveau de performances ou le niveau de service) ou déplace une base de données vers, hors ou entre des pools élastiques. |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#set) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |
|||

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](https://docs.microsoft.com/cli/azure/overview).

Vous trouverez des exemples supplémentaires de scripts CLI SQL Database sur la page [Documentation Azure SQL Database](../sql-database-cli-samples.md).

