---
title: "Didacticiel : Intégration d’Azure Active Directory à Concur | Microsoft Docs"
description: "Apprenez à utiliser Concur avec Azure Active Directory pour activer l’authentification unique, l’approvisionnement automatique et bien plus encore."
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 1eee0a5d-24fa-4986-9aef-3c543cfe3296
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/10/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 325d92e493f6e011367d2c85b52c92838327101e
ms.openlocfilehash: 87f8547486d9c77aeaa49e574b3c05e1a1fd877a
ms.lasthandoff: 02/17/2017


---
# <a name="tutorial-azure-active-directory-integration-with-concur"></a>Didacticiel : Intégration d’Azure Active Directory à Concur
L’objectif de ce didacticiel est de montrer comment intégrer Azure et Concur.  
Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un abonnement Azure valide
* Un client dans Concur

Le scénario décrit dans ce didacticiel se compose des blocs de construction suivants :

* Activation de l’intégration d’applications pour Concur
* Configuration de l’authentification unique (SSO)
* Configuration de l'approvisionnement des utilisateurs
* Affectation d’utilisateurs

![Scénario](./media/active-directory-saas-concur-tutorial/IC769766.png "Scénario")

>[!NOTE]
>La configuration de votre abonnement Concur pour l’authentification unique fédérée via SAML constitue une tâche séparée qui doit être effectuée par Concur à votre demande. 
> 

## <a name="enable-the-application-integration-for-concur"></a>Activer l’intégration d’applications pour Concur
Cette section décrit l’activation de l’intégration d’applications pour Concur.

**Pour activer l’intégration d’applications pour Concur, suivez les étapes ci-dessous :**
1. Dans le volet de navigation gauche du portail Azure Classic, cliquez sur **Active Directory**.
   
  ![Active Directory](./media/active-directory-saas-concur-tutorial/IC700993.png "Active Directory")
2. Dans la liste **Annuaire** , sélectionnez l’annuaire pour lequel vous voulez activer l’intégration d’annuaire.
3. Pour ouvrir la vue des applications, dans la vue d'annuaire, cliquez sur **Applications** dans le menu du haut.
   
  ![Applications](./media/active-directory-saas-concur-tutorial/IC700994.png "Applications")
4. Pour ouvrir la **Galerie d’applications**, cliquez sur **Ajouter une application**, puis sur **Ajouter une application utilisable par mon organisation**.
   
  ![Que souhaitez-vous faire ?](./media/active-directory-saas-concur-tutorial/IC700995.png "Que souhaitez-vous faire ?")
5. Dans la **zone de recherche**, tapez **Concur**.
   
  ![Galerie d’applications](./media/active-directory-saas-concur-tutorial/IC721727.png "Galerie d’applications")
6. Dans le volet des résultats, sélectionnez **Concur**, puis cliquez sur **Terminer** pour ajouter l’application.
   
  ![Concur](./media/active-directory-saas-concur-tutorial/IC721728.png "Concur")
   
## <a name="configure-single-sign-on"></a>Configurer l’authentification unique

Cette section explique comment permettre aux utilisateurs de s’authentifier sur Concur avec leur compte Azure AD en utilisant la fédération basée sur le protocole SAML.

>[!NOTE]
>La configuration de votre abonnement Concur pour l’authentification unique fédérée via SAML constitue une tâche séparée qui doit être effectuée par Concur à votre demande.
> 

**Pour configurer l’authentification unique, procédez comme suit :**

1. Sur la page d’intégration d’applications **Concur** du portail Azure Classic, cliquez sur **Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configurer l’authentification unique**.
   
   ![Configurer l’authentification unique](./media/active-directory-saas-concur-tutorial/IC769767.png "Configurer l’authentification unique")
2. Dans la page **Comment voulez-vous que les utilisateurs se connectent à Concur**, sélectionnez **Authentification unique avec Microsoft Azure AD**, puis cliquez sur **Suivant**.
   
   ![Configurer l’authentification unique](./media/active-directory-saas-concur-tutorial/IC769768.png "Configurer l’authentification unique")
3. Dans la page **Configurer l’URL de l’application**, dans la zone de texte **URL de connexion de Concur**, tapez l’URL de connexion de votre client Concur, puis cliquez sur **Suivant** : 
   
   ![Configurer l’URL de connexion](./media/active-directory-saas-concur-tutorial/IC769769.png "Configurer l’URL de connexion")
4. Dans la page de configuration **Configurer l’authentification unique sur Concur** , procédez comme suit.
   
   ![Configurer l’URL de connexion](./media/active-directory-saas-concur-tutorial/IC769770.png "Configurer l’URL de connexion")
   1. Cliquez sur Télécharger les métadonnées, puis enregistrez le fichier de données sur votre ordinateur.
   2. Contactez l’équipe du support technique Concur pour configurer l’authentification unique de votre client.
   3. Sélectionnez la confirmation de la configuration de l’authentification unique, puis cliquez sur **Terminer** pour fermer la boîte de dialogue **Configurer l’authentification unique**.  
   
   >[!NOTE]
   >La configuration de votre abonnement Concur pour l’authentification unique fédérée via SAML constitue une tâche séparée qui doit être effectuée par Concur à votre demande. 
   > 

## <a name="configure-user-provisioning"></a>Configurer l'approvisionnement de l'utilisateur
Cette section décrit comment activer l’approvisionnement des comptes d’utilisateurs Active Directory sur Concur.

Pour activer des applications dans le service Dépenses, vous devez vous assurer que la bonne configuration est définie et qu’un profil d’administrateur des services Web est utilisé. Ne vous contentez pas d’ajouter le rôle WS Admin au profil d’administrateur existant que vous utilisez pour les fonctions administratives T&E.

Concur Consultants ou l’administrateur des clients doit créer un profil d’administrateur des services Web distinct, et l’administrateur des clients doit utiliser ce profil pour les fonctions d’administrateur des services Web (par exemple, l’activation d’applications). Ces profils doivent être séparés du profil d’administrateur T&E utilisé quotidiennement par l’administrateur des clients (le profil d’administrateur T&E ne doit pas se voir affecter le rôle WS Admin).

Quand vous créez le profil à utiliser pour l’activation de l’application, entrez le nom de l’administrateur des clients dans les champs du profil utilisateur. Cela permet d’affecter la propriété au profil. Une fois le ou les profils créés, le client doit se connecter avec ce profil pour pouvoir cliquer sur le bouton *Activer* d’une application partenaire dans le menu Services Web.

Cette action ne doit pas être exécutée avec le profil utilisé pour l’administration T&E normale, et ce pour les raisons suivantes.

1. Le client doit être celui qui clique sur *Oui* dans la boîte de dialogue qui s’affiche quand une application est activée. Ce clic signifie que le client accepte que l’application partenaire accède aux données. Par conséquent, vous ou le partenaire ne pouvez pas cliquer sur le bouton Oui.
2. Si un administrateur de clients ayant activé une application à l’aide du profil d’administrateur T&E quitte l’entreprise (entraînant la désactivation du profil), toute application activée à l’aide de ce profil cessera de fonctionner jusqu’à son activation au moyen d’un autre profil WS Admin actif. C’est pourquoi vous êtes censé créer des profils WS Admin distincts.
3. Si un administrateur quitte l’entreprise, le nom associé au profil WS Admin peut être remplacé par celui du nouvel administrateur sans affecter l’application activée, dans la mesure où le profil n’a pas besoin d’être désactivé.

**Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :**

1. Connectez-vous à votre client **Concur** .
2. Dans le menu **Administration**, sélectionnez **Web Services**.
   
   ![Client Concur](./media/active-directory-saas-concur-tutorial/IC721729.png "Client Concur")
3. Sur le côté gauche, dans le volet **Web Services**, sélectionnez **Enable Partner Application**.
   
   ![Enable Partner Application](./media/active-directory-saas-concur-tutorial/IC721730.png "Enable Partner Application")
4. Dans la liste **Enable Application**, sélectionnez **Azure Active Directory**, puis cliquez sur **Enable**.
   
   ![Microsoft Azure Active Directory](./media/active-directory-saas-concur-tutorial/IC721731.png "Microsoft Azure Active Directory")
5. Cliquez sur **Yes** pour fermer la boîte de dialogue **Confirm Action**.
   
   ![Confirm Action](./media/active-directory-saas-concur-tutorial/IC721732.png "Confirm Action")
6. Dans le portail Azure Classic, sélectionnez **Concur** dans la liste des applications pour ouvrir la page de boîte de dialogue **Concur**.
7. Pour ouvrir la boîte de dialogue **Configurer l’approvisionnement d’utilisateurs**, cliquez sur **Configurer l’approvisionnement d’utilisateurs**.
8. Entrez le nom d’utilisateur et le mot de passe de l’administrateur Concur, puis cliquez sur **Suivant**.
9. Pour terminer la configuration, dans la page **Confirmation**, cliquez sur le bouton**Terminé**.

Vous pouvez maintenant créer un compte de test, attendre 10 minutes, puis vérifier la synchronisation du compte à Concur.

## <a name="assign-users"></a>Affecter des utilisateurs
Pour tester votre configuration, vous devez autoriser les utilisateurs d’Azure AD concernés à accéder à votre application.

**Pour affecter des utilisateurs à Concur, suivez les étapes ci-dessous :**

1. Dans le portail Azure Classic, créez un compte de test.
2. Dans la page d’intégration d’applications **Concur**, cliquez sur **Affecter des utilisateurs**.
   
   ![Affecter des utilisateurs](./media/active-directory-saas-concur-tutorial/IC769771.png "Affecter des utilisateurs")
3. Sélectionnez votre utilisateur de test, cliquez sur **Affecter**, puis sur **Oui** pour confirmer votre affectation.
   
   ![Oui](./media/active-directory-saas-concur-tutorial/IC767830.png "Oui")

À présent, patientez 10 minutes et vérifiez que le compte est bien synchronisé avec Concur.

Si vous souhaitez tester vos paramètres d’authentification unique, ouvrez le volet d’accès. Pour plus d'informations sur le panneau d'accès, consultez [Présentation du panneau d’accès](active-directory-saas-access-panel-introduction.md).


