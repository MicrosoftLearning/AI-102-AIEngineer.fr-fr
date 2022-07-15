---
lab:
  title: Démarrer avec Cognitive Services
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: a05256a78dee051041320aa3556a43add5596ce9
ms.sourcegitcommit: 5ffc20f6a590fe643c2b695b8dc04589411be36e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/31/2022
ms.locfileid: "145951188"
---
# <a name="get-started-with-cognitive-services"></a>Démarrer avec Cognitive Services

Dans cet exercice, vous allez commencer à utiliser Cognitive Services en créant une ressource **Cognitive Services** dans votre abonnement Azure et en l’utilisant à partir d’une application cliente. L’objectif de l’exercice n’est pas d’acquérir de l’expertise dans un service particulier, mais plutôt de se familiariser avec un modèle général pour l’approvisionnement et l’utilisation des services cognitifs en tant que développeur.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous n’avez pas déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement où vous travaillez sur ce laboratoire, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Clone** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="provision-a-cognitive-services-resource"></a>Approvisionner une ressource Cognitive Services

Azure Cognitive Services est un ensemble de services cloud qui encapsulent des fonctionnalités d’intelligence artificielle que vous pouvez incorporer dans vos applications. Vous pouvez approvisionner des ressources de services cognitifs individuelles pour des API spécifiques (par exemple, **Langue** ou **Vision par ordinateur**), ou vous pouvez approvisionner une ressource **Cognitive Services** générale qui fournit l’accès à plusieurs API via un point de terminaison et une clé uniques. Dans ce cas, vous allez utiliser une seule ressource **Cognitive Services**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Cognitive Services*, puis créez une ressource **Cognitive Services** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S0
3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
5. Accédez à la ressource et affichez sa page **Clés et point de terminaison**. Cette page contient les informations dont vous aurez besoin pour vous connecter à votre ressource et l’utiliser à partir d’applications que vous développez. Plus précisément :
    - *Point de terminaison* HTTP auquel les applications clientes peuvent envoyer des requêtes.
    - Deux *clés* qui peuvent être utilisées pour l’authentification (les applications clientes peuvent utiliser une clé pour s’authentifier).
    - *Emplacement* dans lequel la ressource est hébergée. Cela est nécessaire pour les demandes adressées à certaines API (mais pas toutes).

## <a name="use-a-rest-interface"></a>Utiliser une interface REST

Les API des services cognitifs sont basées sur REST. Vous pouvez donc les utiliser en envoyant des requêtes JSON via HTTP. Dans cet exemple, vous allez explorer une application console qui utilise l’API REST Langue pour effectuer ladétection de **Langue**, mais le principe de base est le même pour toutes les API prises en charge par la ressourceCognitive Services.

> **Remarque** : Dans cet exercice, vous pouvez choisir d’utiliser l’API REST à partir de **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **01-getting-started** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Affichez le contenu du dossier **rest-client**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification de votre ressource Cognitive Services. Enregistrez vos modifications.
3. Notez que le dossier **rest-client** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : rest-client.py

    Ouvrez le fichier de code et passez en revue le code qu’il contient, en notant les détails suivants :
    - Différents espaces de noms sont importés pour activer la communication HTTP
    - Le code de la fonction **Main** récupère le point de terminaison et la clé de votre ressource des services cognitifs. Ceux-ci seront utilisés pour envoyer des requêtes REST au service Analyse de texte.
    - Le programme accepte l’entrée utilisateur et utilise la fonction **GetLanguage** pour appeler l’API REST de détection de langue, Analyse de texte pour votre point de terminaison, Cognitive Services pour détecter la langue du texte entré.
    - La requête envoyée à l’API se compose d’un objet JSON contenant les données d’entrée, dans ce cas, une collection d’objets de **document**, chacune ayant un **ID** et du **texte**.
    - La clé de votre service est incluse dans l’en-tête de demande pour authentifier votre application cliente.
    - La réponse du service est un objet JSON, que l’application cliente peut analyser.
4. Cliquez avec le bouton droit de la souris sur le dossier **rest-client** et ouvrez un terminal intégré. Entrez ensuite la prochaine commande spécifique au langage pour exécuter le programme:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python rest-client.py
    ```

5. Lorsque vous y êtes invité, entrez du texte et passez en revue la langue détectée par le service, qui est retournée dans la réponse JSON. Par exemple, essayez d’entrer « Hello », « Bonjour » et « Hola ».
6. Lorsque vous avez terminé de tester l’application, entrez « quit » pour arrêter le programme.

## <a name="use-an-sdk"></a>Utiliser un Kit de développement logiciel (SDK)

Vous pouvez écrire du code qui consomme directement les API REST des services cognitifs, mais il existe des kits de développement logiciel (SDK) pour de nombreux langages de programmation populaires, notamment Microsoft C#, Python et Node.js. L’utilisation d’un kit de développement logiciel (SDK) peut simplifier considérablement le développement d’applications qui consomment les services cognitifs.

1. Dans Visual Studio Code, dans le volet **Explorateur**, dans le dossier **01-getting-started**, développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier **sdk-client** et ouvrez un terminal intégré. Installez ensuite le package SDK d’analyse de texte en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    ```

3. Affichez le contenu du dossier **sdk-client**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification de votre ressource Cognitive Services. Enregistrez vos modifications.
    
4. Notez que le dossier **sdk-client** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : sdk-client.py

    Ouvrez le fichier de code et passez en revue le code qu’il contient, en notant les détails suivants :
    - L’espace de noms du kit de développement logiciel (SDK) que vous avez installé est importé
    - Le code de la fonction **Main** récupère le point de terminaison et la clé de votre ressource des services cognitifs. Ceux-ci seront utilisés avec le SDK pour créer un client pour le service Analyse de texte.
    - La fonction **GetLanguage** utilise le kit de développement logiciel (SDK) pour créer un client pour le service, puis utilise le client pour détecter la langue du texte entré.
5. Revenez au terminal intégré pour accéder au dossier **sdk-client**, puis entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python sdk-client.py
    ```

6. Lorsque vous y êtes invité, entrez du texte et passez en revue la langue détectée par le service. Par exemple, essayez d’entrer « Goodbye », « Au revoir » et « Hasta la vista ».
7. Lorsque vous avez terminé de tester l’application, entrez « quit » pour arrêter le programme.

> **Remarque** : Certaines langues nécessitant des jeux de caractères Unicode peuvent ne pas être reconnues dans cette application console simple.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur Cognitive Services, consultez la [Documentation Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services).
