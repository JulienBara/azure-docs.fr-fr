---
title: "Équilibrage de la charge des machines virtuelles Linux dans Azure | Microsoft Docs"
description: "Découvrez comment utiliser l’équilibreur de charge Azure pour créer une application hautement disponible et sécurisée entre trois machines virtuelles Linux"
services: virtual-machines-linux
documentationcenter: virtual-machines
author: iainfoulds
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/17/2017
ms.author: iainfou
ms.translationtype: Human Translation
ms.sourcegitcommit: be3ac7755934bca00190db6e21b6527c91a77ec2
ms.openlocfilehash: 5ff735100132e8571871b41ac2309334662adb7f
ms.contentlocale: fr-fr
ms.lasthandoff: 05/03/2017

---

# <a name="how-to-load-balance-linux-virtual-machines-in-azure-to-create-a-highly-available-application"></a>Équilibrage de la charge des machines virtuelles Linux dans Azure pour créer une application hautement disponible
Dans ce didacticiel, vous allez découvrir les différents composants de l’équilibreur de charge Azure qui répartissent le trafic et fournissent une haute disponibilité. Pour voir l’équilibreur de charge en action, vous allez générer une application Node.js qui s’exécutera sur trois machines virtuelles Linux.

Les étapes de ce didacticiel peuvent être effectuées à l’aide de la dernière version [d’Azure CLI 2.0](/cli/azure/install-azure-cli).


## <a name="azure-load-balancer-overview"></a>Vue d’ensemble de l’équilibreur de charge Azure
Un équilibreur de charge Azure est un équilibreur de charge de type Couche 4 (TCP, UDP) qui offre une haute disponibilité en répartissant le trafic entrant entre les machines virtuelles saines. Une sonde d’intégrité d’équilibreur de charge surveille un port donné sur chaque machine virtuelle et ne distribue le trafic que vers une machine virtuelle opérationnelle.

Vous définissez une configuration IP frontale qui contient une ou plusieurs adresses IP publiques. Cette configuration IP frontale permet d’accéder à votre équilibreur de charge et à vos applications via Internet. 

Les machines virtuelles se connectent à un équilibreur de charge à l’aide de leur carte d’interface réseau virtuelle (NIC). Pour distribuer le trafic vers les machines virtuelles, un pool d’adresses principal contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge.

Pour contrôler le flux de trafic, vous définissez les règles d’équilibrage de charge pour les ports et les protocoles spécifiques qui correspondent à vos machines virtuelles.

Si vous avez suivi le didacticiel précédent pour [créer un groupe de machines virtuelles identiques](tutorial-create-vmss.md), un équilibreur de charge a été créé pour vous. Tous ces composants ont été configurés pour vous dans le cadre du groupe identique.


## <a name="create-azure-load-balancer"></a>Créer un équilibreur de charge Azure
Cette section explique en détail comment vous pouvez créer et configurer chaque composant de l’équilibreur de charge. Pour pouvoir créer votre équilibreur de charge, vous devez créer un groupe de ressources avec la commande [az group create](/cli/azure/group#create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroupLoadBalancer* à l’emplacement *westus* :

```azurecli
az group create --name myRGLoadBalancer --location westus
```

### <a name="create-a-public-ip-address"></a>Créer une adresse IP publique
Pour accéder à votre application sur Internet, vous avez besoin d’une adresse IP publique pour l’équilibreur de charge. Créez une adresse IP publique avec la commande [az network public-ip create](/cli/azure/public-ip#create). L’exemple suivant crée une adresse IP publique nommée *myPublicIP* dans le groupe de ressources *myRGLoadBalancer* :

```azurecli
az network public-ip create \
    --resource-group myRGLoadBalancer \
    --name myPublicIP
```

### <a name="create-a-load-balancer"></a>Créer un équilibrage de charge
Créez un équilibrage de charge avec la commande [az network lb create](/cli/azure/network/lb#create). L’exemple suivant crée un équilibreur de charge nommé *myLoadBalancer* et affecte l’adresse *myPublicIP* à la configuration IP frontale :

```azurecli
az network lb create \
    --resource-group myRGLoadBalancer \
    --name myLoadBalancer \
    --frontend-ip-name myFrontEndPool \
    --backend-pool-name myBackEndPool \
    --public-ip-address myPublicIP
```

### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité
Pour permettre à l’équilibrage de charge de surveiller l’état de votre application, vous utilisez une sonde d’intégrité. La sonde d’intégrité ajoute ou supprime dynamiquement des machines virtuelles de la rotation d’équilibrage de charge en fonction de leur réponse aux vérifications d’intégrité. Par défaut, une machine virtuelle est supprimée de la distribution d’équilibrage de charge après deux échecs consécutifs à des intervalles de 15 secondes. Vous créez une sonde d’intégrité selon un protocole ou une page de vérification d’intégrité spécifique pour votre application. 

L’exemple suivant permet de créer une sonde TCP. Vous pouvez également créer des sondes HTTP personnalisées pour des contrôles d’intégrité plus affinés. Lorsque vous utilisez une sonde HTTP personnalisée, vous devez créer la page de contrôle d’intégrité, par exemple *healthcheck.js*. La sonde doit retourner une réponse **HTTP 200 OK** pour l’équilibreur de charge pour assurer la rotation de l’hôte.

Pour créer une sonde d’intégrité TCP, utilisez la commande [az network lb probe create](/cli/azure/network/lb/probe#create). L’exemple suivant permet de créer une sonde d’intégrité nommée *myHealthProbe* :

```azurecli
az network lb probe create \
    --resource-group myRGLoadBalancer \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge
Une règle d’équilibrage de charge est utilisée pour définir la distribution du trafic vers les machines virtuelles. Vous définissez la configuration IP frontale pour le trafic entrant et le pool d’adresses IP principal pour recevoir le trafic, ainsi que le port source et le port de destination requis. Pour veiller à ce que seules les machines virtuelles saines reçoivent le trafic, vous devez également définir la sonde d’intégrité à utiliser.

Utilisez [az network lb rule create](/cli/azure/network/lb/rule#create) pour créer une règle d’équilibrage de charge. L’exemple suivant crée une règle nommée *myLoadBalancerRule*, utilise la sonde d’intégrité *myHealthProbe* et équilibre le trafic sur le port *80* :

```azurecli
az network lb rule create \
    --resource-group myRGLoadBalancer \
    --lb-name myLoadBalancer \
    --name myLoadBalancerRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEndPool \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe
```


## <a name="configure-virtual-network"></a>Configurer un réseau virtuel
Avant de déployer des machines virtuelles et de pouvoir tester votre équilibreur, créez les ressources de réseau virtuel de prise en charge. Pour plus d’informations sur les réseaux virtuels, consultez le didacticiel [Manage Azure Virtual Networks (Gérer les réseaux virtuels Azure)](tutorial-virtual-network.md).

### <a name="create-network-resources"></a>Créer des ressources réseau
Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/vnet#create). L’exemple suivant crée un réseau virtuel nommé *myVnet* avec un sous-réseau nommé *mySubnet* :

```azurecli
az network vnet create \
    --resource-group myRGLoadBalancer \
    --name myVnet \
    --subnet-name mySubnet
```

Pour ajouter un groupe de sécurité réseau, utilisez la commande [az network nsg create](/cli/azure/network/nsg#create). L’exemple suivant crée un groupe de sécurité réseau nommé *myNetworkSecurityGroup* :

```azurecli
az network nsg create \
    --resource-group myRGLoadBalancer \
    --name myNetworkSecurityGroup
```

Créez une règle de groupe de sécurité réseau avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule#create). L’exemple suivant crée une règle de groupe de sécurité réseau nommée *myNetworkSecurityGroupRule* :

```azurecli
az network nsg rule create \
    --resource-group myRGLoadBalancer \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRule \
    --priority 1001 \
    --protocol tcp \
    --destination-port-range 80
```

Les cartes d’interface réseau virtuelles sont créées avec la commande [az network nic create](/cli/azure/network/nic#create). L’exemple suivant crée trois cartes réseau virtuelles. (Une carte d’interface réseau virtuelle pour chaque machine virtuelle que vous créez pour votre application dans les étapes suivantes). Vous pouvez créer des machines virtuelles et des cartes d’interface réseau virtuelles supplémentaires à tout moment et les ajouter à l’équilibreur de charge :

```bash
for i in `seq 1 3`; do
    az network nic create \
        --resource-group myRGLoadBalancer \
        --name myNic$i \
        --vnet-name myVnet \
        --subnet mySubnet \
        --network-security-group myNetworkSecurityGroup \
        --lb-name myLoadBalancer \
        --lb-address-pools myBackEndPool
done
```

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

### <a name="create-cloud-init-config"></a>Créer une configuration cloud-init
Dans le didacticiel précédent [How to customize a Linux virtual machine on first boot (Personnalisation d’une machine virtuelle Linux au premier démarrage)](tutorial-automate-vm-deployment.md), vous avez appris à automatiser la personnalisation des machines virtuelles avec cloud-init. Vous pouvez utiliser le même fichier de configuration cloud-init pour installer NGINX et exécuter une simple application Node.js « Hello World ». Créez un fichier nommé *cloud-init.txt* et collez la configuration suivante :

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

### <a name="create-virtual-machines"></a>Créer des machines virtuelles
Pour améliorer la haute disponibilité de votre application, placez vos machines virtuelles dans un groupe à haute disponibilité. Pour plus d’informations sur les groupes à haute disponibilité, consultez le didacticiel précédent [How to create highly available virtual machines (Création de machines virtuelles hautement disponibles)](tutorial-availability-sets.md).

Créez un groupe à haute disponibilité avec la commande [az vm availability-set create](/cli/azure/vm/availability-set#create). L’exemple suivant permet de créer un groupe à haute disponibilité nommé *myAvailabilitySet* :

```azurecli
az vm availability-set create \
    --resource-group myRGLoadBalancer \
    --name myAvailabilitySet \
    --platform-fault-domain-count 3 \
    --platform-update-domain-count 2
```

À présent, créez les machines virtuelles avec la commande [az vm create](/cli/azure/vm#create). L’exemple suivant crée trois machines virtuelles et génère des clés SSH si elles n’existent pas déjà :

```bash
for i in `seq 1 3`; do
    az vm create \
        --resource-group myRGLoadBalancer \
        --name myVM$i \
        --availability-set myAvailabilitySet \
        --nics myNic$i \
        --image Canonical:UbuntuServer:14.04.4-LTS:latest \
        --admin-username azureuser \
        --generate-ssh-keys \
        --custom-data cloud-init.txt \
        --no-wait
done
```

La création et la configuration des trois machines virtuelles prennent quelques minutes. La sonde d’intégrité de l’équilibrage de charge détecte automatiquement lorsque l’application est en cours d’exécution sur chaque machine virtuelle. Une fois que l’application est en cours d’exécution, la règle d’équilibrage de charge commence à distribuer le trafic.


## <a name="test-load-balancer"></a>Tester l’équilibreur de charge
Obtenez l’adresse IP publique de votre équilibrage de charge avec [az network public-ip show](/cli/azure/network/public-ip#show). L’exemple suivant obtient l’adresse IP pour *myPublicIP* créée précédemment :

```azurecli
az network public-ip show \
    --resource-group myRGLoadBalancer \
    --name myPublicIP \
    --query [ipAddress] \
    --output tsv
```

Vous pouvez alors entrer l’adresse IP publique dans un navigateur web. L’application s’affiche, avec notamment le nom d’hôte de la machine virtuelle sur laquelle l’équilibreur de charge a distribué le trafic comme dans l’exemple suivant :

![Exécution de l’application Node.js](./media/tutorial-load-balancer/running-nodejs-app.png)

Pour visualiser la distribution de trafic par l’équilibreur de charge sur les trois machines virtuelles exécutant votre application, vous pouvez forcer l’actualisation de votre navigateur web.


## <a name="add-and-remove-vms"></a>Ajouter et supprimer des machines virtuelles
Vous devrez peut-être effectuer la maintenance sur la machine virtuelle exécutant votre application, avec par exemple l’installation des mises à jour du système d’exploitation. Pour faire face à une augmentation du trafic vers votre application, vous devrez peut-être ajouter des machines virtuelles supplémentaires. Cette section vous indique comment supprimer ou ajouter une machine virtuelle pour l’équilibrage de charge.

### <a name="remove-a-vm-from-the-load-balancer"></a>Supprimer une machine virtuelle de l’équilibrage de charge
Vous pouvez supprimer une machine virtuelle à partir du pool d’adresses principal avec la commande [az network nic ip-config address-pool remove](/cli/azure/network/nic/ip-config/address-pool#remove). L’exemple suivant supprime la carte réseau virtuelle pour **myVM2** de *myLoadBalancer* :

```azurecli
az network nic ip-config address-pool remove \
    --resource-group myRGLoadBalancer \
    --nic-name myNic2 \
    --ip-config-name ipConfig1 \
    --lb-name myLoadBalancer \
    --address-pool myBackEndPool 
```

Pour visualiser la distribution de trafic par l’équilibreur de charge sur les deux machines virtuelles restantes exécutant votre application, vous pouvez forcer l’actualisation de votre navigateur web. Vous pouvez maintenant effectuer la maintenance sur la machine virtuelle, avec par exemple l’installation des mises à jour du système d’exploitation ou le redémarrage de la machine virtuelle.

### <a name="add-a-vm-to-the-load-balancer"></a>Ajouter une machine virtuelle à l’équilibrage de charge
Après avoir effectué les opérations de maintenance de la machine virtuelle, ou si vous devez développer la capacité, vous pouvez ajouter une machine virtuelle au pool d’adresses principal avec la commande [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#add). L’exemple suivant ajoute la carte réseau virtuelle pour **myVM2** à *myLoadBalancer* :

```azurecli
az network nic ip-config address-pool add \
    --resource-group myRGLoadBalancer \
    --nic-name myNic2 \
    --ip-config-name ipConfig1 \
    --lb-name myLoadBalancer \
    --address-pool myBackEndPool
```


## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à créer un équilibreur de charge pour les machines virtuelles. Passez au didacticiel suivant pour en savoir plus sur les composants de réseau virtuel Azure.

[Gérer la mise en réseau d’une machine virtuelle](tutorial-virtual-network.md)

