---
lab:
  title: Analyser le texte
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 27cd22e81d4fe8fda27e6fc960d3b3aeb8debded
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/12/2022
ms.locfileid: "145195625"
---
# <a name="analyze-text"></a>Analyser le texte

Le service **Langue** est un service cognitif qui prend en charge l’analyse du texte, notamment la détection de langue, l’analyse des sentiments, l’extraction d’expressions clés et la reconnaissance d’entité.

Par exemple, supposons qu’une agence de voyages souhaite traiter les avis d’hôtel soumis au site web de l’entreprise. En utilisant le service Langue, ils peuvent déterminer la langue dans laquelle chaque révision est écrite, le sentiment (positif, neutre ou négatif) des avis, les expressions clés qui peuvent indiquer les principaux sujets abordés dans l’avis et les entités nommées, telles que les lieux, les repères ou les personnes mentionnées dans les avis.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour cette formation

Si vous n’avez pas déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement où vous travaillez sur ce laboratoire, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git: Clone** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="provision-a-cognitive-services-resource"></a>Provisionner une ressource Cognitive Services

Si vous n’en avez pas encore dans votre abonnement, vous devez provisionner une ressource **Cognitive Services**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Cognitive Services*, puis créez une ressource **Cognitive Services** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *entrez un nom unique.*
    - **Niveau tarifaire** : standard S0
3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## <a name="prepare-to-use-the-language-sdk-for-text-analytics"></a>Préparer l’utilisation du kit de développement logiciel (SDK) Langue pour l’analyse de texte

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) d’analyse de texte du service Langue pour analyser les avis sur un hôtel.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **05-analyze-text** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier **text-analysis** et ouvrez un terminal intégré. Installez ensuite le package SDK d’analyse de texte en exécutant la commande appropriée pour votre préférence de langage :
    
    **C#**
    
    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```
    
    **Python**
    
    ```
    pip install azure-ai-textanalytics==5.1.0
    ```
    
3. Affichez le contenu du dossier **text-analysis**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification de votre ressource services cognitifs. Enregistrez vos modifications.

4. Notez que le dossier **text-analysis** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : text-analysis.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’analyse de texte :

    **C#**
    
    ```C#
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**

    ```Python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

5. Dans la fonction **Main**, notez que le code pour charger le point de terminaison et la clé des services cognitifs à partir du fichier de configuration a déjà été fourni. Recherchez ensuite le commentaire **Créer un client à l’aide du point de terminaison et de la clé**, puis ajoutez le code suivant pour créer un client pour l’API Analyse de texte :

    **C#**

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(cogSvcKey);
    Uri endpoint = new Uri(cogSvcEndpoint);
    TextAnalyticsClient CogClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(cog_key)
    cog_client = TextAnalyticsClient(endpoint=cog_endpoint, credential=credential)
    ```

6. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-analysis** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

6. Observez le résultat pour vérifier si le code s'exécute sans erreur, en affichant le contenu de chaque fichier texte d'avis dans le dossier **Avis**. L’application crée correctement un client pour l’API Analyse de texte, mais ne l’utilise pas. Nous le corrigerons dans la procédure suivante.

## <a name="detect-language"></a>Détecter la langue

Maintenant que vous avez créé un client pour l’API Analyse de texte, nous allons l’utiliser pour détecter la langue dans laquelle chaque révision est écrite.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir la langue**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour détecter la langue dans chaque document d’avis :

    **C#**
    
    ```C
    // Get language
    DetectedLanguage detectedLanguage = CogClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**
    
    ```Python
    # Get language
    detectedLanguage = cog_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

    > **Remarque** : *Dans cet exemple, chaque avis est analysé individuellement, ce qui entraîne un appel distinct au service pour chaque fichier. Une autre approche consiste à créer une collection de documents et à les transmettre au service en un seul appel. Dans les deux approches, la réponse du service consiste en une collection de documents ; c'est pourquoi dans le code Python ci-dessus, l'index du premier (et seul) document de la réponse ([0]) est spécifié.*

6. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-analysis** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

7. Observez la sortie, notant que cette fois la langue de chaque avis est identifiée.

## <a name="evaluate-sentiment"></a>Évaluer les sentiments

*L’analyse des sentiments* est une technique couramment utilisée pour classer le texte comme *positif* ou *négatif* (ou *éventuellement neutre* ou *mixte*). Elle est couramment utilisée pour analyser les publications de médias sociaux, les avis sur des produits et d’autres éléments où le sentiment du texte peut fournir des informations utiles.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir le sentiment**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour détecter le sentiment dans chaque document d’avis :

    **C#**
    
    ```C
    // Get sentiment
    DocumentSentiment sentimentAnalysis = CogClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**
    
    ```Python
    # Get sentiment
    sentimentAnalysis = cog_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

2. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-analysis** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

3. Observez le résultat pour vérifier la détection d'un sentiment dans les avis.

## <a name="identify-key-phrases"></a>Identifier les expressions clés

Il peut être utile d’identifier les expressions clés dans un corps de texte pour vous aider à déterminer les sujets principaux qu’il traite.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir les expressions clés**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour détecter les expressions clés dans chaque document d’avis :

    **C#**

    ```C
    // Get key phrases
    KeyPhraseCollection phrases = CogClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```
    
    **Python**
    
    ```Python
    # Get key phrases
    phrases = cog_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

2. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-analysis** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Observez la sortie, notant que chaque document contient des expressions clés qui donnent des informations sur l’avis.

## <a name="extract-entities"></a>Extraire les entités

Souvent, les documents ou d’autres corps de texte mentionnent des personnes, des lieux, des périodes ou d’autres entités. L’API d’analyse de texte peut détecter plusieurs catégories (et sous-catégories) d’entités dans votre texte.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir les entités**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour identifier les entités mentionnées dans chaque avis :

    **C#**
    
    ```C
    // Get entities
    CategorizedEntityCollection entities = CogClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get entities
    entities = cog_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

2. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-analysis** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Observez la sortie, notant les entités détectées dans le texte.

## <a name="extract-linked-entities"></a>Extraire des entités liées

Outre les entités classées, l’API d’analyse de texte peut détecter les entités pour lesquelles il existe des liens connus vers des sources de données, telles que Wikipédia.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir les entités liées**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour identifier les entités liées mentionnées dans chaque avis :

    **C#**
    
    ```C
    // Get linked entities
    LinkedEntityCollection linkedEntities = CogClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get linked entities
    entities = cog_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

2. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-analysis** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Observez la sortie, en notant les entités liées identifiées.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur l’utilisation du service **Langue**, consultez la [documentation relative à l'analyse de texte](https://docs.microsoft.com/azure/cognitive-services/language-service/).
