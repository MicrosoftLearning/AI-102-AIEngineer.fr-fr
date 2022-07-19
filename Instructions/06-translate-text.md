---
lab:
  title: Traduire du texte
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 4ca4394ce5d9456abeabbdded1ee219f271f284e
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195559"
---
# <a name="translate-text"></a>Traduire du texte

Le service **Translator** est un service cognitif qui vous permet de traduire du texte d’une langue vers une autre.

Par exemple, supposons qu’une agence de voyages souhaite examiner les avis sur les hôtels qui ont été soumis au site web de l’entreprise, en standardisant l’anglais comme langue utilisée pour l’analyse. En utilisant le service Translator,elle peut déterminer la langue dans laquelle chaque avis est écrit et, si un avis n’est pas déjà en anglais, le traduire àpartir de la langue source dans laquelle il a été écrit vers l’anglais.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

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
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin de l’une des clés et de l’emplacement dans lequel le service est approvisionné à partir de cette page dans la procédure suivante.

## <a name="prepare-to-use-the-translator-service"></a>Préparer l’utilisation du service Translator

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise l’API REST Traducteur pour traduire les avis sur un hôtel.

> **Remarque** : Vous pouvez choisir d’utiliser l’API à partir de **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **06-translate-text** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Affichez le contenu du dossier **text-translation**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour inclure une **clé** d’authentification dans votre ressource Cognitive Services et l’**emplacement** où elle est déployée (et <u>non</u> le point de terminaison). Vous devez les copier à partir de la page **Clés et point de terminaison** de votre ressource Cognitive Services. Enregistrez vos modifications.
3. Notez que le dossier **text-translation** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : text-translation.py

    Ouvrez le fichier de coder et examinez le code qu’il contient.

4. Dans la fonction **Main**, notez que le code pour charger la clé et la région Cognitive Services à partir du fichier de configuration a déjà été fourni. Le point de terminaison du service de traduction est également spécifié dans votre code.
5. Cliquez avec le bouton droit sur le dossier **text-translation**, ouvrez un terminal intégré, puis entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

6. Observez la sortie comme le code doit s’exécuter sans erreur, affichant le contenu de chaque fichier texte d’avis dans le dossier **avis**. Actuellement, l’application n’utilise pas le service Translator. Nous corrigerons cela dans la procédure suivante.

## <a name="detect-language"></a>Détecter la langue

Le service Translator peut détecter automatiquement la langue source du texte à traduire, mais vous permetégalement de détecter explicitement la langue dans laquelle le texte est écrit.

1. Dans votre fichier de code, recherchez la fonction **GetLanguage**, qui retourne actuellement « en » pour toutes les valeurs de texte.
2. Dans la fonction **GetLanguage**, sous le commentaire **Utiliser la fonction de détection Traducteur**, ajoutez le code suivant pour utiliser l’API REST de Traducteur pour détecter la langue du texte spécifié, en étant prudent de ne pas remplacer le code à la fin de la fonction qui retourne la langue :

**C#**

```C#
// Use the Translator detect function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/detect?api-version=3.0";
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get language
        JArray jsonResponse = JArray.Parse(responseContent);
        language = (string)jsonResponse[0]["language"]; 
    }
}
```

**Python**

```Python
# Use the Translator detect function
path = '/detect'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0'
}

headers = {
'Ocp-Apim-Subscription-Key': cog_key,
'Ocp-Apim-Subscription-Region': cog_region,
'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get language
language = response[0]["language"]
```

3. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-translation** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. Observez la sortie, notant que cette fois la langue de chaque avis est identifiée.

## <a name="translate-text"></a>Traduire le texte

Maintenant que votre application peut déterminer la langue dans laquelle les avis sont écrits, vous pouvez utiliser le service Translator pour traduire les avis dans une langue autre que l’anglais vers l’anglais.

1. Dans votre fichier de code, recherchez la fonction **Translate**, qui retourne et vide la chaîne pour toutes les valeurs de texte.
2. Dans la fonction **Translate**, sous le commentaire **Utiliser la fonction de traduction Traducteur**, ajoutez le code suivant pour utiliser l’API REST de Traducteur pour traduire le texte spécifié de sa langue source vers l’anglais, en étant prudent de ne pas remplacer le code à la fin de la fonction qui retourne la traduction :

**C#**

```C#
// Use the Translator translate function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/translate?api-version=3.0&from=" + sourceLanguage + "&to=en" ;
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get translation
        JArray jsonResponse = JArray.Parse(responseContent);
        translation = (string)jsonResponse[0]["translations"][0]["text"];  
    }
}
```

**Python**

```Python
# Use the Translator translate function
path = '/translate'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0',
    'from': source_language,
    'to': ['en']
}

headers = {
    'Ocp-Apim-Subscription-Key': cog_key,
    'Ocp-Apim-Subscription-Region': cog_region,
    'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get translation
translation = response[0]["translations"][0]["text"]
```

3. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-translation** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. Observez le résultat et notez que les avis dans une langue autre que l’anglais sont traduits en anglais.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur l’utilisation du service **Translator**, consultez la [documentation d’Azure Cognitive Services Translator](https://docs.microsoft.com/azure/cognitive-services/translator/).
