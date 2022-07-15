---
lab:
  title: Créer une compétence personnalisée pour la Recherche cognitive Azure
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 1321115b5c06fb6791f24f9f89311524be14accc
ms.sourcegitcommit: 883a607fbe52f48a9425c38a997f776ecd0130af
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/01/2022
ms.locfileid: "145195566"
---
# <a name="create-a-custom-skill-for-azure-cognitive-search"></a>Créer une compétence personnalisée pour la Recherche cognitive Azure

La Recherche cognitive Azure utilise un pipeline d’enrichissement des compétences cognitives pour extraire des champs générés par l’IA à partir de documents et les inclure dans un index de recherche. Il existe un ensemble complet de compétences intégrées que vous pouvez utiliser, mais si vous avez une exigence spécifique qui n’est pas satisfaite par ces compétences, vous pouvez créer une compétence personnalisée.

Dans cet exercice, vous allez créer une compétence personnalisée qui remplit la fréquence des mots individuels dans un document pour générer une liste des cinq premiers mots les plus utilisés et l’ajouter à une solution de recherche pour Margie’s Travel, une agence de voyage fictive.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour cette formation

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-azure-resources"></a>Créer des ressources Azure

> **Remarque** : Si vous avez déjà effectué l’exercice **[Créer une solution Recherche cognitive Azure](22-azure-search.md)** et que vous disposez toujours de ces ressources Azure dans votre abonnement, vous pouvez ignorer cette section et commencer à la section **Créer une solution de recherche**. Sinon, suivez les étapes ci-dessous pour provisionner les ressources Azure requises.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Affichez les **Groupes de ressources** dans votre abonnement.
3. Si vous utilisez un abonnement restreint dans lequel un groupe de ressources a été fourni pour vous, sélectionnez le groupe de ressources pour afficher ses propriétés. Sinon, créez un groupe de ressources avec le nom de votre choix et accédez-y lors de sa création.
4. Sur la page **Vue d’ensemble** de votre groupe de ressources, notez l’**ID d’abonnement** et l’**Emplacement**. Vous aurez besoin de ces valeurs, ainsi que du nom du groupe de ressources dans les étapes suivantes.
5. Dans Visual Studio Code, développez le dossier **23-custom-search-skill** et sélectionnez **setup.cmd**. Vous allez utiliser ce script de commandes pour exécuter les commandes d’interface de ligne de commande (CLI) Azure requises pour créer les ressources Azure dont vous avez besoin.
6. Cliquez avec le bouton droit de la souris sur le dossier **23-custom-search-skill**, puis sélectionnez **Ouvrir dans le terminal intégré**.
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
12. Dans le terminal du dossier **23-custom-search-skill**, entrez la commande suivante pour exécuter le script :

    ```
    setup
    ```

    > **Remarque** : Le module CLI de recherche est en préversion et peut être bloqué dans l’*Exécution..* processus de Rsync. Si cela se produit pendant plus de 2 minutes, appuyez sur Ctrl+C pour annuler l’opération de longue durée, puis sélectionnez **N** lorsque vous êtes invité à arrêter le script. Il devrait ensuite se terminer correctement.
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

14. Dans le portail Azure, actualisez le groupe de ressources et vérifiez qu’il contient le compte de stockage Azure, la ressource Azure Cognitive Services et la ressource Recherche cognitive Azure.

## <a name="create-a-search-solution"></a>Créer une solution de recherche

Maintenant que vous disposez des ressources Azure nécessaires, vous pouvez créer une solution de recherche avec les composants suivants :

- Une **source de données** qui référence les documents dans votre conteneur de stockage Azure.
- Un **ensemble de compétences** qui définit un pipeline d’enrichissement de compétences pour extraire des champs générés par l’IA à partir des documents.
- Un **index** qui définit un ensemble d’enregistrements de document pouvant faire l’objet d’une recherche.
- Un **indexeur** qui extrait les documents de la source de données, applique l’ensemble de compétences et remplit l’index.

Dans cet exercice, vous allez utiliser l’interface REST Recherche cognitive Azure pour créer ces composants en envoyant des requêtes JSON.

1. Dans Visual Studio Code, dans le dossier **23-custom-search-skill**, développez le dossier **create-search** et sélectionnez **data_source.json**. Ce fichier contient une définition JSON pour une source de données nommée **margies-custom-data**.
2. Remplacez l’espace réservé **YOUR_CONNECTION_STRING** par la chaîne de connexion de votre compte de stockage Azure, qui doit ressembler à ceci :

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Vous trouverez la chaîne de connexion dans la page **Clés d’accès** de votre compte de stockage dans le portail Azure.*

3. Enregistrez et fermez le fichier JSON mis à jour.
4. Dans le dossier **create-search**, ouvrez **skillset.json**. Ce fichier contient une définition JSON pour un ensemble de compétences nommé **margies-custom-skillset**.
5. En haut de la définition de l’ensemble de compétences, dans l’élément **cognitiveServices**, remplacez l’espace réservé **YOUR_COGNITIVE_SERVICES_KEY** par l’une des clés de vos ressources Cognitive Services.

    *Vous trouverez les clés dans la page **Clés et point de terminaison** de votre ressource Cognitive Services dans le portail Azure.*

6. Enregistrez et fermez le fichier JSON mis à jour.
7. Dans le dossier **create-search**, ouvrez **index.json**. Ce fichier contient une définition JSON pour une index nommée **margies-custom-index**.
8. Passez en revue le JSON de l’index, puis fermez le fichier sans apporter de modifications.
9. Dans le dossier **create-search**, ouvrez **indexer.json**. Ce fichier contient une définition JSON pour une indexeur nommé **margies-custom-indexer**.
10. Passez en revue le JSON de l’indexeur, puis fermez le fichier sans apporter de modifications.
11. Dans le dossier **create-search**, ouvrez **create-search.cmd**. Ce script de traitement par lots utilise l’utilitaire cURL pour envoyer les définitions JSON à l’interface REST de votre ressource Recherche cognitive Azure.
12. Remplacez les espaces réservés des variables **YOUR_SEARCH_URL** et **YOUR_ADMIN_KEY** par l’**URL** et l’une des **clés d’administration** de votre ressource Recherche cognitive Azure.

    *Vous trouverez ces valeurs dans les pages **Vue d’ensemble** et **Clés** de votre ressource Recherche cognitive Azure dans le portail Azure.*

13. Enregistrez le fichier de commandes mis à jour.
14. Cliquez avec le bouton droit de la souris sur le dossier **create-search**, puis sélectionnez **Ouvrir dans le terminal intégré**.
15. Dans le volet terminal du dossier **create-search** , entrez la commande suivante pour exécuter le script de commandes.

    ```
    create-search
    ```

16. Une fois le script terminé, dans le portail Azure, sur la page de votre ressource Recherche cognitive Azure, sélectionnez la page **Indexeurs** et attendez que le processus d’indexation se termine.

    *Vous pouvez électionner **Actualiser** pour suivre la progression de l’opération d’indexation. Cette opération peut prendre environ une minute.*

## <a name="search-the-index"></a>Rechercher l’index

Maintenant que vous avez un index, vous pouvez y effectuer des recherches.

1. En haut de l’onglet de votre ressource Recherche cognitive Azure, sélectionnez **Explorateur de recherche**.
2. Dans l’Explorateur de recherche, dans la zone **Chaîne de requête**, entrez la chaîne de requête suivante, puis sélectionnez **Rechercher**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    Cette requête récupère l’**URL**, le **sentiment** et les **phrases clés** pour tous les documents qui mentionnent *Londres* créés par *Reviewer* qui ont une étiquette de **sentiment** positive (en d’autres termes, des critiques positives qui mentionnent Londres)

## <a name="create-an-azure-function-for-a-custom-skill"></a>Créer une fonction Azure pour une compétence personnalisée

La solution de recherche inclut un certain nombre de compétences cognitives intégrées qui enrichissent l’index avec des informations provenant des documents, telles que les scores de sentiment et les listes d’expressions clés vues dans la tâche précédente.

Vous pouvez améliorer davantage l’index en créant des compétences personnalisées. Par exemple, il peut être utile d’identifier les mots utilisés le plus fréquemment dans chaque document, mais aucune compétence intégrée n’offre cette fonctionnalité.

Pour implémenter la fonctionnalité de comptage de mots en tant que compétence personnalisée, vous allez créer une fonction Azure dans votre langage préféré.

> **Remarque** : Dans cet exercice, vous allez créer une fonction de Node.JS simple à l’aide des fonctionnalités de modification de code dans le Portail Azure. Dans une solution de production, vous utilisez généralement un environnement de développement tel que Visual Studio Code pour créer une application de fonction dans votre langage préféré (par exemple C#, Python, Node.JS ou Java) et le publier sur Azure dans le cadre d’un processus de DevOps.

1. Dans le portail Azure, dans la page **Accueil** , créez une ressource **Function App** avec les paramètres suivants :
    - **Abonnement** : *Votre abonnement*
    - **Groupe de ressources** : *Le même groupe de ressources que votre ressource Recherche cognitive Azure*
    - **Nom de l’application de fonction** : *Un nom unique*
    - **Publier** : Code
    - **Pile d’exécution** : Node.js
    - **Version** : 14 LTS
    - **Région** : *La même région que votre ressource Recherche cognitive Azure*

2. Attendez la fin du déploiement, puis accédez à la ressource Function App déployée.
3. Dans le volet de votre application de fonction, sur la gauche, sélectionnez l’onglet **Fonctions**. Créez ensuite une fonction avec les paramètres suivants :
    - **Configurer un environnement de développement**
        - **Environnement de développement** : Développer dans le portail
    - **Sélectionner un modèle**
        - **Modèle** : Déclencheur HTTP
    - **Détails du modèle** :
        - **Nouvelle fonction** : wordcount
        - **Niveau d’autorisation** : Fonction
4. Attendez que la fonction *wordcount* soit créée. Ensuite, dans sa page, sélectionnez l’onglet **Code + Test**.
5. Remplacez le code de fonction par défaut par le code suivant :

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. Enregistrez la fonction, puis ouvrez le volet **Test/Exécution**.
7. Dans le volet **Test/Exécution**, remplacez le paramètre **Body** existant par le code JSON suivant, qui reflète le schéma attendu par une compétence Recherche cognitive Azure dans laquelle les enregistrements contenant des données pour un ou plusieurs documents sont soumis pour traitement :

    ```
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```
    
8. Cliquez sur **Exécuter** et affichez le contenu de la réponse HTTP retourné par votre fonction. Celui-ci reflète le schéma attendu par Recherche cognitive Azure lors de l’utilisation d’une compétence, dans laquelle une réponse est renvoyée pour chaque document. Dans ce cas, la réponse se compose de 10 termes au maximum dans chaque document, dans l’ordre décroissant de leur fréquence d’apparition :

    ```
    {
    "values": [
        {
        "recordId": "a1",
        "data": {
            "text": [
            "tiger",
            "burning",
            "bright",
            "darkness",
            "night"
            ]
        },
        "errors": null,
        "warnings": null
        },
        {
        "recordId": "a2",
        "data": {
            "text": [
            "rain",
            "spain",
            "stays",
            "mainly",
            "plains",
            "thats",
            "youll",
            "find"
            ]
        },
        "errors": null,
        "warnings": null
        }
    ]
    }
    ```

9. Fermez le volet **Test/Exécution** et, dans le volet de la fonction **wordcount**, cliquez sur la fonction **Obtenir l’URL de la fonction**. Copiez ensuite l’URL de la clé par défaut dans le presse-papiers. Vous en aurez besoin dans la procédure suivante.

## <a name="add-the-custom-skill-to-the-search-solution"></a>Ajouter la compétence personnalisée à la solution de recherche

Vous devez maintenant inclure votre fonction en tant que compétence personnalisée dans l’ensemble de compétences de la solution de recherche et mapper les résultats qu’il produit à un champ dans l’index. 

1. Dans Visual Studio Code, dans le dossier **23-custom-search-skill/update-search**, ouvrez le fichier **update-skillset.json**. Ce fichier contient la définition JSON d’un ensemble de compétences.
2. Vérifiez la définition de l’ensemble de compétences. Celle-ci comprend les mêmes compétences qu’auparavant, ainsi qu’une nouvelle compétence **WebApiSkill** nommée **get-top-words**.
3. Modifiez la définition de la compétence **get-top-words** pour définir la valeur **uri** sur l’URL de votre fonction Azure (que vous avez copiée dans le presse-papiers dans la procédure précédente), en remplaçant **YOUR-FUNCTION-APP-URL**.
4. En haut de la définition de l’ensemble de compétences, dans l’élément **cognitiveServices**, remplacez l’espace réservé **YOUR_COGNITIVE_SERVICES_KEY** par l’une des clés de vos ressources Cognitive Services.

    *Vous trouverez les clés dans la page **Clés et point de terminaison** de votre ressource Cognitive Services dans le portail Azure.*

5. Enregistrez et fermez le fichier JSON mis à jour.
6. Dans le dossier **update-search** , ouvrez **update-index.json**. Ce fichier contient la définition JSON de **l’index margies-custom-index**, avec un champ supplémentaire nommé **top_words** en bas de la définition d’index.
7. Passez en revue le JSON de l’index, puis fermez le fichier sans apporter de modifications.
8. Dans le dossier **update-search**, ouvrez **update-indexer.json**. Ce fichier contient une définition JSON pour le **margies-custom-indexer**, avec un mappage supplémentaire pour le champ **top_words** .
9. Passez en revue le JSON de l’indexeur, puis fermez le fichier sans apporter de modifications.
10. Dans le dossier **update-search**, ouvrez **update-search.cmd**. Ce script de traitement par lots utilise l’utilitaire cURL pour envoyer les définitions JSON mises à jour à l’interface REST de votre ressource Recherche cognitive Azure.
11. Remplacez les espaces réservés des variables **YOUR_SEARCH_URL** et **YOUR_ADMIN_KEY** par l’**URL** et l’une des **clés d’administration** de votre ressource Recherche cognitive Azure.

    *Vous trouverez ces valeurs dans les pages **Vue d’ensemble** et **Clés** de votre ressource Recherche cognitive Azure dans le portail Azure.*

12. Enregistrez le fichier de commandes mis à jour.
13. Cliquez avec le bouton droit de la souris sur le dossier **update-search**, puis sélectionnez **Ouvrir dans le terminal intégré**.
14. Dans le volet terminal du dossier **update-search**  entrez la commande suivante pour exécuter le script de commandes.

    ```
    update-search
    ```

15. Une fois le script terminé, dans le portail Azure, sur la page de votre ressource Recherche cognitive Azure, sélectionnez la page **Indexeurs** et attendez que le processus d’indexation se termine.

    *Vous pouvez électionner **Actualiser** pour suivre la progression de l’opération d’indexation. Cette opération peut prendre environ une minute.*

## <a name="search-the-index"></a>Rechercher l’index

Maintenant que vous avez un index, vous pouvez y effectuer des recherches.

1. En haut de l’onglet de votre ressource Recherche cognitive Azure, sélectionnez **Explorateur de recherche**.
2. Dans l’Explorateur de recherche, dans la zone **Chaîne de requête**, entrez la chaîne de requête suivante, puis sélectionnez **Rechercher**.

    ```
    search=Las Vegas&$select=url,top_words
    ```

    Cette requête récupère les champs **URL** et **top_words** pour tous les documents qui mentionnent *Las Vegas*.

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur la Recherche cognitive Azure, consultez la [documentation Recherche cognitive Azure](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface).
