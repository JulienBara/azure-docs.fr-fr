---
title: "Mise à l’échelle d’Azure Service Fabric par programmation | Microsoft Docs"
description: "Faire évoluer un cluster Azure Service Fabric par programmation, selon des déclencheurs personnalisés"
services: service-fabric
documentationcenter: .net
author: mjrousos
manager: jonjung
editor: 
ms.assetid: 
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/10/2017
ms.author: mikerou
translationtype: Human Translation
ms.sourcegitcommit: afe143848fae473d08dd33a3df4ab4ed92b731fa
ms.openlocfilehash: 8d7052fabeb348b4bba744b43d9af78f058175a8
ms.lasthandoff: 03/17/2017


---

# <a name="scale-a-service-fabric-cluster-programmatically"></a>Mettre à l’échelle un cluster Service Fabric par programmation 

Les principes de base de mise à l’échelle d’un cluster Service Fabric dans Azure sont décrits dans la documentation sur la [mise à l’échelle du cluster](./service-fabric-cluster-scale-up-down.md). Cet article décrit comment les clusters Service Fabric s’appuient sur des groupes de machines virtuelles identiques et peuvent être mis à l’échelle manuellement ou avec des règles automatiques. Ce document examine les méthodes de programmation pour coordonner les opérations de mise à l’échelle Azure dans les scénarios plus avancés. 

## <a name="reasons-for-programmatic-scaling"></a>Motifs d’une mise à l’échelle par programmation
Dans de nombreux scénarios, la mise à l’échelle manuelle ou via des règles automatiques est une bonne solution. Cependant, dans d’autres cas de figure, ces deux possibilités ne sont pas indiquées. Les inconvénients potentiels de ces approches sont les suivants :

- La mise à l’échelle manuelle vous oblige à ouvrir une session et à demander explicitement des opérations de mise à l’échelle. Si les opérations de mise à l’échelle sont requises fréquemment ou à des moments imprévisibles, cette approche n’est probablement pas la bonne.
- Lorsque les règles de mise à l’échelle automatique suppriment une instance d’un groupe de machines virtuelles identiques, elles ne suppriment pas automatiquement les connaissances de ce nœud du cluster Service Fabric associé, sauf si le nœud a un niveau de durabilité agent ou or. Comme les règles de mise à l’échelle automatique fonctionnent au niveau du groupe de machines virtuelles (et non au niveau de Service Fabric), elles peuvent supprimer des nœuds Service Fabric sans les arrêter en douceur. Cette suppression brutale laisse l’état de nœud Service Fabric « ghost » derrière elle, après les opérations de mise à l’échelle. Une personne (ou un service) doit nettoyer régulièrement l’état du nœud supprimé dans le cluster Service Fabric.
  - Notez qu’un nœud ayant un niveau de durabilité argent ou or nettoie automatiquement les nœuds supprimés.  
- Bien que les règles de mise à l’échelle automatique prennent en charge [plusieurs mesures](../monitoring-and-diagnostics/insights-autoscale-common-metrics.md), ce groupe reste limité. Si votre scénario requiert une mise à l’échelle en fonction d’une mesure non incluse dans ce groupe, les règles de mise à l’échelle automatique ne sont peut-être pas la bonne solution.

Face à ces limitations, vous pouvez mettre en œuvre des modèles personnalisés de mise à l’échelle automatique. 

## <a name="scaling-apis"></a>API de mise à l’échelle
Il existe des API Azure qui permettent aux applications de travailler par programmation avec les groupes de machines virtuelles identiques et des clusters Service Fabric. Si les options de mise à l’échelle automatique existantes ne conviennent pas à votre scénario, ces API permettent de mettre en œuvre une logique de mise à l’échelle personnalisée. 

Cette fonctionnalité de mise à l’échelle automatique personnalisée permet d’ajouter un service sans état à l’application Service Fabric pour gérer les opérations de mise à l’échelle. Dans la méthode `RunAsync` du service, un ensemble de déclencheurs peut déterminer si la mise à l’échelle est requise (avec vérification de paramètres tels que la taille maximale des clusters et de mise à l’échelle de recharges).   

L’API utilisée pour les interactions des groupes de machines virtuelles identiques (pour vérifier et modifier le nombre actuel d’instances de machine virtuelle) est la [bibliothèque Azure Management Compute](https://www.nuget.org/packages/Microsoft.Azure.Management.Compute.Fluent/1.0.0-beta50). Cette bibliothèque de calcul fluide fournit une API conviviale pour interagir avec les groupes de machines virtuelles identiques.

Pour interagir avec le cluster Service Fabric, utilisez [System.Fabric.FabricClient](/dotnet/api/system.fabric.fabricclient).

Bien sûr, le code de mise à l’échelle n’a pas besoin de s’exécuter comme un service dans le cluster à mettre à l’échelle. `IAzure` et `FabricClient` peuvent se connecter à leurs ressources Azure associées à distance. Donc, le service de mise à l’échelle peut être une application console ou un service Windows exécuté hors de l’application Service Fabric. 

## <a name="credential-management"></a>Gestion des informations d’identification
Un service chargé de gérer la mise à l’échelle doit pouvoir accéder aux ressources d’un groupe de machines virtuelles identiques, sans ouverture de session interactive. L’accès au cluster Service Fabric est simple si le service de mise à l’échelle modifie sa propre application Service Fabric, mais des informations d’identification sont nécessaires pour accéder au groupe identique. Pour vous connecter, vous pouvez utiliser un [principal de service](https://github.com/Azure/azure-sdk-for-net/blob/Fluent/AUTH.md#creating-a-service-principal-in-azure) créé avec [Azure CLI 2.0](https://github.com/azure/azure-cli).

Pour créer un principal de service, procédez comme suit :

1. Connectez-vous à l’interface Azure CLI (`az login`) en tant qu’utilisateur ayant accès au groupe de machines virtuelles identiques.
2. Créez le principal de service avec `az ad sp create-for-rbac`
    1. Notez l’ID d’application (appelé ID client ailleurs), le nom, le mot de passe et le client en vue d’une utilisation ultérieure.
    2. Vous aurez également besoin de votre ID d’abonnement, que vous pouvez afficher avec `az account list`.

La bibliothèque de calcul fluide peut se connecter à l’aide des informations d’identification suivantes :

```C#
var credentials = AzureCredentials.FromServicePrincipal(AzureClientId, AzureClientKey, AzureTenantId, AzureEnvironment.AzureGlobalCloud);
IAzure AzureClient = Azure.Authenticate(credentials).WithSubscription(AzureSubscriptionId);

if (AzureClient?.SubscriptionId == AzureSubscriptionId)
{
    ServiceEventSource.Current.ServiceMessage(Context, "Successfully logged into Azure");
}
else
{
    ServiceEventSource.Current.ServiceMessage(Context, "ERROR: Failed to login to Azure");
}
```

Une fois celle-ci connectée, le nombre d’instances de groupes identiques peut être interrogée avec `AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId).Capacity`.

## <a name="scaling-out"></a>Montée en charge
Le Kit de développement logiciel (SDK) Azure Compute permet d’ajouter des instances au groupe de machines virtuelles identiques, moyennant quelques appels seulement.

```C#
var scaleSet = AzureClient?.VirtualMachineScaleSets.GetById(ScaleSetId);
var newCapacity = Math.Min(MaximumNodeCount, NodeCount.Value + 1);
scaleSet.Update().WithCapacity(newCapacity).Apply(); 
``` 

**Un [bogue](https://github.com/Azure/azure-sdk-for-net/issues/2716) empêche actuellement ce code de fonctionner**, mais un correctif a été intégré, ce qui devrait résoudre le problème dans les prochaines versions publiées de Microsoft.Azure.Management.Compute.Fluent. Lors de la modification des propriétés du groupe de machines virtuelles identiques(comme la capacité) à l’aide de l’API de calcul fluide, les paramètres protégés du modèle Resource Manager du groupe identique sont perdus. L’absence de ces paramètres empêche, entre autres, les services Service Fabric de s’installer correctement sur les nouvelles instances de machine virtuelle.

Une des solutions temporaires consiste à appeler les applets de commande PowerShell à partir du service de mise à l’échelle pour mettre en œuvre la même modification (même si cette approche implique la présence d’outils PowerShell) :

```C#
using (var psInstance = PowerShell.Create())
{
    psInstance.AddScript($@"
        $clientId = ""{AzureClientId}""
        $clientKey = ConvertTo-SecureString -String ""{AzureClientKey}"" -AsPlainText -Force
        $Credential = New-Object -TypeName ""System.Management.Automation.PSCredential"" -ArgumentList $clientId, $clientKey
        Login-AzureRmAccount -Credential $Credential -ServicePrincipal -TenantId {AzureTenantId}
        
        $vmss = Get-AzureRmVmss -ResourceGroupName {ResourceGroup} -VMScaleSetName {NodeTypeToScale}
        $vmss.sku.capacity = {newCapacity}
        Update-AzureRmVmss -ResourceGroupName {ResourceGroup} -Name {NodeTypeToScale} -VirtualMachineScaleSet $vmss
    ");

    psInstance.Invoke();

    if (psInstance.HadErrors)
    {
        foreach (var error in psInstance.Streams.Error)
        {
            ServiceEventSource.Current.ServiceMessage(Context, $"ERROR adding node: {error.ToString()}");
        }
    }                
}
```

Lorsque vous ajoutez manuellement un nœud, l’ajout d’une instance de groupe identique doit suffire pour démarrer un nouveau nœud Service Fabric, car le modèle de groupe identique contient des extensions pour ajouter automatiquement des instances au cluster Service Fabric. 

## <a name="scaling-in"></a>Mise à l'échelle

La mise à l’échelle est similaire à la montée en charge. Les modifications d’un groupe de machines virtuelles identiques sont pratiquement identiques. Mais, comme nous l’avons déjà vu, Service Fabric ne nettoie automatiquement que les nœuds supprimés ayant une durabilité de niveau or ou argent. Par conséquent, dans le cas d’une mise à l’échelle avec une durabilité de niveau bronze, il est nécessaire d’interagir avec le cluster Service Fabric pour arrêter le nœud à supprimer, puis de supprimer son état.

La préparation du nœud à l’arrêt consiste à trouver le nœud à supprimer (le dernier nœud ajouté) et à le désactiver. Pour les nœuds non racines, les nœuds plus récents sont accessibles en comparant le `NodeInstanceId`. 

```C#
using (var client = new FabricClient())
{
    var mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync())
        .Where(n => n.NodeType.Equals(NodeTypeToScale, StringComparison.OrdinalIgnoreCase))
        .Where(n => n.NodeStatus == System.Fabric.Query.NodeStatus.Up)
        .OrderByDescending(n => n.NodeInstanceId)
        .FirstOrDefault();
```

N’oubliez pas que les nœuds *racines* ne semblent pas toujours respecter la convention selon laquelle les ID d’instance supérieurs sont supprimés en premier.

Une fois le nœud à supprimer trouvé, il peut être désactivé et supprimé à l’aide de la même instance `FabricClient` et de l’instance `IAzure` précédente.

```C#
var scaleSet = AzureClient?.VirtualMachineScaleSets.GetById(ScaleSetId);

// Remove the node from the Service Fabric cluster
ServiceEventSource.Current.ServiceMessage(Context, $"Disabling node {mostRecentLiveNode.NodeName}");
await client.ClusterManager.DeactivateNodeAsync(mostRecentLiveNode.NodeName, NodeDeactivationIntent.RemoveNode);

// Wait (up to a timeout) for the node to gracefully shutdown
var timeout = TimeSpan.FromMinutes(5);
var waitStart = DateTime.Now;
while ((mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Up || mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Disabling) &&
        DateTime.Now - waitStart < timeout)
{
    mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync()).FirstOrDefault(n => n.NodeName == mostRecentLiveNode.NodeName);
    await Task.Delay(10 * 1000);
}

// Decrement VMSS capacity
var newCapacity = Math.Max(MinimumNodeCount, NodeCount.Value - 1); // Check min count 

scaleSet.Update().WithCapacity(newCapacity).Apply(); 
```

Une fois l’instance de machine virtuelle supprimée, l’état du nœud Service Fabric peut être supprimé.

```C#
await client.ClusterManager.RemoveNodeStateAsync(mostRecentLiveNode.NodeName);
```

Comme auparavant, vous devez contourner le dysfonctionnement de `IVirtualMachineScaleSet.Update()` jusqu’à la résolution du problème [Azure/azure-sdk-de-net #2716](https://github.com/Azure/azure-sdk-for-net/issues/2716).

## <a name="potential-drawbacks"></a>Inconvénients potentiels

Comme nous l’avons vu dans les extraits de code précédents, créer votre propre service de mise à l’échelle optimise le niveau de contrôle et de personnalisation du comportement de mise à l’échelle de votre application. Cela peut être utile dans les cas qui requièrent un contrôle précis sur le moment ou les modalités de la mise à l’échelle ou la montée en charge d’une application. Toutefois, ce contrôle complique le code. Cette approche signifie que vous devez posséder votre propre code de mise à l’échelle, ce qui n’est pas négligeable.

Le mode de mise à l’échelle de Service Fabric dépend de votre cas de figure. Si la mise à l’échelle est rare, la possibilité d’ajouter ou de supprimer des nœuds manuellement est probablement suffisante. Dans les scénarios plus complexes, les règles de mise à l’échelle automatique et les kits de développement logiciel (SDK) qui assurent la mise à l’échelle par programmation offrent des alternatives puissantes.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à implémenter votre propre logique de mise à l’échelle automatique, vous devez vous familiariser avec les API utiles et les concepts suivants :

- [Augmenter ou diminuer la taille des instances d’un cluster Service Fabric à l’aide de règles de mise à l’échelle automatique](./service-fabric-cluster-scale-up-down.md)
- [Bibliothèques Azure Management fluides pour .NET](https://github.com/Azure/azure-sdk-for-net/tree/Fluent) (utiles pour interagir avec les groupes de machines virtuelles identiques sous-jacents d’un cluster Service Fabric)
- [System.Fabric.FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient) (utile pour interagir avec un cluster Service Fabric et ses nœuds)
