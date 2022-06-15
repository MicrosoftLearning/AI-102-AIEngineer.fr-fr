---
lab:
  title: Créer un modèle de compréhension du langage avec le service Langage (préversion)
ms.openlocfilehash: 92d556e50c8f5c827e16f1e5ad301d3b59c1ad6e
ms.sourcegitcommit: cb2edc850cec8390a81212d9b8f6a279d2f42b4d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/04/2022
ms.locfileid: "145195629"
---
# <a name="create-a-language-understanding-model-with-the-language-service-preview"></a>Créer un modèle de compréhension du langage avec le service Langage (préversion)

> **Remarque** : La fonctionnalité de compréhension du langage courant du service Langage est actuellement en préversion et peut être modifiée. Dans certains cas, l’entraînement du modèle peut échouer. Si cela se produit, réessayez.  

Le service Langage vous permet de définir un *modèle de compréhension du langage courant* que les applications peuvent utiliser pour interpréter l’entrée en langage naturel des utilisateurs, prédire l’*intention* des utilisateurs (les résultats souhaités) et identifier toutes les *entités* auxquelles l’intention doit être appliquée.

Par exemple, on peut s’attendre à ce qu’un modèle de langage courant pour une application d’horloge traite des entrées telles que :

*Quelle heure est-il à Londres ?*

Ce type d’entrée est un exemple d’*énoncé* (quelque chose qu’un utilisateur peut dire ou type), pour lequel l’*intention* souhaitée est d’obtenir l’heure pour un emplacement spécifique (une *entité*) ; dans ce cas, Londres.

> **Remarque** : La tâche d’un modèle de langage courant consiste à prédire l’intention de l’utilisateur et à identifier toutes les entités auxquelles l’intention s’applique. La tâche d’un modèle de langage courant n’est <u>pas</u> d’effectuer réellement les actions requises pour satisfaire l’intention. Par exemple, une application d’horloge peut utiliser un modèle de langage courant pour discerner que l’utilisateur souhaite connaître l’heure à Londres ; mais l’application cliente elle-même doit ensuite implémenter la logique pour déterminer la bonne heure et la présenter à l’utilisateur.

## <a name="create-a-language-service-resource"></a>Créer une ressource du service Language

Pour créer un modèle de langage courant, vous avez besoin d’une ressource du **service Langage** dans une région prise en charge. Au moment de l’écriture, seules les régions USA Ouest 2 et Europe Ouest sont prises en charge.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Langage*, puis créez une ressource du **service Langage** avec les paramètres suivants.

    - **Fonctionnalités** : Utilisez les fonctionnalités par défaut, qui incluent la compréhension du langage courant (préversion) 
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : USA Ouest 2 ou Europe Ouest
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Sélectionnez le niveau Standard (S) (la compréhension du langage courant n’est actuellement pas prise en charge sur le niveau Gratuit).
    - **Conditions juridiques** : _Accepter_ 
    - **Mention sur l’IA responsable** : _J’accepte_
3. Attendez la fin du déploiement, puis visualisez les détails du déploiement.

## <a name="create-a-conversational-language-understanding-project"></a>Créer un projet de compréhension du langage courant

Maintenant que vous avez créé une ressource de création, vous pouvez l’utiliser pour créer un projet de compréhension du langage courant.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Language Studio à l’adresse `https://language.cognitive.azure.com/`, puis connectez-vous en utilisant le compte Microsoft associé à votre abonnement Azure.

2. Si vous êtes invité à choisir une ressource de langue, sélectionnez les paramètres suivants :

    - **Azure Directory** : Annuaire Azure contenant votre abonnement.
    - **Abonnement Azure** : Votre abonnement Azure.
    - **Ressource de langue** : Ressource de langue que vous avez créée précédemment.

3. Si vous n’êtes <u>pas</u> invité à choisir une ressource de langage, cela peut être dû au fait que vous avez déjà attribué une ressource de langage différente, auquel cas :

    1. Dans la barre en haut de la page, cliquez sur le bouton **Paramètres (&#9881;)**.
    2. Dans la page **Paramètres**, affichez l’onglet **Ressources**.
    3. Sélectionnez la ressource de langue que vous venez de créer, puis cliquez sur **Changer de ressource**.
    4. En haut de la page, cliquez sur **Language Studio** pour retourner à la page d’accueil de Language Studio.

4. En haut du portail, dans le menu **Créer**, sélectionnez **Conversational Language Understanding**.

5. Dans la boîte de dialogue **Créer un projet**, dans la page **Entrer les informations de base**, entrez les détails suivants, puis cliquez sur **Suivant** :
    - **Nom** : `Clock`
    - **Description** : `Natural language clock`
    - **Langue principale des énoncés** : anglais
    - **Activer plusieurs langages dans le projet ?** : *Non sélectionné*

7. Dans la page **Vérifier et terminer**, cliquez sur **Créer**.

## <a name="create-intents"></a>Créer des intentions

La première chose que nous allons faire dans le nouveau projet consiste à définir certaines intentions.

> **Conseil** : Lorsque vous travaillez sur votre projet, si des conseils sont affichés, lisez-les et cliquez sur **J’ai compris** pour les ignorer, ou cliquez sur **Ignorer tout**.

1. Dans la page **Générer le schéma**, sous l’onglet **Intentions**, sélectionnez **&#65291; Ajouter** pour ajouter une nouvelle intention nommée **GetTime**.

2. Cliquez sur la nouvelle intention **GetTime** pour la modifier, ajoutez les énoncés suivants comme exemple d’entrée utilisateur :

    `what is the time?`

    `what's the time?`

    `what time is it?`

    `tell me the time`

3. Une fois les énoncés ajoutés, cliquez sur **Enregistrer les modifications** et revenez à la page **Générer le schéma**.

4. Ajoutez une autre nouvelle intention nommée **GetDay** avec les énoncés suivants :

    `what day is it?`

    `what's the day?`

    `what is the day today?`

    `what day of the week is it?`

5. Une fois ces énoncés ajoutés et enregistrés, revenez à la page **Générer le schéma** et ajoutez une autre nouvelle intention nommée **GetDate** avec les énoncés suivants :

    `what date is it?`

    `what's the date?`

    `what is the date today?`

    `what's today's date?`

6. Une fois les énoncés ajoutés, enregistrez-les et effacez le filtre **GetDate** sur la page Énoncés afin de voir tous les énoncés pour toutes les intentions.

## <a name="train-and-test-the-model"></a>Entraîner et tester le modèle

Maintenant que vous avez ajouté certaines intentions, nous allons entraîner le modèle de langage et voir s’il peut correctement les prédire à partir de l’entrée utilisateur.

1. Dans le volet de gauche, sélectionnez la page **Entraîner le modèle**, puis sélectionnez **Démarrer un travail d’entraînement**.

2. Dans la boîte de dialogue **Démarrer un travail d’entraînement**, sélectionnez l’option permettant d’entraîner un nouveau modèle, renommez-le **Horloge** et veillez à ce que l’option d’exécution de l’évaluation lors de l’entraînement soit activée. 

3. Pour commencer le processus d’entraînement de votre modèle, cliquez sur **Entraîner**.

4. Une fois l’entraînement terminé (ce qui peut prendre plusieurs minutes), l’**État** du travail passe à **Apprentissage réussi**.

5. Sélectionnez la page **Afficher les détails du modèle**, puis sélectionnez le modèle **Horloge**. Passez en revue les métriques d’évaluation globales et par intention (*précision*, *rappel* et *score F1*) et la *matrice de confusion* générée par l’évaluation effectuée lors de l’entraînement (notez qu’en raison du petit nombre d’exemples d’énoncés, toutes les intentions ne peuvent pas être incluses dans les résultats).

    >**Remarque** : Pour en savoir plus sur les métriques d’évaluation, consultez la [documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)

5. Dans la page **Déployer un modèle** , sélectionnez **Ajouter un déploiement**.

5. Dans la boîte de dialogue **Ajouter un déploiement**, sélectionnez **Créer un nom de déploiement**, puis entrez **production**

5. Sélectionnez le modèle **Horloge**, puis cliquez sur **Envoyer**. Ce déploiement peut prendre un certain temps.

6. Une fois le modèle déployé, dans la page **Modèle de test**, sélectionnez le modèle **Horloge**.

6. Sur la page **Modèle de test : Horloge**, dans la liste déroulante **Nom du déploiement**, sélectionnez **production**.  

7. Entrez le texte suivant, puis cliquez sur **Exécuter le test** :

    `what's the time now?`

    Passez en revue le résultat retourné, en notant qu’il inclut l’intention prédite (qui doit être **GetTime**) et un score de confiance qui indique la probabilité calculée par le modèle pour l’intention prédite. L’onglet JSON montre la confiance comparative pour chaque intention potentielle (celle qui a le score de confiance le plus élevé est l’intention prédite)

8. Effacez la zone de texte, puis exécutez un autre test avec le texte suivant :

    `tell me the time`

    Là encore, passez en revue l’intention prédite et le score de confiance.

9. Essayez le texte suivant :

    `what's the day today?`

    Espérons que le modèle prédit l’intention **GetDay**.

## <a name="add-entities"></a>Ajouter des entités

Jusqu’à présent, vous avez défini des énoncés simples qui correspondent aux intentions. La plupart des applications réelles incluent des énoncés plus complexes à partir desquels des entités de données spécifiques doivent être extraites pour obtenir plus de contexte pour l’intention.

### <a name="add-a-learned-entity"></a>Ajouter une entité *apprise*

Le type d’entité le plus courant est une entité *apprise*, dans laquelle le modèle apprend à identifier les valeurs d’entité en fonction d’exemples.

1. Dans Language Studio, revenez à la page **Générer le schéma**, puis sous l’onglet **Entités**, sélectionnez **&#65291; Ajouter** pour ajouter une nouvelle entité.

2. Dans la boîte de dialogue **Ajouter une entité**, entrez le nom d’entité **Emplacement** et vérifiez que **Learned** (Apprise) est sélectionné. Cliquez ensuite sur **Ajouter une entité**.

3. Une fois l’entité **Emplacement** créée, revenez à la page **Générer le schéma**, puis sous l’onglet **Intentions**, sélectionnez l’intention **GetTime**.

4. Entrez le nouvel exemple d’énoncé suivant :

    `what time is it in London?`

5. Une fois l’énoncé ajouté, sélectionnez le mot ***Londres** _, puis dans la liste déroulante qui s’affiche, sélectionnez _ *Emplacement** pour indiquer que « Londres » est un exemple d’emplacement.

6. Ajoutez un autre exemple d’énoncé :

    `Tell me the time in Paris?`

7. Une fois l’énoncé ajouté, sélectionnez le mot ***Paris** _, puis mappez-le à l’entité _ *Emplacement**.

8. Ajoutez un autre exemple d’énoncé :

    `what's the time in New York?`

9. Une fois l’énoncé ajouté, sélectionnez le mot ***New York** _, puis mappez-les à l’entité _ *Emplacement**.

10. Cliquez sur **Enregistrer les modifications** pour enregistrer les nouveaux énoncés.

### <a name="add-a-list-entity"></a>Ajouter une entité de *liste*

Dans certains cas, les valeurs valides pour une entité peuvent être limitées à une liste de termes et de synonymes spécifiques, qui peut aider l’application à identifier les instances de l’entité dans les énoncés.

1. Dans Language Studio, revenez à la page **Générer le schéma**, puis sous l’onglet **Entités**, sélectionnez **&#65291; Ajouter** pour ajouter une nouvelle entité.

2. Dans la boîte de dialogue **Ajouter une entité**, entrez le nom d’entité **Jour de la semaine** et sélectionnez le type d’entité **Liste**. Cliquez ensuite sur **Ajouter une entité**.

3. Dans la page de l’entité **Jour de la semaine**, dans la section **Liste**, cliquez sur **&#65291; Ajouter une nouvelle liste**. Entrez ensuite la valeur et le synonyme suivants, puis cliquez sur **Enregistrer** :

    | Clé de liste | Synonymes|
    |-------------------|---------|
    | Dimanche | Dim |

4. Répétez l’étape précédente pour ajouter les composants de liste suivants :

    | Valeur | Synonymes|
    |-------------------|---------|
    | Lundi | Lun |
    | Mardi | Ma, Mar |
    | Mercredi | Mer, Merc |
    | Jeudi | Je, Jeu |
    | Vendredi | Ven |
    | Samedi | Sam |

5. Revenez à la page **Générer le schéma**, puis sous l’onglet **Intentions**, sélectionnez l’intention **GetDate**.

6. Entrez le nouvel exemple d’énoncé suivant :

    `what date was it on Saturday?`

7. Une fois l’énoncé ajouté, sélectionnez le mot ***Samedi** _, puis dans la liste déroulante qui s’affiche, sélectionnez _*Jour de la semaine**.

8. Ajoutez un autre exemple d’énoncé :

    `what date will it be on Friday?`

9. Une fois l’énoncé ajouté, mappez **Vendredi** à l’entité **Jour de la semaine**.

10. Ajoutez un autre exemple d’énoncé :

    `what will the date be on Thurs?`

11. Une fois l’énoncé ajouté, mappez **Jeudi** à l’entité **Jour de la semaine**.

12. Cliquez sur **Enregistrer les modifications** pour enregistrer les nouveaux énoncés.

### <a name="add-a-prebuilt-entity"></a>Ajouter une entité *prédéfinie*

Le service Langage fournit un ensemble d’entités *prédéfinies* couramment utilisées dans les applications conversationnelles.

1. Dans Language Studio, revenez à la page **Générer le schéma**, puis sous l’onglet **Entités**, sélectionnez **&#65291; Ajouter** pour ajouter une nouvelle entité.

2. Dans la boîte de dialogue **Ajouter une entité**, entrez le nom d’entité **Date** et sélectionnez le type d’entité **Prédéfinie**. Cliquez ensuite sur **Ajouter une entité**.

3. Dans la page de l’entité **Date**, dans la section **Prédéfinie**, cliquez sur **&#65291; Ajouter une entité prédéfinie**.

4. Dans la liste **Sélectionner une entité prédéfinie**, sélectionnez **DateTime**, puis cliquez sur **Enregistrer**.

5. Revenez à la page **Générer le schéma**, puis sous l’onglet **Intentions**, sélectionnez l’intention **GetDay**.

6. Entrez le nouvel exemple d’énoncé suivant :

    `what day was 01/01/1901?`

7. Une fois l’énoncé ajouté, sélectionnez ***01/01/1901** _, puis dans la liste déroulante qui s’affiche, sélectionnez _*Date**.

8. Ajoutez un autre exemple d’énoncé :

    `what day will it be on Dec 31st 2099?`

9. Une fois l’énoncé ajouté, mappez **31 déc 2099** à l’entité **Date**.

10. Cliquez sur **Enregistrer les modifications** pour enregistrer les nouveaux énoncés.

### <a name="retrain-the-model"></a>Recycler le modèle

Maintenant que vous avez modifié le schéma, vous devez réentraîner et retester le modèle.

1. Dans la page **Entraîner le modèle**, sélectionnez **Démarrer un travail d’entraînement**.

1. Dans la boîte de dialogue **Démarrer un travail d’entraînement**, sélectionnez l’option pour remplacer un modèle existant et spécifiez le modèle **Horloge**. Vérifiez que l’option d’exécution de l’évaluation lors de l’entraînement est sélectionnée et cliquez sur **Entraîner** pour entraîner le modèle; confirmant que vous souhaitez remplacer le modèle existant.

2. Une l’entraînement terminé, l’**État** du travail est mis à jour vers **Entraînement réussi**. 

2. Sélectionnez la page **Afficher les détails du modèle**, puis sélectionnez le modèle **Horloge**. Passez en revue les métriques d’évaluation globales, par entité et par intention (*précision*, *rappel* et *score F1*) et la *matrice de confusion* générée par l’évaluation effectuée lors de l’entraînement (notez qu’en raison du petit nombre d’exemples d’énoncés, toutes les intentions ne peuvent pas être incluses dans les résultats).

3. Dans la page **Déployer un modèle** , sélectionnez **Ajouter un déploiement**.

3. Dans la boîte de dialogue **Ajouter un déploiement**, sélectionnez **Remplacer un nom de déploiement existant**, puis sélectionnez **production**.

3. Sélectionnez le modèle **Horloge**, puis cliquez sur **Envoyer** pour le déployer. Cette opération peut prendre un certain temps.

4. Une fois le modèle déployé, dans la page **Modèle de test**, sélectionnez le modèle **Horloge**, sélectionnez le déploiement **production**, puis testez-le avec le texte suivant :

    `what's the time in Edinburgh?`

5. Passez en revue le résultat retourné, qui devrait prédire l’intention **GetTime** et une entité **Emplacement** avec la valeur de texte « Édimbourg ».

6. Essayez de tester les énoncés suivants :

    `what time is it in Tokyo?`
    
    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## <a name="use-the-model-from-a-client-app"></a>Utiliser le modèle à partir d’une application cliente

Dans un projet réel, vous affineriez de manière itérative les intentions et les entités, entraîneriez à nouveau et retesteriez jusqu’à ce que vous soyez satisfait des performances prédictives. Ensuite, une fois que vous auriez testé le modèle et que vous seriez satisfait de ses performances prédictives, vous pourriez l’utiliser dans une application cliente en appelant son interface REST. Dans cet exercice, vous allez utiliser l’utilitaire *curl* pour appeler le point de terminaison REST de votre modèle.

1. Dans Language Studio, dans la page **Déployer un modèle**, sélectionnez le modèle **Horloge**. Puis cliquez sur **Obtenir l’URL de prédiction**.

2. Dans la boîte de dialogue **Obtenir l’URL de prédiction**, notez que l’URL du point de terminaison de prédiction est affichée avec un exemple de demande, qui se compose d’une commande **curl** qui envoie une requête HTTP POST au point de terminaison, en spécifiant la clé de votre ressource de langage dans l’en-tête et en incluant une requête et un langage dans les données de requête.

3. Copiez l’exemple de demande et collez-le dans votre éditeur de texte préféré (par exemple Bloc-notes).

4. Remplacez les espaces réservés suivants :
    - **YOUR_QUERY_HERE** : *Quelle heure est-il à Sydney*
    - **QUERY_LANGUAGE_HERE** : *EN*

    La commande doit ressembler au code suivant :

    ```
    curl -X POST "https://some-name.cognitiveservices.azure.com/language/:analyze-conversations?projectName=Clock&deploymentName=production&api-version=2021-11-01-preview" -H "Ocp-Apim-Subscription-Key: 0ab1c23de4f56..."  -H "Apim-Request-Id: 9zy8x76wv5u43...." -H "Content-Type: application/json" -d "{\"verbose\":true,\"query\":\"What's the time in Sydney?\",\"language\":\"EN\"}"
    ```

5. Ouvrez une invite de commandes (Windows) ou un interpréteur de commandes bash (Linux/Mac).

6. Copiez et collez la commande curl modifiée dans votre interface de ligne de commande et exécutez-la.

7. Affichez le code JSON résultant, qui doit inclure l’intention et les entités prédites, comme suit :

    ```
    {"query":"What's the time in Sydney?","prediction":{"topIntent":"GetTime","projectKind":"conversation","intents":[{"category":"GetTime","confidenceScore":0.9998859},{"category":"GetDate","confidenceScore":9.8372206E-05},{"category":"GetDay","confidenceScore":1.5763446E-05}],"entities":[{"category":"Location","text":"Sydney","offset":19,"length":6,"confidenceScore":1}]}}
    ```

8. Passez en revue la réponse JSON retournée par votre modèle pour vous assurer que l’intention avec le score le plus élevé prédite est **GetTime**.

9. Remplacez la requête dans la commande curl par `What's today's date?`, puis exécutez-la et passez en revue le code JSON résultant.

10. Essayez les requêtes suivantes :

    `What day will Jan 1st 2050 be?`

    `What time is it in Glasgow?`

    `What date will next Monday be?`

## <a name="export-the-project"></a>Exporter le projet

Vous pouvez utiliser Language Studio pour développer et tester votre modèle de compréhension du langage, mais dans un processus de développement logiciel pour DevOps, vous devez conserver une définition contrôlée par la source du projet qui peut être incluse dans les pipelines d’intégration et de livraison continues (CI/CD). Bien que vous *puissiez* utiliser l’API REST Language dans des scripts de code pour créer et entraîner le modèle, il est plus simple d’utiliser le portail pour créer le schéma du modèle et de l’exporter en tant que fichier *.json* qui peut être importé et réentraîné dans une autre instance du service Langage. Cette approche vous permet d’utiliser les avantages de productivité de l’interface visuelle Language Studio tout en conservant la portabilité et la reproductibilité du modèle.

1. Dans Language Studio, sur la page **Projets**, sélectionnez le projet **Horloge**. Ne cliquez pas sur **Horloge**, sélectionnez l’icône de cercle pour sélectionner le projet Horloge.

2. Cliquez sur le bouton **&#x2913; Exporter**.

3. Enregistrez le fichier **Clock.json** généré (à l’emplacement de votre choix).

4. Ouvrez le fichier téléchargé dans votre éditeur de code favori (par exemple, Visual Studio Code) pour passer en revue la définition JSON de votre projet.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur l’utilisation du service **Langage** pour créer des solutions de compréhension du langage courant, consultez la [documentation du service Langage](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/overview).
