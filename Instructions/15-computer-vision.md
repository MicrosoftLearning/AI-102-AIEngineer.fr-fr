---
lab:
  title: Analyser des images avec Vision par ordinateur
  module: Module 8 - Getting Started with Computer Vision
ms.openlocfilehash: 5b7f15550844e4bc5efbdb3b8ee00d71760f18c9
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195546"
---
# <a name="analyze-images-with-computer-vision"></a>Analyser des images avec Vision par ordinateur

Vision par ordinateur est une fonctionnalité d’intelligence artificielle qui permet aux systèmes logiciels d’interpréter les entrées visuelles en analysant les images. Dans Microsoft Azure, le service cognitif **Vision par ordinateur** fournit des modèles prédéfinis pour les tâches courantes de vision par ordinateur, notamment l’analyse des images pour suggérer des légendes et des balises, la détection d’objets courants, de points de repère, de célébrités, de marques et la présence de contenu réservé aux adultes. Vous pouvez également utiliser le service Vision par ordinateur pour analyser la couleur et les formats de l’image et générer des images miniatures avec un « rognage intelligent ».

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous n’avez pas déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement où vous travaillez sur ce laboratoire, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

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
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Préparer l’utilisation du kit de développement logiciel (SDK) Vision par ordinateur

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) de vision par ordinateur pour analyser des images.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **15-computer-vision** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit de la souris sur le dossier **image-analysis** et ouvrez un terminal intégré. Installez ensuite le package SDK Vision par ordinateur en exécutant la commande appropriée pour votre préférence de langage :

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
    
3. Affichez le contenu du dossier **image-analysis**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification de votre ressource Cognitive Services. Enregistrez vos modifications.
4. Notez que le dossier **image-analysis** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : image-analysis.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Vision par ordinateur :

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
from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
from msrest.authentication import CognitiveServicesCredentials
```
    
## <a name="view-the-images-you-will-analyze"></a>Afficher les images que vous allez analyser

Dans cet exercice, vous allez utiliser le service Vision par ordinateur pour analyser plusieurs images.

1. Dans Visual Studio Code, développez le dossier **image-analysis** et le dossier **images** qu’il contient.
2. Sélectionnez tour à tour chacun des fichiers image pour les afficher dans Visual Studio Code.

## <a name="analyze-an-image-to-suggest-a-caption"></a>Analyser une image pour suggérer une légende

Vous êtes maintenant prêt à utiliser le Kit de développement logiciel (SDK) pour appeler le service Vision par ordinateur et analyser une image.

1. Dans le fichier de code de votre application cliente (**Program.cs** ou **image-analysis.py**), dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Vision par ordinateur**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet client Vision par ordinateur :

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

2. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code spécifie le chemin d’accès à un fichier image, puis transmet le chemin d’accès de l’image à deux autres fonctions (**AnalyzeImage** et **GetThumbnail**). Ces fonctions ne sont pas encore entièrement implémentées.

3. Dans la fonction **AnalyzeImage**, sous le commentaire **Spécifier les fonctionnalités à récupérer**, ajoutez le code suivant :

**C#**

```C#
// Specify features to be retrieved
List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
{
    VisualFeatureTypes.Description,
    VisualFeatureTypes.Tags,
    VisualFeatureTypes.Categories,
    VisualFeatureTypes.Brands,
    VisualFeatureTypes.Objects,
    VisualFeatureTypes.Adult
};
```

**Python**

```Python
# Specify features to be retrieved
features = [VisualFeatureTypes.description,
            VisualFeatureTypes.tags,
            VisualFeatureTypes.categories,
            VisualFeatureTypes.brands,
            VisualFeatureTypes.objects,
            VisualFeatureTypes.adult]
```
    
4. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir l’analyse de l’image**, ajoutez le code suivant (y compris les commentaires indiquant où vous ajouterez du code ultérieurement.) :

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // get image captions
    foreach (var caption in analysis.Description.Captions)
    {
        Console.WriteLine($"Description: {caption.Text} (confidence: {caption.Confidence.ToString("P")})");
    }

    // Get image tags


    // Get image categories


    // Get brands in the image


    // Get objects in the image


    // Get moderation ratings
    

}            
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

# Get image description
for caption in analysis.description.captions:
    print("Description: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get image categories 


# Get brands in the image


# Get objects in the image


# Get moderation ratings

```
    
5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **image-analysis** et entrez la commande suivante pour exécuter le programme avec l’argument **images/street.jpg** :

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. Observez la sortie, qui doit inclure une légende suggérée pour l’image **street.jpg**.
7. Réexécutez le programme, cette fois avec l’argument **images/building.jpg** pour voir la légende générée pour l’image **building.jpg**.
8. Répétez l’étape précédente pour générer une légende pour le fichier **images/person.jpg**.

## <a name="get-suggested-tags-for-an-image"></a>Obtenir des balises suggérées pour une image

Il peut parfois être utile d’identifier des *balises* pertinentes qui fournissent des indices sur le contenu d’une image.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir des balises d’image**, ajoutez le code suivant :

**C#**

```C
// Get image tags
if (analysis.Tags.Count > 0)
{
    Console.WriteLine("Tags:");
    foreach (var tag in analysis.Tags)
    {
        Console.WriteLine($" -{tag.Name} (confidence: {tag.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image tags
if (len(analysis.tags) > 0):
    print("Tags: ")
    for tag in analysis.tags:
        print(" -'{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers d’images dans le dossier **images**. Observez que, en plus de la légende de l’image, une liste de balises suggérées s’affiche.

## <a name="get-image-categories"></a>Obtenir des catégories d’image

Le service Vision par ordinateur peut suggérer des *catégories* pour les images et peut, dans chaque catégorie, identifier des points de repère ou des célébrités bien connus.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir des catégories d’image (comprenant les célébrités et les points de repère)** , ajoutez le code suivant :

**C#**

```C
// Get image categories (including celebrities and landmarks)
List<LandmarksModel> landmarks = new List<LandmarksModel> {};
List<CelebritiesModel> celebrities = new List<CelebritiesModel> {};
Console.WriteLine("Categories:");
foreach (var category in analysis.Categories)
{
    // Print the category
    Console.WriteLine($" -{category.Name} (confidence: {category.Score.ToString("P")})");

    // Get landmarks in this category
    if (category.Detail?.Landmarks != null)
    {
        foreach (LandmarksModel landmark in category.Detail.Landmarks)
        {
            if (!landmarks.Any(item => item.Name == landmark.Name))
            {
                landmarks.Add(landmark);
            }
        }
    }

    // Get celebrities in this category
    if (category.Detail?.Celebrities != null)
    {
        foreach (CelebritiesModel celebrity in category.Detail.Celebrities)
        {
            if (!celebrities.Any(item => item.Name == celebrity.Name))
            {
                celebrities.Add(celebrity);
            }
        }
    }
}

// If there were landmarks, list them
if (landmarks.Count > 0)
{
    Console.WriteLine("Landmarks:");
    foreach(LandmarksModel landmark in landmarks)
    {
        Console.WriteLine($" -{landmark.Name} (confidence: {landmark.Confidence.ToString("P")})");
    }
}

// If there were celebrities, list them
if (celebrities.Count > 0)
{
    Console.WriteLine("Celebrities:");
    foreach(CelebritiesModel celebrity in celebrities)
    {
        Console.WriteLine($" -{celebrity.Name} (confidence: {celebrity.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image categories (including celebrities and landmarks)
if (len(analysis.categories) > 0):
    print("Categories:")
    landmarks = []
    celebrities = []
    for category in analysis.categories:
        # Print the category
        print(" -'{}' (confidence: {:.2f}%)".format(category.name, category.score * 100))
        if category.detail:
            # Get landmarks in this category
            if category.detail.landmarks:
                for landmark in category.detail.landmarks:
                    if landmark not in landmarks:
                        landmarks.append(landmark)

            # Get celebrities in this category
            if category.detail.celebrities:
                for celebrity in category.detail.celebrities:
                    if celebrity not in celebrities:
                        celebrities.append(celebrity)

    # If there were landmarks, list them
    if len(landmarks) > 0:
        print("Landmarks:")
        for landmark in landmarks:
            print(" -'{}' (confidence: {:.2f}%)".format(landmark.name, landmark.confidence * 100))

    # If there were celebrities, list them
    if len(celebrities) > 0:
        print("Celebrities:")
        for celebrity in celebrities:
            print(" -'{}' (confidence: {:.2f}%)".format(celebrity.name, celebrity.confidence * 100))

```
    
2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**. Observez que, en plus de la légende et des balises d’image, une liste de catégories suggérées s’affiche, avec tous les points de repère ou célébrités reconnus (notamment dans les images **building.jpg** et **person.jpg**).

## <a name="get-brands-in-an-image"></a>Obtenir des marques dans une image

Certaines marques sont visuellement reconnaissables par leur logo, même si le nom de la marque n’est pas affiché. Le service Vision par ordinateur est entraîné pour identifier des milliers de marques connues.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir des marques dans l’image**, ajoutez le code suivant :

**C#**

```C
// Get brands in the image
if (analysis.Brands.Count > 0)
{
    Console.WriteLine("Brands:");
    foreach (var brand in analysis.Brands)
    {
        Console.WriteLine($" -{brand.Name} (confidence: {brand.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get brands in the image
if (len(analysis.brands) > 0):
    print("Brands: ")
    for brand in analysis.brands:
        print(" -'{}' (confidence: {:.2f}%)".format(brand.name, brand.confidence * 100))
```
    
2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**, en observant toutes les marques identifiées (en particulier dans l’image **person.jpg**).

## <a name="detect-and-locate-objects-in-an-image"></a>Détecter et localiser des objets dans une image

*La détection d’objets* est une forme spécifique de vision par ordinateur dans laquelle des objets individuels au sein d’une image sont identifiés et leur emplacement indiqué par un cadre englobant.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir des objets dans l’image**, ajoutez le code suivant :

**C#**

```C
// Get objects in the image
if (analysis.Objects.Count > 0)
{
    Console.WriteLine("Objects in image:");

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.Black);

    foreach (var detectedObject in analysis.Objects)
    {
        // Print object name
        Console.WriteLine($" -{detectedObject.ObjectProperty} (confidence: {detectedObject.Confidence.ToString("P")})");

        // Draw object bounding box
        var r = detectedObject.Rectangle;
        Rectangle rect = new Rectangle(r.X, r.Y, r.W, r.H);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.ObjectProperty,font,brush,r.X, r.Y);

    }
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file);   
}
```

**Python**

```Python
# Get objects in the image
if len(analysis.objects) > 0:
    print("Objects in image:")

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 8))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    color = 'cyan'
    for detected_object in analysis.objects:
        # Print object name
        print(" -{} (confidence: {:.2f}%)".format(detected_object.object_property, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.rectangle
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.object_property,(r.x, r.y), backgroundcolor=color)
    # Save annotated image
    plt.imshow(image)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```
    
2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**, en observant tous les objets détectés. Après chaque exécution, affichez le fichier **objects.jpg** généré dans le même dossier que votre fichier de code pour voir les objets annotés.

## <a name="get-moderation-ratings-for-an-image"></a>Obtenir des évaluations de modération pour une image

Certaines images peuvent ne pas être adaptées à tous les publics, et vous devrez peut-être appliquer une modération pour identifier les images réservées aux adultes ou de nature violente.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir des évaluations de modération**, ajoutez le code suivant :

**C#**

```C
// Get moderation ratings
string ratings = $"Ratings:\n -Adult: {analysis.Adult.IsAdultContent}\n -Racy: {analysis.Adult.IsRacyContent}\n -Gore: {analysis.Adult.IsGoryContent}";
Console.WriteLine(ratings);
```

**Python**

```Python
# Get moderation ratings
ratings = 'Ratings:\n -Adult: {}\n -Racy: {}\n -Gore: {}'.format(analysis.adult.is_adult_content,
                                                                    analysis.adult.is_racy_content,
                                                                    analysis.adult.is_gory_content)
print(ratings)
```
    
2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**, en observant les évaluations de chacune des images.

> **Remarque** : Dans les tâches précédentes, vous avez utilisé une méthode unique pour analyser l’image, puis ajouté de façon incrémentielle du code pour analyser et afficher les résultats. Le Kit de développement logiciel (SDK) fournit également des méthodes individuelles pour suggérer des légendes, identifier des balises, détecter des objets, etc., ce qui signifie que vous pouvez utiliser la méthode la plus appropriée pour retourner uniquement les informations dont vous avez besoin, réduisant ainsi la taille de la charge utile de données qui doit être retournée. Consultez la [documentation du SDK .NET](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet) ou la [documentation du SDK Python](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/computervision?view=azure-python) pour plus d’informations.

## <a name="generate-a-thumbnail-image"></a>Générer une miniature d’image

Dans certains cas, vous devrez peut-être créer une version plus petite d’une image nommée *miniature* en la rognant pour inclure le sujet visuel principal dans de nouvelles dimensions d’image.

1. Dans votre fichier de code, recherchez la fonction **GetThumbnail** ; et sous le commentaire **Générer une miniature**, ajoutez le code suivant :

**C#**

```C
// Generate a thumbnail
using (var imageData = File.OpenRead(imageFile))
{
    // Get thumbnail data
    var thumbnailStream = await cvClient.GenerateThumbnailInStreamAsync(100, 100,imageData, true);

    // Save thumbnail image
    string thumbnailFileName = "thumbnail.png";
    using (Stream thumbnailFile = File.Create(thumbnailFileName))
    {
        thumbnailStream.CopyTo(thumbnailFile);
    }

    Console.WriteLine($"Thumbnail saved in {thumbnailFileName}");
}
```

**Python**

```Python
# Generate a thumbnail
with open(image_file, mode="rb") as image_data:
    # Get thumbnail data
    thumbnail_stream = cv_client.generate_thumbnail_in_stream(100, 100, image_data, True)

# Save thumbnail image
thumbnail_file_name = 'thumbnail.png'
with open(thumbnail_file_name, "wb") as thumbnail_file:
    for chunk in thumbnail_stream:
        thumbnail_file.write(chunk)

print('Thumbnail saved in.', thumbnail_file_name)
```
    
2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**. Ouvrez le fichier **thumbnail.jpg** généré dans le même dossier que votre fichier de code pour chaque image.

## <a name="more-information"></a>Plus d’informations

Dans cet exercice, vous avez exploré certaines des fonctionnalités d’analyse et de manipulation d’images du service Vision par ordinateur. Le service inclut également des fonctionnalités de lecture de texte, de détection des visages et d’autres tâches de vision par ordinateur.

Pour plus d’informations sur l’utilisation du service **Vision par ordinateur**, consultez la [documentation Vision par ordinateur](https://docs.microsoft.com/azure/cognitive-services/computer-vision/).
