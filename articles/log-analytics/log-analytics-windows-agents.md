---
title: "Connecter des ordinateurs Windows à Azure Log Analytics | Microsoft Docs"
description: "Cet article décrit les étapes à suivre pour connecter directement au service Log Analytics les ordinateurs Windows de votre infrastructure locale en utilisant une version personnalisée de Microsoft Monitoring Agent (MMA)."
services: log-analytics
documentationcenter: 
author: bandersmsft
manager: carmonm
editor: 
ms.assetid: 932f7b8c-485c-40c1-98e3-7d4c560876d2
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/12/2017
ms.author: banders
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: 7c28fda22a08ea40b15cf69351e1b0aff6bd0a95
ms.openlocfilehash: 0868eb2269b3675a132e106cd66740b0ce52b00a
ms.lasthandoff: 03/07/2017


---
# <a name="connect-windows-computers-to-the-log-analytics-service-in-azure"></a>Connecter des ordinateurs Windows au service Log Analytics dans Azure

Cet article décrit les étapes à suivre pour connecter à des espaces de travail OMS les ordinateurs Windows de votre infrastructure locale en utilisant une version personnalisée de Microsoft Monitoring Agent (MMA). Vous devez installer et connecter les agents pour tous les ordinateurs que vous souhaitez intégrer afin qu’ils puissent envoyer des données au service Log Analytics, puis les afficher et agir sur ces données. Chaque agent peut fournir des rapports à plusieurs espaces de travail.

Vous pouvez installer des agents à l’aide du programme d’installation, de la ligne de commande, ou de la fonction Desired State Configuration (DSC) d’Azure Automation.  

>[!NOTE]
Pour les ordinateurs virtuels s’exécutant dans Azure, vous pouvez simplifier installation à l’aide de l’[extension de machine virtuelle](log-analytics-azure-vm-extension.md).

Sur les ordinateurs disposant d’une connexion à Internet, l’agent utilisera la connexion à Internet pour envoyer des données à OMS. Pour les ordinateurs qui ne disposent pas de connectivité Internet, vous pouvez utiliser un serveur proxy ou la [passerelle OMS](log-analytics-oms-gateway.md).

Vous pouvez connecter très simplement vos ordinateurs Windows à OMS en 3 étapes :

1. Télécharger le fichier de configuration de l’agent sur le portail OMS
2. Installer l’agent selon la méthode de votre choix
3. Configurer l’agent ou ajouter des espaces de travail supplémentaires, si nécessaire

Le schéma suivant illustre la relation entre vos ordinateurs Windows et OMS après installation et configuration des agents.

![oms-direct-agent-diagram](./media/log-analytics-windows-agents/oms-direct-agent-diagram.png)


## <a name="system-requirements-and-required-configuration"></a>Configuration requise
Avant d’installer ou de déployer des agents, passez en revue les informations suivantes afin d’avoir la certitude de répondre aux conditions requises.

- Vous ne pouvez installer OMS MMA que sur les ordinateurs fonctionnant sous Windows Server 2008 SP 1 ou version ultérieure ou sous Windows 7 SP1 ou version ultérieure.
- Vous avez besoin d’un abonnement OMS.  Pour plus d’informations, consultez l’article [Get started with Log Analytics](log-analytics-get-started.md)(Prise en main de Log Analytics).
- Chaque ordinateur Windows doit pouvoir se connecter à Internet à l’aide du protocole HTTPS ou de la passerelle OMS. Il peut s’agir d’une connexion directe, d’une connexion obtenue par un proxy ou bien établie via la passerelle OMS.
- Vous pouvez installer OMS MMA sur des ordinateurs autonomes, des serveurs et des machines virtuelles. Si vous souhaitez connecter à OMS des machines virtuelles hébergées dans Azure, voir [Connecter des machines virtuelles Azure à Log Analytics](log-analytics-azure-vm-extension.md).
- L’agent doit utiliser le port TCP 443 pour les différentes ressources. Pour plus d’informations, consultez la page [Configure proxy and firewall settings in Log Analytics](log-analytics-proxy-firewall.md)(Configuration des paramètres de proxy et de pare-feu dans Log Analytics).

## <a name="download-the-agent-setup-file-from-oms"></a>Télécharger le fichier de configuration de l’agent sur OMS
1. Dans le portail OMS, sur la page **Vue d’ensemble**, cliquez sur la mosaïque **Paramètres**.  Cliquez sur l’onglet **Sources connectées** , situé en haut de la page.  
    ![onglet Sources connectées](./media/log-analytics-windows-agents/oms-direct-agent-connected-sources.png)
2. Cliquez sur **Serveurs Windows**, puis sur **Télécharger l’Agent Windows** correspondant au type de processeur de votre ordinateur pour télécharger le fichier d’installation.
3. À droite du champ **ID de l’espace de travail**, cliquez sur l’icône de copie et collez l’ID dans le Bloc-notes.
4. À droite du champ **Clé primaire**, cliquez sur l’icône de copie et collez la clé dans le Bloc-notes.     

## <a name="install-the-agent-using-setup"></a>Installation de l’agent à l’aide du programme d’installation
1. Exécutez le programme d’installation pour installer l’agent sur un ordinateur que vous souhaitez gérer.
2. Sur la page d'accueil, cliquez sur **Suivant**.
3. Dans la page Termes du contrat de licence, lisez les conditions de licence et cliquez sur **J’accepte**.
4. Dans la page Dossier de destination, modifiez ou conservez le dossier d’installation par défaut, puis cliquez sur **Suivant**.
5. Dans la page Options d’installation de l’Agent, vous pouvez soit connecter l’agent à Azure Log Analytics (OMS) ou à Operations Manager, soit laisser les options vides si vous souhaitez configurer l’agent ultérieurement. Cliquez sur **Next**.   
    - Si vous choisissez de vous connecter à Azure Log Analytics (OMS), collez l’**ID de l’espace de travail** et la **Clé de l’espace de travail (clé primaire)** que vous avez copiés dans le bloc-notes dans la procédure précédente, puis cliquez sur **Suivant**.  
        ![Coller l’ID et la clé primaire de l’espace de travail](./media/log-analytics-windows-agents/connect-workspace.png)
    - Si vous choisissez de vous connecter à Operations Manager, saisissez le **Nom du groupe d’administration**, le nom du **Serveur d’administration** et le **Port du serveur d’administration**, puis cliquez sur **Suivant**. Sur la page Compte d’action d’agent, choisissez le compte système local ou un compte de domaine local, puis cliquez sur **Suivant**.  
        ![Configuration du groupe d’administration](./media/log-analytics-windows-agents/oms-mma-om-setup01.png)![compte d’action d’agent](./media/log-analytics-windows-agents/oms-mma-om-setup02.png)

6. Sur la page Prêt pour l’installation, passez en revue vos choix, puis cliquez sur **Installer**.
7. Sur la page Configuration effectuée, cliquez sur **Terminer**.
8. Lorsque vous avez terminé, **Microsoft Monitoring Agent** apparaît dans le **Panneau de configuration**. Vous pouvez y contrôler votre configuration et vérifier que l’agent est bien connecté à Operational Insights (OMS). Une fois connecté à OMS, l’agent affiche un message indiquant : **Microsoft Monitoring Agent est bien connecté au service Microsoft Operations Management Suite**

## <a name="install-the-agent-using-the-command-line"></a>Installation de l’agent à l’aide de la ligne de commande
- Modifiez, puis utilisez l’exemple suivant pour installer l’agent à l’aide de la ligne de commande. L’exemple effectue une installation entièrement sans assistance.

    >[!NOTE]
    Si vous souhaitez mettre à niveau un agent, vous devez utiliser l’API de script Log Analytics. Pour mettre à niveau un agent, consultez la section suivante.

    ```
    MMASetup-AMD64.exe /Q:A /R:N /C:"setup.exe /qn ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_ID=<your workspace id> OPINSIGHTS_WORKSPACE_KEY=<your workspace key> AcceptEndUserLicenseAgreement=1"
    ```

L’agent utilise IExpress comme son auto-extracteur à l’aide de la commande `/c`. Vous pouvez afficher les commutateurs de ligne de commande à la page [Commutateurs de ligne de commande pour les packages de mise à jour logicielle IExpress](https://support.microsoft.com/help/197147/command-line-switches-for-iexpress-software-update-packages), puis mettre à jour l’exemple pour répondre à vos besoins.

## <a name="upgrade-the-agent-and-add-a-workspace-using-a-script"></a>Mettre à niveau l’agent et ajouter un espace de travail à l’aide d’un script
Vous pouvez mettre à niveau un agent et ajouter un espace de travail à l’aide de l’API de script Log Analytics avec l’exemple PowerShell suivant.

```
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey)
$mma.ReloadConfiguration()
```

>[!NOTE]
Si vous avez utilisé précédemment la ligne de commande ou un script pour installer ou configurer l’agent, `EnableAzureOperationalInsights` a été remplacé par `AddCloudWorkspace`.

## <a name="install-the-agent-using-dsc-in-azure-automation"></a>Installer l’agent à l’aide de DSC dans Azure Automation

Vous pouvez utiliser l’exemple de script suivant pour installer l’agent à l’aide de DSC dans Azure Automation. L’exemple installe l’agent 64 bits, identifié par la valeur `URI`. Vous pouvez également utiliser la version 32 bits en remplaçant la valeur de l’URI. Les URI pour les deux versions sont :

- Agent Windows 64 bits - https://go.microsoft.com/fwlink/?LinkId=828603
- Agent Windows 32 bits - https://go.microsoft.com/fwlink/?LinkId=828604


>[!NOTE]
Cette procédure et cet exemple de script ne mettent pas à niveau un agent existant.

1. Importez le module DSC xPSDesiredStateConfiguration à partir de [http://www.powershellgallery.com/packages/xPSDesiredStateConfiguration](http://www.powershellgallery.com/packages/xPSDesiredStateConfiguration) dans Azure Automation.  
2.    Créez des ressources variables Azure Automation pour *OPSINSIGHTS_WS_ID* et *OPSINSIGHTS_WS_KEY*. Affectez votre ID d’espace de travail OMS Log Analytics comme valeur *OPSINSIGHTS_WS_ID* et affectez la clé primaire de votre espace de travail comme valeur *OPSINSIGHTS_WS_KEY*.
3.    Utilisez le script ci-dessous et enregistrez-le sous le nom MMAgent.ps1.
4.    Modifiez puis utilisez l’exemple suivant pour installer l’agent à l’aide de DSC dans Azure Automation. Importez MMAgent.ps1 dans Azure Automation à l’aide de l’interface ou de l’applet de commande Azure Automation.
5.    Affectez un nœud à la configuration. En moins de 15 minutes, le nœud vérifiera sa configuration et le MMA sera poussé vers le nœud.

```
Configuration MMAgent
{
    $OIPackageLocalPath = "C:\MMASetup-AMD64.exe"
    $OPSINSIGHTS_WS_ID = Get-AutomationVariable -Name "OPSINSIGHTS_WS_ID"
    $OPSINSIGHTS_WS_KEY = Get-AutomationVariable -Name "OPSINSIGHTS_WS_KEY"


    Import-DscResource -ModuleName xPSDesiredStateConfiguration

    Node OMSnode {
        Service OIService
        {
            Name = "HealthService"
            State = "Running"
            DependsOn = "[Package]OI"
        }

        xRemoteFile OIPackage {
            Uri = "https://go.microsoft.com/fwlink/?LinkId=828603"
            DestinationPath = $OIPackageLocalPath
        }

        Package OI {
            Ensure = "Present"
            Path  = $OIPackageLocalPath
            Name = "Microsoft Monitoring Agent"
            ProductId = "8A7F2C51-4C7D-4BFD-9014-91D11F24AAE2"
            Arguments = '/C:"setup.exe /qn ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_ID=' + $OPSINSIGHTS_WS_ID + ' OPINSIGHTS_WORKSPACE_KEY=' + $OPSINSIGHTS_WS_KEY + ' AcceptEndUserLicenseAgreement=1"'
            DependsOn = "[xRemoteFile]OIPackage"
        }
    }
}


```

### <a name="get-the-latest-productid-value"></a>Obtention de la dernière valeur de ProductId

La `ProductId value` dans le script MMAgent.ps1 est propre à chaque version de l’agent. Lorsqu’une version mise à jour de chaque agent est publiée, la valeur ProductId change. Par conséquent, lors du changement de la valeur ProductId à l’avenir, vous trouverez la version de l’agent à l’aide d’un script simple. Une fois la dernière version de l’agent installée sur un serveur de test, vous pouvez utiliser le script suivant pour obtenir la valeur ProductId installée. À l’aide de la dernière valeur ProductId, vous pouvez mettre à jour la valeur dans le script MMAgent.ps1.

```
$InstalledApplications  = Get-ChildItem hklm:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall


foreach ($Application in $InstalledApplications)

{

     $Key = Get-ItemProperty $Application.PSPath

     if ($Key.DisplayName -eq "Microsoft Monitoring Agent")

     {

        $Key.DisplayName

        Write-Output ("Product ID is: " + $Key.PSChildName.Substring(1,$Key.PSChildName.Length -2))

     }

}  
```

## <a name="configure-an-agent-manually-or-add-additional-workspaces"></a>Configurer un agent manuellement ou ajouter des espaces de travail supplémentaires
Si vous avez installé des agents mais que vous ne les avez pas configurés, ou si vous souhaitez que l’agent fournisse des rapports à plusieurs espaces de travail, vous pouvez utiliser les informations suivantes pour activer un agent ou le reconfigurer. Une fois que vous avez configuré l’agent, il s’inscrira auprès du service de l’agent et obtiendra les informations de configuration nécessaires, ainsi que les packs d’administration contenant les informations de la solution.

1. Après avoir installé l’agent Microsoft Monitoring Agent, ouvrez le **Panneau de configuration**.
2. Ouvrez l’agent **Microsoft Monitoring Agent**, puis cliquez sur l’onglet **Azure Log Analytics (OMS)**.   
3. Cliquez sur **Ajouter** pour ouvrir la boîte de dialogue **Ajouter un espace de travail Log Analytics**.
4. Collez l’**ID de l’espace de travail** et la **Clé de l’espace de travail (clé primaire)** que vous avez copiés dans le bloc-notes au cours d’une étape précédente pour l’espace de travail que vous souhaitez ajouter, puis cliquez sur **OK**.  
    ![Configurer Operational Insights](./media/log-analytics-windows-agents/add-workspace.png)

Une fois que les données des ordinateurs analysés ont été collectées par l’agent, le nombre d’ordinateurs analysés par OMS apparaît dans le portail OMS sous l’onglet **Sources connectées**, dans **Paramètres**, sous **Serveurs connectés**.


## <a name="to-disable-an-agent"></a>Désactivation d’un agent
1. Après avoir installé l’agent, ouvrez le **Panneau de configuration**.
2. Ouvrez l’agent Microsoft Monitoring Agent, puis cliquez sur l’onglet **Azure Log Analytics (OMS)** .
3. Sélectionnez un espace de travail, puis cliquez sur **Supprimer**. Répétez cette étape pour tous les autres espaces de travail.


## <a name="optionally-configure-agents-to-report-to-an-operations-manager-management-group"></a>Vous pouvez éventuellement configurer les agents pour émettre des rapports dans un groupe d’administration Operations Manager

Si vous utilisez Operations Manager dans votre infrastructure informatique, vous pouvez également utiliser l’agent MMA en tant qu’agent Operations Manager.

### <a name="to-configure-mma-agents-to-report-to-an-operations-manager-management-group"></a>Configurer les agents afin qu’ils émettent des rapports dans un groupe d’administration Operations Manager
1.    Sur l’ordinateur sur lequel l’agent est installé, ouvrez le **Panneau de configuration**.  
2.    Ouvrez l’agent **Microsoft Monitoring Agent**, puis cliquez sur l’onglet **Operations Manager**.  
    ![onglet Operations Manager de Microsoft Monitoring Agent](./media/log-analytics-windows-agents/om-mg01.png)
3.    Si vos serveurs Operations Manager sont intégrés à Active Directory, cliquez sur **Met à jour automatiquement les attributions du groupe d’administration à partir d’AD DS**.
4.    Cliquez sur **Ajouter** pour ouvrir la boîte de dialogue **Ajouter un groupe d’administration**.  
    ![Microsoft Monitoring Agent Ajouter un groupe d’administration](./media/log-analytics-windows-agents/oms-mma-om02.png)
5.    Dans la zone **Nom du groupe d’administration** , entrez le nom de votre groupe d’administration.
6.    Dans la zone **Serveur d’administration principal** , entrez le nom d’ordinateur du serveur d’administration principal.
7.    Dans la zone **Port du serveur d’administration** , entrez le numéro de port TCP.
8.    Sous **Compte d’action d’agent**, choisissez le compte système local ou un compte de domaine local.
9.    Cliquez sur **OK** pour fermer la boîte de dialogue **Ajouter un groupe d’administration**, puis cliquez sur **OK** pour fermer la boîte de dialogue **Propriétés de l’agent Microsoft Monitoring Agent**.

## <a name="optionally-configure-agents-to-use-the-oms-gateway"></a>Vous pouvez éventuellement configurer les agents afin qu’ils utilisent la passerelle OMS

Si vos serveurs ou clients ne disposent pas de connexion à Internet, vous pouvez toujours utiliser OMS Log Analytics Forwarder pour qu’ils puissent envoyer des données à OMS avec la passerelle OMS.  Quand vous utilisez la passerelle, toutes les données des agents transitent par un seul serveur ayant accès à Internet. La passerelle transfère directement les données des agents à OMS sans analyser les données transférées.

Consultez [Passerelle OMS](log-analytics-oms-gateway.md) pour en savoir plus sur la passerelle, y compris son installation et sa configuration.

Pour plus d’informations sur la configuration de vos agents pour qu’ils utilisent un serveur proxy (c’est-à-dire ici la passerelle OMS), consultez [Configurer les paramètres de pare-feu et de proxy dans Log Analytics](log-analytics-proxy-firewall.md).

## <a name="optionally-configure-proxy-and-firewall-settings"></a>Configuration des paramètres de pare-feu et de proxy (facultatif)
Si les serveurs proxy ou les pare-feu de votre environnement limitent votre accès à Internet, consultez la page [Configurer les paramètres de pare-feu et de proxy dans Log Analytics](log-analytics-proxy-firewall.md) pour savoir comment configurer vos agents pour communiquer avec le service OMS.

## <a name="next-steps"></a>Étapes suivantes

- [Ajoutez des solutions Log Analytics à partir de la galerie de solutions](log-analytics-add-solutions.md) pour ajouter des fonctionnalités et collecter des données.
- [Configurez les paramètres de proxy et de pare-feu dans Log Analytics](log-analytics-proxy-firewall.md) si votre organisation utilise un serveur proxy ou un pare-feu pour que les agents puissent communiquer avec le service Log Analytics.

