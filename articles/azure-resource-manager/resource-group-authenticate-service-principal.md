---
title: "Créer une identité pour l’application Azure avec PowerShell | Microsoft Docs"
description: "Explique comment utiliser Azure PowerShell pour créer une application et un principal du service Azure Active Directory, et comment accorder à l’application l’accès aux ressources par le biais du contrôle d’accès en fonction du rôle. Cet article montre comment authentifier l’application avec un mot de passe ou un certificat."
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: d2caf121-9fbe-4f00-bf9d-8f3d1f00a6ff
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 04/03/2017
ms.author: tomfitz
translationtype: Human Translation
ms.sourcegitcommit: aaf97d26c982c1592230096588e0b0c3ee516a73
ms.openlocfilehash: eb6bddbe4220418f7c525985ab6a15524589829e
ms.lasthandoff: 04/27/2017


---
# <a name="use-azure-powershell-to-create-a-service-principal-to-access-resources"></a>Créer un principal du service pour accéder aux ressources à l’aide d’Azure PowerShell
> [!div class="op_single_selector"]
> * [PowerShell](resource-group-authenticate-service-principal.md)
> * [Interface de ligne de commande Azure](resource-group-authenticate-service-principal-cli.md)
> * [Portail](resource-group-create-service-principal-portal.md)
> 
> 

Lorsque vous avez une application ou un script qui doit pouvoir accéder à des ressources, vous pouvez configurer une identité pour l’application et authentifier l’application avec ses propres informations d’identification. Cette identité est connue en tant que principal de service. Cette approche vous permet d’effectuer les opérations suivantes :

* Affecter à l’identité de l’application des autorisations différentes de vos propres autorisations. En règle générale, ces autorisations sont strictement limitées à ce que l’application doit faire.
* Utilisez un certificat pour l’authentification lors de l’exécution d’un script sans assistance.

Cette rubrique vous montre comment utiliser [Azure PowerShell](/powershell/azure/overview) pour configurer tout ce dont vous avez besoin pour qu’une application puisse s’exécuter sous ses propres informations d’identification et sous sa propre identité.

## <a name="required-permissions"></a>Autorisations requises
Pour effectuer cette rubrique, vous devez disposer des autorisations suffisantes dans votre annuaire Azure Active Directory et votre abonnement Azure. Plus précisément, vous devez être en mesure de créer une application dans l’annuaire Azure Active Directory et d’affecter un rôle au principal du service. 

Le moyen le plus simple pour vérifier que votre compte dispose des autorisations adéquates est d’utiliser le portail. Consultez [Vérifier l’autorisation requise](resource-group-create-service-principal-portal.md#required-permissions).

Passez maintenant à la section appropriée pour vous authentifier à l’aide d’un [mot de passe](#create-service-principal-with-password) ou d’un [certificat](#create-service-principal-with-certificate).

## <a name="create-service-principal-with-password"></a>Créer un principal du service avec un mot de passe
Le script suivant crée une identité pour votre application et lui attribue le rôle de contributeur pour l’étendue spécifiée :

```powershell
Param (

 # Use to set scope to resource group. If no value is provided, scope is set to subscription.
 [Parameter(Mandatory=$false)]
 [String] $ResourceGroup,

 # Use to set subscription. If no value is provided, default subscription is used. 
 [Parameter(Mandatory=$false)]
 [String] $SubscriptionId,

 [Parameter(Mandatory=$true)]
 [String] $ApplicationDisplayName,

 [Parameter(Mandatory=$true)]
 [String] $Password
 )

 Login-AzureRmAccount
 Import-Module AzureRM.Resources

 if ($SubscriptionId -eq "") 
 {
    $SubscriptionId = (Get-AzureRmContext).Subscription.SubscriptionId
 }
 else
 {
    Set-AzureRmContext -SubscriptionId $SubscriptionId
 }

 if ($ResourceGroup -eq "")
 {
    $Scope = "/subscriptions/" + $SubscriptionId
 }
 else
 {
    $Scope = (Get-AzureRmResourceGroup -Name $ResourceGroup -ErrorAction Stop).ResourceId
 }

 # Create Azure Active Directory application with password
 $Application = New-AzureRmADApplication -DisplayName $ApplicationDisplayName -HomePage ("http://" + $ApplicationDisplayName) -IdentifierUris ("http://" + $ApplicationDisplayName) -Password $Password

 # Create Service Principal for the AD app
 $ServicePrincipal = New-AzureRMADServicePrincipal -ApplicationId $Application.ApplicationId 
 Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id 

 $NewRole = $null
 $Retries = 0;
 While ($NewRole -eq $null -and $Retries -le 6)
 {
    # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
    Sleep 15
    New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId -Scope $Scope | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
    $Retries++;
 }
```

Quelques points à noter concernant le script :

* Pour accorder l’accès aux identités pour l’abonnement par défaut, il est inutile de fournir les paramètres ResourceGroup et SubscriptionId.
* Spécifiez le paramètre ResourceGroup uniquement lorsque vous souhaitez limiter l’étendue de l’attribution de rôles à un groupe de ressources.
* Pour les applications à locataire unique, la page d’accueil et les URI des identificateurs ne sont pas validées.
*  Dans cet exemple, vous ajoutez le principal du service au rôle Contributeur. Pour les autres rôles, voir [RBAC : rôles intégrés](../active-directory/role-based-access-built-in-roles.md).
* Le script reste en veille pendant 15 secondes pour laisser le temps au nouveau principal du service de se propager dans Azure Active Directory. Si votre script n’attend pas suffisamment, une erreur indiquant « PrincipalNotFound : le principal {id} n’existe pas dans le répertoire. » s’affiche.
* Si vous devez accorder au service principal l’accès à plus d’abonnements ou groupes de ressources, exécutez à nouveau l’applet de commande `New-AzureRMRoleAssignment` avec des portées différentes.


### <a name="provide-credentials-through-powershell"></a>Fournir des informations d’identification via PowerShell
Maintenant, vous devez vous connecter en tant qu’application pour effectuer des opérations. Pour le nom d’utilisateur, utilisez le paramètre `ApplicationId` que vous avez créé pour l’application. Utilisez le mot de passe que vous avez indiqué lors de la création du compte. 

```powershell   
$creds = Get-Credential
Login-AzureRmAccount -Credential $creds -ServicePrincipal -TenantId {tenant-id}
```

L’ID client n’est pas sensible, vous pouvez donc l’incorporer directement dans votre script. Pour récupérer l’ID client, utilisez :

```powershell
(Get-AzureRmSubscription -SubscriptionName "Contoso Default").TenantId
```

## <a name="create-service-principal-with-self-signed-certificate"></a>Créer un principal du service avec un certificat auto-signé
Pour générer un certificat auto-signé et un principal de service avec Azure PowerShell 2.0 sur Windows 10 ouWindows Server 2016 Technical Preview, utilisez le script suivant :

```powershell
Param (

 # Use to set scope to resource group. If no value is provided, scope is set to subscription.
 [Parameter(Mandatory=$false)]
 [String] $ResourceGroup,

 # Use to set subscription. If no value is provided, default subscription is used. 
 [Parameter(Mandatory=$false)]
 [String] $SubscriptionId,

 [Parameter(Mandatory=$true)]
 [String] $ApplicationDisplayName
 )

 Login-AzureRmAccount
 Import-Module AzureRM.Resources

 if ($SubscriptionId -eq "") 
 {
    $SubscriptionId = (Get-AzureRmContext).Subscription.SubscriptionId
 }
 else
 {
    Set-AzureRmContext -SubscriptionId $SubscriptionId
 }

 if ($ResourceGroup -eq "")
 {
    $Scope = "/subscriptions/" + $SubscriptionId
 }
 else
 {
    $Scope = (Get-AzureRmResourceGroup -Name $ResourceGroup -ErrorAction Stop).ResourceId
 }

 $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=exampleappScriptCert" -KeySpec KeyExchange
 $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

 # Use Key credentials
 $Application = New-AzureRmADApplication -DisplayName $ApplicationDisplayName -HomePage ("http://" + $ApplicationDisplayName) -IdentifierUris ("http://" + $ApplicationDisplayName) -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore

 $ServicePrincipal = New-AzureRMADServicePrincipal -ApplicationId $Application.ApplicationId 
 Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id 

 $NewRole = $null
 $Retries = 0;
 While ($NewRole -eq $null -and $Retries -le 6)
 {
    # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
    Sleep 15
    New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId -Scope $Scope | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
    $Retries++;
 }
```

Quelques points à noter concernant le script :

* Pour accorder l’accès aux identités pour l’abonnement par défaut, il est inutile de fournir les paramètres ResourceGroup et SubscriptionId.
* Spécifiez le paramètre ResourceGroup uniquement lorsque vous souhaitez limiter l’étendue de l’attribution de rôles à un groupe de ressources.
* Pour les applications à locataire unique, la page d’accueil et les URI des identificateurs ne sont pas validées.
*  Dans cet exemple, vous ajoutez le principal du service au rôle Contributeur. Pour les autres rôles, voir [RBAC : rôles intégrés](../active-directory/role-based-access-built-in-roles.md).
* Le script reste en veille pendant 15 secondes pour laisser le temps au nouveau principal du service de se propager dans Azure Active Directory. Si votre script n’attend pas suffisamment, une erreur indiquant « PrincipalNotFound : le principal {id} n’existe pas dans le répertoire. » s’affiche.
* Si vous devez accorder au service principal l’accès à plus d’abonnements ou groupes de ressources, exécutez à nouveau l’applet de commande `New-AzureRMRoleAssignment` avec des portées différentes.

Si vous **n’avez pas Windows 10 ou Windows Server 2016 Technical Preview**, vous devez télécharger le [générateur de certificat auto-signé](https://gallery.technet.microsoft.com/scriptcenter/Self-signed-certificate-5920a7c6/) depuis le centre de scripts Microsoft. Extrayez son contenu et importez l’applet de commande dont vous avez besoin.

```powershell  
# Only run if you could not use New-SelfSignedCertificate
Import-Module -Name c:\ExtractedModule\New-SelfSignedCertificateEx.ps1
```
  
Dans le script, remplacez les deux lignes suivantes pour générer le certificat.
  
```powershell
New-SelfSignedCertificateEx  -StoreLocation CurrentUser -StoreName My -Subject "CN=exampleapp" -KeySpec "Exchange" -FriendlyName "exampleapp"
$cert = Get-ChildItem -path Cert:\CurrentUser\my | where {$PSitem.Subject -eq 'CN=exampleapp' }
```

### <a name="provide-certificate-through-automated-powershell-script"></a>Fournir un certificat via un script PowerShell automatisé
Chaque fois que vous vous connectez en tant que principal de service, vous devez fournir l’ID de locataire du répertoire de votre application AD. Un locataire est une instance d’Azure Active Directory. Si vous disposez d’un seul abonnement, vous pouvez utiliser :

```powershell
Param (
 
 [Parameter(Mandatory=$true)]
 [String] $CertSubject,
 
 [Parameter(Mandatory=$true)]
 [String] $ApplicationId,

 [Parameter(Mandatory=$true)]
 [String] $TenantId
 )

 $Thumbprint = (Get-ChildItem cert:\CurrentUser\My\ | Where-Object {$_.Subject -match $CertSubject }).Thumbprint
 Login-AzureRmAccount -ServicePrincipal -CertificateThumbprint $Thumbprint -ApplicationId $ApplicationId -TenantId $TenantId
```

L’ID d’application et l’ID de client ne sont pas sensibles, vous pouvez donc les incorporer directement dans votre script. Pour récupérer l’ID client, utilisez :

```powershell
(Get-AzureRmSubscription -SubscriptionName "Contoso Default").TenantId
```

Si vous avez besoin extraire l’ID de l’application, utilisez :

```powershell
(Get-AzureRmADApplication -DisplayNameStartWith {display-name}).ApplicationId
```

## <a name="create-service-principal-with-certificate-from-certificate-authority"></a>Créer un principal du service avec un certificat à partir de l’autorité de certification
Pour utiliser un certificat émis par une autorité de certification afin de créer le principal de service, utilisez le script suivant :

```powershell
Param (
 [Parameter(Mandatory=$true)]
 [String] $ApplicationDisplayName,

 [Parameter(Mandatory=$true)]
 [String] $SubscriptionId,

 [Parameter(Mandatory=$true)]
 [String] $CertPath,

 [Parameter(Mandatory=$true)]
 [String] $CertPlainPassword
 )

 Login-AzureRmAccount
 Import-Module AzureRM.Resources
 Set-AzureRmContext -SubscriptionId $SubscriptionId

 $KeyId = (New-Guid).Guid
 $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force

 $PFXCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($CertPath, $CertPassword)
 $KeyValue = [System.Convert]::ToBase64String($PFXCert.GetRawCertData())

 $KeyCredential = New-Object  Microsoft.Azure.Commands.Resources.Models.ActiveDirectory.PSADKeyCredential
 $KeyCredential.StartDate = $PFXCert.NotBefore
 $KeyCredential.EndDate= $PFXCert.NotAfter
 $KeyCredential.KeyId = $KeyId
 $KeyCredential.CertValue = $KeyValue

 # Use Key credentials
 $Application = New-AzureRmADApplication -DisplayName $ApplicationDisplayName -HomePage ("http://" + $ApplicationDisplayName) -IdentifierUris ("http://" + $KeyId) -KeyCredentials $keyCredential

 $ServicePrincipal = New-AzureRMADServicePrincipal -ApplicationId $Application.ApplicationId 
 Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id 

 $NewRole = $null
 $Retries = 0;
 While ($NewRole -eq $null -and $Retries -le 6)
 {
    # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
    Sleep 15
    New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
    $Retries++;
 }
 
 $NewRole
```

Quelques points à noter concernant le script :

* L’étendue de l’accès est limitée à l’abonnement.
* Pour les applications à locataire unique, la page d’accueil et les URI des identificateurs ne sont pas validées.
*  Dans cet exemple, vous ajoutez le principal du service au rôle Contributeur. Pour les autres rôles, voir [RBAC : rôles intégrés](../active-directory/role-based-access-built-in-roles.md).
* Le script reste en veille pendant 15 secondes pour laisser le temps au nouveau principal du service de se propager dans Azure Active Directory. Si votre script n’attend pas suffisamment, une erreur indiquant « PrincipalNotFound : le principal {id} n’existe pas dans le répertoire. » s’affiche.
* Si vous devez accorder au service principal l’accès à plus d’abonnements ou groupes de ressources, exécutez à nouveau l’applet de commande `New-AzureRMRoleAssignment` avec des portées différentes.

### <a name="provide-certificate-through-automated-powershell-script"></a>Fournir un certificat via un script PowerShell automatisé
Chaque fois que vous vous connectez en tant que principal de service, vous devez fournir l’ID de locataire du répertoire de votre application AD. Un locataire est une instance d’Azure Active Directory.

```powershell
Param (
 
 [Parameter(Mandatory=$true)]
 [String] $CertPath,

 [Parameter(Mandatory=$true)]
 [String] $CertPlainPassword,
 
 [Parameter(Mandatory=$true)]
 [String] $ApplicationId,

 [Parameter(Mandatory=$true)]
 [String] $TenantId
 )

 $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force
 $PFXCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($CertPath, $CertPassword)
 $Thumbprint = $PFXCert.Thumbprint

 Login-AzureRmAccount -ServicePrincipal -CertificateThumbprint $Thumbprint -ApplicationId $ApplicationId -TenantId $TenantId
```

L’ID d’application et l’ID de client ne sont pas sensibles, vous pouvez donc les incorporer directement dans votre script. Pour récupérer l’ID client, utilisez :

```powershell
(Get-AzureRmSubscription -SubscriptionName "Contoso Default").TenantId
```

Si vous avez besoin extraire l’ID de l’application, utilisez :

```powershell
(Get-AzureRmADApplication -DisplayNameStartWith {display-name}).ApplicationId
```

## <a name="change-credentials"></a>Modifier les informations d’identification

Pour modifier les informations d’identification d’une application Active Directory, en raison d’une faille de sécurité ou de l’expiration des informations d’identification, utilisez les applets de commande [Remove-AzureRmADAppCredential](/powershell/resourcemanager/azurerm.resources/v3.3.0/remove-azurermadappcredential) et [New-AzureRmADAppCredential](/powershell/module/azurerm.resources/new-azurermadappcredential).

Pour supprimer toutes les informations d’identification d’une application, utilisez :

```powershell
Remove-AzureRmADAppCredential -ApplicationId 8bc80782-a916-47c8-a47e-4d76ed755275 -All
```

Pour ajouter un mot de passe, utilisez :

```powershell
New-AzureRmADAppCredential -ApplicationId 8bc80782-a916-47c8-a47e-4d76ed755275 -Password p@ssword!
```

Pour ajouter une valeur de certificat, créez un certificat auto-signé, comme indiqué dans cette rubrique. Ensuite, utilisez :

```powershell
New-AzureRmADAppCredential -ApplicationId 8bc80782-a916-47c8-a47e-4d76ed755275 -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
```

## <a name="save-access-token-to-simplify-log-in"></a>Enregistrer le jeton d’accès pour simplifier la connexion
Pour éviter de fournir des informations d’identification au principal de service chaque fois qu’il a besoin de se connecter, vous pouvez enregistrer le jeton d’accès.

Pour utiliser le jeton d’accès actuel dans une session ultérieure, enregistrez le profil.
   
```powershell
Save-AzureRmProfile -Path c:\Users\exampleuser\profile\exampleSP.json
```
   
Ouvrez le profil et examinez-en le contenu. Notez qu’il contient un jeton d’accès. Au lieu de vous reconnecter manuellement, il vous suffit de charger le profil.
   
```powershell
Select-AzureRmProfile -Path c:\Users\exampleuser\profile\exampleSP.json
```

> [!NOTE]
> Le jeton d’accès arrive à expiration. L’utilisation d’un profil enregistré ne fonctionne donc que durant la validité du jeton.
>  

Vous pouvez également appeler des opérations REST à partir de PowerShell pour vous connecter. Dans la réponse d’authentification, vous pouvez récupérer le jeton d’accès en vue de l’utiliser avec d’autres opérations. Pour un exemple de récupération du jeton d’accès en appelant des opérations REST, consultez [Génération d’un jeton d’accès](resource-manager-rest-api.md#generating-an-access-token).

## <a name="debug"></a>Déboguer

Lors de la création d’un principal de service, vous pouvez rencontrer les erreurs suivantes :

* **« Authentication_Unauthorized »** ou **« Aucun abonnement trouvé dans le contexte. »** - Vous voyez cette erreur lorsque votre compte ne possède pas les [autorisations requises](#required-permissions) sur Azure Active Directory pour inscrire une application. En règle générale, vous obtenez cette erreur lorsque seuls des utilisateurs administrateurs dans votre annuaire Azure Active Directory peuvent inscrire des applications et que votre compte n’est pas un compte d’administrateur. Demandez à votre administrateur de vous affecter à un rôle d’administrateur ou d’autoriser les utilisateurs ordinaires à inscrire des applications.

* Le message indique que votre compte **« n’est pas autorisé à effectuer l’action ’Microsoft.Authorization/roleAssignments/write’ sur l’étendue ’/subscriptions/{guid}’ »** - Vous voyez cette erreur lorsque votre compte ne dispose pas d’autorisations suffisantes pour affecter un rôle à une identité. Demandez à votre administrateur d’abonnement de vous ajouter au rôle Administrateur de l’accès utilisateur.

## <a name="sample-applications"></a>Exemples d'applications
Les exemples d’applications suivants montrent comment ouvrir une session en tant que principal du service.

**.NET**

* [Deploy an SSH Enabled VM with a Template with .NET (Déployer une machine virtuelle compatible SSH à l’aide d’un modèle dans .NET)](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-template-deployment/)
* [Manage Azure resources and resource groups with .NET (Gérer des ressources et des groupes de ressources Azure avec .NET)](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-resources-and-groups/)

**Java**

* [Getting Started with Resources - Deploy Using Azure Resource Manager Template - in Java (Découverte des ressources - Déployer à l’aide du modèle Azure Resource Manager dans Java)](https://azure.microsoft.com/documentation/samples/resources-java-deploy-using-arm-template/)
* [Getting Started with Resources - Manage Resource Group - in Java (Découverte des ressources - Gérer un groupe de ressources dans Java)](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource-group//)

**Python**

* [Deploy an SSH Enabled VM with a Template in Python (Déployer une machine virtuelle compatible SSH à l’aide d’un modèle dans Python)](https://azure.microsoft.com/documentation/samples/resource-manager-python-template-deployment/)
* [Managing Azure Resource and Resource Groups with Python (Gérer des ressources et des groupes de ressources Azure avec Python)](https://azure.microsoft.com/documentation/samples/resource-manager-python-resources-and-groups/)

**Node.JS**

* [Deploy an SSH Enabled VM with a Template in Node.js (Déployer une machine virtuelle compatible SSH à l’aide d’un modèle dans Node.js)](https://azure.microsoft.com/documentation/samples/resource-manager-node-template-deployment/)
* [Manage Azure resources and resource groups with Node.js (Gérer des ressources et des groupes de ressources Azure avec Node.js)](https://azure.microsoft.com/documentation/samples/resource-manager-node-resources-and-groups/)

**Ruby**

* [Deploy an SSH Enabled VM with a Template in Ruby (Déployer une machine virtuelle compatible SSH à l’aide d’un modèle dans Ruby)](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-template-deployment/)
* [Managing Azure Resource and Resource Groups with Ruby (Gérer des ressources et des groupes de ressources Azure avec Ruby)](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-resources-and-groups/)

## <a name="next-steps"></a>Étapes suivantes
* Pour obtenir des instructions détaillées sur l’intégration d’une application à Azure pour la gestion des ressources, consultez [Guide du développeur pour l’authentification avec l’API Azure Resource Manager](resource-manager-api-authentication.md).
* Pour obtenir une explication plus détaillée des applications et des principaux du service, consultez la rubrique [Objets principal du service et application](../active-directory/active-directory-application-objects.md). 
* Pour plus d’informations sur l’authentification Azure Active Directory, consultez la rubrique [Scénarios d’authentification pour Azure AD](../active-directory/active-directory-authentication-scenarios.md).


