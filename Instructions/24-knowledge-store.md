---
lab:
  title: Créer une base de connaissances avec la Recherche cognitive Azure
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 8d7008ee592d4a76f24ca26a4b192c7dd5cbe9ea
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195548"
---
# <a name="create-a-knowledge-store-with-azure-cognitive-search"></a>Créer une base de connaissances avec la Recherche cognitive Azure

La Recherche cognitive Azure utilise un pipeline d’enrichissement des compétences cognitives pour extraire des champs générés par l’IA à partir de documents et les inclure dans un index de recherche. Si l’index peut être considéré comme la sortie principale d’un processus d’indexation, les données enrichies qu’il contient peuvent également être utiles à d’autres fins. Par exemple :

- Étant donné que l’index est essentiellement une collection d’objets JSON, chacun représentant un enregistrement indexé, il peut être utile d’exporter les objets en tant que fichiers JSON pour les intégrer dans un processus d’orchestration de données à l’aide d’outils tels que Azure Data Factory.
- Vous pouvez normaliser les enregistrements d’index dans un schéma relationnel de tables pour l’analyse et la création de rapports avec des outils tels que Microsoft Power BI.
- Si vous avez extrait des images incorporées de documents pendant le processus d’indexation, vous souhaiterez peut-être enregistrer ces images en tant que fichiers.

Dans cet exercice, vous allez implémenter une base de connaissances pour *Margie’s Travel*, une agence de voyages fictive qui utilise des informations dans des brochures et des revues d’hôtel pour aider ses clients à planifier leurs voyages.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour cette formation

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-azure-resources"></a>Créer des ressources Azure

> **Remarque** : Si vous avez déjà effectué l’exercice **[Créer une solution Recherche cognitive Azure](22-azure-search.md)** et que vous disposez toujours de ces ressources Azure dans votre abonnement, vous pouvez ignorer cette section et commencer à la section **Créer une solution de recherche**. Sinon, suivez les étapes ci-dessous pour provisionner les ressources Azure requises.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Affichez les **Groupes de ressources** dans votre abonnement.
3. Si vous utilisez un abonnement restreint dans lequel un groupe de ressources vous a été fourni, sélectionnez le groupe de ressources pour afficher ses propriétés. Sinon, créez un groupe de ressources avec le nom de votre choix et accédez-y lors de sa création.
4. Dans la page **Vue d’ensemble** de votre groupe de ressources, notez l’**ID d’abonnement** et l’**Emplacement**. Vous aurez besoin de ces valeurs, ainsi que du nom du groupe de ressources dans les étapes suivantes.
5. Dans Visual Studio Code, développez le dossier **24-knowledge-store** et sélectionnez **setup.cmd**. Vous allez utiliser ce script de commandes pour exécuter les commandes d’interface de ligne de commande (CLI) Azure requises pour créer les ressources Azure dont vous avez besoin.
6. Cliquez avec le bouton droit sur le dossier **24-knowledge-store**, puis sélectionnez **Ouvrir dans le terminal intégré**.
7. Dans le volet Terminal, entrez la commande suivante pour établir une connexion authentifiée à votre abonnement Azure.

    ```
    az login --output none
    ```

8. Lorsque vous y êtes invité, connectez-vous à votre abonnement Azure. Revenez ensuite à Visual Studio Code et attendez la fin du processus de connexion.
9. Exécutez la commande suivante pour lister vos emplacements Azure.

    ```
    az account list-locations -o table
    ```

10. Dans la sortie, recherchez la valeur **Nom** qui correspond à l’emplacement de votre groupe de ressources (par exemple, pour *East US*, le nom correspondant est *eastus*).
11. Dans le script **setup.cmd**, modifiez les déclarations de variables de **subscription_id**, **resource_group** et **location** avec les valeurs appropriées pour votre ID d’abonnement, le nom du groupe de ressources et le nom de l’emplacement. Ensuite, enregistrez vos modifications.
12. Dans le terminal du dossier **24-knowledge-store**, entrez la commande suivante pour exécuter le script :

    ```
    setup
    ```
    > **Remarque** : Le module CLI de recherche est en préversion et peut être bloqué dans le processus *- Exécution ..* processus de Rsync. Si cela se produit pendant plus de 2 minutes, appuyez sur Ctrl+C pour annuler l’opération de longue durée, puis sélectionnez **N** lorsque vous êtes invité à arrêter le script. Il devrait ensuite se terminer correctement.
    >
    > Si le script échoue, vérifiez que vous l’avez enregistré avec les noms de variables corrects et réessayez.

13. Une fois le script terminé, examinez la sortie affichée et notez les informations suivantes sur vos ressources Azure (vous aurez besoin de ces valeurs ultérieurement) :
    - Nom du compte de stockage
    - Chaîne de connexion de stockage
    - Compte Cognitive Services
    - Clé Cognitive Services
    - Point de terminaison du service de recherche
    - Clé d’administration du service de recherche
    - Clé de requête du service de recherche

14. Dans le Portail Azure, actualisez le groupe de ressources et vérifiez qu’il contient le compte de stockage Azure, la ressource Azure Cognitive Services et la ressource Recherche cognitive Azure.

## <a name="create-a-search-solution"></a>Créer une solution de recherche

Maintenant que vous disposez des ressources Azure nécessaires, vous pouvez créer une solution de recherche avec les composants suivants :

- Une **source de données** qui référence les documents dans votre conteneur de stockage Azure.
- Un **ensemble de compétences** qui définit un pipeline d’enrichissement de compétences pour extraire des champs générés par l’IA à partir des documents. L’ensemble de compétences définit également les *projections* qui seront générées dans votre *base de connaissances*.
- Un **index** qui définit un ensemble d’enregistrements de documents pouvant faire l’objet d’une recherche.
- Un **indexeur** qui extrait les documents de la source de données, applique l’ensemble de compétences et remplit l’index. Le processus d’indexation conserve également les projections définies dans l’ensemble de compétences de la base de connaissances.

Dans cet exercice, vous allez utiliser l’interface REST Recherche cognitive Azure pour créer ces composants en envoyant des requêtes JSON.

### <a name="prepare-json-for-rest-operations"></a>Préparer JSON pour les opérations REST

Vous allez utiliser l’interface REST pour envoyer des définitions JSON à vos composants Recherche cognitive Azure.

1. Dans Visual Studio Code, dans le dossier **24-knowledge-store**, développez le dossier **create-search** et sélectionnez **data_source.json**. Ce fichier contient une définition JSON pour une source de données nommée **margies-knowledge-data**.
2. Remplacez l’espace réservé **YOUR_CONNECTION_STRING** par la chaîne de connexion de votre compte de stockage Azure, qui doit ressembler à ceci :

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Vous trouverez la chaîne de connexion sur la page **Clés d’accès** de votre compte de stockage dans le Portail Azure.*

3. Enregistrez et fermez le fichier JSON mis à jour.
4. Dans le dossier **create-search**, ouvrez **skillset.json**. Ce fichier contient une définition JSON pour un ensemble de compétences nommé **margies-knowledge-skillset**.
5. En haut de la définition de l’ensemble de compétences, dans l’élément **cognitiveServices**, remplacez l’espace réservé **YOUR_COGNITIVE_SERVICES_KEY** par l’une des clés de vos ressources Cognitive Services.

    *Vous trouverez les clés dans la page **Clés et point de terminaison** de votre ressource Cognitive Services dans le Portail Azure.*

6. À la fin de la collection de compétences de votre ensemble de compétences, recherchez la compétence **Microsoft.Skills.Util.ShaperSkill** nommée **define-projection**. Cette compétence définit une structure JSON pour les données enrichies qui seront utilisées pour les projections que le pipeline configurera dans la base de connaissances pour chaque document traité par l’indexeur.
7. En bas du fichier d’ensemble de compétences, notez que l’ensemble de compétences inclut également une définition **knowledgeStore**, qui comprend une chaîne de connexion pour le compte de stockage Azure dans lequel la base de connaissances doit être créée, ainsi qu’une collection de **projections**. Cet ensemble de compétences comprend trois *groupes de projection* :
    - Un groupe contenant une projection d’*objet* basée sur la sortie **knowledge_projection** de la compétence de modélisateur dans l’ensemble de compétences.
    - Un groupe contenant une projection de *fichier* basée sur la collection **normalized_images** des données d’image extraites des documents.
    - Un groupe contenant les projections de *table* suivantes :
        - **KeyPhrases** : contient une colonne clé générée automatiquement et une colonne **keyPhrase** mappée à la sortie de la collection **knowledge_projection/key_phrases/** de la compétence de modélisateur.
        - **Locations** : contient une colonne clé générée automatiquement et une colonne **location** mappée à la sortie de la collection **knowledge_projection/key_phrases/** de la compétence de modélisateur.
        - **ImageTags** : contient une colonne clé générée automatiquement et une colonne **tag** mappée à la sortie de la collection **knowledge_projection/image_tags/** de la compétence de modélisateur.
        - **Docs**: contient une colonne clé générée automatiquement et toutes les valeurs de sortie **knowledge_projection** de la compétence de modélisateur qui ne sont pas déjà attribuées à une table.
8. Remplacez l’espace réservé **YOUR_CONNECTION_STRING** pour la valeur **storageConnectionString** par la chaîne de connexion de votre compte de stockage.
9. Enregistrez et fermez le fichier JSON mis à jour.
10. Dans le dossier **create-search**, ouvrez **index.json**. Ce fichier contient une définition JSON pour un index nommé **margies-knowledge-index**.
11. Passez en revue le JSON de l’index, puis fermez le fichier sans apporter de modifications.
12. Dans le dossier **create-search**, ouvrez **indexer.json**. Ce fichier contient une définition JSON pour un indexeur nommé **margies-knowledge-indexer**.
13. Passez en revue le JSON de l’indexeur, puis fermez le fichier sans apporter de modifications.

### <a name="submit-rest-requests"></a>Envoyer des requêtes REST

Maintenant que vous avez préparé les objets JSON qui définissent vos composants de solution de recherche, vous pouvez soumettre les documents JSON à l’interface REST pour les créer.

1. Dans le dossier **create-search**, ouvrez **create-search.cmd**. Ce script de traitement par lots utilise l’utilitaire cURL pour envoyer les définitions JSON à l’interface REST de votre ressource Recherche cognitive Azure.
2. Remplacez les espaces réservés des variables **YOUR_SEARCH_URL** et **YOUR_ADMIN_KEY** par l’**URL** et l’une des **clés d’administration** de votre ressource Recherche cognitive Azure.

    *Vous trouverez ces valeurs dans les pages **Vue d’ensemble** et **Clés** de votre ressource Recherche cognitive Azure dans le Portail Azure.*

3. Enregistrez le fichier de commandes mis à jour.
4. Cliquez avec le bouton droit sur le dossier **create-search**, puis sélectionnez **Ouvrir dans le terminal intégré**.
5. Dans le volet Terminal du dossier **create-search**, entrez la commande suivante pour exécuter le script de commandes.

    ```
    create-search
    ```

6. Une fois le script terminé, dans le Portail Azure, sur la page de votre ressource Recherche cognitive Azure, sélectionnez la page **Indexeurs** et attendez que le processus d’indexation se termine.

    *Vous pouvez sélectionner **Actualiser** pour suivre la progression de l’opération d’indexation. Cette opération peut prendre environ une minute.*

    > **Conseil** : Si le script échoue, vérifiez les espaces réservés que vous avez ajoutés dans les fichiers **data_source.json** et **skillset.json**, ainsi que le fichier **create-search.cmd**. Après avoir corrigé les erreurs, vous devrez peut-être utiliser l’interface utilisateur du Portail Azure pour supprimer les composants créés dans votre ressource de recherche avant de réexécurer le script.

## <a name="view-the-knowledge-store"></a>Afficher la base de connaissances

Une fois que vous avez exécuté un indexeur qui utilise un ensemble de compétences pour créer une base de connaissances, les données enrichies extraites par le processus d’indexation sont conservées dans les projections de la base de connaissances.

### <a name="view-object-projections"></a>Afficher les projections d’objets

Les projections d’*objets* définies dans l’ensemble de compétences de Margie’s Travel se composent d’un fichier JSON pour chaque document indexé. Ces fichiers sont stockés dans un conteneur d’objets blob dans le compte de stockage Azure spécifié dans la définition de l’ensemble de compétences.

1. Dans le Portail Azure, affichez le compte de stockage Azure que vous avez créé précédemment.
2. Sélectionnez l’onglet **Explorateur de stockage** (dans le volet de gauche) pour afficher le compte de stockage dans l’interface de l’Explorateur de stockage du Portail Azure.
2. Développez **CONTENEURS D’OBJETS BLOB** pour afficher les conteneurs dans le compte de stockage. Outre le conteneur **Margie** dans lequel les données sources sont stockées, il doit y avoir deux nouveaux conteneurs : **margies-images** et **margies-knowledge**. Ceux-ci ont été créés par le processus d’indexation.
3. Sélectionnez le conteneur **margies-knowledge**. Il doit contenir un dossier pour chaque document indexé.
4. Ouvrez l’un des dossiers, puis téléchargez et ouvrez le fichier **knowledge-projection.js** qu’il contient. Chaque fichier JSON contient une représentation d’un document indexé, comprenant les données enrichies extraites par l’ensemble de compétences, comme indiqué ici.

```
{
    "file_id":"abcd1234....",
    "file_name":"Margies Travel Company Info.pdf",
    "url":"https://store....blob.core.windows.net/margies/...pdf",
    "language":"en",
    "sentiment":0.83164644241333008,
    "key_phrases":[
        "Margie’s Travel",
        "Margie's Travel",
        "best travel experts",
        "world-leading travel agency",
        "international reach"
        ],
    "locations":[
        "Dubai",
        "Las Vegas",
        "London",
        "New York",
        "San Francisco"
        ],
    "image_tags":[
        "outdoor",
        "tree",
        "plant",
        "palm"
        ]
}
```

La possibilité de créer des projections d’*objets* de ce type vous permet de générer des objets de données enrichis qui peuvent être incorporés dans une solution d’analyse de données d’entreprise, par exemple en ingérant les fichiers JSON dans un pipeline Azure Data Factory en vue d’un traitement plus poussé ou d’un chargement dans un entrepôt de données.

### <a name="view-file-projections"></a>Afficher des projections de fichiers

Les projections de *fichiers* définies dans l’ensemble de compétences créent des fichiers JPEG pour chaque image extraite des documents pendant le processus d’indexation.

1. Dans l’interface de l’Explorateur de stockage du Portail Azure, sélectionnez le conteneur d’objets blob **margies-images**. Ce conteneur contient un dossier pour chaque document qui contenait des images.
2. Ouvrez l’un des dossiers et affichez son contenu : chaque dossier contient au moins un fichier \*.jpg.
3. Ouvrez l’un des fichiers image pour vérifier qu’ils contiennent des images extraites des documents.

La capacité à générer des projections de *fichiers* comme celle-ci permet d’envisager l’indexation comme un moyen efficace d’extraire des images incorporées à partir d’un grand volume de documents.

### <a name="view-table-projections"></a>Afficher des projections de tables

Les projections de *tables* définies dans l’ensemble de compétences forment un schéma relationnel de données enrichies.

1. Dans l’interface de l’Explorateur de stockage du Portail Azure, développez **TABLES**.
2. Sélectionnez la table **Docs** pour en afficher les colonnes. Les colonnes incluent des colonnes de table de stockage Azure standard. Pour les masquer, modifiez les **Options de colonne** pour sélectionner uniquement les colonnes suivantes :
    - **document_id** (colonne clé générée automatiquement par le processus d’indexation)
    - **file_id** (URL du fichier encodé)
    - **file_name** (nom de fichier extrait des métadonnées du document)
    - **language** (langue dans laquelle le document est écrit)
    - **sentiment** (score de sentiment calculé pour le document)
    - **url** (URL de l’objet blob de document dans le stockage Azure)
3. Affichez les autres tables qui ont été créées par le processus d’indexation :
    - **ImageTags** (contient une ligne pour chaque balise d’image individuelle avec le **document_id** correspondant au document dans lequel la balise apparaît).
    - **Keyphrase** (contient une ligne pour chaque expression clé individuelle avec le **document_id** correspondant au document dans lequel l’expression apparaît).
    - **Locations** (contient une ligne pour chaque emplacement individuel avec le **document_id** correspondant au document dans lequel l’emplacement apparaît).

La capacité de créer des projections de *tables* vous permet de créer des solutions d’analyse et de création de rapports qui interrogent le schéma relationnel, par exemple, à l’aide de Microsoft Power BI. Les colonnes clés générées automatiquement peuvent être utilisées pour joindre les tables dans des requêtes, par exemple pour retourner tous les emplacements mentionnés dans un document spécifique.

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur la création de bases de connaissances avec la Recherche cognitive Azure, consultez l'article Base de [connaissances dans Recherche cognitive Azure](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro).
