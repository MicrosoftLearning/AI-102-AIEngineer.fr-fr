---
lab:
  title: Créer une application de Language Understanding
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: d8a32a2b6404e81d8a5d69cef874fad209de1bfc
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195621"
---
# <a name="create-a-language-understanding-app"></a>Créer une application de Language Understanding

Le service Language Understanding vous permet de définir une application qui encapsule un modèle de langage que les applications peuvent utiliser pour interpréter l’entrée en langage naturel des utilisateurs, prédire l’*intention* des utilisateurs (ce qu’ils veulent obtenir) et identifier toutes les *entités* auxquelles l’intention doit être appliquée.

Par exemple, on peut s'attendre à ce qu'une application de compréhension de la langue pour une application d'horloge traite des entrées telles que :

*Quelle heure est-il à Londres ?*

Ce type d’entrée est un exemple d’*énoncé* (quelque chose qu’un utilisateur peut dire ou taper), pour lequel l’*intention* souhaitée est d’obtenir le temps dans un emplacement spécifique (une *entité*) ; dans ce cas, Londres.

> **Remarque** : La tâche de l’application de compréhension du langage consiste à prédire l’intention de l’utilisateur et à identifier toutes les entités auxquelles l’intention s’applique. Ce <u>n’est pas</u> son travail d’effectuer réellement les actions nécessaires pour satisfaire l’intention. Par exemple, l’application d’horloge peut utiliser une application linguistique pour discerner que l’utilisateur souhaite connaître l’heure à Londres ; mais l’application cliente elle-même doit ensuite implémenter la logique pour déterminer le bon temps et la présenter à l’utilisateur.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour cette formation

Si vous n’avez pas déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement où vous travaillez sur ce laboratoire, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-language-understanding-resources"></a>Créer des ressources Language Understanding

Pour utiliser le service Language Understanding, vous avez besoin de deux types de ressources :

- Une ressource *création* : utilisée pour définir, entraîner et tester l’application de compréhension du langage. Il doit s’agir d’une ressource **Language Understanding - Authoring** dans votre abonnement Azure.
- Une ressource de *prédiction* : utilisée pour publier l’application de compréhension du langage et traiter les demandes des applications clientes qui l’utilisent. Il peut s’agir d’une ressource **Language Understanding** ou **Cognitive Services** dans votre abonnement Azure.

     > **Important** : Les ressources de création doivent être créées dans l’une des trois *régions* (Europe, Australie ou États-Unis). Les applications Language Understanding créés dans les ressources de création européennes ou australiennes ne peuvent être déployés que vers des ressources de prédiction en Europe ou en Australie respectivement ; les modèles créés dans les ressources de création américaines peuvent être déployés vers des ressources de prédiction dans tout emplacement Azure autre que l’Europe et l’Australie. Consultez la [documentation sur les régions de création et de publication](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions) pour de plus amples détails sur la correspondance entre les emplacements de création et de prédiction.

Si vous ne disposez pas déjà de ressources de création et de prédiction Language Understanding :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Language Understanding*, puis créez une ressource **Language Understanding** avec les paramètres suivants.

    *Vérifiez que vous sélectionnez **Language Understanding**, <u>pas</u> Language Understanding (Azure Cognitive Services)*

    - **Options de création** : Les deux
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Nom** : *Entrez un nom unique.*
    - **Emplacement de création** : *Sélectionnez l’emplacement de votre choix.*
    - **Création - Niveau tarifaire** : F0
    - **Emplacement de prédiction** : *Le même que votre emplacement de création*
    - **Niveau tarifaire de prédiction** : F0
3. Attendez que les ressources soient créées. Notez que deux ressources Language Understanding sont provisionnées : une pour la création et une autre pour la prédiction. Vous pouvez voir ces deux ressources en accédant au groupe de ressources dans lequel vous les avez créées. Si vous sélectionnez **Accéder à la ressource**, cela ouvre la ressource de *création*.

## <a name="create-a-language-understanding-app"></a>Créer une application Language Understanding

Maintenant que vous avez créé une ressource de création, vous pouvez l’utiliser pour créer une application Language Understanding.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Language Understanding à l’adresse `https://www.luis.ai`.
2. Connectez-vous avec le compte Microsoft associé à votre abonnement Azure. Si vous vous connectez au portail Language Understanding pour la première fois, il se peut que vous deviez accorder à l’application des autorisations pour accéder aux détails de votre compte. Suivez alors les étapes de *Bienvenue* en sélectionnant votre abonnement Azure et la ressource de création que vous venez de créer.

    > **Remarque** : Si votre compte est associé à plusieurs abonnements dans différents répertoires, vous devrez peut-être basculer vers le répertoire contenant l’abonnement dans lequel vous avez approvisionné vos ressources Language Understanding.

3. Ouvrez la page **Applications de conversation** et assurez-vous que votre abonnement et votre ressource de création Language Understanding sont sélectionnés. Créez ensuite une nouvelle application de conversation avec les paramètres suivants :
    - **Nom** : Horloge
    - **Culture** : Anglais (*si cette option n’est pas disponible, laissez-la vide*)
    - **Description** : Horloge de langage naturel
    - **Ressource de prédiction** : *votre ressource de prédiction Language Understanding*

    Si votre application **Horloge** n’est pas ouverte automatiquement, ouvrez-la.
    
    Si un panneau contenant des conseils pour la création d’une application de Language Understanding efficace s’affiche, fermez-le.

## <a name="create-intents"></a>Créer des intentions

La première chose que nous allons faire dans la nouvelle application consiste à définir certaines intentions.

1. Dans la page **Intentions**, sélectionnez **&#65291; Créer** pour créer une intention nommée **GetTime**.
2. Dans l’intention **GetTime**, ajoutez les énoncés suivants comme exemple d’entrée utilisateur :

    *il est quelle heure ?*

    *quelle heure est-il ?*

3. Une fois que vous avez ajouté ces énoncés, revenez à la page **Intentions** et ajoutez une autre nouvelle intention nommée **GetDay** avec les énoncés suivants :

    *on est quel jour ?*

    *quel jour sommes-nous ?*

4. Une fois que vous avez ajouté ces énoncés, revenez à la page **Intentions** et ajoutez une autre nouvelle intention nommée **GetDate** avec les énoncés suivants :

    *qu’est-ce que la date aujourd’hui ?*

    *quelle est la date ?*

5. Une fois que vous avez ajouté ces énoncés, revenez à la page **Intentions** et sélectionnez l’intention **None**. Il s’agit d’un secours pour l’entrée qui ne correspond à aucune des intentions que vous avez définies dans votre modèle de langage.
6. Ajoutez les énoncés exemples suivants à l’intention **None** :

    *hello*

    *goodbye*

## <a name="train-and-test-the-app"></a>Entraîner et tester l’application

Maintenant que vous avez ajouté certaines intentions, nous allons entraîner l’application et voir si elle peut correctement les prédire à partir de l’entrée utilisateur.

1. En haut à droite du portail, sélectionnez **Entraîner** pour entraîner l’application.
2. Lorsque l’application est entraînée, sélectionnez **Test** pour afficher le panneau Test, puis entrez l’énoncé de test suivant :

    *quelle heure est-il maintenant ?*

    Passez en revue le résultat retourné, en notant qu’il inclut l’intention prédite (qui doit être **GetTime**) et un score de confiance qui indique la probabilité calculée par le modèle pour l’intention prédite.

3. Essayez l’énoncé de test suivant :

    *donnez moi l’heure*

    Là encore, passez en revue l’intention prédite et le score de confiance.

4. Essayez l’énoncé de test suivant :

    *qu’est-ce qui est aujourd’hui ?*

    Espérons que le modèle prédit l’intention **GetDay**.

5. Enfin, essayez cet énoncé de test :

    *salut*

    Cela doit renvoyer l’intention **None**.

6. Fermez le panneau Test.

## <a name="add-entities"></a>Ajouter des entités

Jusqu’à présent, vous avez défini des énoncés simples qui mappent aux intentions. La plupart des applications réelles incluent des énoncés plus complexes à partir desquels des entités de données spécifiques doivent être extraites pour obtenir plus de contexte pour l’intention.

### <a name="add-a-machine-learned-entity"></a>Ajoutez une entité *issue du machine learning*

Le type d’entité le plus courant est une entité *issue du machine learning*, dans laquelle l’application apprend à identifier les valeurs d’entité en fonction d’exemples.

1. Dans la page **Entités** , sélectionnez **&#65291; Créer** pour créer une entité.
2. Dans la boîte de dialogue **Créer une entité** qui s’affiche, créez une entité **Issue du machine learning** nommée **Emplacement**.
3. Une fois l’entité **Emplacement** créée, revenez à la page **Intentions** et sélectionnez l’intention **GetTime**.
4. Entrez le nouvel exemple d’énoncé suivant :

    *quelle heure est-il à Londres ?*

5. Lorsque l’énoncé a été ajouté, sélectionnez le mot ***londres** _, puis dans la liste déroulante qui s’affiche, sélectionnez _ *Emplacement** pour indiquer que « londres » est un exemple d’emplacement.
6. Ajoutez un autre exemple d’énoncé :

    *quelle est l’heure actuelle à New York ?*

7. Lorsque l’énoncé a été ajouté, sélectionnez les mots ***new york** _, puis mappez-les à l’entité _ *Emplacement**.

### <a name="add-a-list-entity"></a>Ajouter une entité de *liste*

Dans certains cas, les valeurs valides pour une entité peuvent être limitées à une liste de termes et de synonymes spécifiques, qui peut aider l’application à identifier les instances de l’entité dans les énoncés.

1. Dans la page **Entités** , sélectionnez **&#65291; Créer** pour créer une entité.
2. Dans la boîte de dialogue **Créer une entité**, créez une entité **Liste** nommée **Jour de la semaine**.
3. Ajoutez les **Valeurs normalisées** et les **synonymes** suivants :

    | Valeurs normalisées | Synonymes|
    |-------------------|---------|
    | sunday | Dim |
    | monday | Lun |
    | tuesday | Mars |
    | wednesday | Mer |
    | thursday | Jeu |
    | friday | Ven |
    | saturday | Sam |

3. Une fois l’entité **Jour de la semaine** créée, revenez à la page **Intentions** et sélectionnez l’intention **GetDate**.
4. Entrez le nouvel exemple d’énoncé suivant :

    *quelle date était-ce samedi ?*

5. Lorsque l’énoncé a été ajouté, vérifiez que **samedi** a été automatiquement mappé à l’entité **Week-end**. Si ce n’est pas le cas, sélectionnez le mot **_samedi_ *_, puis, dans la liste déroulante qui s’affiche, sélectionnez _* Jour de la semaine**.
6. Ajoutez un autre exemple d’énoncé :

    *quelle date sera-t-il vendredi ?*

7. Lorsque l’énoncé a été ajouté, vérifiez que **vendredi** est mappé à l’entité **Jour de la semaine**.

### <a name="add-a-regex-entity"></a>Ajouter une entité *Regex*

Parfois, les entités ont un format spécifique, tel qu’un numéro de série, un code de formulaire ou une date. Vous pouvez définir une expression régulière (*regex*) qui décrit un format attendu pour aider votre application à identifier les valeurs d’entité correspondantes.

1. Dans la page **Entités** , sélectionnez **&#65291; Créer** pour créer une entité.
2. Dans la boîte de dialogue **Créer une entité** , créez une entité **Regex** nommée **Date** avec l’expression régulière suivante :

    ```
    [0-9]{2}/[0-9]{2}/[0-9]{4}
    ```

    > **Remarque** : Il s’agit d’une expression régulière simple qui recherche deux chiffres suivis d’un « / », de deux autres chiffres, d’un autre « / » et de quatre chiffres (par exemple *01/11/2020*). Il autorise les dates non valides, telles que *56/00/9999* ; mais il est important de se rappeler que l’entité regex est utilisée pour identifier l’entrée de données *prévue* comme une date - pas pour valider les valeurs de date.

3. Une fois l’entité **Date** créée, revenez à la page **Intentions** et sélectionnez l’intention **GetDay**.
4. Entrez le nouvel exemple d’énoncé suivant :

    *quel jour était le 01/01/1901 ?*

5. Lorsque l’énoncé a été ajouté, vérifiez que **01/01/1901** a été automatiquement mappé à l’entité **Date**. Si ce n’est pas le cas, sélectionnez **_01/01/1901_ *_, puis dans la liste déroulante qui s’affiche, sélectionnez _* Date**.
6. Ajoutez un autre exemple d’énoncé :

    *quel jour sera-t-il le 12/12/2099 ?*

7. Lorsque l’énoncé a été ajouté, vérifiez que **12/12/2099** est mappé à l’entité **Date**.

### <a name="retrain-the-app"></a>Entraîner à nouveau l’application

Maintenant que vous avez modifié le modèle de langage, vous devez réentraîner et retester l’application.

1. En haut à droite du portail, sélectionnez **Entraîner** pour entraîner à nouveau l’application.
2. Lorsque l’application est entraînée, sélectionnez **Test** pour afficher le panneau Test, puis entrez l’énoncé de test suivant :

    *quelle heure est-il à Édimbourg ?*

3. Passez en revue le résultat retourné, ce qui devrait prédire l’intention **GetTime**. Sélectionnez ensuite **Inspecter** et dans le panneau d’inspection supplémentaire qui s’affiche, examinez la section **Entités ML**. Le modèle doit avoir prédit que « édimbourg » est une instance d’une entité **Emplacement**.
4. Essayez de tester les énoncés suivants :

    *quelle date sera-t-il vendredi ?*

    *qu’est-ce que la date Jeu ?*

    *quel était le jour le 01/01/2020 ?*

5. Lorsque vous avez terminé les tests, fermez le panneau d’inspection, mais laissez le panneau de test ouvert.

## <a name="perform-batch-testing"></a>Effectuer des tests par lots

Vous pouvez utiliser le volet de test pour tester des énoncés individuels de manière interactive, mais pour les modèles de langage plus complexes, il est généralement plus efficace d’effectuer des *tests des lots*.

1. Dans Visual Studio Code, ouvrez le fichier **batch-test.json** dans le dossier **09-luis-app**. Ce fichier se compose d’un document JSON qui contient plusieurs cas de test pour le modèle de langage d’horloge que vous avez créé.
2. Dans le portail Language Understanding, dans le volet de test, sélectionnez **Volet de test des lots**. Sélectionnez ensuite **&#65291; Importer** et importer le fichier **batch-test.json**, en affectant le nom **clock-test**.
3. Dans le volet de test des lots, exécutez le test **clock-test**.
4. Une fois le test terminé, sélectionnez **Afficher les résultats**.
5. Dans la page résultats, affichez la matrice de confusion qui représente les résultats de prédiction. Elle affiche des prédictions positives, faux positifs, négatives et faux négatifs pour l’intention ou l’entité sélectionnée dans la liste à droite.

    ![Matrice de confusion pour un test par lot de compréhension du langage](./images/luis-confusion-matrix.jpg)

    > **Remarque** : Chaque énoncé est marqué comme *positif* ou *négatif* pour chaque intention , par exemple « quelle heure est-il ? » doit être marqué comme *positif* pour l’intention **GetTime** et *négatif* pour l’intention **GetDate**. Les points de la matrice de confusion montrent quels énoncés ont été prédits correctement (*true*) et incorrectement (*false*) comme *positifs* et *négatifs* pour l’intention sélectionnée.

6. Avec l’intention **GetDate** sélectionnée, sélectionnez l’un des points de la matrice de confusion pour afficher les détails de la prédiction, y compris l’énoncé et le score de confiance pour la prédiction. Sélectionnez ensuite les intentions **GetDay**, **GetTime** et **None** et affichez leurs résultats de prédiction. L’application doit avoir réussi à prédire les intentions correctement.

    > **Remarque** : L’interface utilisateur peut ne pas effacer les points sélectionnés précédemment.

7. Sélectionnez l’entité **Emplacement** et affichez les résultats de prédiction dans la matrice de confusion. Notez en particulier les prédictions qui étaient des *faux négatifs* : il s’agissait de cas où l’application n’a pas pu détecter l’emplacement spécifié dans l’énoncé, indiquant que vous devrez peut-être ajouter d’autres exemples d’énoncés aux intentions et réentraîner le modèle.
8. Fermez le volet de test des lots.

## <a name="publish-the-app"></a>Publier l’application

Dans un projet réel, vous affineriez de manière itérative les intentions et les entités, entraîneriez à nouveau et retesteriez jusqu’à ce que vous soyez satisfait des performances prédictives. Ensuite, vous pouvez publier l’application pour les applications clientes à utiliser.

1. En haut à droite du portail Language Understanding, sélectionnez **Publier**.
2. Sélectionnez **Emplacement de production** et publiez l’application.
3. Une fois la publication terminée, en haut du portail Language Understanding, sélectionnez **Gérer**.
4. Dans la page **Paramètres**, notez l’**ID de l’application**. Les applications clientes ont besoin de cela pour utiliser votre application.
5. Sur la page **Ressources Azure**, notez la **Clé primaire**, la **Clé secondaire** et l’**URL du point de terminaison** pour la ressource de prédiction via laquelle l’application peut être consommée. Les applications clientes ont besoin du point de terminaison et de l’une des clés pour se connecter à la ressource de prédiction et être authentifiées.
6. Dans Visual Studio Code, dans le dossier **09-luis-app**, sélectionnez le fichier batch **GetIntent.cmd** et affichez le code qu’il contient. Ce script de ligne de commande utilise cURL pour appeler l’API REST Language Understanding pour le point de terminaison d’application et de prédiction spécifié.
7. Remplacez les valeurs d’espace réservé dans le script par l’**ID d’application**, l’**URL du point de terminaison** et la **Clé primaire** ou la **Clé secondaire** de votre application Language Understanding, puis enregistrez le fichier mis à jour.
8. Cliquez avec le bouton droit de la souris sur le dossier **09-luis-app** et ouvrez un terminal intégré. Entrez ensuite la commande suivante (veillez à inclure les guillemets !) :

    ```
    GetIntent "What's the time?"
    ```

9. Passez en revue la réponse JSON retournée par votre application, qui doit indiquer l’intention de scoring supérieure prédite pour votre entrée (qui doit être **GetTime**).
10. Essayez la commande suivante :

    ```
    GetIntent "What's today's date?"
    ```

11. Examinez la réponse et vérifiez qu’elle prédit l’intention **GetDate**.
12. Essayez la commande suivante :

    ```
    GetIntent "What time is it in Sydney?"
    ```

13. Examinez la réponse et vérifiez qu’elle inclut une entité **Emplacement**.

14. Essayez les commandes suivantes et examinez les réponses :

    ```
    GetIntent "What time is it in Glasgow?"
    ```

    ```
    GetIntent "What's the time in Nairobi?"
    ```

    ```
    GetIntent "What's the UK time?"
    ```
15. Essayez quelques variantes supplémentaires : l’objectif est de générer au moins des réponses qui prédit correctement l’intention **GetTime**, mais qui ne parviennent pas à détecter une entité **Emplacement**.

    Gardez le terminal ouvert. Nous y reviendrons sous peu.

## <a name="apply-active-learning"></a>Appliquer l’*apprentissage actif*

Vous pouvez améliorer une application Language Understanding en fonction des énoncés historiques soumis au point de terminaison. Cette pratique est appelée *apprentissage actif*.

Dans la procédure précédente, vous avez utilisé cURL pour envoyer des demandes au point de terminaison de votre application. Ces demandes incluent l’option permettant de consigner les requêtes, ce qui permet à l’application de les suivre pour une utilisation dans l’apprentissage actif.

1. Dans le portail Language Understanding, sélectionnez **Générer** et afficher la page **Révision des énoncés de point de terminaison**. Cette page répertorie les énoncés journalisés que le service a marqué pour révision.
2. Pour les énoncés pour lesquels l’intention et une nouvelle entité d’emplacement (qui n’ont pas été incluses dans les énoncés d’apprentissage d’origine) sont correctement prédites, sélectionnez **&#10003;** pour confirmer l’entité, puis utilisez l’icône **&#10514;** pour ajouter l’énoncé à l’intention comme exemple d’apprentissage.
3. Recherchez un exemple d’énoncé dans lequel l’intention **GetTime** a été correctement identifiée, mais une entité **Emplacement** <u>n’a pas</u> été identifiée, et sélectionnez le nom de l’emplacement et mappez-le à l’entité **Emplacement**. Utilisez ensuite l’icône **&#10514;** pour ajouter l’énoncé à l’intention comme exemple d’apprentissage.
4. Accédez à la page **Intentions** et ouvrez l’intention **GetTime** pour confirmer que les énoncés suggérés ont été ajoutés.
5. En haut du portail Language Understanding, sélectionnez **Entraîner** pour réentraîner l’application.
6. En haut à droite du portail Language Understanding, sélectionnez **Publier** et republiez l’application dans **Emplacement de production**.
7. Revenez au terminal du dossier **09-luis-app** et utilisez la commande **GetIntent** pour envoyer l’énoncé que vous avez ajouté et corrigé pendant l’apprentissage actif.
8. Vérifiez que le résultat inclut désormais l’entité **Emplacement**. Essayez ensuite un autre énoncé qui utilise la même formulation, mais spécifie un autre emplacement (par exemple, *Berlin*).

## <a name="export-the-app"></a>Exporter l’application

Vous pouvez utiliser le portail Language Understanding pour développer et tester votre application de langage, mais dans un processus de développement logiciel pour DevOps, vous devez conserver une définition contrôlée par la source de l’application qui peut être incluse dans les pipelines d’intégration continue et de livraison (CI/CD). Bien que vous *puissiez* utiliser le SDK Language Understanding ou l’API REST dans des scripts de code pour créer et entraîner l’application, il est plus simple d’utiliser le portail pour créer l’application et de l’exporter en tant que fichier *.lu* qui peut être importé et réentraîné dans une autre instance Language Understanding. Cette approche vous permet d’utiliser les avantages de productivité du portail tout en conservant la portabilité et la reproductibilité de l’application.

1. Dans le portail Language Understanding, sélectionnez **Gérer**.
2. Dans la page **Versions**, sélectionnez la version actuelle de l’application (il ne doit y en avoir qu’une).
3. Dans la liste déroulante **Exporter** , sélectionnez **Exporter en tant que LU**. Ensuite, lorsque vous y êtes invité, enregistrez le fichier dans le dossier **09-luis-app**.
4. Dans Visual Studio Code, ouvrez le fichier **.lu** que vous venez d’exporter et télécharger (si vous êtes invité à rechercher le marketplace pour une extension qui peut la lire, ignorez l’invite). Notez que le format LU est lisible par l’homme, ce qui permet de documenter efficacement la définition de votre application Language Understanding dans un environnement de développement d’équipe.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur l’utilisation du service **Language Understanding**, consultez la [documentation Language Understanding](https://docs.microsoft.com/azure/cognitive-services/luis/).
