---
lab:
  title: Lire le texte contenu dans des images
  module: Module 11 - Reading Text in Images and Documents
ms.openlocfilehash: 1199e4e4f44a98fc5f900fa1ad021384b56f0c2b
ms.sourcegitcommit: e242452e8125a2622093980048f1e2cacb8b893d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/25/2022
ms.locfileid: "145757488"
---
# <a name="read-text-in-images"></a>Lire le texte contenu dans des images

La reconnaissance optique de caractères (OCR) est un sous-ensemble de la vision par ordinateur qui traite de la lecture de texte dans des images et des documents. Le service **Vision par ordinateur** fournit deux API pour lire du texte, que vous allez explorer dans cet exercice.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour cette formation

Si vous ne l’avez pas déjà fait, vous devez cloner le référentiel de code pour ce cours :

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
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
    - **Niveau tarifaire** : Standard S0
3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Préparer l’utilisation du pit de développement logiciel (SDK) de vision par ordinateur

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) de vision par ordinateur pour lire du texte.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **20-ocr** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier **create-search**, puis sélectionnez Ouvrir dans le terminal intégré. Installez ensuite le package SDK de vision par ordinateur en exécutant la commande appropriée pour votre préférence de langage :

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```

3. Affichez le contenu du dossier **read-text**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification de votre ressource services cognitifs. Enregistrez vos modifications.
4. Notez que le dossier **read-text** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : read-text.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) de vision par ordinateur :

**C#**

```C#
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```

5. Dans le fichier de code de votre application cliente, dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client de vision par ordinateur**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet client vision par ordinateur :

**C#**

```C#
// Authenticate Computer Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Computer Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```
    
## <a name="use-the-ocr-api"></a>Utiliser l’API OCR

L’API **OCR** est une API de reconnaissance optique de caractères optimisée pour la lecture de petites et moyennesquantités de texte imprimé dans images au format *.jpg*, *.png*, *.gif*,et *.bmp*. Il prend en charge un large éventail de langues et en plus de lire du texte dans l’image, il peut déterminer l’orientation de chaque région de texte et retourner des informations sur l’angle de rotation du texte par rapport à l’image

1. Dans le fichier de code de votre application, dans la fonction **Main** , examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **1**. Ce code appelle la fonction **GetTextOcr**, en passant le chemin d’accès à un fichier image.
2. Dans le dossier **read-text/images**, ouvrez **Lincoln.jpg** pour afficher l’image que votre code traitera.
3. De retour dans le fichier de code, recherchez la fonction **GetTextOcr** et, sous le code existant qui imprime un message dans la console, ajoutez le code suivant :

**C#**

```C#
// Use OCR API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var ocrResults = await cvClient.RecognizePrintedTextInStreamAsync(detectOrientation:false, image:imageData);

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Magenta, 3);

    foreach(var region in ocrResults.Regions)
    {
        foreach(var line in region.Lines)
        {
            // Show the position of the line of text
            int[] dims = line.BoundingBox.Split(",").Select(int.Parse).ToArray();
            Rectangle rect = new Rectangle(dims[0], dims[1], dims[2], dims[3]);
            graphics.DrawRectangle(pen, rect);

            // Read the words in the line of text
            string lineText = "";
            foreach(var word in line.Words)
            {
                lineText += word.Text + " ";
            }
            Console.WriteLine(lineText.Trim());
        }
    }

    // Save the image with the text locations highlighted
    String output_file = "ocr_results.jpg";
    image.Save(output_file);
    Console.WriteLine("Results saved in " + output_file);
}
```

**Python**

```Python
# Use OCR API to read text in image
with open(image_file, mode="rb") as image_data:
    ocr_results = cv_client.recognize_printed_text_in_stream(image_data)

# Prepare image for drawing
fig = plt.figure(figsize=(7, 7))
img = Image.open(image_file)
draw = ImageDraw.Draw(img)

# Process the text line by line
for region in ocr_results.regions:
    for line in region.lines:

        # Show the position of the line of text
        l,t,w,h = list(map(int, line.bounding_box.split(',')))
        draw.rectangle(((l,t), (l+w, t+h)), outline='magenta', width=5)

        # Read the words in the line of text
        line_text = ''
        for word in line.words:
            line_text += word.text + ' '
        print(line_text.rstrip())

# Save the image with the text locations highlighted
plt.axis('off')
plt.imshow(img)
outputfile = 'ocr_results.jpg'
fig.savefig(outputfile)
print('Results saved in', outputfile)
```

4. Examinez le code que vous avez ajouté à la fonction **GetTextOcr**. Il détecte les régions de texte imprimé à partir d’un fichier image et, pour chaque région, il extrait les lignes de texte et met en surbrillance leur position sur l’image. Il extrait ensuite les mots de chaque ligne et les imprime.
5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **clock-client** et entrez la commande suivante pour exécuter le programme :

**C#**

```
dotnet run
```

*La sortie C# peut afficher des avertissements sur les fonctions asynchrones utilisant maintenant l'opérateur **await**. Vous pouvez les ignorer.*

**Python**

```
python read-text.py
```

6. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui est le texte extrait de l’image.
7. Affichez le fichier **ocr_results.jpg** généré dans le même dossier que votre fichier de code pour afficher les lignes de texte annotées dans l’image.

## <a name="use-the-read-api"></a>Utiliser l’API Read

L’API **Read** utilise un modèle de reconnaissance de texte plus récent que l’API OCR et s’améliore pour les images plus volumineuses qui contiennent beaucoup de texte. Il prend également en charge l’extraction de texte à partir de fichiers *.pdf* et peut reconnaître le texte imprimé et le texte manuscrit dans plusieurs langues.

L’API **Read** utilise un modèle d’opération asynchrone, dans lequel une demande de démarrage de la reconnaissance de texte est envoyée ; et l’ID d’opération renvoyé par la requête peut ensuite être utilisé pour vérifier la progression et récupérer les résultats.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **2**. Ce code appelle la fonction **GetTextRead**, en passant le chemin d’accès à un fichier de document PDF.
2. Dans le dossier **read-text/images**, cliquez avec le bouton droit sur **Rome.pdf**, puis sélectionnez **Révéler dans l'Explorateur de fichiers**. Ensuite, dans Explorateur de fichiers, ouvrez le fichier PDF pour l’afficher.
3. De retour dans le fichier de code dans Visual Studio Code, recherchez la fonction **GetTextRead** et, sous le code existant qui imprime un message dans la console, ajoutez le code suivant :

**C#**

```C#
// Use Read API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // Get the async operation ID so we can check for the results
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // Wait for the asynchronous operation to complete
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // If the operation was successfuly, process the text line by line
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);
            }
        }
    }
}  
```

**Python**

```Python
# Use Read API to read text in image
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # Get the async operation ID so we can check for the results
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # Wait for the asynchronous operation to complete
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # If the operation was successfuly, process the text line by line
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
```
    
4. Examinez le code que vous avez ajouté à la fonction **GetTextRead**. Il envoie une demande d’opération de lecture, puis vérifie à plusieurs reprises l’état jusqu’à ce que l’opération soit terminée. S’il réussit, le code traite les résultats en itérant sur chaque page, puis sur chaque ligne.
5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **clock-client** et entrez la commande suivante pour exécuter le programme :

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. Lorsque vous y êtes invité, entrez **2** et observez la sortie, qui est le texte extrait du document.

## <a name="read-handwritten-text"></a>Lisez le texte manuscrit

En plus du texte imprimé, l’API **Read** peut extraire du texte manuscrit en anglais.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **3**. Ce code appelle la fonction **GetTextRead**, en passant le chemin vers un fichier image.
2. Dans le dossier **read-text/images**, ouvrez **Note.jpg** pour afficher l’image que votre code traitera.
3. Revenez au terminal intégré pour le dossier **read-text**, puis entrez la commande suivante pour exécuter le programme :

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. Lorsque vous y êtes invité, entrez **3** et observez la sortie, qui est le texte extrait du document.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur l’utilisation du service **Vision par ordinateur** pour lire du texte, consultez la [documentation Vision par ordinateur](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text).
