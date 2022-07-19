---
lab:
  title: Extraire les données de formulaires
  module: Module 11 - Reading Text in Images and Documents
ms.openlocfilehash: 540fdc49b9efcf335d43cdd7a6db405c255cd058
ms.sourcegitcommit: de1f38bbe53ec209b42cd89516813773e2f3479b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/17/2022
ms.locfileid: "145195549"
---
# <a name="extract-data-from-forms"></a>Extraire les données de formulaires 

Supposons qu'une entreprise demande actuellement à ses employés d'acheter manuellement des feuilles de commande et de saisir les données dans une base de données. Ils souhaitent utiliser des services IA pour améliorer le processus d’entrée de données. Vous décidez de créer un modèle Machine Learning qui lit le formulaire et produit des données structurées qui peuvent être utilisées pour mettre à jour automatiquement une base de données.

**Form Recognizer** est un service cognitif qui permet aux utilisateurs de créer des logiciels de traitement de données automatisé. Ce logiciel peut extraire du texte, des paires clé/valeur et des tables à partir de documents de formulaire à l’aide de la reconnaissance optique de caractères (OCR). Form Recognizer possède des modèles prédéfinis pour reconnaître les factures, les reçus et les cartes de visite. Le service offre également la possibilité d’entraîner des modèles personnalisés. Dans cet exercice, nous allons nous concentrer sur la création de modèles personnalisés.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour cette formation

Si vous ne l’avez pas déjà fait, vous devez cloner le référentiel de code pour ce cours :

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-a-form-recognizer-resource"></a>Créer une ressource Form Recognizer

Pour utiliser le service Form Recognizer, vous devez créer une ressource Form Recognizer ou Cognitive Services dans votre abonnement Azure. Vous utilisez le portail Azure pour créer un ressource.

1.  Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.

2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Form Recognizer* et créez une ressource **Form Recognizer** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *entrez un nom unique.*
    - **Niveau tarifaire** : F0

    > **Remarque** : Si vous avez déjà un service de reconnaissance de formulaire F0 dans votre abonnement, sélectionnez **S0** pour celui-ci.

3. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin du **point de terminaison** et de l’une des **clés** de cette page pour gérer l’accès à partir de votre code ultérieurement. 

## <a name="gather-documents-for-training"></a>Collecter des documents pour l’entraînement

![Image d’une facture.](../21-custom-form/sample-forms/Form_1.jpg)  

Vous allez utiliser les exemples de formulaires du dossier **21-custom-form/sample-forms** dans ce référentiel, qui contiennent tous les fichiers que vous devez entraîner et tester un modèle.

1. Dans Visual Studio Code, dans le dossier **21-custom-form**, développez le dossier **sample-forms**. Notez que les fichiers se terminent par **.json** et **.jpg** dans le dossier.

    Vous utiliserez les fichiers **.jpg** pour entraîner votre modèle.  

    Les fichiers **.json** ont été générés pour vous et contiennent des informations d’étiquette. Les fichiers seront chargés dans votre conteneur de stockage d’objets blob en même temps que les formulaires. 

2. Retournez au portail Azure sur la page [https://portal.azure.com](https://portal.azure.com).

3. Affichez le **Groupe de ressources** dans lequel vous avez créé la ressource Form Recognizer précédemment.

4. Sur la page **Vue d’ensemble** de votre groupe de ressources, notez l’**ID d’abonnement** et l’**Emplacement**. Vous aurez besoin de ces valeurs, ainsi que du nom de votre **groupe de ressources** dans les étapes suivantes.

![Un exemple de la page de groupe de ressources.](./images/resource_group_variables.png)

5. Dans Visual Studio Code, dans le volet Explorateur, cliquez avec le bouton droit de la souris sur le dossier **21-custom-form**, puis sélectionnez **Ouvrir dans le terminal intégré**.

6. Dans le volet Terminal, entrez la commande suivante pour établir une connexion authentifiée à votre abonnement Azure.
    
```
az login --output none
```

7. Lorsque vous y êtes invité, connectez-vous à votre abonnement Azure. Revenez ensuite à Visual Studio Code et attendez la fin du processus de connexion.

8. Exécutez la commande suivante pour lister vos emplacements Azure.

```
az account list-locations -o table
```

9. Dans la sortie, recherchez la valeur **Nom** qui correspond à l’emplacement de votre groupe de ressources (par exemple, pour *East US*, le nom correspondant est *eastus*).

    > **Important** : Enregistrez la valeur **Nom** et utilisez-la à l’étape 12.

10. Dans le volet Explorateur, dans le dossier **21-custom-form**, sélectionnez **setup.cmd**. Vous allez utiliser ce script de commandes pour exécuter les commandes d’interface de ligne de commande (CLI) Azure requises pour créer les autres ressources Azure dont vous avez besoin.

11. Dans le script **setup.cmd**, passez en revue les commandes **rem**. Ces commentaires décrivent le programme que le script s’exécute. Le programme va : 
    - Créer un compte de stockage dans votre groupe de ressources Azure
    - Télécharger des fichiers de votre dossier _sampleforms_ local vers un conteneur appelé _sampleforms_ dans le compte de stockage
    - Imprimer un URI de signature d’accès partagé

12. Modifiez les déclarations de variable **subscription_id**, **resource_group** et **location** avec les valeurs appropriées pour le nom de l’abonnement, du groupe de ressources et de l’emplacement où vous avez déployé la ressource Form Recognizer. Ensuite, **enregistrez** vos modifications.

    Laissez la variable **expiry_date** telle qu’elle est pour l’exercice. Cette variable est utilisée lors de la génération de l’URI SAP (Shared Access Signature). Dans la pratique, vous devez définir une date d’expiration appropriée pour votre SAP. Vous en découvrirez davantage sur les SAP [ici](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works).  

13. Dans le terminal du dossier **21-custom-form**, entrez la commande suivante pour exécuter le script :

```
setup
```

14. Une fois le script terminé, passez en revue la sortie affichée et notez l’URI SAP de votre ressource Azure.

> **Important** : Avant de passer à l’opération, collez l’URI SAP quelque part, vous pourrez le récupérer ultérieurement (par exemple, dans un nouveau fichier texte dans Visual Studio Code).

15. Dans le portail Azure, actualisez le groupe de ressources et vérifiez qu'il contient le compte Stockage Azure qui vient d'être créé. Ouvrez le compte de stockage et, dans le volet de gauche, sélectionnez **Navigateur de stockage (préversion)** . Ensuite, dans le navigateur de stockage, développez les **CONTENEURS BLOB** et sélectionnez le conteneur **sampleforms** pour vérifier que les fichiers ont été chargés à partir de votre dossier local **21-custom-form/sample-forms**.

## <a name="train-a-model-using-the-form-recognizer-sdk"></a>Entraîner un modèle à l’aide du kit de développement logiciel (SDK) Form Recognizer

Vous allez maintenant entraîner un modèle à l’aide des fichiers **.jpg** et **.json** .

1. Dans Visual Studio Code, dans le dossier **21-custom-form/sample-forms**, ouvrez **fields.json** et passez en revue le document JSON qu’il contient. Ce fichier définit les champs dans lesquels vous allez entraîner un modèle à extraire des formulaires.
2. Ouvrez **Form_1.jpg.labels.json** et passez en revue le JSON qu’il contient. Ce fichier identifie l’emplacement et les valeurs des champs nommés dans le document d’entraînement **Form_1.jpg**.
3. Ouvrez **Form_1.jpg.ocr.json** et passez en revue le JSON qu’il contient. Ce fichier contient une représentation JSOn de la disposition de texte de **Form_1.jpg**, y compris l’emplacement de toutes les zones de texte trouvées dans le formulaire.

    *Les fichiers d'information sur les champs ont été fournis pour vous dans cet exercice. Pour vos propres projets, vous pouvez créer ces fichiers à l'aide de [Form Recognizer Studio](https://formrecognizer.appliedai.azure.com/studio). Lorsque vous utilisez l'outil, vos fichiers d'information de champ sont automatiquement créés et stockés dans votre compte de stockage connecté.*

4. Dans Visual Studio Code, dans le dossier **de 21-custom-form**, développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
5. Cliquez avec le bouton droit sur le dossier **create-search,** puis sélectionnez Ouvrir dans le terminal intégré.

6. Installez le package Form Recognizer en exécutant la commande appropriée pour votre préférence de langage :

**C#**

```
dotnet add package Azure.AI.FormRecognizer --version 3.0.0 
```

**Python**

```
pip install azure-ai-formrecognizer==3.0.0
```

7. Affichez le contenu du dossier **train-model**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#**  : appsettings.json
    - **Python** : .env

8. Modifiez le fichier de configuration, en modifiant les paramètres à refléter :
    - Le **point de terminaison** pour votre ressource Form Recognizer.
    - Une **clé** pour votre ressource Form Recognizer.
    - L’**URI SAP** de votre conteneur d’objets blob.

9. Notez que le dossier **train-model** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : train-model.py

    Ouvrez le fichier de code et passez en revue le code qu’il contient, en notant les détails suivants :
    - Les espaces de noms du package que vous avez installé sont importés
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **Client** authentifié.
    - Le code utilise le client d’entraînement pour entraîner un modèle à l’aide des images dans votre conteneur de stockage d’objets blob, accessible à l’aide de l’URI SAP que vous avez généré.

10. Dans le dossier **train-model**, ouvrez le fichier de code de l’application d’entraînement :

    - **C#** : Program.cs
    - **Python** : train-model.py

11. Revenez au terminal intégré pour le dossier **train-model**, puis entrez la commande suivante pour exécuter le programme :

**C#**

```
dotnet run
```

**Python**

```
python train-model.py
```

12. Attendez la fin du programme, puis passez en revue la sortie du modèle.
13. Notez l’ID de modèle dans la sortie du terminal. Vous en aurez besoin pour la partie suivante du labo. 

## <a name="test-your-custom-form-recognizer-model"></a>Tester votre modèle Form Recognizer personnalisé 

1. Dans le dossier **21-custom-form**, dans le sous-dossier de votre langage préféré (**C-Sharp** ou **Python**), développez le dossier **test-model**.

2. Cliquez avec le bouton droit sur le dossier **train-model** et sélectionnez **ouvrir un terminal intégré**.

3. Dans le terminal du dossier **test-model**, installez le package Form Recognizer en exécutant la commande appropriée pour votre préférence de langage :

**C#**

```
dotnet add package Azure.AI.FormRecognizer --version 3.0.0 
```

**Python**

```
pip install azure-ai-formrecognizer==3.0.0
```

*Cela n’est pas obligatoire si vous avez utilisé pip précédemment pour installer le package dans l’environnement Python. Cependant, il peut être utile de vérifier s'il est installé.*

4. Dans le même terminal pour le dossier **test-model**, installez la bibliothèque Tabulate. Cela fournit votre sortie dans une table :

**C#**

```
Install-Package Tabulate.NET -Version 1.0.5
```

**Python**

```
pip install tabulate
```

5. Dans le dossier **test-model**, modifiez le fichier de configuration (**appsettings.json** ou **.env**, selon votre préférence de langage) pour ajouter les valeurs suivantes :
    - Votre point de terminaison Form Recognizer.
    - Votre clé Form Recognizer.
    - L’ID de modèle généré lorsque vous avez entraîné le modèle (vous pouvez le trouver en basculant le terminal vers la console **cmd** pour le dossier **train-model**). **Enregistrez** les changements apportés.

6. Dans le dossier **test-model**, ouvrez le fichier de code de votre application cliente (*Program.cs* pour C#, *test-model.py* pour Python) et examinez le code qu’il contient, en notant les détails suivants :
    - Les espaces de noms du package que vous avez installé sont importés
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **Client** authentifié.
    - Le client est ensuite utilisé pour extraire les champs de formulaire et les valeurs de l’image **test1.jpg** .
    

7. Revenez au terminal intégré pour le dossier **train-model**, puis entrez la commande suivante pour exécuter le programme :

**C#**

```
dotnet run
```

**Python**

```
python test-model.py
```
    
8. Affichez la sortie et observez comment la sortie du modèle fournit des noms de champs tels que « CompanyPhoneNumber » et « DatedAs ».   

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur le service Form Recognizer, consultez l'article [Documentation Azure Form Recognizer](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/).
