---
services: virtual-machines
title: "Configuration de la CLI Azure pour la gestion des services"
author: squillace
solutions: 
manager: timlt
editor: tysonn
ms.service: virtual-machine
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: linux
ms.workload: infrastructure
ms.date: 04/13/2015
ms.author: rasquill
translationtype: Human Translation
ms.sourcegitcommit: 0d8472cb3b0d891d2b184621d62830d1ccd5e2e7
ms.openlocfilehash: 737e4be21eefcbd6fbd31d15a6f77770cd59ca67
ms.lasthandoff: 03/21/2017


---
## <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure
Les étapes suivantes vous aident à utiliser facilement l’interface de ligne de commande Azure avec la version la plus récente et l’abonnement approprié. Si vous devez installer la CLI Azure et la connecter à votre compte, consultez la page [Interface de ligne de commande Azure](../articles/cli-install-nodejs.md).

### <a name="step-1-update-azure-cli-version"></a>Étape 1 : mise à jour de l’interface de ligne de commande Azure
Pour utiliser l’interface de ligne de commande Azure avec le mode de gestion des services, vous devez avoir une version récente. Pour vérifier votre version, tapez `azure --version`. Le résultat suivant doit s’afficher :

    $ azure --version
    0.8.17 (node: 0.10.25)

Si vous souhaitez mettre à jour votre version de l’interface de ligne de commande, consultez la page [Interface de ligne de commande](https://github.com/Azure/azure-xplat-cli).

### <a name="step-2-set-the-azure-account-and-subscription"></a>Étape 2 : configuration de votre compte et de votre abonnement Azure
Une fois que vous avez connecté votre interface de ligne de commande Azure au compte que vous souhaitez utiliser, vous pouvez avoir plusieurs abonnements. Le cas échéant, vous devez examiner les abonnements disponibles pour votre compte en saisissant la chaîne `azure account list`, puis sélectionner l’abonnement que vous souhaitez utiliser en indiquant `azure account set <subscription id or name> true`, où *id ou nom d’abonnement* correspond à l’ID ou au nom de l’abonnement que vous souhaitez utiliser dans la session active. Un résultat tel que celui-ci doit s’afficher :

    $ azure account set "Visual Studio Ultimate with MSDN" true
    info:    Executing command account set
    info:    Setting subscription to "Visual Studio Ultimate with MSDN" with id "0e220bf6-5caa-4e9f-8383-51f16b6c109f".
    info:    Changes saved
    info:    account set command OK

> [!NOTE]
> Si vous ne disposez pas d'un compte Azure, mais d'un abonnement MSDN, vous pouvez obtenir des crédits Azure gratuits en activant vos [avantages d'abonné MSDN ici](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ; ou vous pouvez utiliser le compte gratuit. Les deux possibilités permettent d’accéder à Azure.
> 
> 


