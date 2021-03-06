---
title: "Se connecter à Azure SQL Database à l’aide de Java | Microsoft Docs"
description: "Présente un exemple de code Java que vous pouvez utiliser pour vous connecter à Azure SQL Database et l’interroger."
services: sql-database
documentationcenter: 
author: ajlam
manager: jhubbard
editor: 
ms.assetid: 
ms.service: sql-database
ms.custom: quick start connect
ms.workload: drivers
ms.tgt_pltfrm: na
ms.devlang: java
ms.topic: article
ms.date: 04/17/2017
ms.author: andrela;carlrab;sstein
ms.translationtype: Human Translation
ms.sourcegitcommit: abdbb9a43f6f01303844677d900d11d984150df0
ms.openlocfilehash: 6ba2880b1ce9eed0f5c3b8e3ed4255c7e4ec7f29
ms.contentlocale: fr-fr
ms.lasthandoff: 04/20/2017


---
# <a name="azure-sql-database-use-java-to-connect-and-query-data"></a>Azure SQL Database : utilisez Java pour vous connecter et interroger des données

Ce guide de démarrage rapide indique comment utiliser [Java](https://docs.microsoft.com/sql/connect/jdbc/microsoft-jdbc-driver-for-sql-server) pour se connecter à une base de données Azure SQL, puis utiliser les instructions Transact-SQL pour interroger, insérer, mettre à jour et supprimer des données dans la base de données à partir des plateformes Mac OS, Ubuntu Linux et Windows.

Ce guide de démarrage rapide utilise comme point de départ les ressources créées dans l’une de ces instructions de démarrage rapide :

- [Créer une base de données - Portail](sql-database-get-started-portal.md)
- [Créer une base de données - CLI](sql-database-get-started-cli.md)

## <a name="install-java-software"></a>Installer le logiciel Java

Les étapes de cette section supposent que vous êtes familiarisé avec le développement à l’aide de Java et que vous ne savez pas utiliser la base de données SQL Azure. Si vous êtes novice en développement avec Java, consultez [Créer une application à l’aide de SQL Server](https://www.microsoft.com/en-us/sql-server/developer-get-started/) et sélectionnez **Java**, puis sélectionnez votre système d’exploitation.

### <a name="mac-os"></a>**Mac OS**
Ouvrez votre terminal et accédez au répertoire dans lequel vous envisagez de créer votre projet Java. Entrez les commandes suivantes pour installer **brew** et **Maven**. 

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
brew install maven
```

### <a name="linux-ubuntu"></a>**Linux (Ubuntu)**
Ouvrez votre terminal et accédez au répertoire dans lequel vous envisagez de créer votre projet Java. Entrez les commandes suivantes pour installer **Maven**. 

```bash
sudo apt-get install maven
```

### <a name="windows"></a>**Windows**
Installez [Maven](https://maven.apache.org/download.cgi) à l’aide du programme d’installation officiel. Vous pouvez utiliser Maven pour gérer les dépendances, générer, tester et exécuter votre projet Java. 

## <a name="get-connection-information"></a>Obtenir des informations de connexion

Obtenez les informations de connexion requises pour la connexion à la base de données SQL Azure. Vous aurez besoin du nom du serveur complet, du nom de la base de données et des informations de connexion dans les procédures suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez **Bases de données SQL** dans le menu de gauche, puis cliquez sur votre base de données dans la page **Bases de données SQL**. 
3. Sur la page **Vue d’ensemble** de votre base de données, vérifiez le nom complet du serveur, comme indiqué dans l’image ci-dessous. Vous pouvez pointer sur le nom du serveur pour afficher l’option **Cliquez pour copier**. 

   ![server-name](./media/sql-database-connect-query-dotnet/server-name.png) 

4. Si vous avez oublié vos informations de connexion à votre serveur Azure SQL Database, accédez à la page du serveur SQL Database pour afficher le nom de l’administrateur du serveur et, si nécessaire, réinitialiser le mot de passe.
5. Cliquez sur **Afficher les chaînes de connexion de la base de données**.

6. Passez en revue la chaîne de connexion **JDBC** complète.

    ![Chaîne de connexion JDBC](./media/sql-database-connect-query-jdbc/jdbc-connection-string.png)    

### <a name="create-maven-project"></a>**Créer un projet Maven**
À partir du terminal, créez un nouveau projet Maven. 
```bash
mvn archetype:generate "-DgroupId=com.sqldbsamples" "-DartifactId=SqlDbSample" "-DarchetypeArtifactId=maven-archetype-quickstart" "-Dversion=1.0.0"
```

Ajoutez **Microsoft JDBC Driver pour SQL Server** aux dépendances dans ***pom.xml***. 

```xml
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>6.1.0.jre8</version>
</dependency>
```

## <a name="select-data"></a>Sélectionner des données

Utilisez le code suivant pour rechercher les 20 premiers produits par catégorie à l’aide de la classe [connexion](https://docs.microsoft.com/sql/connect/jdbc/working-with-a-connection) et d’une instruction Transact-SQL [SELECT](https://docs.microsoft.com/sql/t-sql/queries/select-transact-sql). Remplacez les paramètres du nom d’hôte, du nom de la base de données, de l’utilisateur et du mot de passe par les valeurs spécifiées lors de la création de la base de données avec les exemples de données AdventureWorksLT. 

```java
package com.sqldbsamples;

import java.sql.Connection;
import java.sql.Statement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.DriverManager;

public class App {

    public static void main(String[] args) {
    
        // Connect to database
        String hostName = "your_server.database.windows.net";
        String dbName = "your_database";
        String user = "your_username";
        String password = "your_password";
        String url = String.format("jdbc:sqlserver://%s.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;", hostName, dbName, user, password);
        Connection connection = null;

        try {
                connection = DriverManager.getConnection(url);
                String schema = connection.getSchema();
                System.out.println("Successful connection - Schema: " + schema);

                System.out.println("Query data example:");
                System.out.println("=========================================");

                // Create and execute a SELECT SQL statement.
                String selectSql = "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName " 
                    + "FROM [SalesLT].[ProductCategory] pc "  
                    + "JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid";
                
                try (Statement statement = connection.createStatement();
                    ResultSet resultSet = statement.executeQuery(selectSql)) {

                        // Print results from select statement
                        System.out.println("Top 20 categories:");
                        while (resultSet.next())
                        {
                            System.out.println(resultSet.getString(1) + " "
                                + resultSet.getString(2));
                        }
                }
        }
        catch (Exception e) {
                e.printStackTrace();
        }
    }
}
```

## <a name="insert-data"></a>Insertion des données

Utilisez le code suivant pour insérer un nouveau produit dans la table SalesLT.Product à l’aide de la classe [Instructions préparées](https://docs.microsoft.com/sql/connect/jdbc/using-statements-with-sql) avec une instruction Transact-SQL [INSERT](https://docs.microsoft.com/sql/t-sql/statements/insert-transact-sql). Remplacez les paramètres du nom d’hôte, du nom de la base de données, de l’utilisateur et du mot de passe par les valeurs spécifiées lors de la création de la base de données avec les exemples de données AdventureWorksLT. 

```java
package com.sqldbsamples;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.DriverManager;

public class App {

    public static void main(String[] args) {
    
        // Connect to database
        String hostName = "your_server.database.windows.net";
        String dbName = "your_database";
        String user = "your_username";
        String password = "your_password";
        String url = String.format("jdbc:sqlserver://%s.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;", hostName, dbName, user, password);
        Connection connection = null;

        try {
                connection = DriverManager.getConnection(url);
                String schema = connection.getSchema();
                System.out.println("Successful connection - Schema: " + schema);

                System.out.println("Insert data example:");
                System.out.println("=========================================");

                // Prepared statement to insert data
                String insertSql = "INSERT INTO SalesLT.Product (Name, ProductNumber, Color, " 
                    + " StandardCost, ListPrice, SellStartDate) VALUES (?,?,?,?,?,?);";

                java.util.Date date = new java.util.Date();
                java.sql.Timestamp sqlTimeStamp = new java.sql.Timestamp(date.getTime());

                try (PreparedStatement prep = connection.prepareStatement(insertSql)) {
                        prep.setString(1, "BrandNewProduct");
                        prep.setInt(2, 200989);
                        prep.setString(3, "Blue");
                        prep.setDouble(4, 75);
                        prep.setDouble(5, 80);
                        prep.setTimestamp(6, sqlTimeStamp);

                        int count = prep.executeUpdate();
                        System.out.println("Inserted: " + count + " row(s)");
                }
        }
        catch (Exception e) {
                e.printStackTrace();
        }
    }
}
```
## <a name="update-data"></a>Mettre à jour des données

Utilisez le code suivant pour mettre à jour le nouveau produit que vous avez précédemment ajouté en utilisant la classe [Instructions préparées](https://docs.microsoft.com/sql/connect/jdbc/using-statements-with-sql) avec une instruction Transact-SQL [UPDATE](https://docs.microsoft.com/sql/t-sql/queries/update-transact-sql) pour mettre à jour les données dans votre base de données SQL Azure. Remplacez les paramètres du nom d’hôte, du nom de la base de données, de l’utilisateur et du mot de passe par les valeurs spécifiées lors de la création de la base de données avec les exemples de données AdventureWorksLT. 

```java
package com.sqldbsamples;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.DriverManager;

public class App {

    public static void main(String[] args) {
    
        // Connect to database
        String hostName = "your_server.database.windows.net";
        String dbName = "your_database";
        String user = "your_username";
        String password = "your_password";
        String url = String.format("jdbc:sqlserver://%s.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;", hostName, dbName, user, password);
        Connection connection = null;

        try {
                connection = DriverManager.getConnection(url);
                String schema = connection.getSchema();
                System.out.println("Successful connection - Schema: " + schema);

                System.out.println("Update data example:");
                System.out.println("=========================================");

                // Prepared statement to update data
                String updateSql = "UPDATE SalesLT.Product SET ListPrice = ? WHERE Name = ?";

                try (PreparedStatement prep = connection.prepareStatement(updateSql)) {
                        prep.setString(1, "500");
                        prep.setString(2, "BrandNewProduct");

                        int count = prep.executeUpdate();
                        System.out.println("Updated: " + count + " row(s)")
                }
        }
        catch (Exception e) {
                e.printStackTrace();
        }
    }
}
```



## <a name="delete-data"></a>Suppression de données

Utilisez le code suivant pour supprimer le nouveau produit que vous avez ajouté précédemment à l’aide de la classe [Instructions préparées](https://docs.microsoft.com/sql/connect/jdbc/using-statements-with-sql) avec une instruction Transact-SQL [DELETE](https://docs.microsoft.com/sql/t-sql/statements/delete-transact-sql). Remplacez les paramètres du nom d’hôte, du nom de la base de données, de l’utilisateur et du mot de passe par les valeurs spécifiées lors de la création de la base de données avec les exemples de données AdventureWorksLT. 

```java
package com.sqldbsamples;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.DriverManager;

public class App {

    public static void main(String[] args) {
    
        // Connect to database
        String hostName = "your_server.database.windows.net";
        String dbName = "your_database";
        String user = "your_username";
        String password = "your_password";
        String url = String.format("jdbc:sqlserver://%s.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;", hostName, dbName, user, password);
        Connection connection = null;

        try {
                connection = DriverManager.getConnection(url);
                String schema = connection.getSchema();
                System.out.println("Successful connection - Schema: " + schema);

                System.out.println("Delete data example:");
                System.out.println("=========================================");

                // Prepared statement to delete data
                String deleteSql = "DELETE SalesLT.Product WHERE Name = ?";

                try (PreparedStatement prep = connection.prepareStatement(deleteSql)) {
                        prep.setString(1, "BrandNewProduct");

                        int count = prep.executeUpdate();
                        System.out.println("Deleted: " + count + " row(s)");
                }
        }        
        catch (Exception e) {
                e.printStackTrace();
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

- Le référentiel GitHub du [Pilote JDBC Microsoft pour SQL Server](https://github.com/microsoft/mssql-jdbc).
- [Problèmes avec les fichiers/posez des questions](https://github.com/microsoft/mssql-jdbc/issues).
- Pour vous connecter et envoyer des requêtes à l’aide de SQL Server Management Studio, consultez [Se connecter à la base de données SQL avec SQL Server Management Studio et exécuter un exemple de requête T-SQL](sql-database-connect-query-ssms.md).
- Pour vous connecter et exécuter des requêtes à l’aide de Visual Studio Code, consultez l’article décrivant comment [se connecter et effectuer des requêtes avec Visual Studio Code](sql-database-connect-query-vscode.md).
- Pour vous connecter et exécuter des requêtes avec .NET, consultez la page [Se connecter et effectuer des requêtes avec .NET](sql-database-connect-query-dotnet.md).
- Pour vous connecter et exécuter des requêtes avec PHP, consultez la page [se connecter et effectuer des requêtes avec PHP](sql-database-connect-query-php.md).
- Pour vous connecter et exécuter des requêtes avec Node.js, consultez la page [se connecter et effectuer des requêtes avec Node.js](sql-database-connect-query-nodejs.md).
- Pour vous connecter et exécuter des requêtes avec Python, consultez la page [se connecter et effectuer des requêtes avec Python](sql-database-connect-query-python.md).
- Pour vous connecter et exécuter des requêtes avec Ruby, consultez la page [se connecter et effectuer des requêtes avec Ruby](sql-database-connect-query-ruby.md).


