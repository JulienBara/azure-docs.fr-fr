---
title: "Questions sur le service de base de données DocumentDB - Forum Aux Questions (FAQ) | Microsoft Docs"
description: "Obtenez des réponses aux questions les plus fréquentes à propos du service de base de données de documents NoSQL DocumentDB Azure pour JSON. Répondez aux questions sur la capacité, les niveaux de performance et l’évolutivité de la base de données."
keywords: "Questions sur la base de données, Forum aux questions, documentdb, azure, Microsoft azure"
services: documentdb
author: mimig1
manager: jhubbard
editor: monicar
documentationcenter: 
ms.assetid: b68d1831-35f9-443d-a0ac-dad0c89f245b
ms.service: documentdb
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/08/2017
ms.author: mimig
translationtype: Human Translation
ms.sourcegitcommit: 97acd09d223e59fbf4109bc8a20a25a2ed8ea366
ms.openlocfilehash: 8ebc1aa663f298d1f3f495523d85bda8777d5d29
ms.lasthandoff: 03/10/2017


---
# <a name="frequently-asked-questions-about-documentdb"></a>Forum Aux Questions (FAQ) sur DocumentDB
## <a name="database-questions-about-microsoft-azure-documentdb-fundamentals"></a>Questions à propos des concepts de base de Microsoft Azure DocumentDB
### <a name="what-is-microsoft-azure-documentdb"></a>Qu'est-ce que Microsoft Azure DocumentDB ?
Microsoft Azure DocumentDB est un service de base de données NoSQL orienté documents, à l’échelle mondiale, très rapide, qui propose des requêtes enrichies plutôt que des données sans schéma. Il offre des options de configuration et des performances fiables, tout en permettant un développement rapide. Tout cela est rendu possible grâce à une plateforme gérée, soutenue par la puissance et la portée de Microsoft Azure. DocumentDB est la solution idéale pour les applications web, mobiles, de jeu et IoT lorsque le débit, la haute disponibilité, la faible latence et le modèle de données sans schéma sont primordiaux. DocumentDB offre une flexibilité des schémas et une indexation riche par le biais d'un modèle de données JSON natif. Il inclut également un support transactionnel multi-documents avec JavaScript intégré.  

Pour plus de questions, de réponses et d’instructions sur le déploiement et l’utilisation de ce service, consultez la [page de documentation de DocumentDB](https://azure.microsoft.com/documentation/services/documentdb/).

### <a name="what-kind-of-database-is-documentdb"></a>Quel type de base de données est DocumentDB ?
DocumentDB est une base de données NoSQL orientée documents qui stocke les données au format JSON.  DocumentDB prend en charge les structures imbriquées et autonomes qui peuvent être interrogées grâce à une [syntaxe de requête SQL](documentdb-sql-query.md)DocumentDB enrichie. DocumentDB offre un traitement transactionnel haute performance JavaScript côté serveur par le biais de [procédures stockées, de déclencheurs et de fonctions définies par l’utilisateur](documentdb-programming.md). La base de données prend également en charge des niveaux de cohérence ajustables avec les [niveaux de performances](documentdb-performance-levels.md)associés.

### <a name="do-documentdb-databases-have-tables-like-a-relational-database-rdbms"></a>Les bases de données DocumentDB ont-elles des tables comme une base de données relationnelle (SGBDR) ?
Non, DocumentDB stocke les données dans des collections sous forme de documents JSON.  Pour plus d’informations sur les ressources de DocumentDB, consultez la rubrique [Modèle de ressource et concepts de DocumentDB](documentdb-resources.md). Pour plus d’informations sur les différences entre les solutions NoSQL comme DocumentDB et les solutions relationnelles, consultez [NoSQL et SQL](documentdb-nosql-vs-sql.md).

### <a name="do-documentdb-databases-support-schema-free-data"></a>Les bases de données DocumentDB prennent-elles en charge les données sans schéma ?
Oui, DocumentDB permet aux applications de stocker des documents JSON arbitrairement sans définition de schéma ni indice. Les données peuvent être interrogées immédiatement via l'interface de requête SQL de DocumentDB.   

### <a name="does-documentdb-support-acid-transactions"></a>DocumentDB prend-il en charge les transactions ACID ?
Oui, DocumentDB prend en charge les transactions entre documents exprimées en procédures stockées et déclencheurs JavaScript. Les transactions sont étendues à une seule partition au sein de chaque collection et exécutées intégralement avec des sémantiques ACID, isolées du code et des requêtes utilisateur en cours d’exécution.  Si des exceptions surviennent lors de l’exécution du code d’application JavaScript côté serveur, la transaction entière est annulée. Pour plus d’informations sur les transactions, consultez [Transactions de programme de base de données](documentdb-programming.md#database-program-transactions).

### <a name="what-are-the-typical-use-cases-for-documentdb"></a>Dans quels cas utilise-t-on généralement DocumentDB ?
DocumentDB est une option conseillée pour les nouvelles applications web, mobiles, de jeu et IoT dans lesquelles la mise à l’échelle automatique, les performances prévisibles, la commande rapide de temps de réponse en millisecondes, et la capacité à interroger sur des données sans schéma sont importants. DocumentDB permet un développement rapide et prend en charge l'itération continue des modèles de données d'application. Les applications qui gèrent du contenu généré par l’utilisateur et des données sont [communément utilisées dans DocumentDB](documentdb-use-cases.md).  

### <a name="how-does-documentdb-offer-predictable-performance"></a>Comment DocumentDB offre-t-il des performances prévisibles ?
Une [unité de requête](documentdb-request-units.md) est la mesure du débit dans DocumentDB. 1 RU correspond au débit de la requête GET d’un document d’1 Ko. Chaque opération dans DocumentDB, y compris les lectures, les écritures, les requêtes SQL et les exécutions de procédures stockées, comporte une valeur d’unité de requête déterministe basée sur le débit nécessaire pour terminer l’opération. Au lieu de penser à l’UC, à l’E/S et à la mémoire, et à la façon dont ils impactent le débit de votre application, vous pouvez penser en termes de mesure d’unité de requête unique.

Chaque collection DocumentDB peut être réservée avec un débit approvisionné en termes d’unités de requête de débit par seconde. Pour les applications quelle que soit leur échelle, vous pouvez évaluer les requêtes individuelles pour mesurer leur valeur d’unités de requête, et approvisionner les collections pour gérer la somme totale des unités de requête sur l’ensemble des requêtes. Vous pouvez également mettre à l’échelle le débit de votre collection à mesure de l’évolution des besoins de votre application. Pour plus d’informations sur les unités de requête et pour obtenir de l’aide afin de déterminer vos besoins en matière de collections, consultez [Estimation des besoins de débit](documentdb-request-units.md#estimating-throughput-needs) et essayez la [calculatrice de débit](https://www.documentdb.com/capacityplanner).

### <a name="is-documentdb-hipaa-compliant"></a>DocumentDB est-il conforme à HIPAA ?
Oui, DocumentDB est conforme à HIPAA. HIPAA établit les conditions requises pour l’utilisation, la divulgation et la protection des informations de santé identifiables de façon individuelle. Pour plus d’informations, consultez le [Centre de gestion de la confidentialité de Microsoft](https://www.microsoft.com/en-us/TrustCenter/Compliance/HIPAA).

### <a name="what-are-the-storage-limits-of-documentdb"></a>Quelles sont les limites de stockage de DocumentDB ?
Il n’existe aucune limite à la quantité totale de données qu’une collection peut stocker dans DocumentDB.

### <a name="what-are-the-throughput-limits-of-documentdb"></a>Quelles sont les limites de débit de DocumentDB ?
Il n’existe aucune limite à la quantité totale de débit qu’une collection peut prendre en charge dans DocumentDB, si votre charge de travail peut être distribuée à peu près uniformément entre un nombre suffisant de clés de partition.

### <a name="how-much-does-microsoft-azure-documentdb-cost"></a>Combien coûte Microsoft Azure DocumentDB ?
Pour plus d’informations, consultez la page [Tarification de DocumentDB](https://azure.microsoft.com/pricing/details/documentdb/) . Les frais d’utilisation de DocumentDB sont déterminés par le nombre de collections configurées, le nombre d’heures durant lequel les collections sont en ligne et le débit configuré pour chaque collection.

### <a name="is-there-a-free-account-available"></a>Un compte gratuit est-il disponible ?
Si vous débutez avec Azure, vous pouvez vous inscrire pour bénéficier d’un [compte Azure gratuit](https://azure.microsoft.com/free/), qui vous offre 30 jours et 200 USD pour essayer tous les services Azure. Ou, si vous possédez un abonnement Visual Studio, vous pouvez bénéficier de [150 USD de crédits Azure gratuits par mois](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/), à utiliser sur n’importe quel service Azure.  

Vous pouvez également utiliser [l’émulateur Azure DocumentDB](documentdb-nosql-local-emulator.md) pour développer et tester votre application localement, sans créer d’abonnement Azure et sans frais. Lorsque vous êtes satisfait du fonctionnement de votre application dans l’émulateur DocumentDB, vous pouvez commencer à utiliser un compte Azure DocumentDB dans le cloud.

### <a name="how-can-i-get-additional-help-with-documentdb"></a>Comment puis-je obtenir une aide supplémentaire avec DocumentDB ?
Si vous avez besoin d’aide, contactez-nous sur [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-documentdb) ou planifiez une conversation 1 à 1 avec l’équipe d’ingénierie de DocumentDB en envoyant un message à [askdocdb@microsoft.com](mailto:askdocdb@microsoft.com). Pour vous tenir informé des dernières nouveautés et fonctionnalités de DocumentDB, suivez-nous sur [Twitter](https://twitter.com/DocumentDB).

## <a name="set-up-microsoft-azure-documentdb"></a>Configuration de Microsoft Azure DocumentDB
### <a name="how-do-i-sign-up-for-microsoft-azure-documentdb"></a>Comment m’inscrire à Microsoft Azure DocumentDB ?
Microsoft Azure DocumentDB est disponible dans le [portail Azure][azure-portal].  Vous devez d'abord disposer d'un abonnement à Microsoft Azure.  Une fois inscrit à Microsoft Azure, vous pouvez ajouter un compte DocumentDB à votre abonnement Azure. Pour obtenir des instructions sur l’ajout d’un compte DocumentDB, consultez la page [Création d’un compte de base de données DocumentDB](documentdb-create-account.md).   

### <a name="what-is-a-master-key"></a>Qu'est-ce que la clé principale ?
Une clé principale est un jeton de sécurité permettant d'accéder à toutes les ressources d'un compte. Les personnes disposant de cette clé ont un accès en lecture et en écriture à toutes les ressources du compte de la base de données. Faites preuve de précaution lorsque vous distribuez des clés principales. La clé principale primaire et la clé principale secondaire sont disponibles dans le panneau **Clés** du [portail Azure][azure-portal]. Pour plus d’informations sur les clés, consultez la rubrique [Affichage, copie et régénération de clés d’accès](documentdb-manage-account.md#keys).

### <a name="how-do-i-create-a-database"></a>Comment créer une base de données ?
Vous pouvez créer des bases de données à l’aide du [portail Azure](), comme décrit dans la rubrique [Création d’une collection et d’une base de données DocumentDB](documentdb-create-collection.md), à l’aide d’un des [Kits de développement logiciel (SDK) DocumentDB](documentdb-sdk-dotnet.md) ou au moyen des [API REST](https://msdn.microsoft.com/library/azure/dn781481.aspx).  

### <a name="what-is-a-collection"></a>Qu'est-ce qu'une collection ?
Une collection est un conteneur de documents JSON. Elle est associée à une logique d'application JavaScript. Une collection est une entité facturable, où le [coût](documentdb-performance-levels.md) est déterminé par le débit et le stockage utilisés. Les collections peuvent couvrir une ou plusieurs partitions/serveurs et peuvent être mises à l’échelle pour gérer des volumes de stockage ou de débit quasi-illimités.

Les collections sont également les entités de facturation de DocumentDB. Chaque collection est facturée sur la base du débit approvisionné et de l’espace de stockage utilisé. Pour plus d’informations, consultez [Tarification de DocumentDB](https://azure.microsoft.com/pricing/details/documentdb/).  

### <a name="how-do-i-set-up-users-and-permissions"></a>Comment configurer des utilisateurs et des autorisations ?
Vous pouvez créer des utilisateurs et des autorisations en utilisant un [Kit de développement logiciel (SDK) DocumentDB](documentdb-sdk-dotnet.md) ou des [API REST](https://msdn.microsoft.com/library/azure/dn781481.aspx).   

## <a name="database-questions-about-developing-against-microsoft-azure-documentdb"></a>Questions à propos du développement avec Microsoft Azure DocumentDB
### <a name="how-to-do-i-start-developing-against-documentdb"></a>Comment développer avec DocumentDB ?
[Kits de développement logiciel (SDK)](documentdb-sdk-dotnet.md) sont disponibles pour .NET, Python, Node.js, JavaScript et Java.  Les développeurs peuvent également utiliser les [API RESTful HTTP](https://msdn.microsoft.com/library/azure/dn781481.aspx) pour interagir avec les ressources DocumentDB sur différentes plateformes et dans de nombreux langages.

Des exemples de kits de développement logiciel (SDK) DocumentDB [.NET](documentdb-dotnet-samples.md), [Java](https://github.com/Azure/azure-documentdb-java), [Node.js](documentdb-nodejs-samples.md) et [Python](documentdb-python-samples.md) sont disponibles sur GitHub.

### <a name="does-documentdb-support-sql"></a>DocumentDB prend-il en charge SQL ?
Le langage de requête SQL DocumentDB est un sous-ensemble amélioré des fonctionnalités de requête prises en charge par SQL. Le langage de requête SQL de DocumentDB fournit des opérateurs relationnels et hiérarchiques enrichis ainsi qu’une extensibilité par le biais de fonctions JavaScript définies par l’utilisateur. La syntaxe JSON permet la modélisation de documents JSON en tant qu’arborescences avec des étiquettes en tant que nœuds d’arborescence. Cette capacité est exploitée par les techniques d’indexation automatique de DocumentDB et le langage de requête SQL de ce dernier.  Pour plus d’informations sur l’utilisation de la syntaxe SQL, consultez l’article relatif à [l’interrogation de DocumentDB][query].

### <a name="does-documentdb-support-sql-aggregation-functions"></a>DocumentDB prend-il en charge les fonctions d’agrégation SQL ?
DocumentDB prend en charge l’agrégation à faible latence à n’importe quelle échelle via les fonctions d’agrégation `COUNT`, `MIN`, `MAX`, `AVG` et `SUM` via la syntaxe SQL. Pour plus d’informations, consultez [Fonctions d’agrégation](documentdb-sql-query.md#Aggregates).

### <a name="what-are-the-data-types-supported-by-documentdb"></a>Quels sont les types de données pris en charge par DocumentDB ?
Les types de données primitifs pris en charge dans DocumentDB sont identiques à ceux pris en charge dans JSON. JSON dispose d'un système de type simple constitué de chaînes, de chiffres (double précision IEEE754), ainsi que de valeurs booléennes (true, false et null). DocumentDB prend en charge en mode natif les types de données spatiales Point, Polygon et LineString exprimées sous la forme GeoJSON. Des types de données plus complexes, tels que DateTime, Guid, Int64 et Geometry peuvent être représentés dans JSON et DocumentDB en créant des objets imbriqués à l'aide de l'opérateur { } et des tableaux à l'aide de l'opérateur [ ].

### <a name="how-does-documentdb-provide-concurrency"></a>Quels sont les avantages concurrentiels de DocumentDB ?
DocumentDB prend en charge le contrôle d’accès concurrentiel optimiste via les balises d’entité HTTP ou etags. Chaque ressource DocumentDB dispose d’une balise etag et l’etag est définie sur le serveur à chaque fois qu’un document est mis à jour. L’en-tête de l’etag et la valeur actuelle sont inclus dans tous les messages de réponse. Les balises etags peuvent être utilisées avec l’en-tête If-Match pour permettre au serveur de déterminer si une ressource doit être mise à jour. La valeur If-Match est la valeur etag utilisée pour la vérification. Si la valeur etag correspond à la valeur etag du serveur, la ressource sera mise à jour. Si la balise etag n’est plus active, le serveur rejette l’opération avec un code de réponse « HTTP 412 Échec de la condition préalable ». Le client doit alors récupérer à nouveau la ressource pour obtenir la valeur etag actuelle pour la ressource. En outre, les balises etags peuvent être utilisées avec un en-tête If-None-Match pour déterminer si la nouvelle extraction d’une ressource est nécessaire.

Pour utiliser l’accès concurrentiel optimiste dans .NET, utilisez la classe [AccessCondition](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.accesscondition.aspx) . Pour voir un exemple .NET, consultez [Program.cs](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/DocumentManagement/Program.cs) dans l’exemple DocumentManagement sur github.

### <a name="how-do-i-perform-transactions-in-documentdb"></a>Comment effectuer des transactions dans DocumentDB ?
DocumentDB prend en charge les transactions intégrées au langage via les déclencheurs et procédures stockées JavaScript. Toutes les opérations de base de données dans les scripts sont exécutées avec l’isolement de capture instantanée étendu à la collection s’il s’agit d’une collection à partition unique, ou à des documents ayant la même valeur de clé de partition au sein d’une collection, si la collection est partitionnée. Une capture instantanée des versions des documents (ETags) est prise au début de la transaction et validée uniquement si le script fonctionne. Si le JavaScript renvoie une erreur, la transaction est annulée. Pour plus d’informations, consultez la rubrique [Programmation côté serveur dans DocumentDB](documentdb-programming.md) .

### <a name="how-can-i-bulk-insert-documents-into-documentdb"></a>Comment insérer des documents en bloc dans DocumentDB ?
Il existe trois façons d’insérer des documents en bloc dans DocumentDB :

* L’outil de migration de données, décrit dans la rubrique [Importation de données vers DocumentDB](documentdb-import-data.md).
* L’Explorateur de documents dans le portail Azure, décrit dans la rubrique [Ajouter des documents en bloc à l’aide de l’Explorateur de documents](documentdb-view-json-document-explorer.md#bulk-add-documents).
* Les procédures stockées, décrites dans la rubrique [Programmation DocumentDB côté serveur](documentdb-programming.md).

### <a name="does-documentdb-support-resource-link-caching"></a>DocumentDB prend-il en charge la mise en cache des liens de ressource ?
Oui. DocumentDB étant un service RESTful, les liens de ressource sont immuables et peuvent être mis en cache. Les clients DocumentDB peuvent spécifier un en-tête « If-None-Match » pour des lectures en comparaison avec des ressources telles que des documents ou des collections. Ils peuvent également mettre à jour les copies locales uniquement lorsque la version du serveur a été modifiée.

### <a name="is-a-local-instance-of-documentdb-available"></a>Une instance locale de DocumentDB est-elle disponible ?
Oui. [L’émulateur Azure DocumentDB](documentdb-nosql-local-emulator.md) fournit une émulation haute fidélité du service DocumentDB. Il prend en charge les mêmes fonctionnalités qu’Azure DocumentDB, notamment la prise en charge de la création et de l’interrogation des documents JSON, l’approvisionnement et la mise à l’échelle des collections, et l’exécution des procédures stockées et des déclencheurs. Vous pouvez développer et tester des applications à l’aide de l’émulateur DocumentDB et les déployer vers Azure à l’échelle mondiale en apportant une seule modification à la configuration du point de terminaison de connexion pour DocumentDB.

## <a name="database-questions-about-developing-against-api-for-mongodb"></a>Questions à propos du développement avec l’API pour MongoDB
### <a name="what-is-documentdbs-api-for-mongodb"></a>Qu’est-ce que l’API pour MongoDB de DocumentDB ?
L’API Microsoft Azure DocumentDB pour MongoDB est une couche de compatibilité qui permet aux applications de communiquer facilement et en toute transparence avec le moteur de base de données DocumentDB natif en utilisant des API et des pilotes MongoDB communautaires existants. Les développeurs peuvent maintenant utiliser les chaînes et compétences de l’outil MongoDB pour créer des applications qui tirent parti de DocumentDB et exploitent des fonctionnalités uniques, comme l’indexation automatique, la maintenance de sauvegarde, les contrats SLA, etc..

### <a name="how-to-do-i-connect-to-my-api-for-mongodb-database"></a>Comment me connecter à mon API de base de données MongoDB ?
Pour se connecter à l’API de DocumentDB pour MongoDB, le plus rapide est d’accéder au [portail Azure](https://portal.azure.com). Naviguez jusqu’à votre compte. Dans la barre de *navigation gauche*, cliquez sur *Démarrage rapide*. *Démarrage rapide* est le meilleur moyen d’obtenir des extraits de code pour vous connecter à votre base de données. 

DocumentDB se conforme à des normes et des exigences strictes en matière de sécurité. Les comptes DocumentDB requièrent une authentification et une communication sécurisée via *SSL*. Assurez-vous donc d’utiliser TLSv1.2.

Pour plus d’informations, consultez [Connexion à votre API de base de données MongoDB](documentdb-connect-mongodb-account.md).

### <a name="are-there-additional-error-codes-for-an-api-for-mongodb-database"></a>Existe-t-il des codes d’erreur supplémentaires pour une API de base de données MongoDB ?
L’API pour MongoDB a ses propres codes d’erreur spécifiques en plus des codes d’erreur MongoDB habituels.


| Error               | Code  | Description  | Solution  |
|---------------------|-------|--------------|-----------|
| TooManyRequests     | 16500 | Le nombre total d’unités de requête consommées a dépassé le taux d’unités de requête configuré pour la collection et a été limité. | Envisagez de mettre à l’échelle le débit de la collection à partir du portail Azure ou faites une nouvelle tentative. |
| ExceededMemoryLimit | 16501 | En tant que service mutualisé, l’opération a dépassé les unités de mémoire du client. | Réduire l’étendue de l’opération via un critère de requête plus restrictif ou contactez le support à partir du [portail Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). <br><br>*Ex :  &nbsp;&nbsp;&nbsp;&nbsp;db.getCollection('users').aggregate([<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{$match: {name: "Andy"}}, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{$sort: {age: -1}}<br>&nbsp;&nbsp;&nbsp;&nbsp;])*) |

[azure-portal]: https://portal.azure.com
[query]: documentdb-sql-query.md

