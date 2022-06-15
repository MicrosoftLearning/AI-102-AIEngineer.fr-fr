---
lab:
  title: Traduire du contenu vocal
  module: Module 4 - Building Speech-Enabled Applications
ms.openlocfilehash: 45c8d0d31bee5901247b22917d8dbad8c587a0bd
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/12/2022
ms.locfileid: "145195605"
---
# <a name="translate-speech"></a>Traduire du contenu vocal

Le service **Speech** inclut une API de **traduction vocale** que vous pouvez utiliser pour traduire une langue parlée. Par exemple, supposons que vous souhaitiez développer une application de traduction que des utilisateurs peuvent utiliser lorsqu’ils voyagent dans des pays où ils ne parlent pas la langue locale. Ils seraient en mesure de dire des phrases telles que « Où se trouve la gare ? » ou « J’ai besoin de trouver une pharmacie » dans leur propre langue et de les traduire dans la langue locale.

**Remarque** : Cet exercice nécessite que vous utilisiez un ordinateur avec des haut-parleurs ou des écouteurs. Pour une expérience optimale, un microphone est également nécessaire. Certains environnements virtuels hébergés peuvent être en mesure de capturer l’audio à partir de votre microphone local, mais si cela ne fonctionne pas (ou si vous n’avez pas de microphone du tout), vous pouvez utiliser un fichier audio fourni pour l’entrée vocale. Suivez attentivement les instructions, car vous devez choisir différentes options selon que vous utilisez un microphone ou le fichier audio.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="provision-a-cognitive-services-resource"></a>Approvisionner une ressource Cognitive Services

Si vous n’en avez pas encore dans votre abonnement, vous devez approvisionner une ressource **Cognitive Services**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Cognitive Services*, puis créez une ressource **Cognitive Services** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S0
3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin de l’une des clés et de l’emplacement dans lequel le service est approvisionné à partir de cette page dans la procédure suivante.

## <a name="prepare-to-use-the-speech-translation-service"></a>Préparer l’utilisation du service Traduction vocale

Dans cet exercice, vous allez finaliser une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) Speech pour reconnaître, traduire et synthétiser du contenu vocal.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **08-speech-translation** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit de la souris sur le dossier **translator** et ouvrez un terminal intégré. Installez ensuite le package SDK Speech en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.19.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.19.0
    ```

3. Affichez le contenu du dossier **translator**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour inclure une **clé** d’authentification dans votre ressource Cognitive Services et l’**emplacement** où elle est déployée. Enregistrez vos modifications.
4. Notez également que le dossier **translator** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : translator.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Speech :

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. Dans la fonction **Main**, notez que le code pour charger la clé et la région Cognitive Services à partir du fichier de configuration a déjà été fourni. Vous devez utiliser ces variables pour créer un objet **SpeechTranslationConfig** pour votre ressource Cognitive Services, que vous utiliserez pour traduire des entrées parlées. Ajoutez le code suivant sous le commentaire **Configurer la traduction** :

    **C#**
    
    ```C#
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```
    
    **Python**
    
    ```Python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(cog_key, cog_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

6. Vous utiliserez l’objet **SpeechTranslationConfig** pour traduire la parole en texte, mais vous utiliserez également un objet **SpeechConfig** pour synthétiser des traductions en parole. Ajoutez le code suivant sous le commentaire **Configurer Speech** :

    **C#**
    
    ```C#
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    ```
    
    **Python**
    
    ```Python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    ```

7. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **translator** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

8. Si vous utilisez C#, vous pouvez ignorer tous les avertissements concernant l’utilisation de l’opérateur **await** dans les méthodes asynchrones. Nous les corrigerons ultérieurement. Le code doit afficher un message indiquant qu’il est prêt à traduire à partir de la langue en-US. Appuyez sur ENTRÉE pour mettre fin au programme.

## <a name="implement-speech-translation"></a>Implémenter la traduction vocale

Maintenant que vous disposez d’un objet **SpeechTranslationConfig** pour le service Speech dans votre ressource Cognitive Services, vous pouvez utiliser l’API **Traduction vocale** pour reconnaître et traduire le contenu vocal.

### <a name="if-you-have-a-working-microphone"></a>Si vous disposez d’un microphone de travail

1. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **Translate** pour traduire l’entrée parlée.
2. Dans la fonction **Translate**, sous le commentaire **Traduire du contenu vocal**, ajoutez le code suivant pour créer un client **TranslationRecognizer** qui peut être utilisé pour reconnaître et traduire du contenu vocal à l’aide du microphone système par défaut pour l’entrée.

    **C#**
    
    ```C#
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **Remarque** : Le code de votre application traduit l’entrée en trois langues dans un seul appel. Seule la traduction de la langue spécifique est affichée, mais vous pouvez récupérer l’une des traductions en spécifiant le code de la langue cible dans la collection de **traductions** du résultat.

3. Passez maintenant à la section **Exécuter le programme** ci-dessous.

### <a name="alternatively-use-audio-input-from-a-file"></a>Vous pouvez également utiliser l’entrée audio à partir d’un fichier

1. Dans la fenêtre de terminal, entrez la commande suivante pour installer une bibliothèque que vous pouvez utiliser pour lire le fichier audio :

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

2. Dans le fichier de code de votre programme, sous les importations d’espaces de noms existants, ajoutez le code suivant pour importer la bibliothèque que vous venez d’installer :

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

3. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **Translate** pour traduire l’entrée parlée. Puis, dans la fonction **Translate**, sous le commentaire **Traduire du contenu vocal**, ajoutez le code suivant pour créer un client **TranslationRecognizer** qui peut être utilisé pour reconnaître et traduire du contenu vocal à partir d’un fichier.

    **C#**
    
    ```C#
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **Remarque** : Le code de votre application traduit l’entrée en trois langues dans un seul appel. Seule la traduction de la langue spécifique est affichée, mais vous pouvez récupérer l’une des traductions en spécifiant le code de la langue cible dans la collection de **traductions** du résultat.

### <a name="run-the-program"></a>Exécuter le programme

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **translator** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

2. Lorsque vous y êtes invité, entrez un code de langue valide (*fr*, *es* ou *hi*), puis, si vous utilisez un microphone, parlez distinctement et dites « où se trouve la gare ? » ou une autre phrase que vous pourriez utiliser lors d’un voyage à l’étranger. Le programme doit transcrire votre entrée parlée et la traduire dans la langue que vous avez spécifiée (français, espagnol ou hindi). Répétez ce processus, en essayant chaque langue prise en charge par l’application. Lorsque vous avez terminé, appuyez sur ENTRÉE pour terminer le programme.

    > **Remarque** : TranslationRecognizer vous donne environ 5 secondes pour parler. S’il ne détecte aucune entrée parlée, il génère un résultat « Aucune correspondance ».
    >
    > La traduction en hindi peut ne pas toujours être affichée correctement dans la fenêtre Console en raison de problèmes d’encodage de caractères.

## <a name="synthesize-the-translation-to-speech"></a>Synthétiser la traduction en parole

Pour le moment, votre application traduit l’entrée parlée en texte, ce qui peut être suffisant si vous devez demander de l’aide pendant un voyage. Toutefois, il serait préférable d’avoir la traduction parlée à haute voix dans une voix appropriée.

1. Dans la fonction **Translate**, sous le commentaire **Synthétiser la traduction**, ajoutez le code suivant pour utiliser un client **SpeechSynthesizer** afin de synthétiser la traduction en tant que parole par le biais du haut-parleur par défaut :

    **C#**
    
    ```C#
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **translator** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

3. Lorsque vous y êtes invité, entrez un code de langue valide (*fr*, *es* ou *hi*), puis parlez distinctement dans le microphone et dites une phrase que vous pourriez utiliser lors d’un voyage à l’étranger. Le programme doit transcrire votre entrée parlée et répondre avec une traduction parlée. Répétez ce processus, en essayant chaque langue prise en charge par l’application. Lorsque vous avez terminé, appuyez sur ENTRÉE pour terminer le programme.

    > **Remarque** *Dans cet exemple, vous avez utilisé un objet **SpeechTranslationConfig** pour traduire de la parole en texte, puis utilisé un objet **SpeechConfig** afin de synthétiser la traduction en parole. De fait, vous pouvez utiliser l’objet **SpeechTranslationConfig** pour synthétiser directement la traduction, mais cela ne fonctionne uniquement que lorsque vous traduisez vers une seule langue, et retourne un flux audio généralement enregistré en tant que fichier plutôt qu’envoyé vers un haut-parleur.*

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur l’utilisation de l’API **Traduction vocale**, consultez la [documentation Traduction vocale](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-translation).
