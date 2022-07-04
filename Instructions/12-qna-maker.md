---
lab:
  title: Créer une solution de réponses aux questions
  module: Module 6 - Building a QnA Solution
ms.openlocfilehash: 42b3b6e6955188d3d442a4f587ead6afe0f389c6
ms.sourcegitcommit: a879eae298aaf2ab6415fc4fdb67f52f592d907a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2022
ms.locfileid: "146902479"
---
# <a name="create-a-question-answering-solution"></a>Créer une solution de réponses aux questions

L’un des scénarios conversationnels les plus courants consiste à fournir une prise en charge par le biais d’un base de connaissances de questions fréquentes (FAQ). De nombreuses organisations publient les FAQ sous forme de documents ou de pages web, ce qui fonctionne bien pour un petit ensemble de paires de questions et de réponses, mais les documents volumineux peuvent être difficiles à consulter et prendre du temps.

Le service **Langue** comprend une fonction de *réponse aux questions* qui vous permet de créer une base de connaissances de paires de questions et de réponses qui peuvent être interrogées à l'aide d'une entrée en langue naturel. Il est le plus souvent utilisé comme une ressource qu'un robot peut utiliser pour rechercher les réponses aux questions soumises par les utilisateurs.

> **Remarque** : La fonctionnalité de réponses aux questions dans Language Service est une version plus récente du service QnA Maker, qui est toujours disponible en tant que service distinct.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous n’avez pas déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement où vous travaillez sur ce laboratoire, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-a-language-resource"></a>Créer une ressource Langage

Pour créer et héberger un base de connaissances pour répondre aux questions, vous avez besoin d’une ressource de **service de langue** dans votre abonnement Azure.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Langue*, puis créez une ressource de **service de langue**.
3. Cliquez sur **Sélectionner** sur le bloc **Réponses aux questions personnalisées**. Cliquez ensuite sur **Continuez pour créer votre ressource**. Vous devez entrer les paramètres suivants :
    
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quel emplacement disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S
    - **Emplacement de la Recherche Azure**\* : *Choisissez un emplacement dans la même région du monde que votre ressource linguistique*.
    - **Niveau tarifaire Recherche Azure** : Gratuit (F) (*Si ce niveau n'est pas disponible, sélectionnez Basique (B)* )
    - **Conditions légales** : _J’accepte_ 
    - **Mention sur l’IA responsable** : _J’accepte_
    
    \*Les réponses aux questions personnalisées utilisent Recherche Azure pour indexer et interroger le base de connaissances des questions et réponses.

4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.

## <a name="create-a-question-answering-project"></a>Créer un projet de réponses aux questions

Pour créer une base de connaissances pour répondre aux questions dans votre ressource de language, vous pouvez utiliser le portail Language Studio pour créer un projet de réponse aux questions. Dans ce cas, vous allez créer une base de connaissances contenant des questions et des réponses sur [Microsoft Learn](https://docs.microsoft.com/learn).

1. Dans un nouvel onglet de navigateur, accédez au portail *Language Studio* à l’adresse `https://language.azure.com`, puis connectez-vous en utilisant le compte Microsoft associé à votre abonnement Azure.
2. Si vous êtes invité à choisir une ressource de langue, sélectionnez les paramètres suivants :
    - **Azure Directory** : Annuaire Azure contenant votre abonnement.
    - **Abonnement Azure** : Votre abonnement Azure.
    - **Ressource de langue** : Ressource de langue que vous avez créée précédemment.
3. Si vous n’êtes <u>pas</u> invité à choisir une ressource de langue, c’est peut-être parce que vous avez plusieurs ressources de langue dans votre abonnement, auquel cas :
    1. Dans la barre en haut de la page, cliquez sur le bouton **Paramètres (&#9881;)**.
    2. Dans la page **Paramètres**, affichez l’onglet **Ressources**.
    3. Sélectionnez la ressource de langue que vous venez de créer, puis cliquez sur **Changer de ressource**.
    4. En haut de la page, cliquez sur **Language Studio** pour revenir à la page d’accueil de Language Studio.
3. En haut du portail, dans le menu **Créer**, sélectionnez **Réponses aux questions personnalisées**.
4. Dans l’Assistant **Créer un projet**, dans la page **Choisir un paramètre de langue**, sélectionnez l’option permettant de définir la langue de tous les projets de cette ressource, puis sélectionnez **Anglais** comme langue. Cliquez ensuite sur **Suivant**.
5. Dans la page **Entrer les informations de base**, entrez les détails suivants, puis cliquez sur **Suivant** :
    - **Nom** LearnFAQ
    - **Description** : FAQ pour Microsoft Learn
    - **Réponse par défaut quand aucune réponse n’est retournée** : Désolé, je ne comprends pas la question
6. Dans la page **Vérifier et terminer**, cliquez sur **Créer un projet**.

## <a name="add-a-sources-to-the-knowledge-base"></a>Ajoutez une source à la base de connaissances

Vous pouvez créer une base de connaissances à partir de zéro, mais il est courant de commencer par importer les questions et les réponses d'une page ou d'un document de FAQ existant. Dans ce cas, vous allez importer des données à partir d’une page web de FAQ existante pour Microsoft Learn, et vous importerez également certaines questions et réponses prédéfinies de type « chit-chat » pour prendre en charge les échanges conversationnels courants.

1. Dans la page **Gérer les sources** de votre projet de réponse aux questions, dans la liste **&#9547; Ajouter une source**, sélectionnez **URL**. Ensuite, dans la boîte de dialogue **Ajouter des URL**, cliquez sur **&#9547; Ajouter une URL** et ajoutez l’URL suivante, puis cliquez sur **Ajouter tout** pour l’ajouter au base de connaissances :
    - **Nom** : `Learn FAQ Page`
    - **URL** : `https://docs.microsoft.com/en-us/learn/support/faq`
2. Dans la page **Gérer les sources** de votre projet de réponse aux questions, dans la liste **&#9547; Ajouter une source**, sélectionnez **Chitchat**. Dans la boîte de dialogue **Ajouter une conversation de chit-chat**, sélectionnez **Convivial**, puis cliquez sur **Ajouter une conversation de chit-chat**.

## <a name="edit-the-knowledge-base"></a>Modifier la base de connaissances

Votre base de connaissances a été rempli avec des paires de questions et réponses de la FAQ Microsoft Learn, complétée par un ensemble de paires de questions et de réponses de type *chit-chat*. Vous pouvez étendre le base de connaissances en ajoutant des paires de questions et réponses supplémentaires.

1. Dans votre projet **LearnFAQ** dans Language Studio, sélectionnez la page **Modifier base de connaissances** pour afficher les paires de questions et réponses existantes (si certains conseils sont affichés, lisez-les **, cliquez dessus** pour les ignorer ou cliquez sur **Ignorer tout**)
2. Dans le base de connaissances, sélectionnez **&#65291; Ajouter une paire de questions**.
3. Dans la zone **Question**, tapez `What is Microsoft certification?` et appuyez sur **Entrée****.
4. Sélectionnez **&#65291; Ajouter une autre expression**, tapez `How can I demonstrate my Microsoft technology skills?` et **appuyez sur Entrée**.
5. Dans la zone **Réponse**, tapez `The Microsoft Certified Professional program enables you to validate and prove your skills with Microsoft technologies.`. Ensuite, appuyez sur **Envoyer** pour ajouter la question (y compris une formulation alternative) et réponse à la base de connaissances.

    Dans certains cas, il est judicieux d’autoriser l’utilisateur à suivre la réponse pour créer une invite *multitour* qui permet à l’utilisateur d’affiner de manière itérative la question pour obtenir la réponse dont il a besoin.

6. Sous la réponse que vous avez entrée pour la question de certification, sélectionnez **&#65291; Ajouter des invites de suivi**.
7. Dans la boîte de dialogue **Invite de suivi**, entrez les paramètres suivants, puis cliquez sur **Ajouter une invite** :
    - **Texte affiché dans l’invite à l’utilisateur** : `Learn more about certification`.
    - Sélectionnez **Créer un lien vers une nouvelle paire**, puis entrez ce texte : `You can learn more about certification on the [Microsoft certification page](https://docs.microsoft.com/learn/certifications/).`
    - **Afficher dans le flux contextuel uniquement** : Sélectionné. *Cette option garantit que la réponse n’est jamais retournée dans le contexte d’une question de suivi de la question de certification d’origine.*

## <a name="train-and-test-the-knowledge-base"></a>Entraîner et tester la base de connaissances

Maintenant que vous avez une base de connaissances, vous pouvez la tester dans le portail Language Studio.

1. En haut à droite de la page, cliquez sur **Enregistrer les modifications**.
2. Une fois les modifications enregistrées, cliquez sur **Tester** pour ouvrir le volet de test.
3. Dans le volet de test, en haut, *désélectionnez* l’option permettant d’afficher des réponses courtes. Dans la partie inférieure, entrez le message `Hello`. Une réponse appropriée doit être retournée.
4. Dans la partie inférieure du volet de test, entrez le message `What is Microsoft Learn?`. Une réponse appropriée du FAQ devrait être retournée.
5. Entrez le message `Thanks!` Une réponse de conversation appropriée doit être retournée.
6. Entrez le message `Tell me about certification`. La réponse que vous avez créée doit être retournée avec un lien d’invite de suivi.
7. Sélectionnez le lien de suivi **En savoir plus sur la certification**. La réponse de suivi avec un lien vers la page de certification doit être retournée.
8. Lorsque vous avez fini de tester la base de connaissances, fermez le volet de test.

## <a name="deploy-and-test-the-knowledge-base"></a>Déployer et tester la base de connaissances

La base de connaissances fournit un service de bout en bout que les applications clientes peuvent utiliser pour répondre aux questions. Vous êtes maintenant prêt à publier votre base de connaissances et à accéder à son interface REST à partir d’un client.

1. Dans le projet **LearnFAQ** dans Language Studio, sélectionnez la page **Déployer base de connaissances**.
2. En haut de la page, cliquez sur **Déployer**. Cliquez ensuite sur **Déployer** pour confirmer que vous souhaitez déployer le base de connaissances.
3. Une fois le déploiement terminé, cliquez sur **Obtenir l’URL de prédiction** pour afficher le point de terminaison REST de votre base de connaissances, puis copiez-le dans le Presse-papiers (mais ne fermez pas encore la boîte de dialogue).
4. Dans Visual Studio Code, dans le dossier **12-qna**, ouvrez **ask-question.cmd**. Ce script utilise *Curl* pour appeler l’interface REST d’un point de terminaison de réponse aux questions.
5. Dans le script, remplacez *YOUR_PREDICTION_ENDPOINT* par le point de terminaison de prédiction que vous avez copié (en veillant à ce qu’il soit placé entre guillemets).
6. Revenez au navigateur et dans la boîte de dialogue **Obtenir l’URL de prédiction**, notez que l’exemple de requête inclut une valeur pour le paramètre **Ocp-Apim-Subscription-Key**, qui ressemble à *ab12c345de678fg9hijk01lmno2pqrs34*. Il s’agit de la clé d’autorisation de votre ressource. Copiez-le dans le Presse-papiers, puis cliquez sur **Obtenir** pour fermer la boîte de dialogue.
7. Revenez à Visual Studio Code et, dans le script **ask-question.cmd**, remplacez *YOUR_KEY* par la clé que vous avez copiée (en veillant à ce qu’elle soit placée entre guillemets).
8. Notez que la commande Curl dans le script envoie un paramètre de **question** avec la valeur **Qu’est-ce qu’un parcours d'apprentissage ?** .
9. Vérifiez que l’intégralité du script ressemble au code suivant, puis enregistrez le fichier.

    ```
    @echo off
    SETLOCAL ENABLEDELAYEDEXPANSION

    rem Set variables
    set prediction_url="https://some-name.cognitiveservices.azure.com/language/......."
    set key="ab12c345de678fg9hijk01lmno2pqrs34"

    curl -X POST !prediction_url! -H "Ocp-Apim-Subscription-Key: !key!" -H "Content-Type: application/json" -d "{'question': 'What is a learning Path?' }"
    ```

10. Dans Visual Studio Code, dans le volet Explorateur, cliquez avec le bouton droit de la souris sur le dossier **ask-question.cmd**, puis sélectionnez **Ouvrir dans le terminal intégré**.
11. Dans le volet terminal, entrez la commande `ask-question.cmd` pour exécuter le script et afficher la réponse JSON retournée par le service, qui doit contenir une réponse appropriée à la question *Qu’est-ce qu’un parcours d’apprentissage ?*

## <a name="create-a-bot-for-the-knowledge-base"></a>Créer un bot pour la base de connaissances

Le plus souvent, les applications clientes utilisées pour récupérer des réponses à partir d’un base de connaissances sont des bots.

1. Revenez à Language Studio dans le navigateur, puis, dans la page **Déployer une base de connaissances**, sélectionnez **Créer un bot**. Cela ouvre le portail Azure dans un nouvel onglet de navigateur afin que vous puissiez créer un bot d'application Web dans votre abonnement Azure.
2. Dans le portail Azure, créez un bot d’application web avec les paramètres suivants (dont la plupart sont préremplis pour vous) :

    *Si certaines valeurs sont manquantes, actualisez votre navigateur.*  

  - **Descripteur du bot** : *nom unique de votre bot*
  - **Abonnement** : *votre abonnement Azure*
  - **Groupe de ressources** : *groupe de ressources contenant votre ressource linguistique*
  - **Emplacement** : *Même emplacement que votre service Analyse de texte*.
  - **Niveau tarifaire** : F0
  - **Nom de l’application** : identique au **Descripteur de bot** avec un ID unique *.azurewebsites.net* ajouté automatiquement
  - **Langage du Kit de développement logiciel (SDK)**  : *choisissez C# ou Node.js*
  - **Clé d’authentification QnA** : *doit être automatiquement définie sur la clé d’authentification de votre base de connaissances QnA*
  - **Plan App Service/Emplacement** : *Il peut être réglé automatiquement sur un plan et un emplacement appropriés, s'il en existe un. Sinon, créez un plan*
  - **Application Insights** : désactivé
  - **ID et mot de passe de l’application Microsoft** : ID et mot de passe d’application créés automatiquement.
3. Attendez que votre bot soit créé. Ensuite, cliquez sur **Aller à la ressource** (ou alternativement, sur la page d’accueil, cliquez sur **Groupes de ressources**, ouvrez le groupe de ressources où vous avez créé le bot de l’application Web, et cliquez dessus).
4. Dans le panneau de votre bot, affichez la page **Test dans chat Web** et attendez que le bot affiche le message **Bonjour et bienvenue !** (l’initialisation peut prendre quelques secondes).
5. Utilisez l’interface de test de conversation pour vous assurer que votre bot répond aux questions à partir de votre base de connaissances comme prévu. Par exemple, essayez d’envoyer `What is Microsoft certification?`.

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur les réponses aux questions de service de langue, consultez la [documentation du service de langue](https://docs.microsoft.com/en-us/azure/cognitive-services/language-service/question-answering/overview).
