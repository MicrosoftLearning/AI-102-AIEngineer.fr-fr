---
lab:
  title: Détecter, analyser et reconnaître des visages
  module: Module 10 - Detecting, Analyzing, and Recognizing Faces
ms.openlocfilehash: b9565f41eb67b916278508c729860a3471a9e0bd
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195586"
---
# <a name="detect-analyze-and-recognize-faces"></a>Détecter, analyser et reconnaître des visages

La possibilité de détecter, d’analyser et de reconnaître les visages humains est une fonctionnalité d’IA fondamentale. Dans cet exercice, vous allez explorer deux services Azure Cognitive Services que vous pouvez utiliser pour travailler avec les visages dans des images : le service **Vision par ordinateur** et le service **Visage**.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous ne l’avez pas déjà fait, vous devez cloner le référentiel de code pour ce cours :

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

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) Vision par ordinateur pour analyser les visages dans une image.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **19-face** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier **Vision par ordinateur** et ouvrez un terminal intégré. Installez ensuite le package SDK Vision par ordinateur en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. Affichez le contenu du dossier **computer-vision**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

4. Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification de votre ressource Cognitive Services. Enregistrez vos modifications.

5. Notez que le dossier **computer-vision** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : detect-faces.py

6. Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Vision par ordinateur :

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

## <a name="view-the-image-you-will-analyze"></a>Afficher l’image que vous allez analyser

Dans cet exercice, vous allez utiliser le service Vision par ordinateur pour analyser une image avec des personnes.

1. Dans Visual Studio Code, développez le dossier **computer-vision** et le dossier **images** qu’il contient.
2. Sélectionnez l’image **people.jpg** pour l’afficher.

## <a name="detect-faces-in-an-image"></a>Détecter des visages dans une image

Vous êtes maintenant prêt à utiliser le Kit de développement logiciel (SDK) pour appeler le service Vision par ordinateur et détecter les visages dans une image.

1. Dans le fichier de code de votre application cliente (**Program.cs** ou **detect-faces.py**), dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Vision par ordinateur**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet client Vision par ordinateur :

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

2. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code spécifie le chemin d’accès à un fichier image, puis transmet le chemin d’accès de l’image à une fonction nommée **AnalyzeFaces**. Cette fonction n’est pas encore entièrement implémentée.

3. Dans la fonction **AnalyzeFaces**, sous le commentaire **Spécifier les fonctionnalités à récupérer (visages)** , ajoutez le code suivant :

    **C#**

    ```C#
    // Specify features to be retrieved (faces)
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (faces)
    features = [VisualFeatureTypes.faces]
    ```

4. Dans la fonction **AnalyzeFaces**, sous le commentaire **Obtenir l’analyse de l’image**, ajoutez le code suivant :

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // Get faces
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // Draw and annotate each face
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person aged approximately {face.Age}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # Get faces
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person aged approximately {}'.format(face.age)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **computer-vision** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. Observez la sortie, qui doit indiquer le nombre de visages détectés.
7. Affichez le fichier **detected_faces.jpg** généré dans le même dossier que votre fichier de code pour voir les visages annotés. Dans ce cas, votre code a utilisé les attributs du visage pour estimer l’âge de chaque personne dans l’image et les coordonnées du cadre englobant pour dessiner un rectangle autour de chaque visage.

## <a name="prepare-to-use-the-face-sdk"></a>Préparer l’utilisation du Kit de développement logiciel (SDK) Visage

Bien que le service **Vision par ordinateur** offre une détection des visages de base (ainsi que de nombreuses autres fonctionnalités d’analyse d’image), le service **Visage** apporte des fonctionnalités plus complètes pour l’analyse et la reconnaissance faciales.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **19-face** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier **face-api** et ouvrez un terminal intégré. Installez ensuite le package SDK Visage en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. Affichez le contenu du dossier **face-api**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

4. Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification de votre ressource Cognitive Services. Enregistrez vos modifications.

5. Notez que le dossier **face-api** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : analyze-faces.py

6. Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Vision par ordinateur :

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. Dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Visage**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet **FaceClient** :

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code affiche un menu qui vous permet d’appeler des fonctions dans votre code pour explorer les fonctionnalités du service Visage. Vous allez implémenter ces fonctions dans le reste de cet exercice.

## <a name="detect-and-analyze-faces"></a>Détecter et analyser les visages

L’une des fonctionnalités les plus fondamentales du service Visage consiste à détecter les visages dans une image et à déterminer leurs attributs, tels que l’âge, les expressions émotionnelles, la couleur des cheveux, la présence de lunettes, etc.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **1**. Ce code appelle la fonction **DetectFaces**, en passant le chemin vers un fichier image.
2. Recherchez la fonction **DetectFaces** dans le fichier de code et, sous le commentaire **Spécifier les fonctionnalités faciales à récupérer**, ajoutez le code suivant :

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Age,
        FaceAttributeType.Emotion,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.age,
                FaceAttributeType.emotion,
                FaceAttributeType.glasses]
    ```

3. Dans la fonction **DetectFaces**, sous le code que vous venez d’ajouter, recherchez le commentaire **Obtenir des visages** et ajoutez le code suivant :

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            // Get face properties
            Console.WriteLine($"\nFace ID: {face.FaceId}");
            Console.WriteLine($" - Age: {face.FaceAttributes.Age}");
            Console.WriteLine($" - Emotions:");
            foreach (var emotion in face.FaceAttributes.Emotion.ToRankedList())
            {
                Console.WriteLine($"   - {emotion}");
            }

            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            print('\nFace ID: {}'.format(face.face_id))
            detected_attributes = face.face_attributes.as_dict()
            age = 'age unknown' if 'age' not in detected_attributes.keys() else int(detected_attributes['age'])
            print(' - Age: {}'.format(age))

            if 'emotion' in detected_attributes:
                print(' - Emotions:')
                for emotion_name in detected_attributes['emotion']:
                    print('   - {}: {}'.format(emotion_name, detected_attributes['emotion'][emotion_name]))
            
            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. Examinez le code que vous avez ajouté à la fonction **DetectFaces**. Il analyse un fichier image et détecte les visages qu’il contient, y compris les attributs tels que l’âge, les émotions et la présence de lunettes. Les détails de chaque visage sont affichés, notamment un identificateur de visage unique affecté à chaque visage; et l’emplacement des visages est indiqué sur l’image à l’aide de rectangles englobants.
5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **face-api** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    *La sortie C# peut afficher des avertissements concernant les fonctions asynchrones utilisant maintenant l’opérateur **await**. Vous pouvez les ignorer.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui doit inclure l’ID et les attributs de chaque visage détecté.
7. Affichez le fichier **detected_faces.jpg** généré dans le même dossier que votre fichier de code pour voir les visages annotés.

## <a name="compare-faces"></a>Comparer des visages

Une tâche courante consiste à comparer les visages et à trouver des visages similaires. Dans ce scénario, vous n’avez pas besoin d’*identifier* la personne dans les images. Déterminez simplement si plusieurs images montrent la *même* personne (ou au moins quelqu’un qui lui ressemble). 

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **2**. Ce code appelle la fonction **CompareFaces**, en passant le chemin d’accès à deux fichiers image (**person1.jpg** et **people.jpg**).
2. Recherchez la fonction **CompareFaces** dans le fichier de code et, sous le code existant qui imprime un message dans la console, ajoutez le code suivant :

**C#**

```C
// Determine if the face in image 1 is also in image 2
DetectedFace image_i_face;
using (var image1Data = File.OpenRead(image1))
{    
    // Get the first face in image 1
    var image1_faces = await faceClient.Face.DetectWithStreamAsync(image1Data);
    if (image1_faces.Count > 0)
    {
        image_i_face = image1_faces[0];
        Image img1 = Image.FromFile(image1);
        Graphics graphics = Graphics.FromImage(img1);
        Pen pen = new Pen(Color.LightGreen, 3);
        var r = image_i_face.FaceRectangle;
        Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        String output_file = "face_to_match.jpg";
        img1.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file); 

        //Get all the faces in image 2
        using (var image2Data = File.OpenRead(image2))
        {    
            var image2Faces = await faceClient.Face.DetectWithStreamAsync(image2Data);

            // Get faces
            if (image2Faces.Count > 0)
            {

                var image2FaceIds = image2Faces.Select(f => f.FaceId).ToList<Guid?>();
                var similarFaces = await faceClient.Face.FindSimilarAsync((Guid)image_i_face.FaceId,faceIds:image2FaceIds);
                var similarFaceIds = similarFaces.Select(f => f.FaceId).ToList<Guid?>();

                // Prepare image for drawing
                Image img2 = Image.FromFile(image2);
                Graphics graphics2 = Graphics.FromImage(img2);
                Pen pen2 = new Pen(Color.LightGreen, 3);
                Font font2 = new Font("Arial", 4);
                SolidBrush brush2 = new SolidBrush(Color.Black);

                // Draw and annotate each face
                foreach (var face in image2Faces)
                {
                    if (similarFaceIds.Contains(face.FaceId))
                    {
                        // Draw and annotate face
                        var r2 = face.FaceRectangle;
                        Rectangle rect2 = new Rectangle(r2.Left, r2.Top, r2.Width, r2.Height);
                        graphics2.DrawRectangle(pen2, rect2);
                        string annotation = "Match!";
                        graphics2.DrawString(annotation,font2,brush2,r2.Left, r2.Top);
                    }
                }

                // Save annotated image
                String output_file2 = "matched_faces.jpg";
                img2.Save(output_file2);
                Console.WriteLine(" Results saved in " + output_file2);   
            }
        }

    }
}
```

**Python**

```Python
# Determine if the face in image 1 is also in image 2
with open(image_1, mode="rb") as image_data:
    # Get the first face in image 1
    image_1_faces = face_client.face.detect_with_stream(image=image_data)
    image_1_face = image_1_faces[0]

    # Highlight the face in the image
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_1)
    draw = ImageDraw.Draw(image)
    color = 'lightgreen'
    r = image_1_face.face_rectangle
    bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
    draw = ImageDraw.Draw(image)
    draw.rectangle(bounding_box, outline=color, width=5)
    plt.imshow(image)
    outputfile = 'face_to_match.jpg'
    fig.savefig(outputfile)

# Get all the faces in image 2
with open(image_2, mode="rb") as image_data:
    image_2_faces = face_client.face.detect_with_stream(image=image_data)
    image_2_face_ids = list(map(lambda face: face.face_id, image_2_faces))

    # Find faces in image 2 that are similar to the one in image 1
    similar_faces = face_client.face.find_similar(face_id=image_1_face.face_id, face_ids=image_2_face_ids)
    similar_face_ids = list(map(lambda face: face.face_id, similar_faces))

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_2)
    draw = ImageDraw.Draw(image)

    # Draw and annotate matching faces
    for face in image_2_faces:
        if face.face_id in similar_face_ids:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline='lightgreen', width=10)
            plt.annotate('Match!',(r.left, r.top + r.height + 15), backgroundcolor='white')

    # Save annotated image
    plt.imshow(image)
    outputfile = 'matched_faces.jpg'
    fig.savefig(outputfile)
```

3. Examinez le code que vous avez ajouté à la fonction **CompareFaces**. Il trouve le premier visage dans l’image 1 et l’annote dans un nouveau fichier image nommé **face_to_match.jpg**. Ensuite, il trouve tous les visages dans l’image 2 et utilise leurs ID de visage pour trouver ceux qui sont similaires à l’image 1. Les visages similaires sont annotés et enregistrés dans une nouvelle image nommée **matched_faces.jpg**.
4. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **face-api** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    *La sortie C# peut afficher des avertissements concernant les fonctions asynchrones utilisant maintenant l’opérateur **await**. Vous pouvez les ignorer.*

    **Python**

    ```
    python analyze-faces.py
    ```
    
5. Lorsque vous y êtes invité, entrez **2** et observez la sortie. Affichez ensuite les fichiers **face_to_match.jpg** et **matched_faces.jpg** générés dans le même dossier que votre fichier de code pour voir les visages correspondants.
6. Modifiez le code dans la fonction **Main** de l’option de menu **2** pour comparer **person2.jpg** à **people.jpg**, puis réexécutez le programme et affichez les résultats.

## <a name="train-a-facial-recognition-model"></a>Entraîner un modèle de reconnaissance faciale

Il peut exister des scénarios dans lesquels vous devez conserver un modèle de personnes spécifiques dont les visages peuvent être reconnus par une application IA. Par exemple, pour stimuler un système de sécurité biométrique qui utilise la reconnaissance faciale pour vérifier l’identité des employés dans un bâtiment sécurisé.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **3**. Ce code appelle la fonction **TrainModel**, en passant le nom et l’ID d’un nouveau **PersonGroup** qui sera inscrit dans votre ressource Cognitive Services et une liste des noms des employés.
2. Dans le dossier **face-api/images**, observez qu’il existe des dossiers portant les mêmes noms que les employés. Chaque dossier contient plusieurs images de l’employé nommé.
3. Recherchez la fonction **TrainModel** dans le fichier de code et, sous le code existant qui imprime un message dans la console, ajoutez le code suivant :

**C#**

```C
// Delete group if it already exists
var groups = await faceClient.PersonGroup.ListAsync();
foreach(var group in groups)
{
    if (group.PersonGroupId == groupId)
    {
        await faceClient.PersonGroup.DeleteAsync(groupId);
    }
}

// Create the group
await faceClient.PersonGroup.CreateAsync(groupId, groupName);
Console.WriteLine("Group created!");

// Add each person to the group
Console.Write("Adding people to the group...");
foreach(var personName in imageFolders)
{
    // Add the person
    var person = await faceClient.PersonGroupPerson.CreateAsync(groupId, personName);

    // Add multiple photo's of the person
    string[] images = Directory.GetFiles("images/" + personName);
    foreach(var image in images)
    {
        using (var imageData = File.OpenRead(image))
        { 
            await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(groupId, person.PersonId, imageData);
        }
    }

}

    // Train the model
Console.WriteLine("Training model...");
await faceClient.PersonGroup.TrainAsync(groupId);

// Get the list of people in the group
Console.WriteLine("Facial recognition model trained with the following people:");
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    Console.WriteLine($"-{person.Name}");
}
```

**Python**

```Python
# Delete group if it already exists
groups = face_client.person_group.list()
for group in groups:
    if group.person_group_id == group_id:
        face_client.person_group.delete(group_id)

# Create the group
face_client.person_group.create(group_id, group_name)
print ('Group created!')

# Add each person to the group
print('Adding people to the group...')
for person_name in image_folders:
    # Add the person
    person = face_client.person_group_person.create(group_id, person_name)

    # Add multiple photo's of the person
    folder = os.path.join('images', person_name)
    person_pics = os.listdir(folder)
    for pic in person_pics:
        img_path = os.path.join(folder, pic)
        img_stream = open(img_path, "rb")
        face_client.person_group_person.add_face_from_stream(group_id, person.person_id, img_stream)

# Train the model
print('Training model...')
face_client.person_group.train(group_id)

# Get the list of people in the group
print('Facial recognition model trained with the following people:')
people = face_client.person_group_person.list(group_id)
for person in people:
    print('-', person.name)
```

4. Examinez le code que vous avez ajouté à la fonction **TrainModel**. Il effectue les tâches suivantes :
    - Obtient une liste des groupes **PersonGroup** inscrits dans le service et supprime celui spécifié s’il en existe un.
    - Crée un groupe avec l’ID et le nom spécifiés.
    - Ajoute une personne au groupe pour chaque nom spécifié et ajoute les multiples images de chaque personne.
    - Entraîne un modèle de reconnaissance faciale basé sur les personnes nommées dans le groupe et leurs images de visage.
    - Récupère une liste des personnes nommées dans le groupe et les affiche.
5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **face-api** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    *La sortie C# peut afficher des avertissements concernant les fonctions asynchrones utilisant maintenant l’opérateur **await**. Vous pouvez les ignorer.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Lorsque vous y êtes invité, entrez **3**, observez la sortie, et notez que le **PersonGroup** créé comprend deux personnes.

## <a name="recognize-faces"></a>Reconnaître des visages

Maintenant que vous avez défini un **PeopleGroup** et entraîné un modèle de reconnaissance faciale, vous pouvez l’utiliser pour reconnaître les individus nommés dans une image.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **4**. Ce code appelle la fonction **RecognizeFaces**, en passant le chemin d’accès à un fichier image (**people.jpg**) et l’ID du **PeopleGroup** à utiliser pour l’identification des visages.
2. Recherchez la fonction **RecognizeFaces** dans le fichier de code et, sous le code existant qui imprime un message dans la console, ajoutez le code suivant :

**C#**

```C
// Detect faces in the image
using (var imageData = File.OpenRead(imageFile))
{    
    var detectedFaces = await faceClient.Face.DetectWithStreamAsync(imageData);

    // Get faces
    if (detectedFaces.Count > 0)
    {
        
        // Get a list of face IDs
        var faceIds = detectedFaces.Select(f => f.FaceId).ToList<Guid?>();

        // Identify the faces in the people group
        var recognizedFaces = await faceClient.Face.IdentifyAsync(faceIds, groupId);

        // Get names for recognized faces
        var faceNames = new Dictionary<Guid?, string>();
        if (recognizedFaces.Count> 0)
        {
            foreach(var face in recognizedFaces)
            {
                var person = await faceClient.PersonGroupPerson.GetAsync(groupId, face.Candidates[0].PersonId);
                Console.WriteLine($"-{person.Name}");
                faceNames.Add(face.FaceId, person.Name);

            }
        }

        
        // Annotate faces in image
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen penYes = new Pen(Color.LightGreen, 3);
        Pen penNo = new Pen(Color.Magenta, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Cyan);
        foreach (var face in detectedFaces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            if (faceNames.ContainsKey(face.FaceId))
            {
                // If the face is recognized, annotate in green with the name
                graphics.DrawRectangle(penYes, rect);
                string personName = faceNames[face.FaceId];
                graphics.DrawString(personName,font,brush,r.Left, r.Top);
            }
            else
            {
                // Otherwise, just annotate the unrecognized face in magenta
                graphics.DrawRectangle(penNo, rect);
            }
        }

        // Save annotated image
        String output_file = "recognized_faces.jpg";
        image.Save(output_file);
        Console.WriteLine("Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Detect faces in the image
with open(image_file, mode="rb") as image_data:

    # Get faces
    detected_faces = face_client.face.detect_with_stream(image=image_data)

    # Get a list of face IDs
    face_ids = list(map(lambda face: face.face_id, detected_faces))

    # Identify the faces in the people group
    recognized_faces = face_client.face.identify(face_ids, group_id)

    # Get names for recognized faces
    face_names = {}
    if len(recognized_faces) > 0:
        print(len(recognized_faces), 'faces recognized.')
        for face in recognized_faces:
            person_name = face_client.person_group_person.get(group_id, face.candidates[0].person_id).name
            print('-', person_name)
            face_names[face.face_id] = person_name

    # Annotate faces in image
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    for face in detected_faces:
        r = face.face_rectangle
        bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
        draw = ImageDraw.Draw(image)
        if face.face_id in face_names:
            # If the face is recognized, annotate in green with the name
            draw.rectangle(bounding_box, outline='lightgreen', width=3)
            plt.annotate(face_names[face.face_id],
                        (r.left, r.top + r.height + 15), backgroundcolor='white')
        else:
            # Otherwise, just annotate the unrecognized face in magenta
            draw.rectangle(bounding_box, outline='magenta', width=3)

    # Save annotated image
    plt.imshow(image)
    outputfile = 'recognized_faces.jpg'
    fig.savefig(outputfile)

    print('\nResults saved in', outputfile)
```
    
3. Examinez le code que vous avez ajouté à la fonction **RecognizeFaces**. Il trouve les visages dans l’image et crée une liste de leurs ID. Ensuite, il utilise le groupe de personnes pour essayer d’identifier les visages dans la liste des ID de visage. Les visages reconnus sont annotés avec le nom de la personne identifiée et les résultats sont enregistrés dans **recognized_faces.jpg**.
4. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **face-api** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    *La sortie C# peut afficher des avertissements concernant les fonctions asynchrones utilisant maintenant l’opérateur **await**. Vous pouvez les ignorer.*

    **Python**

    ```
    python analyze-faces.py
    ```

5. Lorsque vous y êtes invité, entrez **4** et observez la sortie. Affichez ensuite le fichier **recognized_faces.jpg** généré dans le même dossier que votre fichier de code pour afficher les personnes identifiées.
6. Modifiez le code dans la fonction **Main** de l’option de menu **4** pour reconnaître les visages dans **people2.jpg**, puis réexécutez le programme et affichez les résultats. Les mêmes personnes devraient être reconnues.

## <a name="verify-a-face"></a>Vérifier un visage

La reconnaissance faciale est souvent utilisée pour la vérification d’identité. Avec le service Visage, vous pouvez vérifier un visage dans une image en le comparant à un autre visage ou à partir d’une personne inscrite dans un **PersonGroup**.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **5**. Ce code appelle la fonction **VerifyFace**, en passant le chemin d’accès à un fichier image (**person1.jpg**), un nom et l’ID du **PeopleGroup** à utiliser pour l’identification des visages.
2. Recherchez la fonction **VerifyFace** dans le fichier de code et, sous le commentaire **Obtenir l’ID de la personne dans le groupe de personnes** (au-dessus du code qui imprime le résultat), ajoutez le code suivant :

**C#**

```C
// Get the ID of the person from the people group
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    if (person.Name == personName)
    {
        Guid personId = person.PersonId;

        // Get the first face in the image
        using (var imageData = File.OpenRead(personImage))
        {    
            var faces = await faceClient.Face.DetectWithStreamAsync(imageData);
            if (faces.Count > 0)
            {
                Guid faceId = (Guid)faces[0].FaceId;

                //We have a face and an ID. Do they match?
                var verification = await faceClient.Face.VerifyFaceToPersonAsync(faceId, personId, groupId);
                if (verification.IsIdentical)
                {
                    result = "Verified";
                }
            }
        }
    }
}
```

**Python**

```Python
# Get the ID of the person from the people group
people = face_client.person_group_person.list(group_id)
for person in people:
    if person.name == person_name:
        person_id = person.person_id

        # Get the first face in the image
        with open(person_image, mode="rb") as image_data:
            faces = face_client.face.detect_with_stream(image=image_data)
            if len(faces) > 0:
                face_id = faces[0].face_id

                # We have a face and an ID. Do they match?
                verification = face_client.face.verify_face_to_person(face_id, person_id, group_id)
                if verification.is_identical:
                    result = 'Verified'
```

3. Examinez le code que vous avez ajouté à la fonction **VerifyFace**. Il recherche l’ID de la personne dans le groupe avec le nom spécifié. Si la personne existe, le code obtient l’ID de visage du premier visage dans l’image. Enfin, s’il existe un visage dans l’image, le code le vérifie en le comparant à l’ID de la personne spécifiée.
4. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **face-api** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python analyze-faces.py
    ```

5. Lorsque vous y êtes invité, entrez **5** et observez le résultat.
6. Modifiez le code dans la fonction **Main** de l’option de menu **5** pour expérimenter différentes combinaisons de noms et d’images **person1.jpg** et **person2.jpg**.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur l’utilisation du service **Vision par ordinateur** pour la détection des visages, consultez la documentation [Vision par ordinateur](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Pour en savoir plus sur le service **Visage**, consultez la [documentation Visage](https://docs.microsoft.com/azure/cognitive-services/face/).
