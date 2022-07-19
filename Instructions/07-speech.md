---
lab:
  title: Reconnaître et synthétiser du contenu vocal
  module: Module 4 - Building Speech-Enabled Applications
ms.openlocfilehash: 4f65f068cab76299f838153d32a2d4e8979a1253
ms.sourcegitcommit: 3374bf6b03869daf624f3916bc34510fcbe580e0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/18/2022
ms.locfileid: "145195552"
---
# <a name="recognize-and-synthesize-speech"></a>Reconnaître et synthétiser du contenu vocal

Le service **Speech** est un service cognitif Azure qui fournit des fonctionnalités vocales, notamment :

- Une API de *reconnaissance vocale* qui vous permet d’implémenter la reconnaissance vocale (conversion de mots parlés audibles en texte).
- Une API de *synthèse vocale* qui vous permet d’implémenter la synthèse vocale (conversion de texte en parole audible).

Dans cet exercice, vous allez utiliser ces deux API pour implémenter une application d’horloge parlante.

**Remarque** : cet exercice nécessite que vous utilisiez un ordinateur avec des haut-parleurs ou des écouteurs. Pour une expérience optimale, un microphone est également nécessaire. Certains environnements virtuels hébergés peuvent être en mesure de capturer l’audio à partir de votre microphone local, mais si cela ne fonctionne pas (ou si vous n’avez pas de microphone du tout), vous pouvez utiliser un fichier audio fourni pour l’entrée vocale. Suivez attentivement les instructions, car vous devez choisir différentes options selon que vous utilisez un microphone ou le fichier audio.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour cette formation

Si vous n’avez pas déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement où vous travaillez sur ce laboratoire, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

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
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin de l’une des clés et de l’emplacement dans lequel le service est approvisionné à partir de cette page dans la procédure suivante.

## <a name="prepare-to-use-the-speech-service"></a>Préparer l’utilisation du service Speech

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) Speech pour reconnaître et synthétiser du contenu vocal.

**Remarque** : vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **07-speech** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit de la souris sur le dossier **speaking-clock** et ouvrez un terminal intégré. Installez ensuite le package SDK Speech en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.19.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.19.0
    ```

3. Affichez le contenu du dossier **speaking-clock**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour inclure une **clé** d’authentification dans votre ressource Cognitive Services et l’**emplacement** où elle est déployée. Enregistrez vos modifications.
4. Notez que le dossier **speaking-clock** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : speaking-clock.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Speech :

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. Dans la fonction **Main**, notez que le code pour charger la clé et la région Cognitive Services à partir du fichier de configuration a déjà été fourni. Vous devez utiliser ces variables pour créer un objet **SpeechConfig** pour votre ressource Cognitive Services. Ajoutez le code suivant sous le commentaire **Configurer le service Speech** :

    **C#**
    
    ```C#
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```
    
    **Python**
    
    ```Python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

6. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

7. Si vous utilisez C#, vous pouvez ignorer tous les avertissements concernant l’utilisation de l’opérateur **await** dans les méthodes asynchrones. Nous les corrigerons ultérieurement. Le code doit afficher la région de la ressource du service Speech que l’application utilisera.

## <a name="recognize-speech"></a>Reconnaissance vocale

Maintenant que vous disposez d’un objet **SpeechConfig** pour le service Speech dans votre ressource Cognitive Services, vous pouvez utiliser l’API **Reconnaissance vocale** pour reconnaître le discours audible et le transcrire en texte.

### <a name="if-you-have-a-working-microphone"></a>Si vous disposez d’un microphone de travail

1. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **TranscribeCommand** pour accepter l’entrée parlée.
2. Dans la fonction **TranscribeCommand**, sous le commentaire **Configurer la reconnaissance vocale**, ajoutez le code approprié ci-dessous pour créer un client **SpeechRecognizer** qui peut être utilisé pour reconnaître et transcrire du contenu vocal à l’aide du microphone système par défaut :

    **C#**
    
    ```C#
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```
    
    **Python**
    
    ```Python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

3. Passez maintenant à la section **Ajouter du code pour traiter la commande transcrite** ci-dessous.

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

3. Dans la fonction **Main**, notez que le code utilise la fonction **TranscribeCommand** pour accepter l’entrée parlée. Puis, dans la fonction **TranscribeCommand**, sous le commentaire **Configurer la reconnaissance vocale**, ajoutez le code approprié ci-dessous pour créer un client **SpeechRecognizer** qui peut être utilisé pour reconnaître et transcrire du contenu vocal à partir d’un fichier audio :

    **C#**

    ```C#
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech recognition
    audioFile = 'time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

### <a name="add-code-to-process-the-transcribed-command"></a>Ajouter du code pour traiter la commande transcrite

1. Dans la fonction **TranscribeCommand**, sous le commentaire **Traiter l’entrée vocale**, ajoutez le code suivant pour écouter l’entrée parlée, en étant prudent de ne pas remplacer le code à la fin de la fonction qui retourne la commande :

    **C#**
    
    ```C#
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**
    
    ```Python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

2. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Si vous utilisez un microphone, parlez clairement et dites « quelle heure est-il ? ». Le programme doit transcrire votre entrée parlée et afficher l’heure (en fonction de l’heure locale de l’ordinateur sur lequel le code est en cours d’exécution, qui peut ne pas être l’heure exacte de votre emplacement).

    SpeechRecognizer vous donne environ 5 secondes pour parler. S’il ne détecte aucune entrée parlée, il génère un résultat « Aucune correspondance ».

    Si SpeechRecognizer rencontre une erreur, il génère un résultat « Annulé ». Le code de l’application affiche ensuite le message d’erreur. La cause la plus probable est une clé ou une région incorrecte dans le fichier de configuration.

## <a name="synthesize-speech"></a>Synthèse vocale

Votre application d’horloge parlante accepte l’entrée parlée, mais elle ne parle pas. Nous allons résoudre ce problème en ajoutant du code pour synthétiser du contenu vocal.

1. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **TellTime** pour indiquer l’heure actuelle à l’utilisateur.
2. Dans la fonction **TellTime**, sous le commentaire **Configurer la synthèse vocale**, ajoutez le code suivant pour créer un client **SpeechSynthesizer** qui peut être utilisé pour générer la sortie parlée :

    **C#**
    
    ```C#
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```
    
    **Python**
    
    ```Python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```
    
    > **Remarque** : *la configuration audio par défaut utilise l’appareil audio système par défaut pour la sortie. Vous n’avez donc pas besoin de fournir explicitement une **AudioConfig**. Si vous devez rediriger la sortie audio vers un fichier, vous pouvez utiliser une **AudioConfig** avec un FilePath.*

3. Dans la fonction **TellTime**, sous le commentaire **Synthétiser la sortie parlée**, ajoutez le code suivant pour générer la sortie parlée, en étant prudent de ne pas remplacer le code à la fin de la fonction qui imprime la réponse :

    **C#**
    
    ```C#
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

4. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

5. Lorsque vous y êtes invité, parlez clairement dans le microphone et dites « quelle heure est-il ? ». Le programme doit parler et vous indiquer l’heure.

## <a name="use-a-different-voice"></a>Utiliser une autre voix

Votre application d’horloge parlante utilise une voix par défaut, que vous pouvez modifier. Le service Speech prend en charge une gamme de voix *standard* ainsi que des voix *neuronales* quasi humaines. Vous pouvez également créer des voix *personnalisées*.

> **Remarque** : pour accéder à la liste des voix standard et neuronales, consultez l'article [Prise en charge des langues et de la voix](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#text-to-speech) pour le service Speech.

1. Dans la fonction **TellTime**, sous le commentaire **Configurer la synthèse vocale**, modifiez le code comme suit pour spécifier une autre voix avant de créer le client **SpeechSynthesizer** :

   **C#**

    ```C#
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```
    
    **Python**
    
    ```Python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

2. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Lorsque vous y êtes invité, parlez clairement dans le microphone et dites « quelle heure est-il ? ». Le programme doit parler dans la voix spécifiée, vous indiquant l’heure.

## <a name="use-speech-synthesis-markup-language"></a>Utiliser le langage de balisage de synthèse vocale

Le langage SSML (Speech Synthesis Markup Language) vous permet de personnaliser la façon dont vos paroles sont synthétisées à l’aide d’un format XML.

1. Dans la fonction **TellTime**, remplacez tout le code actuel sous le commentaire **Synthétiser la sortie parlée** par le code suivant (laissez le code sous le commentaire **Imprimer la réponse**) :

   **C#**

    ```C#
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**
    
    ```Python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Lorsque vous y êtes invité, parlez clairement dans le microphone et dites « quelle heure est-il ? ». Le programme doit parler dans la voix qui est spécifiée dans le langage SSML (en remplaçant la voix spécifiée dans l’objet SpeechConfig). Il doit vous indiquer l’heure puis, après une pause, vous dire qu’il est temps de conclure ce labo, ce qui est le cas !

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur l’utilisation des API **Reconnaissance vocale** et **Synthèse vocale**, consultez les articles [Documentation sur la reconnaissance vocale](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-to-text) et [Documentation sur la synthèse vocale](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-text-to-speech).
