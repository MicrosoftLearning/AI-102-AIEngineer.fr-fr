---
lab:
  title: Créer une solution Recherche cognitive Azure
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: bba5786ed34cbbe806c74e2b2eec6286cb92554b
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/12/2022
ms.locfileid: "145195582"
---
# <a name="create-an-azure-cognitive-search-solution"></a>Créer une solution Recherche cognitive Azure

Toutes les organisations s’appuient sur des informations pour prendre des décisions, répondre à des questions et fonctionner efficacement. Le problème pour la plupart des organisations n’est pas un manque d’informations, mais la difficulté de trouver et d’extraire les informations à partir de l’ensemble des documents, bases de données et autres sources dans lesquelles les informations sont stockées.

Par exemple, supposons que *Margie’s Travel* est une agence de voyages spécialisée dans l’organisation de voyages dans des villes du monde entier. Au fil du temps, l’entreprise a accumulé de grandes quantités d’informations dans des documents tels que des brochures, ainsi que des critiques d’hôtels soumises par des clients. Ces données sont une source précieuse d’insights pour les agents de voyages et les clients lorsqu’ils planifient des voyages, mais le volume de données peut compliquer la recherche d’informations pertinentes pour répondre à une question de client spécifique.

Pour relever ce défi, Margie’s Travel peut utiliser la recherche cognitive Azure pour implémenter une solution dans laquelle les documents sont indexés et enrichis à l’aide de compétences cognitives basées sur l’IA pour faciliter leur recherche.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous n’avez pas déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement où vous travaillez sur ce laboratoire, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-azure-resources"></a>Créer des ressources Azure

La solution que vous allez créer pour Margie’s Travel nécessite les ressources suivantes dans votre abonnement Azure :

- Une ressource **Recherche cognitive Azure**, qui va gérer l’indexation et l’interrogation.
- Une ressource **Cognitive Services**, qui fournit les services d’IA pour les compétences que votre solution de recherche peut utiliser pour enrichir les données de la source de données avec des Insights générés par IA.
- Un **compte de stockage** avec un conteneur blob dans lequel les documents à rechercher sont stockés.

> **Important** : Vos ressources Recherche cognitive Azure et Cognitive Services doivent se trouver au même emplacement !

### <a name="create-an-azure-cognitive-search-resource"></a>Créer une ressource Recherche cognitive Azure

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *recherche*, puis créez une ressource **Recherche cognitive Azure** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Nom du service** : *Entrez un nom unique*
    - **Emplacement** : *Sélectionnez un emplacement. Remarquez que vos ressources Recherche cognitive Azure et Cognitive Services doivent se trouver au même emplacement*
    - **Niveau tarifaire** : De base

3. Attendez la fin du déploiement, puis accédez à la ressource déployée.
4. Passez en revue la page **Vue d’ensemble** du panneau de votre ressource Recherche cognitive Azure dans le Portail Azure. Ici, vous pouvez utiliser une interface visuelle pour créer, tester, gérer et surveiller les différents composants d’une solution de recherche ; y compris les sources de données, les index, les indexeurs et les ensembles de compétences.

### <a name="create-a-cognitive-services-resource"></a>Créer une ressource Cognitive Services

Si vous n’en avez pas encore dans votre abonnement, vous devez provisionner une ressource **Cognitive Services**. Votre solution de recherche va l’utiliser pour enrichir les données dans le magasin de données avec des insights générés par IA.

1. Revenez à la page d’accueil du portail Azure, puis sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Cognitive Services*, puis créez une ressource **Cognitive Services** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *le même groupe de ressources que votre ressource Recherche cognitive Azure*
    - **Région** : *la même région que votre ressource Recherche cognitive Azure*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S0
2. Cochez les cases nécessaires et créez la ressource.
3. Attendez la fin du déploiement, puis visualisez les détails du déploiement.

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

1. Revenez à la page d’accueil du portail Azure, puis sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Compte de stockage*, puis créez une ressource **Compte de stockage** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : **Le même groupe de ressources que vos ressources Recherche cognitive Azure et Cognitive Services*
    - **Nom du compte de stockage** : *Entrez un nom unique*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Performances** : standard
    - **Réplication** : Stockage localement redondant (LRS)
2. Attendez la fin du déploiement, puis accédez à la ressource déployée.
3. Dans la page **Vue d’ensemble**, notez l’**ID d’abonnement** : il identifie l’abonnement dans lequel le compte de stockage est provisionné.
4. Dans la page **Clés d’accès**, notez que deux clés ont été générées pour votre compte de stockage. Sélectionnez ensuite **Afficher les clés** pour afficher les clés.

    > **Conseil** : Laissez le panneau **Compte de stockage** ouvert. Vous aurez besoin de l’ID d’abonnement et de l’une des clés de la procédure suivante.

## <a name="upload-documents-to-azure-storage"></a>Charger des documents sur Stockage Azure

Maintenant que vous disposez des ressources requises, vous pouvez charger certains documents sur votre compte Stockage Azure.

1. Dans Visual Studio Code, dans l’onglet **Explorer** développez le dossier **22-create-a-search-solution** et sélectionnez **UploadDocs.cmd**.
2. Modifiez le fichier de commandes pour remplacer les espaces réservés **YOUR_SUBSCRIPTION_ID**, **YOUR_AZURE_STORAGE_ACCOUNT_NAME** et **YOUR_AZURE_STORAGE_KEY** avec l'ID d'abonnement, le nom du compte de stockage Azure et les valeurs de clé du compte de stockage Azure appropriés pour le compte de stockage que vous avez créé précédemment.
3. Enregistrez vos modifications, puis cliquez avec le bouton droit de la souris sur le dossier **22-create-a-search-solution** et ouvrez un terminal intégré.
4. Entrez la commande suivante pour vous connecter à votre abonnement Azure à l’aide d’Azure CLI.

    ```
    az login
    ```

Un onglet de navigateur web s’ouvre et vous invite à vous connecter à Azure. Pour ce faire, fermez l’onglet du navigateur et revenez à Visual Studio Code.

5. Entrez la commande suivante pour exécuter le fichier de commandes. Cela crée un conteneur d’objets blob dans votre compte de stockage et charge les documents dans le dossier de **données** vers celui-ci.

    ```
    UploadDocs
    ```

## <a name="index-the-documents"></a>Indexer les documents

Maintenant que vous disposez des documents en place, vous pouvez créer une solution de recherche en les indexant.

1. Dans le portail Azure, accédez à votre ressource Recherche cognitive Azure. Ensuite, dans sa page **Vue d’ensemble**, sélectionnez **Importer des données**.
2. Dans la page **Se connecter à vos données**, dans la liste **Source de données**, sélectionnez **Stockage Blob Azure**. Renseignez ensuite les détails du magasin de données avec les valeurs suivantes :
    - **Source de données** : Stockage Blob Azure
    - **Nom de la source de données** : margies-data
    - **Données à extraire** : Contenu et métadonnées
    - **Mode d’analyse** : Par défaut
    - **Chaîne de connexion** : *Sélectionnez **Choisir une connexion existante**. Sélectionnez ensuite le compte de stockage, puis le conteneur **margies** créé par le script UploadDocs.cmd.*
    - **Authentification d’identité managée** : Aucun
    - **Nom du conteneur** : margies
    - **Dossier d’objets blob** : *laissez ce champ vide*
    - **Description** : Brochures et évaluations sur le site web Margie’s Travel.
3. Passez à l’étape suivante (*Ajouter des compétences cognitives*).
4. Dans la section **Attacher Cognitive Services**, sélectionnez votre ressource Cognitive Services.
5. Dans la section **Ajouter des enrichissements** :
    - Remplacez le **Nom de l’ensemble de compétences** par **margies-skillset**.
    - Sélectionnez l’option **Activer la reconnaissance optique de caractères et fusionner tout le texte dans le champ merged_content**.
    - Vérifiez que le **champ Données sources** est défini sur **merged_content**.
    - Laissez le **niveau de granularité de l’enrichissement** comme **Champ source**, qui est défini sur l’ensemble des contenus du document indexé. Notez cependant que vous pouvez modifier cette option pour extraire des informations à des niveaux plus précis, tels que des pages ou des phrases.
    - Sélectionnez les champs enrichis suivants :

        | Compétence cognitive | Paramètre | Nom du champ |
        | --------------- | ---------- | ---------- |
        | Extraire les noms d’emplacement | | locations |
        | Extraire les phrases clés | | keyphrases |
        | Détecter la langue | | langage |
        | Générer les balises des images | | imageTags |
        | Générer les légendes des images | | imageCaption |

6. Vérifiez vos sélections (il peut être difficile de les modifier ultérieurement). Passez ensuite à l’étape suivante (*Personnaliser l’index cible*).
7. Remplacez le **nom de l’index** par **margies-index**.
8. Vérifiez que la **Clé** est définie sur **metadata_storage_path**, et laissez le **Nom du suggesteur** vide et le **Mode de recherche** par défaut.
9. Apportez les modifications suivantes aux champs d’index, en laissant tous les autres champs avec leurs paramètres par défaut (**IMPORTANT** : vous devrez peut-être faire défiler vers la droite pour afficher la table entière) :

    | Nom du champ | Récupérable | Filtrable | Triable | À choix multiples | Peut faire l’objet d’une recherche |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | metadata_author | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | langage | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | |

11. Vérifiez vos sélections, en veillant à ce que les options **Récupérables**, **Filtrables**, **Triables**, **À choix multiples** et avec une **Possibilité de recherche** soient sélectionnées pour chaque champ (il peut être difficile de les modifier ultérieurement). Passez ensuite à l’étape suivante (*Créer un indexeur*).
12. Remplacez le **Nom de l’index** par **margies-index**.
13. Laissez la **Planification** définie sur **Une fois**.
14. Développez les options **Avancées**, puis vérifiez que l’option **Clés d’encodage en base-64** est sélectionnée (en général, les clés d’encodage rendent l’index plus efficace).
15. Sélectionnez **Envoyer** pour créer la source de données, les compétences, l’index et l’indexeur. L’indexeur est exécuté automatiquement et exécute le pipeline d’indexation, qui :
    1. Extrait les champs de métadonnées des documents et le contenu de la source de données
    2. Exécute l’ensemble de compétences de compétences cognitives pour générer des champs enrichis supplémentaires
    3. Mappe les champs extraits à l’index.
16. Dans la moitié inférieure de la page **Vue d’ensemble** de votre ressource Recherche cognitive Azure, affichez l’onglet **Indexeurs**, qui devrait montrer l’index **margies-indexer** nouvellement créé. Attendez quelques minutes, puis cliquez sur **&orarr; Actualiser** jusqu’à ce que l’**État** indique la réussite.

## <a name="search-the-index"></a>Rechercher l’index

Maintenant que vous avez un index, vous pouvez y effectuer des recherches.

1. En haut de la page **Vue d’ensemble** de votre ressource Recherche cognitive Azure, sélectionnez **Rechercher dans l’explorateur**.
2. Dans l’Explorateur de recherche, dans la zone **Chaîne de requête**, entrez `*` (un astérisque unique), puis sélectionnez **Rechercher**.

    Cette requête récupère tous les documents dans l’index au format JSON. Examinez les résultats et notez les champs de chaque document, qui contiennent du contenu, des métadonnées et des données enrichies extraites par les compétences cognitives que vous avez sélectionnées.

3. Modifiez la chaîne de requête par `search=*&$count=true` et lancez la recherche.

    Cette fois, les résultats incluent un champ **@odata.count** en haut des résultats qui indique le nombre de documents retournés par la recherche.

4. Essayez la chaîne de requête suivante :

    ```
    search=*&$count=true&$select=metadata_storage_name,metadata_author,locations
    ```

    Cette fois, les résultats incluent uniquement le nom de fichier, l’auteur et tous les emplacements mentionnés dans le contenu du document. Le nom de fichier et l’auteur se trouvent dans les champs **metadata_storage_name** et **metadata_author**, qui ont été extraits du document source. Le champ **emplacements** a été généré par une compétence cognitive.

5. Essayez la chaîne de requête suivante :

    ```
    search="New York"&$count=true&$select=metadata_storage_name,keyphrases
    ```

    Cette recherche recherche des documents qui mentionnent « New York » dans l’un des champs pouvant faire l’objet d’une recherche et retourne le nom de fichier et les expressions clés du document.

6. Essayons une chaîne de requête supplémentaire :

    ```
    search="New York"&$count=true&$select=metadata_storage_name&$filter=metadata_author eq 'Reviewer'
    ```

    Cette requête retourne le nom de fichier des documents créés par le *Réviseur* qui mentionnent « New York ».

## <a name="explore-and-modify-definitions-of-search-components"></a>Explorer et modifier les définitions des composants de recherche

Les composants de la solution de recherche sont basés sur des définitions JSON que vous pouvez afficher et modifier dans le portail Azure.

Bien que vous puissiez utiliser le portail pour créer et modifier des solutions de recherche, il est souvent souhaitable de définir les objets de recherche au format JSON et d’utiliser l’interface REST Azure Cognitive Service pour les créer et les modifier.

### <a name="get-the-endpoint-and-key-for-your-azure-cognitive-search-resource"></a>Obtenir le point de terminaison et la clé de votre ressource Recherche cognitive Azure

1. Dans le portail Azure, revenez à la page **Vue d’ensemble** de votre ressource Recherche cognitive Azure. Puis, dans la section supérieure de la page, recherchez l’**URL** de votre ressource (qui ressemble à **https://resource_name.search.windows.net** ) et copiez-la dans le presse-papiers.
2. Dans Visual Studio Code, dans le volet Explorateur, développez le dossier **22-create-a-search-solution** et son sous-dossier **modify-search**, puis sélectionnez **modify-search.cmd** pour l’ouvrir. Vous allez utiliser ce fichier de script pour exécuter des commandes *cURL* qui envoient JSON à l’interface REST d’Azure Cognitive Service.
3. Dans **modify-search.cmd**, remplacez l’espace réservé **YOUR_SEARCH_URL** par l’URL que vous avez copiée dans le presse-papiers.
4. Dans le portail Azure, affichez la page **Clés** de votre ressource Recherche cognitive Azure et copiez la **Clé d’administration principale** dans le presse-papiers.
5. Dans Visual Studio Code, remplacez l’espace réservé **YOUR_ADMIN_KEY** par la clé que vous avez copiée dans le presse-papiers.
6. Enregistrez les modifications apportées à **modify-search.cmd** (mais ne l’exécutez pas encore !)

### <a name="review-and-modify-the-skillset"></a>Passer en revue et modifier l’ensemble de compétences

1. Dans Visual Studio Code, dans le dossier **modify-search**, ouvrez **skillset.json**. Cela montre une définition JSON pour **margies-skillset**.
2. En haut de la définition de l’ensemble de compétences, notez l’objet **cognitiveServices** , qui est utilisé pour connecter votre ressource Cognitive Services à l’ensemble de compétences.
3. Dans le portail Azure, ouvrez votre ressource Cognitive Services (<u>non</u> votre ressource Recherche cognitive Azure !) et affichez sa page **Clés**. Copiez ensuite la **Clé 1** dans le presse-papiers.
4. Dans Visual Studio Code, dans **skillset.json**, remplacez l’espace réservé **YOUR_COGNITIVE_SERVICES_KEY** par la clé Cognitive Services que vous avez copiée dans le presse-papiers.
5. Faites défiler le fichier JSON en notant qu’il inclut des définitions pour les compétences que vous avez créées à l’aide de l’interface utilisateur Recherche cognitive Azure dans le portail Azure. En bas de la liste des compétences, une compétence supplémentaire a été ajoutée avec la définition suivante :

    ```
    {
        "@odata.type": "#Microsoft.Skills.Text.V3.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "sentiment",
                "targetName": "sentimentLabel"
            }
        ]
    }
    ```

La nouvelle compétence est nommée **get-sentiment** et, pour chaque niveau de **document** d’un document, elle évalue le texte trouvé dans le champ **merged_content** du document indexé (qui inclut le contenu source ainsi que tout texte extrait d’images dans le contenu). Il utilise la **langue** extraite du document (avec une valeur par défaut anglais) et évalue une étiquette pour le sentiment du contenu. Les valeurs de l’étiquette de sentiment peuvent être « positives », « négatives », « neutres » ou « mixtes ». Cette étiquette est ensuite générée sous la forme d’un nouveau champ nommé **sentimentLabel**.

6. Enregistrez les modifications que vous avez apportées à **skillset.json**.

### <a name="review-and-modify-the-index"></a>Passer en revue et modifier l’index

1. Dans Visual Studio Code, dans le dossier **modify-search**, ouvrez **index.json**. Cela montre une définition JSON pour **margies-index**.
2. Faites défiler l’index et affichez les définitions de champ. Certains champs sont basés sur les métadonnées et le contenu dans le document source, et d’autres sont les résultats des compétences dans l’ensemble de compétences.
3. À la fin de la liste des champs que vous avez définis dans le portail Azure, notez que deux champs supplémentaires ont été ajoutés :

    ```
    {
        "name": "sentiment",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. Le champ **sentiment** sera utilisé pour ajouter la sortie de la compétence **get-sentiment** qui a été ajoutée à l’ensemble de compétences. Le champ **url** sera utilisé pour ajouter l’URL de chaque document indexé à l’index, en fonction de la valeur **metadata_storage_path** extraite de la source de données. Notez que l’index inclut déjà le champ **metadata_storage_path** , mais qu’il est utilisé comme clé d’index et codé en base 64, ce qui le rend efficace en tant que clé, mais que les applications clientes doivent le décoder s’ils souhaitent utiliser la valeur d’URL réelle en tant que champ. L’ajout d’un deuxième champ pour la valeur non codée résout ce problème.

### <a name="review-and-modify-the-indexer"></a>Passer en revue et modifier l’indexeur

1. Dans Visual Studio Code, dans le dossier **modify-search**, ouvrez **indexer.json**. Cela montre une définition JSON pour **margies-indexer**, qui mappe les champs extraits du contenu et des métadonnées du document (dans la section **fieldMappings**) et les valeurs extraites par les compétences de l’ensemble de compétences (dans la section **outputFieldMappings** ), aux champs dans l’index.
3. Dans la liste **fieldMappings**, notez le mappage de la valeur **metadata_storage_path** au champ clé codé en base 64. Cela a été créé lorsque vous avez affecté le **metadata_storage_path** en tant que clé et sélectionné l’option pour encoder la clé dans le portail Azure. En outre, un nouveau mappage mappe explicitement la même valeur au champ **url**, mais sans le codage en base 64 :

    ```
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }
    
    ```

Tous les autres champs de métadonnées et de contenu du document source sont implicitement mappés aux champs du même nom dans l’index.

4. Passez en revue la section **ouputFieldMappings**, qui mappe les sorties des compétences de l’ensemble de compétences aux champs d’index. La plupart de ces choix reflètent les choix que vous avez effectués dans l’interface utilisateur, mais le mappage suivant a été ajouté pour mapper la valeur **sentimentLabel** extraite par votre compétence de sentiment au champ de **sentiment** que vous avez ajouté à l’index :

    ```
    {
        "sourceFieldName": "/document/sentimentLabel",
        "targetFieldName": "sentiment"
    }
    ```

### <a name="use-the-rest-api-to-update-the-search-solution"></a>Utiliser l’API REST pour mettre à jour la solution de recherche

1. Cliquez avec le bouton droit de la souris sur le dossier **modify-search** et ouvrez un terminal intégré.
2. Dans le volet terminal du dossier **modify-search**, entrez la commande suivante pour exécuter le script **modify-search.cmd**, qui soumet les définitions JSON à l’interface REST et lance l’indexation.

    ```
    modify-search
    ```

3. Une fois le script terminé, revenez à la page **Vue d’ensemble** de votre ressource Recherche cognitive Azure dans le portail Azure et affichez la page **Indexeurs**. Sélectionnez **Actualiser** régulièrement pour suivre la progression de l’opération d’indexation. Cette opération peut prendre environ une minute.

    *Il peut y avoir quelques avertissements pour quelques documents qui sont trop grands pour évaluer le sentiment. Souvent, l'analyse des sentiments est effectuée au niveau de la page ou de la phrase plutôt qu'au niveau du document complet ; mais dans ce scénario, la plupart des documents, en particulier les critiques d'hôtels, sont suffisamment courts pour que des scores de sentiments utiles au niveau du document puissent être évalués.*

### <a name="query-the-modified-index"></a>Interroger l’index modifié

1. En haut de l’onglet de votre ressource Recherche cognitive Azure, sélectionnez **Explorateur de recherche**.
2. Dans l’Explorateur de recherche, dans la zone **Chaîne de requête**, entrez la chaîne de requête suivante, puis sélectionnez **Rechercher**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    Cette requête récupère l’**URL**, le **sentiment** et les **phrases clés** pour tous les documents qui mentionnent *Londres* créés par *Reviewer* qui ont une étiquette de **sentiment** positive (en d’autres termes, des critiques positives qui mentionnent Londres)

3. Fermez la page **Explorateur de recherche** pour revenir à la page **Vue d’ensemble**.

## <a name="create-a-search-client-application"></a>Créer une application cliente de recherche

Maintenant que vous avez un index utile, vous pouvez l’utiliser à partir d’une application cliente. Vous pouvez le faire en consommant l’interface REST, en envoyant des demandes et en recevant des réponses au format JSON via HTTP ; ou vous pouvez utiliser le kit de développement logiciel (SDK) pour votre langage de programmation préféré. Dans cet exercice, nous allons utiliser le kit de développement logiciel (SDK).

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

### <a name="get-the-endpoint-and-keys-for-your-search-resource"></a>Obtenir le point de terminaison et les clés de votre ressource de recherche

1. Dans le portail Azure, dans la page **Vue d’ensemble** de votre ressource Recherche cognitive Azure, notez la valeur de l’**URL**, qui doit être similaire à **https://*your_resource_name*.search.windows.net**. Il s’agit du point de terminaison de votre ressource de recherche.
2. Dans la page **Clés** , notez qu’il existe deux clés d’**administration** et une seule clé de **requête**. Une clé d’*administration* est utilisée pour créer et gérer des ressources de recherche. Une clé de *requête* est utilisée par les applications clientes qui doivent uniquement effectuer des requêtes de recherche.

    *Vous aurez besoin du point de terminaison et de la clé de requête pour votre application cliente.*

### <a name="prepare-to-use-the-azure-cognitive-search-sdk"></a>Préparer l’utilisation du kit de développement logiciel (SDK) Recherche cognitive Azure

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **22-create-a-search-solution** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit de la souris sur le dossier **margies-travel** et ouvrez un terminal intégré. Installez ensuite le package SDK Recherche cognitive Azure en exécutant la commande appropriée pour votre préférence de langage :

    **C#**
    
    ```
    dotnet add package Azure.Search.Documents --version 11.1.1
    ```
    
    **Python**
    
    ```
    pip install azure-search-documents==11.0.0
    ```
    
3. Affichez le contenu du dossier **margies-travel**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et la **clé de requête** de votre ressource Recherche cognitive Azure. Enregistrez vos modifications.

### <a name="explore-code-to-search-an-index"></a>Explorer le code pour rechercher un index

Le dossier **margies-travel** contient des fichiers de code pour une application web (une application web Microsoft C# *ASP.NET Razor* ou une application *Flask* Python), qui inclut des fonctionnalités de recherche.

1. Ouvrez le fichier de code suivant dans l’application web, en fonction de votre choix de langage de programmation :
    - **C#**  : Pages/Index.cshtml.cs
    - **Python** : app.py
2. En haut du fichier de code, recherchez le commentaire **Importer les espaces de noms de recherche** et notez les espaces de noms importés pour travailler avec le kit SDK Recherche cognitive Azure :
3. Dans la fonction **search_query**, recherchez le commentaire **Créer un client de recherche** et notez que le code crée un objet **SearchClient** à l’aide du point de terminaison et de la clé de requête pour votre ressource Recherche cognitive Azure :
4. Dans la fonction **search_query**, recherchez le commentaire **Envoyer une requête de recherche** et passez en revue le code pour lancer une recherche pour le texte spécifié avec les options suivantes :
    - Un *mode de recherche* qui nécessite que **tous** les mots individuels dans le texte de recherche soient trouvés.
    - Le nombre total de documents trouvés par la recherche est inclus dans les résultats.
    - Les résultats sont filtrés pour inclure uniquement les documents qui correspondent à l’expression de filtre fournie.
    - Les résultats sont triés dans l’ordre de tri spécifié.
    - Chaque valeur discrète du champ **metadata_author** est retournée en tant que *facette* qui peut être utilisée pour afficher des valeurs prédéfinies pour le filtrage.
    - Jusqu’à trois extraits des champs **merged_content** et **imageCaption** avec les termes de recherche mis en surbrillance sont inclus dans les résultats.
    - Les résultats incluent uniquement les champs spécifiés.

### <a name="explore-code-to-render-search-results"></a>Explorer le code pour afficher les résultats de la recherche

L’application web inclut déjà du code pour traiter et afficher les résultats de la recherche.

1. Ouvrez le fichier de code suivant dans l’application web, en fonction de votre choix de langage de programmation :
    - **C#**  : Pages/Index.cshtml
    - **Python** : templates/search.html
2. Examinez le code, qui affiche la page sur laquelle les résultats de la recherche sont affichés. Observez que :
    - La page commence par un formulaire de recherche que l’utilisateur peut utiliser pour soumettre une nouvelle recherche (dans la version Python de l’application, ce formulaire est défini dans le modèle **base.html** ), qui est référencé au début de la page.
    - Un deuxième formulaire est ensuite rendu, ce qui permet à l’utilisateur d’affiner les résultats de la recherche. Code de ce formulaire :
        - Récupère et affiche le nombre de documents à partir des résultats de la recherche.
        - Récupère les valeurs de facette du champ **metadata_author** et les affiche en tant que liste d’options pour le filtrage.
        - Crée une liste déroulante d’options de tri pour les résultats.
    - Le code effectue ensuite une itération dans les résultats de la recherche, en montrant chaque résultat comme suit :
        - Affichez le champ **metadata_storage_name** (nom de fichier) en tant que lien vers l’adresse dans le champ de l’**URL**.
        - Affichage de *surbrillances* pour les termes de recherche trouvés dans les champs **merged_content** et **imageCaption** pour vous aider à afficher les termes de recherche dans le contexte.
        - Affichez les champs **metadata_author**, **metadata_storage_size**, **metadata_storage_last_modified** et **langage** .
        - Affichez l’étiquette de **sentiment** du document. Peut être positif, négatif, neutre ou mixte.
        - Affichez les cinq premiers **keyphrases** (le cas échéant).
        - Affichez les cinq premiers **emplacements** (le cas échéant).
        - Affichez les cinq premiers **imagesTags** (le cas échéant).

### <a name="run-the-web-app"></a>Exécuter l’application web

 1. revenez au terminal intégré pour le dossier **margies-travel**, puis entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. Dans le message qui s’affiche lorsque l’application démarre correctement, suivez le lien vers l’application web en cours d’exécution ( *http://localhost:5000/* ou *http://127.0.0.1:5000/* ) pour ouvrir le site Margies Travel dans un navigateur web.
3. Dans le site web Margie’s Travel, entrez **hôtel Londres** dans la zone de recherche, puis cliquez sur **Rechercher**.
4. Examinez les résultats de recherche. Ils incluent le nom de fichier (avec un lien hypertexte vers l’URL de fichier), un extrait du contenu du fichier avec les termes de recherche (*hôtel* et *Londres*) mis en évidence, et d’autres attributs du fichier à partir des champs d’index.
5. Notez que la page de résultats inclut certains éléments d’interface utilisateur qui vous permettent d’affiner les résultats. notamment :
    - Un *filtre* basé sur une valeur de facette pour le champ **metadata_author**. Cela montre comment utiliser des champs *à choix multiples* pour renvoyer une liste de *facettes* - champs avec un petit ensemble de valeurs discrètes qui peuvent s’afficher comme valeurs de filtre potentielles dans l’interface utilisateur.
    - Possibilité d’*ordonner* les résultats en fonction d’un champ spécifié et d’une direction de tri (croissant ou décroissant). L’ordre par défaut est basé sur la *pertinence*, qui est calculé en tant que valeur **search.score()** basée sur un *profil de score* qui évalue la fréquence et l’importance des termes de recherche dans les champs d’index.
6. Sélectionnez le filtre **Réviseur** et l’option de tri **Positif à négatif**, puis sélectionnez **Affiner les résultats**.
7. Notez que les résultats sont filtrés pour inclure uniquement les révisions et triés en fonction de l’étiquette de sentiment.
8. Dans la zone de **Recherche**, entrez une nouvelle recherche d’**hôtel calme à New York** et passez en revue les résultats.
9. Essayez les termes de recherche suivants :
    - **Tour de Londres** (observez que ce terme est identifié comme une *expression clé* dans certains documents).
    - **gratte-ciel** (observez que ce mot n’apparaît pas dans le contenu réel des documents, mais se trouve dans les *légendes d’image* et les *balises d’image* qui ont été générées pour les images dans certains documents).
    - **Désert de Mojave** (observez que ce terme est identifié comme un *emplacement* dans certains documents).
10. Fermez l’onglet du navigateur contenant le site web Margie’s Travel et revenez à Visual Studio Code. Ensuite, dans le terminal Python du dossier **margies-travel** (où l’application dotnet ou flask est en cours d’exécution), entrez CTRL + C pour arrêter l’application.

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur la Recherche cognitive Azure, consultez la [documentation Recherche cognitive Azure](https://docs.microsoft.com/azure/search/search-what-is-azure-search).
