---
lab:
  title: Gérer la sécurité de Cognitive Services
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 9c8de44265ffa0846b6860fd7d416bb3be547ed9
ms.sourcegitcommit: 5ffc20f6a590fe643c2b695b8dc04589411be36e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/31/2022
ms.locfileid: "145951180"
---
# <a name="manage-cognitive-services-security"></a>Gérer la sécurité de Cognitive Services

La sécurité est une considération essentielle pour toute application. En tant que développeur, vous devez vous assurer que l'accès aux ressources telles que les services cognitifs est limité aux seules personnes qui en ont besoin.

L’accès aux services cognitifs est généralement contrôlé par le biais de clés d’authentification générées lorsque vous créez initialement une ressource relative aux services cognitifs.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="provision-a-cognitive-services-resource"></a>Approvisionner une ressource Cognitive Services

Si vous n’en avez pas encore dans votre abonnement, vous devez provisionner une ressource **Cognitive Services**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Cognitive Services*, puis créez une ressource **Cognitive Services** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S0
3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.

## <a name="manage-authentication-keys"></a>Gérer les clés d'authentification

Lorsque vous avez créé votre ressource de service cognitif, deux clés d’authentification ont été générées. Vous pouvez les gérer dans le portail Azure ou en utilisant l'interface de ligne de commande (CLI) Azure.

1. Dans le Portail Azure, accédez à votre ressource de services cognitifs et affichez sa page **Clés et point de terminaison**. Cette page contient les informations dont vous aurez besoin pour vous connecter à votre ressource et l’utiliser à partir d’applications que vous développez. Plus précisément :
    - *Point de terminaison* HTTP auquel les applications clientes peuvent envoyer des requêtes.
    - Deux *clés* qui peuvent être utilisées pour l’authentification (les applications clientes peuvent utiliser l’une des clés. Une pratique courante consiste à en utiliser une pour le développement, et une autre pour la production. Vous pouvez facilement régénérer la clé de développement une fois que les développeurs ont terminé leur travail pour empêcher l’accès continu.
    - *Emplacement* dans lequel la ressource est hébergée. Cela est nécessaire pour les demandes adressées à certaines API (mais pas toutes).
2. Dans Visual Studio Code, faites un clic droit sur le dossier **02-cognitive-security** et ouvrez un terminal intégré. Ensuite, entrez la commande suivante pour vous connecter à votre abonnement Azure à l’aide d’Azure CLI.

    ```
    az login
    ```

    Un onglet de navigateur web s’ouvre et vous invite à vous connecter à Azure. Pour ce faire, fermez l’onglet du navigateur et revenez à Visual Studio Code.

    > **Conseil** : Si vous avez plusieurs abonnements, vous devez vous assurer que vous travaillez dans celui qui contient votre ressource de services cognitifs.  Utilisez cette commande pour déterminer votre abonnement actuel : son ID unique est la valeur **id** dans le JSON qui est retourné.

    > **Avertissement** : Si vous obtenez un échec de vérification de certificat pour `az login`, essayez d’attendre quelques minutes et réessayez.
    >
    > ```
    > az account show
    > ```
    >
    > Si vous devez modifier l’abonnement, exécutez cette commande, en modifiant *&lt;Your_Subscription_Id&gt;* en l’ID d’abonnement approprié.
    >
    > ```
    > az account set --subscription <Your_Subscription_Id>
    > ```
    >
    > Vous pouvez également spécifier explicitement l’ID d’abonnement en tant que paramètre *--subscription* dans chaque commande Azure CLI qui suit.

3. Vous pouvez maintenant utiliser la commande suivante pour obtenir la liste des clés des services cognitifs, en remplaçant *&lt;resourceName&gt;* par le nom de votre ressource et *&lt;resourceGroup&gt;* par le nom du groupe de ressources dans lequel vous l’avez créé.

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

La commande retourne une liste des clés de votre ressource de services cognitifs : il existe deux clés nommées **key1** et **key2**.

4. Pour tester votre service cognitif, vous pouvez utiliser **curl**, un outil de ligne de commande pour les requêtes HTTP. Dans le **dossier 02-cognitive-security**, ouvrez **rest-test.cmd** et modifiez la commande **curl** qu'il contient (illustrée ci-dessous), en remplaçant *&lt;yourEndpoint&gt;* et *&lt;yourKey&gt;* par votre URI de point de terminaison et votre clé **Key1** pour utiliser l'API Analyse de texte dans votre ressource de services cognitifs.

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.0/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':[{'id':1,'text':'hello'}]}"
    ```

5. Enregistrez vos modifications, puis dans le terminal intégré du dossier **02-cognitive-security**, exécutez la commande suivante :

    ```
    rest-test
    ```

La commande retourne un document JSON contenant des informations sur la langue détectée dans les données d’entrée (qui doivent être anglais).

6. Si une clé devient compromise ou si les développeurs n’ont plus besoin d’accès, vous pouvez la régénérer dans le portail ou à l’aide d’Azure CLI. Exécutez la commande suivante pour régénérer votre **clé key1** (en remplaçant *&lt;resourceName&gt;* et *&lt;resourceGroup&gt;* par les informations de votre ressource).

    ```
    az cognitiveservices account keys regenerate --name <resourceName> --resource-group <resourceGroup> --key-name key1
    ```

La liste des clés de votre ressource de services cognitifs est retournée : notez que **key1** a changé depuis la dernière récupération.

7. Réexécutez la commande **rest-test** avec l’ancienne clé (vous pouvez utiliser la clé **^** pour parcourir les commandes précédentes) et vérifier qu’elle échoue maintenant.
8. Modifiez la commande *curl* dans **rest-test.cmd** en remplaçant la clé par la nouvelle valeur **key1**, puis enregistrez les modifications. Réexécutez ensuite la commande **rest-test** et vérifiez qu’elle réussit.

> **Conseil** : Dans cet exercice, vous avez utilisé les noms complets des paramètres Azure CLI, tels que **--resource-group**.  Vous pouvez également utiliser des alternatives plus courtes, telles que **-g**, pour rendre vos commandes moins détaillées (mais un peu plus difficile à comprendre).  La [référence des commandes CLI de Cognitive Services](https://docs.microsoft.com/cli/azure/cognitiveservices?view=azure-cli-latest) répertorie les options de paramètre pour chaque commande CLI de services cognitifs.

## <a name="secure-key-access-with-azure-key-vault"></a>Sécuriser l’accès à la clé avec Azure Key Vault

Vous pouvez développer des applications qui consomment des services cognitifs à l’aide d’une clé pour l’authentification. Toutefois, cela signifie que le code de l’application doit être en mesure d’obtenir la clé. Une option consiste à stocker la clé dans une variable d’environnement ou un fichier de configuration où l’application est déployée, mais cette approche laisse la clé vulnérable à un accès non autorisé. Une meilleure approche lors du développement d’applications sur Azure consiste à stocker la clé en toute sécurité dans Azure Key Vault et à fournir l’accès à la clé via une *identité managée* (en d’autres termes, un compte d’utilisateur utilisé par l’application elle-même).

### <a name="create-a-key-vault-and-add-a-secret"></a>Créer un coffre de clés et ajouter un secret

Tout d’abord, vous devez créer un coffre de clés et ajouter un *secret* pour la clé des services cognitifs.

1. Notez la valeur **key1** de votre ressource de services cognitifs (ou copiez-la dans le Presse-papiers).
2. Dans le Portail Azure, dans la page **Accueil**, sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Key Vault* et créez une ressource **Key Vault** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Le même groupe de ressources que votre ressource de service cognitif*
    - **Nom du coffre de clés** : *Entrez un nom unique*
    - **Région** : *La même région que votre ressource de service cognitif*
    - **Niveau tarifaire** : Standard
3. Attendez que le déploiement soit terminé, puis allez dans votre ressource de coffre de clés.
4. Dans le volet de navigation gauche, sélectionnez **Secrets** (dans la section Paramètres).
5. Sélectionnez **+ Générer/Importer** et ajoutez un nouveau secret avec les paramètres suivants :
    - **Options de chargement** : Manuelle
    - **Nom** : Cognitive-Services-Key *(il est important de le faire correspondre exactement, car plus tard, vous allez exécuter du code qui récupère le secret en fonction de ce nom)*
    - **Valeur** : *Votre clé de services cognitifs **key1***

### <a name="create-a-service-principal"></a>Créer un principal du service

Pour accéder au secret dans le coffre de clés, votre application doit utiliser un principal de service qui a accès au secret. Vous allez utiliser l’interface de ligne de commande Azure (CLI) pour créer le principal de service, rechercher son ID d’objet et accorder l’accès au secret dans Azure Vault.

1. Revenez à Visual Studio Code et, dans le terminal intégré du dossier **02-cognitive-security**, exécutez la commande Azure CLI suivante, en remplaçant *&lt;spName&gt;* par un nom approprié pour une identité d’application (par exemple, *ai-app*). Remplacez également *&lt;subscriptionId&gt;* et *&lt;resourceGroup&gt;* par les valeurs appropriées pour votre ID d’abonnement et le groupe de ressources contenant vos ressources de services cognitifs et de coffre de clés :

    > **Conseil** : Si vous n’êtes pas sûr de votre ID d’abonnement, utilisez la commande **az account show** pour récupérer vos informations d’abonnement : l’ID d’abonnement est l’attribut **id** dans la sortie.

    ```
    az ad sp create-for-rbac -n "api://<spName>" --role owner --scopes subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    ```

La sortie de cette commande inclut des informations sur votre nouveau principal de service. Il doit ressembler à ceci :

    ```
    {
        "appId": "abcd12345efghi67890jklmn",
        "displayName": "ai-app",
        "name": "http://ai-app",
        "password": "1a2b3c4d5e6f7g8h9i0j",
        "tenant": "1234abcd5678fghi90jklm"
    }
    ```

Notez les valeurs **appId**, **password**, et **tenant**, vous en aurez besoin plus tard (si vous fermez ce terminal, vous ne serez pas en mesure de récupérer le mot de passe ; il est donc important de noter les valeurs maintenant.Vous pouvez coller la sortie dans un nouveau fichier texte dans Visual Studio Code pour vous assurer de trouver les valeurs dont vous avez besoin plus tard !)

2. Pour obtenir **l’ID d’objet** de votre principal de service, exécutez la commande Azure CLI suivante, en remplaçant *&lt;appId&gt;* par la valeur de l’ID d’application de votre principal de service.

    ```
    az ad sp show --id <appId> --query objectId --out tsv
    ```

3. Pour attribuer à votre nouveau principal de service la permission d'accéder aux secrets de votre Key Vault, exécutez la commande Azure CLI suivante, en remplaçant *&lt;keyVaultName&gt;* par le nom de votre ressource Azure Key Vault et *&lt;objectId&gt;* par la valeur de l'ID d'objet de votre principal de service.

    ```
    az keyvault set-policy -n <keyVaultName> --object-id <objectId> --secret-permissions get list
    ```

### <a name="use-the-service-principal-in-an-application"></a>Utiliser le principal de service dans une application

Vous êtes maintenant prêt à utiliser l’identité du principal de service dans une application, afin qu’elle puisse accéder à la clé secrète de vos services cognitifs dans votre coffre de clés et l’utiliser pour vous connecter à votre ressource.

> **Remarque** : Dans cet exercice, nous allons stocker les informations d’identification du principal de service dans la configuration de l’application et les utiliser pour authentifier une identité **ClientSecretCredential** dans votre code d’application. Cela est parfait pour le développement et le test, mais dans une application de production réelle, un administrateur affecterait une *identité managée* à l’application afin qu’elle utilise l’identité du principal de service pour accéder aux ressources, sans mettre en cache ou stocker le mot de passe.

1. Dans Visual Studio Code, dans le dossier **02-cognitive-security**, développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit de la souris sur le dossier **keyvault-client** et ouvrez un terminal intégré. Ensuite, installez les packages que vous devez utiliser Azure Key Vault et l’API Analyse de texte dans votre ressource de services cognitifs en exécutant la commande appropriée pour votre préférence de langue :

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    dotnet add package Azure.Identity --version 1.5.0
    dotnet add package Azure.Security.KeyVault.Secrets --version 4.2.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    pip install azure-identity==1.5.0
    pip install azure-keyvault-secrets==4.2.0
    ```

3. Affichez le contenu du dossier **keyvault-client**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter les paramètres suivants :
    
    - Le **point de terminaison** de votre ressource de services cognitifs
    - Nom de votre ressource **Azure Key Vault**
    - ID de **locataire** pour votre principal de service
    - **appid** pour votre principal de service
    - **Mot de passe** pour votre principal de service

     Enregistrez vos modifications.
4. Notez que le dossier **keyvault-client** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : keyvault-client.py

    Ouvrez le fichier de code et passez en revue le code qu’il contient, en notant les détails suivants :
    - L’espace de noms du kit de développement logiciel (SDK) que vous avez installé est importé
    - Le code de la fonction **Main** récupère les paramètres de configuration de l’application, puis utilise les informations d’identification du principal de service pour obtenir la clé des services cognitifs à partir du coffre de clés.
    - La fonction **GetLanguage** utilise le kit de développement logiciel (SDK) pour créer un client pour le service, puis utilise le client pour détecter la langue du texte entré.
5. Revenez au terminal intégré pour accéder au dossier **keyvault-client**, puis entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python keyvault-client.py
    ```

6. Lorsque vous y êtes invité, entrez du texte et passez en revue la langue détectée par le service. Par exemple, essayez d’entrer « Hello », « Bonjour » et « Gracias ».
7. Lorsque vous avez terminé de tester l’application, entrez « quit » pour arrêter le programme.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur la sécurisation des services cognitifs, consultez la [documentation sur la sécurité des services cognitifs](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-security).
