---
lab:
  title: Utiliser les services Speech et Language Understanding
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 31b88612bce38780f828859f8e2b166dbe8df34d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195619"
---
# <a name="use-the-speech-and-language-understanding-services"></a>Utiliser les services Speech et Language Understanding

Vous pouvez intégrer le service Speech avec le service Language Understanding pour créer des applications qui peuvent déterminer intelligemment les intentions des utilisateurs à partir d’une entrée parlée.

> **Remarque** : Cet exercice fonctionne mieux si vous avez un microphone. Certains environnements virtuels hébergés peuvent être en mesure de capturer l’audio à partir de votre microphone local, mais si cela ne fonctionne pas (ou si vous n’avez pas de microphone du tout), vous pouvez utiliser un fichier audio fourni pour l’entrée vocale. Suivez attentivement les instructions, car vous devez choisir différentes options selon que vous utilisez un microphone ou le fichier audio.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-language-understanding-resources"></a>Créer des ressources Language Understanding

Si vous avez déjà des ressources de création et de prédiction Language Understanding dans votre abonnement Azure, vous pouvez les utiliser dans cet exercice. Dans le cas contraire, suivez ces instructions pour les créer.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Language Understanding*, puis créez une ressource **Language Understanding** avec les paramètres suivants :
    - **Options de création** : Les deux
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Nom** : *Entrez un nom unique.*
    - **Emplacement de création** : *Sélectionnez l’emplacement de votre choix.*
    - **Création - Niveau tarifaire** : F0
    - **Emplacement de prédiction** : *Choisissez le <u>même emplacement</u> que celui sélectionné pour la création.*
    - **Niveau tarifaire de prédiction** : F0 (*Si F0 n’est pas disponible, choisissez S0*)

3. Attendez que les ressources soient créées. Notez que deux ressources Language Understanding sont provisionnées : une pour la création et une autre pour la prédiction. Vous pouvez voir ces deux ressources en accédant au groupe de ressources dans lequel vous les avez créées.

## <a name="prepare-a-language-understanding-app"></a>Préparer une application Language Understanding

Si vous avez déjà une application **Horloge** à partir d’un exercice précédent, ouvrez-la dans le portail Language Understanding à l’adresse `https://www.luis.ai`. Dans le cas contraire, suivez ces instructions pour la créer.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Language Understanding à l’adresse `https://www.luis.ai`.
2. Connectez-vous avec le compte Microsoft associé à votre abonnement Azure. Si vous vous connectez au portail Language Understanding pour la première fois, il se peut que vous deviez accorder à l’application des autorisations pour accéder aux détails de votre compte. Suivez alors les étapes de *Bienvenue* en sélectionnant votre abonnement Azure et la ressource de création que vous venez de créer.
3. Ouvrez la page **Applications de conversation**, en regard de **&#65291;Nouvelle application**, affichez la liste déroulante et sélectionnez **Importer en tant que LU**.
Accédez au sous-dossier **11-luis-speech** dans le dossier de projet contenant les fichiers lab de cet exercice, puis sélectionnez **Clock.lu**. Spécifiez ensuite un nom unique pour l’application horloge.
4. Si un panneau contenant des conseils pour la création d’une application de Language Understanding efficace s’affiche, fermez-le.

## <a name="train-and-publish-the-app-with-speech-priming"></a>Effectuer l'apprentissage et publier l’application avec *Speech Priming*

1. En haut du portail Language Understanding, sélectionnez **Effectuer l'apprentissage** pour effectuer l'apprentissage de l’application si ce n’est pas déjà le cas.
2. En haut à droite du portail Language Understanding, sélectionnez **Publier**. Sélectionnez ensuite l’**Emplacement de production** et modifiez les paramètres pour activer **Speech Priming** (cela entraîne de meilleures performances pour la reconnaissance vocale).
3. Une fois la publication terminée, en haut du portail Language Understanding, sélectionnez **Gérer**.
4. Dans la page **Paramètres**, notez l’**ID de l’application**. Les applications clientes ont besoin de cela pour utiliser votre application.
5. Dans la page **Ressources Azure**, sous **Ressources de prédiction**, si aucune ressource de prédiction n’est répertoriée, ajoutez-en une dans votre abonnement Azure.
6. Notez la **clé primaire**, la **clé secondaire** et l’**emplacement** (et <u>non</u> le point de terminaison !) pour la ressource de prédiction. Les applications clientes du SDK Speech ont besoin de l’emplacement et de l’une des clés pour se connecter à la ressource de prédiction et être authentifiées.

## <a name="configure-a-client-application-for-language-understanding"></a>Configurer une application cliente pour Language Understanding

Dans cet exercice, vous allez créer une application cliente qui accepte l’entrée parlée et utilise votre application Language Understanding pour prédire l’intention de l’utilisateur.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python** dans cet exercice. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langue préférée.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **11-luis-speech** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langue.
2. Affichez le contenu du dossier **speaking-clock-client**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour inclure l’**ID d’application** de votre application Language Understanding, et l’**emplacement** (<u>pas</u> le point de terminaison complet, par exemple, *eastus*) et l’une des **clés** de sa ressource de prédiction (à partir de la page **Gérer** pour votre application dans le portail Language Understanding).

## <a name="install-sdk-packages"></a>Installer des packages SDK

Pour utiliser le kit de développement logiciel (SDK) Speech avec le service Language Understanding, vous devez installer le package SDK Speech pour votre langage de programmation.

1. Dans Visual Studio, cliquez avec le bouton droit sur le dossier **speaking-clock-client** et ouvrez un terminal intégré. Installez ensuite le package SDK Language Understanding en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

2. En outre, si votre système n’a <u>pas</u> de microphone de travail, vous devrez utiliser un fichier audio pour fournir une entrée parlée pour votre application. Dans ce cas, utilisez les commandes suivantes pour installer un package supplémentaire afin que votre programme puisse lire le fichier audio (vous pouvez ignorer cela si vous envisagez d’utiliser un microphone) :

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

3. Notez que le dossier **speaking-clock-client** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : speaking-clock-client.py

4. Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Speech :

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Intent;
    ```

    **Python**

    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. En outre, si votre système <u>n’a pas</u> de microphone de travail, sous les importations d’espace de noms existants, ajoutez le code suivant pour importer la bibliothèque que vous utiliserez pour lire un fichier audio :

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

## <a name="create-an-intentrecognizer"></a>Créer un *IntentRecognizer*

La classe **IntentRecognizer** fournit un objet client que vous pouvez utiliser pour obtenir des prédictions Language Understanding à partir d’une entrée parlée.

1. Dans la fonction **Main**, notez que le code pour charger l’ID d’application, la région de prédiction et la clé à partir du fichier de configuration ont déjà été fournis. Recherchez ensuite le commentaire **Configurer le service de reconnaissance vocale et obtenez le module de reconnaissance d’intention**, puis ajoutez le code suivant selon que vous utiliserez un microphone ou un fichier audio pour l’entrée vocale :

    ### <a name="if-you-have-a-working-microphone"></a>**Si vous disposez d’un microphone de travail :**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

    ### <a name="if-you-need-to-use-an-audio-file"></a>**Si vous devez utiliser un fichier audio :**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    string audioFile = "time-in-london.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    System.Threading.Thread.Sleep(2000);
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    audioFile = 'time-in-london.wav'
    playsound(audioFile)
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

## <a name="get-a-predicted-intent-from-spoken-input"></a>Obtenir une intention prédite à partir d’une entrée parlée

Vous êtes maintenant prêt à implémenter du code qui utilise le kit de développement logiciel (SDK) Speech pour obtenir une intention prédite à partir d’une entrée parlée.

1. Dans la fonction **Main**, immédiatement sous le code que vous venez d’ajouter, recherchez le commentaire **Obtenir le modèle à partir de l’AppID et ajoutez les intentions que nous voulons utiliser**, et ajoutez le code suivant pour obtenir votre modèle Language Understanding (en fonction de son ID d’application) et spécifiez les intentions que nous voulons que le module de reconnaissance identifie.

    **C#**

    ```C#
    // Get the model from the AppID and add the intents we want to use
    var model = LanguageUnderstandingModel.FromAppId(luAppId);
    recognizer.AddIntent(model, "GetTime", "time");
    recognizer.AddIntent(model, "GetDate", "date");
    recognizer.AddIntent(model, "GetDay", "day");
    recognizer.AddIntent(model, "None", "none");
    ```

    *Notez que vous pouvez spécifier un ID basé sur une chaîne pour chaque intention*

    **Python**

    ```Python
    # Get the model from the AppID and add the intents we want to use
    model = speech_sdk.intent.LanguageUnderstandingModel(app_id=lu_app_id)
    intents = [
        (model, "GetTime"),
        (model, "GetDate"),
        (model, "GetDay"),
        (model, "None")
    ]
    recognizer.add_intents(intents)
    ```

2. Sous le commentaire **Traiter la saisie vocale**, ajoutez le code suivant, qui utilise le module de reconnaissance pour appeler de façon asynchrone le service Language Understanding avec une entrée parlée et récupérer la réponse. Si la réponse inclut une intention prédite, la requête parlée, l’intention prédite et la réponse JSON complète s’affichent. Sinon, le code gère la réponse en fonction de la raison retournée.

**C#**

```C
// Process speech input
string intent = "";
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
if (result.Reason == ResultReason.RecognizedIntent)
{
    // Intent was identified
    intent = result.IntentId;
    Console.WriteLine($"Query: {result.Text}");
    Console.WriteLine($"Intent Id: {intent}.");
    string jsonResponse = result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult);
    Console.WriteLine($"JSON Response:\n{jsonResponse}\n");
    
    // Get the first entity (if any)

    // Apply the appropriate action
    
}
else if (result.Reason == ResultReason.RecognizedSpeech)
{
    // Speech was recognized, but no intent was identified.
    intent = result.Text;
    Console.Write($"I don't know what {intent} means.");
}
else if (result.Reason == ResultReason.NoMatch)
{
    // Speech wasn't recognized
    Console.WriteLine($"Sorry. I didn't understand that.");
}
else if (result.Reason == ResultReason.Canceled)
{
    // Something went wrong
    var cancellation = CancellationDetails.FromResult(result);
    Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");
    if (cancellation.Reason == CancellationReason.Error)
    {
        Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
    }
}
```

**Python**

```Python
# Process speech input
intent = ''
result = recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedIntent:
    intent = result.intent_id
    print("Query: {}".format(result.text))
    print("Intent: {}".format(intent))
    json_response = json.loads(result.intent_json)
    print("JSON Response:\n{}\n".format(json.dumps(json_response, indent=2)))
    
    # Get the first entity (if any)
    
    # Apply the appropriate action
    
elif result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    # Speech was recognized, but no intent was identified.
    intent = result.text
    print("I don't know what {} means.".format(intent))
elif result.reason == speech_sdk.ResultReason.NoMatch:
    # Speech wasn't recognized
    print("Sorry. I didn't understand that.")
elif result.reason == speech_sdk.ResultReason.Canceled:
    # Something went wrong
    print("Intent recognition canceled: {}".format(result.cancellation_details.reason))
    if result.cancellation_details.reason == speech_sdk.CancellationReason.Error:
        print("Error details: {}".format(result.cancellation_details.error_details))
```

Le code que vous avez ajouté identifie jusqu’à présent l’*intention*, mais certaines intentions peuvent référencer des *entités*. Vous devez donc ajouter du code pour extraire les informations d’entité du JSON retourné par le service.

3. Dans le code que vous venez d’ajouter, recherchez le commentaire **Obtenir la première entité (le cas échéant)** , et ajoutez le code suivant sous celui-ci :

**C#**

```C
// Get the first entity (if any)
JObject jsonResults = JObject.Parse(jsonResponse);
string entityType = "";
string entityValue = "";
if (jsonResults["entities"].HasValues)
{
    JArray entities = new JArray(jsonResults["entities"][0]);
    entityType = entities[0]["type"].ToString();
    entityValue = entities[0]["entity"].ToString();
    Console.WriteLine(entityType + ": " + entityValue);
}
```

**Python**

```Python
# Get the first entity (if any)
entity_type = ''
entity_value = ''
if len(json_response["entities"]) > 0:
    entity_type = json_response["entities"][0]["type"]
    entity_value = json_response["entities"][0]["entity"]
    print(entity_type + ': ' + entity_value)
```
    
Votre code utilise désormais l’application Language Understanding pour prédire une intention ainsi que toutes les entités détectées dans l’énoncé d’entrée. Votre application cliente doit maintenant utiliser cette prédiction pour déterminer et effectuer l’action appropriée.

4. Sous le code que vous venez d’ajouter, recherchez le commentaire **Appliquer l’action appropriée** et ajoutez le code suivant, qui recherche les intentions prises en charge par l’application (**GetTime**, **GetDate** et **GetDay**) et détermine si des entités pertinentes ont été détectées, avant d’appeler une fonction existante pour produire une réponse appropriée.

**C#**

```C#
// Apply the appropriate action
switch (intent)
{
    case "time":
        var location = "local";
        // Check for entities
        if (entityType == "Location")
        {
            location = entityValue;
        }
        // Get the time for the specified location
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;
    case "day":
        var date = DateTime.Today.ToShortDateString();
        // Check for entities
        if (entityType == "Date")
        {
            date = entityValue;
        }
        // Get the day for the specified date
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;
    case "date":
        var day = DateTime.Today.DayOfWeek.ToString();
        // Check for entities
        if (entityType == "Weekday")
        {
            day = entityValue;
        }

        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;
    default:
        // Some other intent (for example, "None") was predicted
        Console.WriteLine("You said " + result.Text.ToLower());
        if (result.Text.ToLower().Replace(".", "") == "stop")
        {
            intent = result.Text;
        }
        else
        {
            Console.WriteLine("Try asking me for the time, the day, or the date.");
        }
        break;
}
```

**Python**

```Python
# Apply the appropriate action
if intent == 'GetTime':
    location = 'local'
    # Check for entities
    if entity_type == 'Location':
        location = entity_value
    # Get the time for the specified location
    print(GetTime(location))

elif intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if entity_type == 'Date':
        date_string = entity_value
    # Get the day for the specified date
    print(GetDay(date_string))

elif intent == 'GetDate':
    day = 'today'
    # Check for entities
    if entity_type == 'Weekday':
        # List entities are lists
        day = entity_value
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('You said {}'.format(result.text))
    if result.text.lower().replace('.', '') == 'stop':
        intent = result.text
    else:
        print('Try asking me for the time, the day, or the date.')
```

## <a name="run-the-client-application"></a>Exécution de l’application cliente

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock-client** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock-client.py
    ```

2. Si vous utilisez un microphone, parlez à haute voix pour tester l’application. Par exemple, essayez ce qui suit (en réexécutant le programme chaque fois) :

    *Quelle heure est-il ?*
    
    *Quelle heure est-il ?*

    *Quel jour sommes-nous ?*

    *Quelle heure est-il à Londres ?*

    *Quelle est la date ?*

    *Quelle est la date du dimanche ?*

> **Remarque** : La logique de l’application est délibérément simple et a un certain nombre de limitations, mais doit servir à tester la possibilité pour le modèle Language Understanding de prédire les intentions à partir d’une entrée parlée à l’aide du kit de développement logiciel (SDK) Speech. Vous avez peut-être des difficultés à reconnaître l’intention **GetDay** avec une entité de date spécifique en raison de la difficulté de verbaliser une date au format *MM/DD/AAAA* !

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur l’intégration de Speech et de Language Understanding, consultez la [documentation Speech](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition).
