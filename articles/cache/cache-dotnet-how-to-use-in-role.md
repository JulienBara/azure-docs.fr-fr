---
title: "Utilisation de In-Role Cache (.NET) | Microsoft Docs"
description: "Découvrez comment utiliser Azure In-Role Cache. Les exemples sont écrits en C# et utilisent l&quot;API .NET."
services: cache
documentationcenter: .net
author: steved0x
manager: douge
editor: 
ms.assetid: 4dad61ec-422a-4618-8054-84ae4ce652a2
ms.service: cache
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/17/2017
ms.author: sdanie
ROBOTS: NOINDEX
translationtype: Human Translation
ms.sourcegitcommit: bb1ca3189e6c39b46eaa5151bf0c74dbf4a35228
ms.openlocfilehash: e8347c21610118af4ebaa80b24edd0838bc58cf8
ms.lasthandoff: 03/18/2017

---
# <a name="how-to-use-in-role-cache-for-azure-cache"></a>Utilisation de In-Role Cache pour Azure Cache
> [!IMPORTANT]
> Conformément à notre [annonce](https://azure.microsoft.com/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/) de l’année dernière, le service de cache géré d’Azure et le service In-Role Cache d’Azure **ont été supprimés** le 30 novembre 2016. Nous vous recommandons d’utiliser [Cache Redis Azure](https://azure.microsoft.com/services/cache/). Pour obtenir des informations sur la migration, consultez [Migrer un Service de cache géré vers le Cache Redis Azure](../redis-cache/cache-migrate-to-redis.md).
> 
> 

Ce guide décrit la prise en main de **In-Role Cache pour Azure Cache**. Les exemples sont écrits en C\# et utilisent l’API .NET. Les scénarios présentés comprennent la **configuration d’un cluster de cache**, la **configuration de clients de cache**, l’**ajout et la suppression d’objets dans le cache, le stockage de l’état de session ASP.NET dans le cache**, ainsi que l’**activation du cache de la sortie de pages ASP.NET avec le cache**. Pour plus d’informations sur l’utilisation de In-Role Cache, consultez la section [Étapes suivantes][Next Steps].



<a name="what-is"></a>

## <a name="what-is-in-role-cache"></a>Présentation de In-Role Cache
In-Role Cache offre une couche de mise en cache à vos applications Azure. La mise en cache améliore les performances en stockant de façon temporaire les informations en mémoire à partir d'autres sources principales. Elle peut réduire les coûts associés aux transactions de bases de données dans le cloud. In-Role Cache comprend les fonctionnalités suivantes :

* Fournisseurs ASP.NET préconfigurés pour la mise en cache de l'état de session et de la sortie de pages, permettant d'accélérer les applications Web sans avoir à modifier le code d'application.
* Mise en cache de tout objet géré, pouvant être sérialisé, par exemple : objets CLR, lignes, XML, données binaires.
* Modèle de développement cohérent dans Azure et Windows Server AppFabric.

In-Role Cache vous permet d’assurer la mise en cache en utilisant une portion de la mémoire des machines virtuelles qui hébergent les instances de rôle dans vos services cloud Azure (aussi appelés services hébergés). Vous disposez ainsi d’une plus grande flexibilité en termes d’options de déploiement : les caches peuvent être très volumineux et ne pas avoir de quotas spécifiques.

> [!IMPORTANT]
> À compter du Kit SDK Azure 2.6, In-Role Cache utilise le Kit SDK Stockage Microsoft Azure version 4.3. Dans les versions précédentes du Kit SDK Azure, In-Role Cache utilisait le Kit Stockage Azure version 1.7. Les applications utilisant In-Role Cache avec les versions antérieures du Kit SDK Azure 2.6 doivent migrer vers le Kit SDK Azure 2.6 avant la désactivation de la version de Stockage Azure du 18-08-2011 le 1er août 2016. Pour plus d’informations, consultez les [Notes de publication du Kit SDK Azure 2.6 - In-Role Cache](../app-service-web/azure-sdk-dotnet-release-notes-2-6.md#in-role-cache-updates) et la rubrique [Mise à jour de Suppression de la version du service Stockage Microsoft Azure : extension jusqu’en 2016](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/10/19/microsoft-azure-storage-service-version-removal-update-extension-to-2016.aspx).
> 
> 

La mise en cache sur les instances de rôles présente certains avantages :

* Vous ne payez pas de frais supplémentaires pour la mise en cache. Vous ne payez que pour les ressources de calcul qui hébergent le cache.
* Les limitations et les quotas sont éliminés.
* Le contrôle et l'isolement sont de meilleur niveau. 
* Les performances sont améliorées.
* La taille des caches est automatiquement ajustée lorsque les rôles sont étendus ou réduits. La mémoire disponible pour le cache est étendue ou réduite de façon efficace lorsque des instances de rôle sont ajoutées ou supprimées.
* Débogage complet au moment du développement. 
* Prend en charge le protocole memcache.

En outre, la mise en cache sur les instances de rôle comporte ces options configurables :

* Configuration d'un rôle dédié à la mise en cache ou colocation de la mise en cache dans des rôles existants. 
* Mise à disposition du cache pour plusieurs clients dans le même déploiement de service cloud.
* Création de plusieurs caches nommés avec des propriétés différentes.
* Configuration facultative de la haute disponibilité sur certains caches.
* Utilisation de capacités étendues de mise en cache, comme les régions, les balises et les notifications.

Ce guide offre un aperçu de la prise en main de In-Role Cache. Pour plus d’informations sur les fonctions qui ne sont pas présentées dans ce guide de prise en main, consultez [À propos de In-Role Cache pour Azure Cache][Overview of In-Role Cache].

<a name="getting-started-cache-role-instance"></a>

## <a name="getting-started-with-in-role-cache"></a>Prise en main de In-Role Cache
In-Role Cache permet d'activer la mise en cache à l'aide de la mémoire qui se trouve sur les machines virtuelles qui hébergent vos instances de rôle. Les instances de rôle qui hébergent vos caches sont appelées **clusters de cache**. Il existe deux topologies de déploiement pour la mise en cache sur les instances de rôle :

* **Rôle dédié** : les instances de rôle sont utilisées exclusivement pour la mise en cache.
* **Rôle en colocation** : le cache partage les ressources de la machine virtuelle (bande passante, processeur, mémoire) avec l'application.

Pour utiliser la mise en cache sur les instances de rôle, vous devez configurer un cluster de cache, puis configurer les clients du cache afin qu'ils puissent accéder au cluster de cache.

* [Configuration du cluster de cache][Configure the cache cluster]
* [Configuration des clients de cache][Configure the cache clients]

<a name="enable-caching"></a>

## <a name="configure-the-cache-cluster"></a>Configuration du cluster de cache
Pour configurer un cluster de cache **Rôle en colocation** , sélectionnez le rôle dans lequel vous voulez héberger le cluster de cache. Cliquez avec le bouton droit sur les propriétés du rôle dans l’**Explorateur de solutions** et sélectionnez **Propriétés**.

![RoleCache1][RoleCache1]

Basculez vers l’onglet **Mise en cache**, cochez la case **Activer la mise en cache** et spécifiez les options de mise en cache souhaitées. Quand la mise en cache est activée sur un **Rôle de travail** ou sur un **Rôle Web ASP.NET**, la configuration par défaut est la mise en cache **Rôle en colocation** avec 30 % de la mémoire des instances de rôle alloués à la mise en cache. Un cache par défaut est configuré automatiquement, et d'autres caches nommés supplémentaires peuvent être créés si nécessaire. Ces caches se partageront la mémoire allouée.

![RoleCache2][RoleCache2]

Pour configurer un cluster de cache **Rôle dédié**, ajoutez un **Rôle de travail de cache** à votre projet.

![RoleCache7][RoleCache7]

Quand un **Rôle de travail de cache** est ajouté à un projet, la configuration par défaut est la mise en cache **Rôle dédié**.

![RoleCache8][RoleCache8]

Une fois que la mise en cache est activée, le compte de stockage du cluster de cache peut être configuré. In-Role Cache nécessite un compte de stockage Azure. Ce compte de stockage est utilisé pour contenir les données de configuration sur le cluster de cache, accessible depuis toutes les machines virtuelles qui composent le cluster de cache. Ce compte de stockage est spécifié sous l’onglet **Caching** de la page des propriétés du rôle de cluster de cache, juste au-dessus de **Paramètres du cache nommé**.

![RoleCache10][RoleCache10]

> Si ce compte de stockage n'est pas configuré, les rôles ne peuvent pas démarrer. 
> 
> 

La taille du cache est déterminée par la taille de la machine virtuelle du rôle, le nombre d'instances du rôle, et si le cluster de cache est configuré comme rôle dédié ou comme cluster de cache de rôle en colocation.

> Cette section contient une présentation simplifiée sur la configuration de la taille du cache. Pour plus d’informations sur la taille du cache et sur d’autres facteurs de planification de la capacité, consultez [Planification de la capacité pour le service de cache géré Azure][In-Role Cache Capacity Planning Considerations].
> 
> 

Pour configurer la taille de la machine virtuelle et le nombre d’instances de rôle, cliquez avec le bouton droit sur les propriétés du rôle dans **Explorateur de solutions** et choisissez **Propriétés**.

![RoleCache1][RoleCache1]

Passez sur l'onglet **Configuration** . La valeur par défaut de **Nombre d’instances** est 1 et **Taille de la machine virtuelle** par défaut est **Petite**.

![RoleCache3][RoleCache3]

La mémoire totale par taille de machine virtuelle se présente comme suit : 

* **Petite**: 1,75 Go
* **Moyenne**3,5 Go
* **Grande**: 7 Go
* **Très grande**: 14 Go

> Cette taille représente la quantité totale de mémoire disponible pour la machine virtuelle, partagée par le système d'exploitation, le processus de cache, les données en cache et l'application. Pour plus d’informations sur la configuration de la taille des machines virtuelles, consultez [Tailles de services cloud][How to Configure Virtual Machine Sizes]. Notez que le cache n'est pas pris en charge sur les machines virtuelles de taille **ExtraSmall** .
> 
> 

Lorsque la mise en cache **Rôle en colocation** est spécifiée, la taille du cache est déterminée par le pourcentage spécifié de mémoire de la machine virtuelle. Lorsque la mise en cache **Rôle dédié** est spécifiée, toute la mémoire disponible sur la machine virtuelle est utilisée pour la mise en cache. Si deux instances de rôle sont configurées, la mémoire combinée des machines virtuelles est utilisée. Ceci forme un cluster de cache dans lequel la mémoire de cache disponible est distribuée sur plusieurs instances de rôle, mais présentée aux clients du cache comme une seule et même ressource. La taille du cache augmente de façon similaire lorsque vous configurez d'autres instances de rôle. Afin de déterminer les paramètres nécessaires pour fournir un cache de la taille souhaitée, vous pouvez utiliser la feuille de calcul de planification de la capacité, qui est présentée dans la page [Planification de la capacité pour le service de cache géré Azure][In-Role Cache Capacity Planning Considerations].

Une fois le cluster de cache configuré, vous pouvez configurer les clients du cache pour permettre l'accès au cache.

<a name="NuGet"></a>

## <a name="configure-the-cache-clients"></a>Configuration des clients de cache
Pour accéder à un cache In-Role Cache, les clients doivent se trouver dans le même déploiement. Si le cluster de cache est un cluster de cache de rôle dédié, les clients sont d'autres rôles dans le déploiement. Si le cluster de cache est un cluster de cache de rôle en colocation, les clients peuvent être soit d'autres rôles dans le déploiement, soit les rôles qui hébergent le cluster de cache. Le package NuGet fourni permet de configurer chaque rôle client qui accède au cache. Pour configurer un rôle pour qu’il accède à un cluster de cache à l’aide du package Caching NuGet, cliquez avec le bouton droit sur le projet de rôle dans l’**Explorateur de solutions** et choisissez **Gérer les packages NuGet**. 

![RoleCache4][RoleCache4]

Sélectionnez **In-Role Cache**, cliquez sur **Installer**, puis sur **J’accepte**.

> Si **In-Role Cache** ne figure pas dans la liste, tapez **WindowsAzure.Caching** dans la zone de texte **Recherche en ligne** et sélectionnez-le dans les résultats.
> 
> 

![RoleCache5][RoleCache5]

Le package NuGet fait plusieurs choses : il ajoute la configuration nécessaire aux fichiers de configuration du rôle, il ajoute un paramètre de niveau de diagnostic du client de cache au fichier ServiceConfiguration.cscfg de l'application Azure, et il ajoute également les références d'assembly nécessaires.

> Pour les rôles web ASP.NET, le package Caching NuGet ajoute également deux sections commentées au fichier web.config. La première section active l'état de session à stocker dans le cache, et la seconde section active la mise en cache de la sortie de pages ASP.NET. Pour plus d’informations, consultez [Stockage de l’état de session ASP.NET dans le cache] et [Stockage de la mise en cache de sortie de pages ASP.NET dans le cache][How To: Store ASP.NET Page Output Caching in the Cache].
> 
> 

Le package NuGet ajoute les éléments de configuration suivants au fichier web.config ou app.config de votre rôle. Une section **dataCacheClients** et une section **cacheDiagnostics** sont ajoutées sous l’élément **configSections**. Si aucun élément **configSections** n’est présent, un élément configSections enfant de l’élément **configuration** est créé.

    <configSections>
      <!-- Existing sections omitted for clarity. -->

      <section name="dataCacheClients" 
               type="Microsoft.ApplicationServer.Caching.DataCacheClientsSection, Microsoft.ApplicationServer.Caching.Core" 
               allowLocation="true" 
               allowDefinition="Everywhere" />

      <section name="cacheDiagnostics" 
               type="Microsoft.ApplicationServer.Caching.AzureCommon.DiagnosticsConfigurationSection, Microsoft.ApplicationServer.Caching.AzureCommon" 
               allowLocation="true" 
               allowDefinition="Everywhere" />
    </configSections>

Ces nouvelles sections comprennent des références à un élément **dataCacheClients** et à un élément **cacheDiagnostics**. Ces éléments sont également ajoutés à l'élément **configuration** .

    <dataCacheClients>
      <dataCacheClient name="default">
        <autoDiscover isEnabled="true" identifier="[cache cluster role name]" />
        <!--<localCache isEnabled="true" sync="TimeoutBased" objectCount="100000" ttlValue="300" />-->
      </dataCacheClient>
    </dataCacheClients>
    <cacheDiagnostics>
      <crashDump dumpLevel="Off" dumpStorageQuotaInMB="100" />
    </cacheDiagnostics>

Une fois la configuration ajoutée, remplacez **[cache cluster role name]** par le nom du rôle qui héberge le cluster de cache.

> Si **[cache cluster role name]** n’est pas remplacé par le nom du rôle qui héberge le cluster de cache, une exception **TargetInvocationException** est retournée lors de l’accès au cache avec une exception interne **DatacacheException** avec le message « No such role exists ».
> 
> 

Le package NuGet ajoute également un paramètre **ClientDiagnosticLevel** à la section **ConfigurationSettings** du rôle de client de cache dans ServiceConfiguration.cscfg. L’exemple suivant représente la section **WebRole1** d’un fichier ServiceConfiguration.cscfg, où **ClientDiagnosticLevel** a la valeur 1, ce qui correspond à la valeur par défaut de **ClientDiagnosticLevel**.

    <Role name="WebRole1">
      <Instances count="1" />
      <ConfigurationSettings>
        <!-- Existing settings omitted for clarity. -->
        <Setting name="Microsoft.WindowsAzure.Plugins.Caching.ClientDiagnosticLevel" 
                 value="1" />
      </ConfigurationSettings>
    </Role>

> In-Role Cache comporte un niveau de diagnostic au niveau du serveur de cache et du client de cache. Le niveau de diagnostic est un paramètre unique qui configure le niveau des informations de diagnostic collectées pour la mise en cache. Pour plus d’informations, consultez [Dépannage et diagnostics d’Azure In-Role Cache][Troubleshooting and Diagnostics for In-Role Cache].
> 
> 

Le package NuGet ajoute également des références aux assemblys suivants :

* Microsoft.ApplicationServer.Caching.Client.dll
* Microsoft.ApplicationServer.Caching.Core.dll
* Microsoft.WindowsFabric.Common.dll
* Microsoft.WindowsFabric.Data.Common.dll
* Microsoft.ApplicationServer.Caching.AzureCommon.dll
* Microsoft.ApplicationServer.Caching.AzureClientHelper.dll

Si votre projet est un rôle Web ASP.NET, la référence d'assembly suivante est également ajoutée :

* Microsoft.Web.DistributedCache.dll.

Une fois que votre projet client est configuré pour la mise en cache, vous pouvez utiliser les techniques décrites dans les sections suivantes pour utiliser votre cache.

<a name="working-with-caches"></a>

## <a name="working-with-caches"></a>Utilisation des caches
Les étapes de cette section décrivent des tâches courantes avec la mise en cache.

* [Création d’un objet DataCache][How To: Create a DataCache Object]
* [Ajout et récupération d’un objet dans le cache][How To: Add and Retrieve an Object from the Cache]
* [Spécification de l’expiration d’un objet dans le cache][How To: Specify the Expiration of an Object in the Cache]
* [Stockage de l’état de session ASP.NET dans le cache][Stockage de l’état de session ASP.NET dans le cache]
* [Stockage de la mise en cache de sortie de pages ASP.NET dans le cache][How To: Store ASP.NET Page Output Caching in the Cache]

<a name="create-cache-object"></a>

## <a name="how-to-create-a-datacache-object"></a>Création d'un objet DataCache
Pour utiliser un cache par programmation, vous avez besoin d'une référence au cache. Ajoutez l'élément suivant en haut de tout fichier dont vous souhaitez qu'il utilise In-Role Cache :

    using Microsoft.ApplicationServer.Caching;

> Si Visual Studio ne reconnaît pas les types dans l'instruction d'utilisation même après installation du package Caching NuGet, qui ajoute les références nécessaires, assurez-vous que le profil cible du projet est .NET Framework 4.0 ou version ultérieure et sélectionnez un des profils qui ne spécifie pas **Client Profile**. Pour obtenir des instructions sur la configuration des clients du cache, consultez [Configuration des clients de cache][Configure the cache clients].
> 
> 

Il existe plusieurs méthodes pour créer un objet **DataCache** . La première consiste à créer simplement un **DataCache**en transmettant le nom du cache voulu.

    DataCache cache = new DataCache("default");

Une fois que le **DataCache** est instancié, vous pouvez l'utiliser pour interagir avec le cache, comme décrit dans les sections suivantes.

Pour utiliser la seconde méthode, créez un objet **DataCacheFactory** dans votre application à l'aide du constructeur par défaut. Le client du cache utilise alors les paramètres présents dans le fichier de configuration. Appelez soit la méthode **GetDefaultCache** de la nouvelle instance de **DataCacheFactory**, qui retourne un objet **DataCache**, soit la méthode **GetCache**, puis transmettez le nom de votre cache. Ces méthodes renvoient un objet **DataCache** qui peut être utilisé pour accéder au cache par programme.

    // Cache client configured by settings in application configuration file.
    DataCacheFactory cacheFactory = new DataCacheFactory();
    DataCache cache = cacheFactory.GetDefaultCache();
    // Or DataCache cache = cacheFactory.GetCache("MyCache");
    // cache can now be used to add and retrieve items.    

<a name="add-object"></a>

## <a name="how-to-add-and-retrieve-an-object-from-the-cache"></a>Ajout et récupération d'un objet dans le cache
Pour ajouter un élément au cache, vous pouvez utiliser la méthode **Add** ou **Put**. La méthode **Add** ajoute l'objet spécifié au cache, indexé par la valeur du paramètre de clé.

    // Add the string "value" to the cache, keyed by "item"
    cache.Add("item", "value");

Si un objet avec la même clé est déjà présent dans le cache, une **DataCacheException** est levée avec le message suivant :

> ErrorCode:SubStatus: tentative de création d'un objet avec une clé qui existe déjà dans le cache. La mise en cache accepte uniquement des valeurs de clé uniques pour les objets.
> 
> 

Pour récupérer un objet avec une clé particulière, vous pouvez utiliser la méthode **Get** . Si l'objet existe, il est renvoyé ; sinon, la valeur null est renvoyée.

    // Add the string "value" to the cache, keyed by "key"
    object result = cache.Get("Item");
    if (result == null)
    {
        // "Item" not in cache. Obtain it from specified data source
        // and add it.
        string value = GetItemValue(...);
        cache.Add("item", value);
    }
    else
    {
        // "Item" is in cache, cast result to correct type.
    }

La méthode **Put** ajoute l'objet avec la clé spécifiée au cache s'il n'existe pas ou remplace l'objet s'il existe.

    // Add the string "value" to the cache, keyed by "item". If it exists,
    // replace it.
    cache.Put("item", "value");

<a name="specify-expiration"></a>

## <a name="how-to-specify-the-expiration-of-an-object-in-the-cache"></a>Spécification de l'expiration d'un objet dans le cache
Par défaut, les éléments du cache expirent 10 minutes après qu'ils y ont été placés. Cette valeur peut être configurée dans le paramètre **Time to Live (min)** des propriétés du rôle qui héberge le cluster de cache.

![RoleCache6][RoleCache6]

Il existe trois types de **Type d’expiration** : **Aucun**, **Absolu** et **Fenêtre coulissante**. Ils définissent comment **Durée de vie (min)** est utilisé pour déterminer l’expiration. Le **Type d’expiration** par défaut est **Absolu**, ce qui signifie que le compte à rebours pour l’expiration d’un élément commence quand l’élément est placé dans le cache. Une fois que le délai spécifié pour un élément est écoulé, l'élément expire. Si vous spécifiez **Fenêtre coulissante**, le décompte de l’expiration d’un élément est réinitialisé à chaque accès de l’élément dans le cache, et l’élément n’expire pas tant que la durée spécifiée depuis son dernier accès n’est pas écoulée. Si vous spécifiez **Aucun**, vous devez affecter la valeur **0** à **Durée de vie (min)**. Les éléments n’expirent pas et restent valides tant qu’ils se trouvent dans le cache.

Si vous souhaitez une durée d’expiration plus longue ou plus courte que celle configurée dans les propriétés du rôle, cette durée spécifique peut être indiquée lors de l’ajout ou de la mise à jour d’un élément dans le cache à l’aide des surcharges **Add** et **Put**, qui peuvent prendre le paramètre **TimeSpan**. Dans l’exemple suivant, la chaîne **value** est ajoutée au cache, indexée par **item**, avec un délai d’expiration de 30 minutes.

    // Add the string "value" to the cache, keyed by "item"
    cache.Add("item", "value", TimeSpan.FromMinutes(30));

Pour afficher l’intervalle restant du délai d’expiration d’un élément dans le cache, vous pouvez utiliser la méthode **GetCacheItem** pour récupérer un objet **DataCacheItem** contenant des informations sur l’élément dans le cache, notamment l’intervalle restant du délai d’expiration.

    // Get a DataCacheItem object that contains information about
    // "item" in the cache. If there is no object keyed by "item" null
    // is returned. 
    DataCacheItem item = cache.GetCacheItem("item");
    TimeSpan timeRemaining = item.Timeout;

<a name="store-session"></a>

## <a name="how-to-store-aspnet-session-state-in-the-cache"></a>Stockage de l'état de session ASP.NET dans le cache
Le fournisseur de l'état de session pour In-Role Cache est un mécanisme de stockage externe aux processus pour les applications ASP.NET. Il vous permet de stocker l'état de votre session dans un cache Azure plutôt qu'en mémoire ou dans une base de données SQL Server. Pour utiliser le fournisseur de l’état de session de la mise en cache, configurez votre cluster de cache, puis configurez votre application ASP.NET pour la mise en cache à l’aide du package Caching NuGet, comme indiqué dans [Prise en main de In-Role Cache][Getting Started with In-Role Cache]. Lorsque le package Cache NuGet est installé, il ajoute dans le fichier web.config une section placée en commentaire contenant la configuration requise pour que votre application ASP.NET utilise le fournisseur d'état de session pour In-Role Cache.

    <!--Uncomment this section to use In-Role Cache for session state caching
    <system.web>
      <sessionState mode="Custom" customProvider="AFCacheSessionStateProvider">
        <providers>
          <add name="AFCacheSessionStateProvider" 
            type="Microsoft.Web.DistributedCache.DistributedCacheSessionStateStoreProvider,
                  Microsoft.Web.DistributedCache" 
            cacheName="default" 
            dataCacheClientName="default"/>
        </providers>
      </sessionState>
    </system.web>-->

> Si votre fichier web.config ne contient pas cette section mise en commentaire, après l’installation du package Caching NuGet, assurez-vous que la dernière version de NuGet Package Manager est installée depuis [NuGet Package Manager Installation][NuGet Package Manager Installation], puis désinstallez et réinstallez le package.
> 
> 

Pour activer le fournisseur de l'état de session pour In-Role Cache, supprimez les commentaires de le section spécifiée. Le cache par défaut est spécifié dans l'extrait fourni. Pour utiliser un autre cache, spécifiez le cache voulu dans l'attribut **cacheName** .

Pour plus d’informations sur l’utilisation du fournisseur de l’état de session du service de mise en cache, consultez [Fournisseur d’état de session pour Azure In-Role Cache][Session State Provider for In-Role Cache].

<a name="store-page"></a>

## <a name="how-to-store-aspnet-page-output-caching-in-the-cache"></a>Stockage de la mise en cache de sortie de pages ASP.NET dans le cache
Le fournisseur de caches de sortie pour In-Role Cache est un mécanisme de stockage hors processus pour les données de cache de sortie. Ces données concernent spécialement les réponses HTTP complètes (mise en cache de la sortie de pages). Le fournisseur se connecte au nouveau point d'extension du fournisseur de caches de sortie introduit dans ASP.NET 4. Pour utiliser le fournisseur de caches de sortie, configurez votre cluster de cache, puis configurez votre application ASP.NET pour la mise en cache à l’aide du package Caching NuGet, comme indiqué dans [Prise en main de In-Role Cache][Getting Started with In-Role Cache]. Lorsque le package Cache NuGet est installé, il ajoute dans le fichier web.config la section commentée suivante, qui contient la configuration requise pour que votre application ASP.NET utilise le fournisseur de caches de sortie pour In-Role Cache.

    <!--Uncomment this section to use In-Role Cache for output caching
    <caching>
      <outputCache defaultProvider="AFCacheOutputCacheProvider">
        <providers>
          <add name="AFCacheOutputCacheProvider" 
            type="Microsoft.Web.DistributedCache.DistributedCacheOutputCacheProvider,
                  Microsoft.Web.DistributedCache" 
            cacheName="default" 
            dataCacheClientName="default" />
        </providers>
      </outputCache>
    </caching>-->

> Si votre fichier web.config ne contient pas cette section mise en commentaire, après l’installation du package Caching NuGet, assurez-vous que la dernière version de NuGet Package Manager est installée depuis [NuGet Package Manager Installation][NuGet Package Manager Installation], puis désinstallez et réinstallez le package.
> 
> 

Pour activer le fournisseur de caches de sortie pour In-Role Cache, supprimez les commentaires de la section spécifiée. Le cache par défaut est spécifié dans l'extrait fourni. Pour utiliser un autre cache, spécifiez le cache voulu dans l'attribut **cacheName** .

Ajoutez une directive **OutputCache** à chaque page pour laquelle vous voulez mettre en cache la sortie.

    <%@ OutputCache Duration="60" VaryByParam="*" %>

Dans cet exemple, les données de page mises en cache resteront dans le cache pendant 60 secondes et une version différente de la page sera mise en cache pour chaque combinaison de paramètres. Pour plus d’informations sur les options disponibles, consultez [Directive OutputCache][OutputCache Directive].

Pour plus d’informations sur l’utilisation du fournisseur de cache de sortie pour In-Role Cache, consultez [Fournisseur de cache de sortie pour In-Role Cache][Output Cache Provider for In-Role Cache].

<a name="next-steps"></a>

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris les bases de In-Role Cache, suivez ces liens pour apprendre des tâches de mise en cache plus complexes.

* Consultez la référence MSDN : [In-Role Cache][In-Role Cache]
* Découvrez comment effectuer la migration vers In-Role Cache : [Migration vers Azure In-Role Cache][Migrate to In-Role Cache]
* Consultez les exemples : [Exemples Azure In-Role Cache][In-Role Cache Samples]
* Regardez la session [Maximum Performance: Accelerate Your Cloud Services Applications with Azure Caching (Performances maximales : accélérez vos applications de service cloud avec Azure Caching)][Maximum Performance: Accelerate Your Cloud Services Applications with Azure Caching] du TechEd 2013 sur In-Role Cache.

<!-- INTRA-TOPIC LINKS -->
[Next Steps]: #next-steps
[What is In-Role Cache?]: #what-is
[Create an Azure Cache]: #create-cache
[Which type of caching is right for me?]: #choosing-cache
[Getting Started with the In-Role Cache Service]: #getting-started-cache-service
[Prepare Your Visual Studio Project to Use In-Role Cache]: #prepare-vs
[Configure Your Application to Use Caching]: #configure-app
[Getting Started with In-Role Cache]: #getting-started-cache-role-instance
[Configure the cache cluster]: #enable-caching
[Configure the desired cache size]: #cache-size
[Configure the cache clients]: #NuGet
[Working with Caches]: #working-with-caches
[How To: Create a DataCache Object]: #create-cache-object
[How To: Add and Retrieve an Object from the Cache]: #add-object
[How To: Specify the Expiration of an Object in the Cache]: #specify-expiration
[Stockage de l’état de session ASP.NET dans le cache]: #store-session
[How To: Store ASP.NET Page Output Caching in the Cache]: #store-page
[Target a Supported .NET Framework Profile]: #prepare-vs-target-net

<!-- IMAGES --> 
[RoleCache1]: ./media/cache-dotnet-how-to-use-in-role/cache8.png
[RoleCache2]: ./media/cache-dotnet-how-to-use-in-role/cache9.png
[RoleCache3]: ./media/cache-dotnet-how-to-use-in-role/cache10.png
[RoleCache4]: ./media/cache-dotnet-how-to-use-in-role/cache11.png
[RoleCache5]: ./media/cache-dotnet-how-to-use-in-role/cache12.png
[RoleCache6]: ./media/cache-dotnet-how-to-use-in-role/cache13.png
[RoleCache7]: ./media/cache-dotnet-how-to-use-in-role/cache14.png
[RoleCache8]: ./media/cache-dotnet-how-to-use-in-role/cache15.png
[RoleCache10]: ./media/cache-dotnet-how-to-use-in-role/cache17.png

<!-- LINKS -->
[How to Configure Virtual Machine Sizes]: http://go.microsoft.com/fwlink/?LinkId=164387
[How to: Configure a Cache Client Programmatically]: http://msdn.microsoft.com/library/windowsazure/gg618003.aspx
[How to: Set a Page's Cacheability Programmatically]: http://msdn.microsoft.com/library/z852zf6b.aspx
[How to: Set the Cacheability of an ASP.NET Page Declaratively]: http://msdn.microsoft.com/library/zd1ysf1y.aspx
[In-Role Cache Capacity Planning Considerations]: http://go.microsoft.com/fwlink/?LinkId=252651
[In-Role Cache Samples]: http://msdn.microsoft.com/library/jj189876.aspx
[In-Role Cache]: http://go.microsoft.com/fwlink/?LinkId=252658
[In-Role Cache]: http://www.microsoft.com/showcase/Search.aspx?phrase=azure+caching
[Maximum Performance: Accelerate Your Cloud Services Applications with Azure Caching]: http://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/WAD-B326#fbid=kmrzkRxQ6gU
[Migrate to In-Role Cache]: http://msdn.microsoft.com/library/hh914163.aspx
[NuGet Package Manager Installation]: http://go.microsoft.com/fwlink/?LinkId=240311
[Output Cache Provider for In-Role Cache]: http://msdn.microsoft.com/library/windowsazure/gg185662.aspx
[OutputCache Directive]: http://go.microsoft.com/fwlink/?LinkId=251979
[Overview of In-Role Cache]: http://go.microsoft.com/fwlink/?LinkId=254172
[Session State Provider for In-Role Cache]: http://msdn.microsoft.com/library/windowsazure/gg185668.aspx
[Team Blog]: http://blogs.msdn.com/b/windowsazure/
[Troubleshooting and Diagnostics for In-Role Cache]: http://msdn.microsoft.com/library/windowsazure/hh914135.aspx
[Azure AppFabric Cache: Caching Session State]: http://www.microsoft.com/showcase/details.aspx?uuid=87c833e9-97a9-42b2-8bb1-7601f9b5ca20
[Azure Shared Caching]: http://msdn.microsoft.com/library/windowsazure/gg278356.aspx

[Which Azure Cache offering is right for me?]: cache-faq.md#which-azure-cache-offering-is-right-for-me


