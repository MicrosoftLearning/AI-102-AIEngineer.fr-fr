---
lab:
  title: Création d’une application cliente Conversational Language Understanding (Préversion)
ms.openlocfilehash: 486a6716a189156739e9107824e561f7bb8fa95c
ms.sourcegitcommit: 2c92ea1491cac72a4765755cf2bc6d8b5f657a46
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/09/2022
ms.locfileid: "145195631"
---
# <a name="create-a-language-service-client-application"></a>Créer une application cliente Language Service

La fonctionnalité de Language Understanding de conversation du service cognitif Azure pour le langage vous permet de définir un modèle de langage courant que les applications clientes peuvent utiliser pour interpréter les entrées en langage naturel des utilisateurs, prédire l’*intention* des utilisateurs (ce qu’ils veulent obtenir) et identifier les *entités* auxquelles l’intention doit être appliquée. Vous pouvez créer des applications clientes qui consomment des modèles de compréhension du langage courant directement par le biais d’interfaces REST ou à l’aide de kits de développement logiciel (SDK) spécifiques au langage.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.

2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).

3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.

4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-language-service-resources"></a>Créer des ressources du service de langage

Si vous disposez déjà d’une ressource de service langage dans votre abonnement Azure, vous pouvez l’utiliser dans cet exercice. Dans le cas contraire, suivez ces instructions pour la créer.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.

2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *service de langage*, puis créez une ressource du **service de langage** avec les paramètres suivants :

    - **Fonctionnalités par défaut** : All
    - **Fonctionnalités personnalisées** : aucune
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *westus2*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : *S*
    - **Conditions légales** : *activez la case à cocher pour confirmer*
    - **Avis d’IA responsable** : *activez la case à cocher pour confirmer*

3. Attendez que la ressource soit créée. Vous pouvez visualiser votre ressource en naviguant vers le groupe de ressources dans lequel vous l’avez créée.

## <a name="import-train-and-publish-a-conversational-language-understanding-model"></a>Importer, entraîner et publier un modèle de compréhension du langage courant

Si vous avez déjà un projet **Horloge** d’un laboratoire ou d’un exercice précédent, vous pouvez l’utiliser dans cet exercice. Dans le cas contraire, suivez ces instructions pour le créer.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Language Studio - Préversion à l’adresse `https://language.cognitive.azure.com`.

2. Connectez-vous avec le compte Microsoft associé à votre abonnement Azure. Si c’est la première fois que vous vous connectez au portail Language Service, vous devrez peut-être accorder à l’application certaines autorisations pour accéder aux détails de votre compte. Suivez alors les étapes de *Bienvenue* en sélectionnant votre abonnement Azure et la ressource de création que vous venez de créer.

3. Ouvrez la page **Compréhension du langage courant**.

3. En regard de **&#65291;Créer un projet**, affichez la liste déroulante et sélectionnez **Importer**. Cliquez sur **Choisir un fichier** , puis accédez au sous-dossier **10b-clu-client-(preview)** dans le dossier du projet contenant les fichiers lab de cet exercice. Sélectionnez **Clock.json**, cliquez sur **Ouvrir**, puis sur **Terminé**.

4. Si un volet contenant des astuces pour créer une application de service de langage efficace s’affiche, fermez-le.

5. À gauche du portail Language Studio, sélectionnez **Entraîner le modèle** pour entraîner l’application. Cliquez sur **Démarrer un travail d’entraînement, nommez**, nommez le modèle **Horloge** et vérifiez que l’évaluation avec l’entraînement est activée. L’entraînement peut prendre plusieurs minutes.

    > **Remarque** : Étant donné que le nom du modèle **Horloge** est codé en dur dans le code clock-client (utilisé plus loin dans le labo), mettez en majuscules et attribuez le nom exactement comme décrit. 

6. À gauche du portail Language Studio, sélectionnez **Déployer le modèle** et utilisez **Ajouter un déploiement** pour créer un déploiement pour le modèle Horloge nommé **production**.

    > **Remarque** : Étant donné que le nom du déploiement **production** est codé en dur dans le code clock-client (utilisé plus loin dans le labo), mettez en majuscules et attribuez le nom exactement comme décrit. 

7. Une fois le déploiement terminé, sélectionnez le déploiement de **production** , puis cliquez sur **Obtenir l’URL de prédiction**. Les applications clientes ont besoin de l’URL du point de terminaison pour utiliser votre modèle déployé.

8. À gauche du portail Language Studio, sélectionnez **Paramètres du projet** et notez la **Clé primaire**. Les applications clientes ont besoin de la clé primaire pour utiliser votre modèle déployé.

9. Les applications clientes ont besoin d’informations à partir de l’URL de prédiction et de la clé du service de langage pour se connecter à votre modèle déployé et être authentifiées.

## <a name="prepare-to-use-the-language-service-sdk"></a>Préparer l’utilisation du kit de développement logiciel (SDK) du service de langage

Dans cet exercice, vous allez finaliser une application cliente partiellement implémentée qui utilise le modèle Horloge (modèle de compréhension du langage courant publié) pour prédire les intentions de l’entrée utilisateur et répondre de manière appropriée.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **.NET** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **10b-clu-client-(preview)** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.

2. Cliquez avec le bouton droit de la souris sur le dossier **clock-client**, puis sélectionnez **Ouvrir dans le terminal intégré**. Installez ensuite le package SDK Language Service courant en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-language-conversations --pre
    python -m pip install python-dotenv
    python -m pip install python-dateutil
    ```

3. Affichez le contenu du dossier **clock-client**, et notez qu’il contient un fichier pour les paramètres de configuration :

    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour inclure l’**URL du point de terminaison** et la **Clé primaire** de votre ressource de langue. Vous trouverez les valeurs requises dans le portail Azure ou dans Language Studio comme suit :

    - Portail Azure : Ouvrez votre ressource langage. Sous **Gestion des ressources**, sélectionnez **Clés et points de terminaison**. Copiez les valeurs **CLÉ 1** et **Point de terminaison** dans votre fichier de paramètres de configuration.
    - Language Studio : Ouvrez votre projet **Horloge**. Le point de terminaison du service de langage se trouve dans la page **Déployer le modèle** sous **Obtenir l’URL de prédiction**, et la **clé primaire** se trouve dans la page **Paramètres du projet**. La partie point de terminaison de service de langage de l’URL de prédiction se termine par **.cognitiveservices.azure.com/** . Par exemple : `https://ai102-langserv.cognitiveservices.azure.com/`.

4. Notez que le dossier **clock-client** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : clock-client.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) de service de langage :

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    from azure.ai.language.conversations.models import ConversationAnalysisOptions
    ```

## <a name="get-a-prediction-from-the-conversational-language-model"></a>Obtenir une prédiction à partir du modèle de langage courant

Vous êtes maintenant prêt à implémenter du code qui utilise le kit de développement logiciel (SDK) pour obtenir une prédiction à partir de votre modèle de langage courant.

1. Dans la fonction **Principal**, notez que le code pour charger le point de terminaison et la clé de la prédiction à partir du fichier de configuration a déjà été fourni. Recherchez ensuite le commentaire **Créer un client pour le modèle de service de langage** et ajoutez le code suivant pour créer un client de prédiction pour votre application Language Service :

    **C#**

    ```C#
    // Create a client for the Language service model
    Uri lsEndpoint = new Uri(predictionEndpoint);
    AzureKeyCredential lsCredential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient lsClient = new ConversationAnalysisClient(lsEndpoint, lsCredential);
    ```

    **Python**

    ```Python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

2. Notez que le code de la fonction **Main** invite l’utilisateur à entrer une entrée utilisateur jusqu’à ce que l’utilisateur entre « quit ». Dans cette boucle, recherchez le commentaire **Appeler le modèle de service de langage pour obtenir l’intention et les entités** et ajouter le code suivant :

    **C#**

    ```C#
    // Call the Language service model to get intent and entities
    var lsProject = "Clock";
    var lsSlot = "production";

    ConversationsProject conversationsProject = new ConversationsProject(lsProject, lsSlot);
    Response<AnalyzeConversationResult> response = await lsClient.AnalyzeConversationAsync(userText, conversationsProject);

    ConversationPrediction conversationPrediction = response.Value.Prediction as ConversationPrediction;

    Console.WriteLine(JsonConvert.SerializeObject(conversationPrediction, Formatting.Indented));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].Confidence > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }

    var entities = conversationPrediction.Entities;
    ```

    **Python**

    ```Python
    # Call the Language service model to get intent and entities
    convInput = ConversationAnalysisOptions(
        query = userText
        )
    
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        result = client.analyze_conversations(
            convInput, 
            project_name=cls_project, 
            deployment_name=deployment_slot
            )

    # list the prediction results
    top_intent = result.prediction.top_intent
    entities = result.prediction.entities

    print("view top intent:")
    print("\ttop intent: {}".format(result.prediction.top_intent))
    print("\tcategory: {}".format(result.prediction.intents[0].category))
    print("\tconfidence score: {}\n".format(result.prediction.intents[0].confidence_score))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity.category))
        print("\ttext: {}".format(entity.text))
        print("\tconfidence score: {}".format(entity.confidence_score))

    print("query: {}".format(result.query))
    ```

    L’appel au modèle de service de langage retourne une prédiction/un résultat, qui inclut l’intention supérieure (la plus probable) ainsi que toutes les entités détectées dans l’énoncé d’entrée. Votre application cliente doit maintenant utiliser cette prédiction pour déterminer et effectuer l’action appropriée.

3. Recherchez le commentaire **Appliquer l’action appropriée** et ajoutez le code suivant, qui recherche les intentions prises en charge par l’application (**GetTime**, **GetDate** et **GetDay**) et détermine si des entités pertinentes ont été détectées, avant d’appeler une fonction existante pour produire une réponse appropriée.

    **C#**

    ```C#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a location entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Location")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        location = entity.Text;
                    }
                    
                }

            }

            // Get the time for the specified location
            var getTimeTask = Task.Run(() => GetTime(location));
            string timeResponse = await getTimeTask;
            Console.WriteLine(timeResponse);
            break;

        case "GetDay":
            var date = DateTime.Today.ToShortDateString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Date entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Date")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        date = entity.Text;
                    }
                }
            }
            // Get the day for the specified date
            var getDayTask = Task.Run(() => GetDay(date));
            string dayResponse = await getDayTask;
            Console.WriteLine(dayResponse);
            break;

        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Weekday entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Weekday")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        day = entity.Text;
                    }
                }
                
            }
            // Get the date for the specified day
            var getDateTask = Task.Run(() => GetDate(day));
            string dateResponse = await getDateTask;
            Console.WriteLine(dateResponse);
            break;

        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**

    ```Python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity.category:
                    # ML entities are strings, get the first one
                    location = entity.text
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity.category:
                    # Regex entities are strings, get the first one
                    date_string = entity.text
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity.category:
                # List entities are lists
                    day = entity.text
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

4. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **clock-client** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python clock-client.py
    ```

5. Lorsque vous y êtes invité, entrez des énoncés pour tester l’application. Par exemple, essayez :

    *Bonjour*
    
    *Quelle heure est-il ?*

    *Quelle heure est-il à Londres ?*

    *Quelle est la date ?*

    *Quelle est la date du dimanche ?*

    *Quel jour sommes-nous ?*

    *Quel jour sera le 01/01/2025 ?*

> **Remarque** : La logique de l’application est délibérément simple et présente un certain nombre de limitations. Par exemple, lorsque vous obtenez l’heure, seul un ensemble restreint de villes est pris en charge et l’heure d’été est ignorée. L’objectif est de voir un exemple de modèle classique pour l’utilisation du service de langage dans lequel votre application doit :
>
>   1. Se connecter à un point de terminaison de prédiction.
>   2. Envoyer un énoncé pour obtenir une prédiction.
>   3. Implémenter une logique pour répondre de manière appropriée à l’intention et aux entités prédites.

6. Une fois que vous avez terminé le test, saisissez *quit*.

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur la création d’un client Language Service, consultez la [documentation du développeur](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)
