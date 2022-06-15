---
lab:
  title: Créer une application cliente Language Understanding
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 36f75e47910707c959495140be43c4196649d471
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195558"
---
# <a name="create-a-language-understanding-client-application"></a>Créer une application cliente Language Understanding

Le service Language Understanding vous permet de définir une application qui encapsule un modèle de langage que les applications peuvent utiliser pour interpréter l’entrée en langage naturel des utilisateurs, prédire l’*intention* des utilisateurs (ce qu’ils veulent obtenir) et identifier toutes les *entités* auxquelles l’intention doit être appliquée. Vous pouvez créer des applications clientes qui consomment des applications Language Understanding directement par le biais d’interfaces REST ou à l’aide de kits de développement logiciel (SDK) spécifiques au langage.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-language-understanding-resources"></a>Créer des ressources Language Understanding

Si vous disposez déjà de ressources de création et de prédiction Language Understanding dans votre abonnement Azure, vous pouvez les utiliser dans cet exercice. Dans le cas contraire, suivez ces instructions pour les créer.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Language Understanding*, puis créez une ressource **Language Understanding** avec les paramètres suivants :
    - **Options de création** : Les deux
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Nom** : *Entrez un nom unique.*
    - **Emplacement de création** : *Sélectionnez l’emplacement de votre choix.*
    - **Création - Niveau tarifaire** : F0
    - **Emplacement de prédiction** : *Choisissez le <u>même emplacement</u> que celui sélectionné pour la création.*
    - **Niveau tarifaire de prédiction** : F0 (*Si F0 n’est pas disponible, choisissez S0*)

3. Attendez que les ressources soient créées. Notez que deux ressources Language Understanding sont provisionnées : une pour la création et une autre pour la prédiction. Vous pouvez voir ces deux ressources en accédant au groupe de ressources dans lequel vous les avez créées.

## <a name="import-train-and-publish-a-language-understanding-app"></a>Importer, entraîner et publier une application Language Understanding

Si vous avez déjà une application **Horloge** d’un exercice précédent, vous pouvez l’utiliser dans cet exercice. Dans le cas contraire, suivez ces instructions pour la créer.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Language Understanding à l’adresse `https://www.luis.ai`.
2. Connectez-vous avec le compte Microsoft associé à votre abonnement Azure. Si vous vous connectez au portail Language Understanding pour la première fois, il se peut que vous deviez accorder à l’application des autorisations pour accéder aux détails de votre compte. Suivez alors les étapes de *Bienvenue* en sélectionnant votre abonnement Azure et la ressource de création que vous venez de créer.
3. Ouvrez la page **Applications de conversation**, en regard de **&#65291;Nouvelle application**, affichez la liste déroulante et sélectionnez **Importer en tant que LU**.
Accédez au sous-dossier **10-luis-client** dans le dossier de projet contenant les fichiers lab de cet exercice, puis sélectionnez **Clock.lu**. Spécifiez ensuite un nom unique pour l’application horloge.
4. Si un panneau contenant des conseils pour la création d’une application de Language Understanding efficace s’affiche, fermez-le.
5. En haut du portail Language Understanding, sélectionnez **Entraîner** pour entraîner l’application.
6. En haut à droite du portail Language Understanding, sélectionnez **Publier** et publiez l’application dans **Emplacement de production**.
7. Une fois la publication terminée, en haut du portail Language Understanding, sélectionnez **Gérer**.
8. Dans la page **Paramètres**, notez l’**ID de l’application**. Les applications clientes en ont besoin pour utiliser votre application.
9. Dans la page **Ressources Azure**, sous **Ressources de prédiction**, si aucune ressource de prédiction n’est répertoriée, ajoutez-en une dans votre abonnement Azure.
10. Notez la **clé primaire**, la **clé secondaire** et l’**URL de point de terminaison** pour la ressource de prédiction. Les applications clientes ont besoin du point de terminaison et de l’une des clés pour se connecter à la ressource de prédiction et être authentifiées.

## <a name="prepare-to-use-the-language-understanding-sdk"></a>Préparer l’utilisation du Kit de développement logiciel (SDK) Language Understanding

Dans cet exercice, vous allez finaliser une application cliente partiellement implémentée qui utilise l’application horloge Language Understanding pour prédire les intentions de l’entrée utilisateur et répondre de manière appropriée.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **10-luis-client** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langue.
2. Cliquez avec le bouton droit de la souris sur le dossier **clock-client** et ouvrez un terminal intégré. Installez ensuite le package SDK Language Understanding en exécutant la commande appropriée pour votre préférence de langage :

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
```

*Outre le package (de prédiction) **Runtime**, il existe un package de **création** que vous pouvez utiliser pour écrire du code afin de créer et de gérer des modèles Language Understanding.*

**Python**

```
pip install azure-cognitiveservices-language-luis==0.7.0
```
    
*Le package SDK Python inclut des classes pour la **prédiction** et la **création**.*

3. Affichez le contenu du dossier **clock-client**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour inclure l’**ID d’application** de votre application Language Understanding, et l’**URL de point de terminaison** et l’une des **clés** de sa ressource de prédiction (à partir de la page **Gérer** pour votre application dans le portail Language Understanding).

4. Notez que le dossier **clock-client** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : clock-client.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) de prédiction Language Understanding :

**C#**

```C#
// Import namespaces
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
```

**Python**

```Python
# Import namespaces
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
```

## <a name="get-a-prediction-from-the-language-understanding-app"></a>Obtenir une prédiction à partir de l’application Language Understanding

Vous êtes maintenant prêt à implémenter du code qui utilise le kit de développement logiciel (SDK) pour obtenir une prédiction à partir de votre application Language Understanding.

1. Dans la fonction **Main**, notez que le code pour charger l’ID d’application, le point de terminaison et la clé de la prédiction à partir du fichier de configuration a déjà été fourni. Recherchez ensuite le commentaire **Créer un client pour l’application LU** et ajoutez le code suivant pour créer un client de prédiction pour votre application Language Understanding :

**C#**

```C#
// Create a client for the LU app
var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.ApiKeyServiceClientCredentials(predictionKey);
var luClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
```

**Python**

```Python
# Create a client for the LU app
credentials = CognitiveServicesCredentials(lu_prediction_key)
lu_client = LUISRuntimeClient(lu_prediction_endpoint, credentials)
```

2. Notez que le code de la fonction **Main** invite à entrer une entrée utilisateur jusqu’à ce que l’utilisateur entre « quit ». Dans cette boucle, recherchez le commentaire **Appeler l’application LU pour obtenir l’intention et les entités** et ajoutez le code suivant :

**C#**

```C#
// Call the LU app to get intent and entities
var slot = "Production";
var request = new PredictionRequest { Query = userText };
PredictionResponse predictionResponse = await luClient.Prediction.GetSlotPredictionAsync(luAppId, slot, request);
Console.WriteLine(JsonConvert.SerializeObject(predictionResponse, Formatting.Indented));
Console.WriteLine("--------------------\n");
Console.WriteLine(predictionResponse.Query);
var topIntent = predictionResponse.Prediction.TopIntent;
var entities = predictionResponse.Prediction.Entities;
```

**Python**

```Python
# Call the LU app to get intent and entities
request = { "query" : userText }
slot = 'Production'
prediction_response = lu_client.prediction.get_slot_prediction(lu_app_id, slot, request)
top_intent = prediction_response.prediction.top_intent
entities = prediction_response.prediction.entities
print('Top Intent: {}'.format(top_intent))
print('Entities: {}'.format (entities))
print('-----------------\n{}'.format(prediction_response.query))
```

L’appel à l’application Language Understanding retourne une prédiction, qui inclut l’intention supérieure (la plus probable) ainsi que toutes les entités détectées dans l’énoncé d’entrée. Votre application cliente doit maintenant utiliser cette prédiction pour déterminer et effectuer l’action appropriée.

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
            if (entities.ContainsKey("Location"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Location"].ToString());
                // ML entities are strings, get the first one
                location = entityJson[0].ToString();
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
            if (entities.ContainsKey("Date"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Date"].ToString());
                // Regex entities are strings, get the first one
                date = entityJson[0].ToString();
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
            if (entities.ContainsKey("Weekday"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Weekday"].ToString());
                // List entities are lists
                day = entityJson[0][0].ToString();
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
        if 'Location' in entities:
            # ML entities are strings, get the first one
            location = entities['Location'][0]
    # Get the time for the specified location
    print(GetTime(location))

elif top_intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if len(entities) > 0:
        # Check for a Date entity
        if 'Date' in entities:
            # Regex entities are strings, get the first one
            date_string = entities['Date'][0]
    # Get the day for the specified date
    print(GetDay(date_string))

elif top_intent == 'GetDate':
    day = 'today'
    # Check for entities
    if len(entities) > 0:
        # Check for a Weekday entity
        if 'Weekday' in entities:
            # List entities are lists
            day = entities['Weekday'][0][0]
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('Try asking me for the time, the day, or the date.')
```
    
4. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **clock-client** et entrez la commande suivante pour exécuter le programme :

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

    *Quelle heure est-il à Londres ?*

    *Quelle est la date ?*

    *Quelle est la date de dimanche ?*

    *Quel jour sommes-nous ?*

    *Quel jour sera le 01/01/2025 ?*

> **Remarque** : La logique de l’application est délibérément simple et présente un certain nombre de limitations. Par exemple, lorsque vous obtenez l’heure, seul un ensemble restreint de villes est pris en charge et l’heure d’été est ignorée. L’objectif est de voir un exemple de modèle classique pour l’utilisation du service Language Understanding dans lequel votre application doit :
>
>   1. Se connecter à un point de terminaison de prédiction.
>   2. Envoyer un énoncé pour obtenir une prédiction.
>   3. Implémenter une logique pour répondre de manière appropriée à l’intention et aux entités prédites.

6. Une fois que vous avez terminé le test, saisissez *quit*.

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur la création d’un client Language Understanding, consultez la [documentation du développeur](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)
