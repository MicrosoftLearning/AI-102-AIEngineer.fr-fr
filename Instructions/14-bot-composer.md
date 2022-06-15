---
lab:
  title: Créer un bot avec Bot Framework Composer
  module: Module 7 - Conversational AI and the Azure Bot Service
ms.openlocfilehash: f25609df8d9abc29e691bd83d0470561c9e3e4b0
ms.sourcegitcommit: b934aa694b86756d8b297a384cc6b707f0536e57
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2021
ms.locfileid: "145195569"
---
# <a name="create-a-bot-with-bot-framework-composer"></a>Créer un bot avec Bot Framework Composer

Bot Framework Composer est un concepteur graphique qui vous permet de créer rapidement et facilement des chatbots sophistiqués sans programmation. Composer est un outil open source qui présente un canevas visuel pour la création de bots.

## <a name="prepare-to-develop-a-bot"></a>Préparer le développement d’un bot

Commençons par préparer les services et outils dont vous avez besoin pour développer un bot.

### <a name="get-an-openweather-api-key"></a>Obtenir une clé API OpenWeather

Dans cet exercice, vous allez créer un bot qui utilise le service OpenWeather pour récupérer les conditions météorologiques dans la ville indiquée par l’utilisateur. Vous devez disposer d’une clé API pour que le service fonctionne.

1. Dans un navigateur web, accédez au site OpenWeather à l’adresse `https://openweathermap.org/price`.
2. Demandez une clé API gratuite et créez un compte OpenWeather (si vous n’en avez pas déjà).
3. Après l’inscription, affichez la page **Clés API** pour afficher votre clé API.

### <a name="update-bot-framework-composer"></a>Mettre à jour Bot Framework Composer

Vous allez utiliser Bot Framework Composer pour créer votre bot. Cet outil est mis à jour régulièrement. Nous allons donc vérifier que la dernière version est installée.

> **Remarque** : Les mises à jour peuvent inclure des modifications apportées à l’interface utilisateur qui affectent les instructions de cet exercice.

1. Démarrez **Bot Framework Composer** et, si vous n’êtes pas automatiquement invité à installer une mise à jour, utilisez l’option **Rechercher les mises à jour** dans le menu **Aide** pour rechercher les mises à jour.
2. Si une mise à jour est disponible, choisissez de procéder à l’installation de la mise à jour après la fermeture de l’application. Fermez ensuite Bot Framework Composer et installez la mise à jour pour l’utilisateur actuellement connecté. Redémarrez Bot Framework Composer une fois l’installation terminée. L’installation peut prendre quelques minutes.
3. Vérifiez que la version de Bot Framework Composer est au moins **2.0.0**.

## <a name="create-a-bot"></a>Créer un bot

Vous êtes maintenant prêt à utiliser Bot Framework Composer pour créer un bot.

### <a name="create-a-bot-and-customize-the-welcome-dialog-flow"></a>Créer un bot et personnaliser le flux de dialogue de bienvenue

1. Démarrez Bot Framework Composer s’il n’est pas déjà ouvert.
2. Dans l’écran **Accueil**, sélectionnez **Nouveau**. Créez un bot vide, nommez-le **BotMétéo** et enregistrez-le dans un dossier local.
3. Fermez le volet **Démarrage** s’il est ouvert, puis, dans le volet de navigation de gauche, sélectionnez **Salutations** pour ouvrir le canevas de création et voir l’activité *ConversationUpdate*. Celle-ci est appelée quand un utilisateur rejoint initialement une conversation avec le bot. L’activité se compose d’un flux d’actions.
4. Dans le volet de propriétés à droite, sélectionnez le titre **Salutations** en haut pour le modifier. Remplacez-le par **AccueilUtilisateurs**.
5. Dans le canevas de création, sélectionnez l’action **Envoyer une réponse**. Ensuite, dans le volet des propriétés, modifiez le texte par défaut de *Bonjour votre bot*  en `Hi! I'm WeatherBot.`
6. Dans le canevas de création, sélectionnez le symbole final **+** (juste au-dessus du cercle qui marque la <u>fin</u> du flux de dialogue), puis ajoutez une nouvelle action **Poser une question** avec une réponse **Texte**.

    Cette nouvelle action crée deux nœuds dans le flux de dialogue. Le premier nœud définit une invite avec laquelle le bot pose une question à l’utilisateur, et le deuxième représente la réponse qui sera reçue de l’utilisateur. Dans le volet des propriétés, ces nœuds possèdent les onglets **Réponse du bot** et **Entrée utilisateur** correspondants.

7. Dans le volet des propriétés, sous l’onglet **Réponse du bot**, ajoutez une réponse avec le texte `What's your name?`. Ensuite, sous l’onglet **Entrée utilisateur**, définissez la valeur **Propriété** sur `user.name` pour définir une variable à laquelle vous pourrez accéder ultérieurement dans la conversation du bot.
8. De retour dans le canevas de création, sélectionnez le symbole **+** sous l’action **Entrée utilisateur (texte)** que vous venez d’ajouter, puis ajoutez une action **Envoyer une réponse**.
9. Sélectionnez l’action **Envoyer une réponse** que vous venez d’ajouter, puis, dans le volet de propriétés, définissez la valeur de texte sur `Hello ${user.name}, nice to meet you!`.

    Le flux d’activité complet doit se présenter comme suit :

    ![Flux de dialogue qui accueille un utilisateur et demande son nom](./images/welcomeUsers.png)

### <a name="test-the-bot"></a>Tester le bot

Votre bot de base est terminé. Nous allons à présent le tester.

1. Sélectionnez **Démarrer le bot** en haut à droite de Composer, puis attendez le temps que votre bot soit compilé et démarré. Cette opération peut prendre plusieurs minutes.

    - Si un message du pare-feu Windows s’affiche, activez l’accès pour tous les réseaux.

2. Dans le volet **Gestionnaire de runtime de bot local**, sélectionnez **Ouvrir un chat web**.
3. Dans le volet de chat web **BotMétéo**, après un court temps d’attente, vous voyez le message d’accueil ainsi que le message vous invitant à entrer votre prénom.  Entrez votre prénom, puis appuyez sur **Entrée**.
4. Le bot doit répondre avec **Bonjour *votre_prénom*, ravi de vous rencontrer !** .
5. Fermez le panneau de chat web.
6. En haut à droite de Composer, en regard de **&#8635; redémarrer le bot**, cliquez sur **<u>=</u>** pour ouvrir le volet **Gestionnaire de runtime de bot local** et utilisez l’icône ⏹ pour arrêter le bot.

## <a name="add-a-dialog-to-get-the-weather"></a>Ajouter un dialogue pour obtenir la météo

Vous disposez maintenant d’un bot opérationnel. Vous pouvez à présent développer ses capacités en ajoutant des dialogues pour des interactions spécifiques. Dans le cas présent, vous allez ajouter un dialogue déclenché quand l’utilisateur mentionne « météo ».

### <a name="add-a-dialog"></a>Ajouter un dialogue

Tout d’abord, vous devez définir un flux de dialogue qui sera utilisé pour traiter les questions sur la météo.

1. Dans le volet de navigation de Composer, maintenez la souris sur le nœud de niveau supérieur (**BotMétéo**) et, dans le menu **...** , sélectionnez **+ Ajouter un dialogue** comme illustré ici :

    ![Menu Ajouter un dialogue](./images/add-dialog.png)

    Créez ensuite un dialogue nommé **ObtenirMétéo** avec la description **Obtenir les conditions météorologiques actuelles pour le code postal indiqué**.
2. Dans le volet de navigation, sélectionnez le nœud **BeginDialog** pour le nouveau dialogue **ObtenirMétéo**. Ensuite, dans le canevas de création, utilisez le symbole **+** pour ajouter une action **Poser une question** dans le cas d’une réponse **Texte**.
3. Dans le volet des propriétés, sous l’onglet **Réponse du bot**, ajoutez la réponse `Enter your city.`.
4. Dans l’onglet **Entrée utilisateur**, définissez le champ **Propriété** sur `dialog.city`, puis définissez le champ **Format de sortie** sur l’expression `=trim(this.value)` pour supprimer tous les espaces superflus autour de la valeur fournie par l’utilisateur.

    À ce stade, le flux d’activité doit ressembler à ceci :

    ![Flux de dialogue avec une action « Envoyer une réponse »](./images/getWeather-dialog-1.png)

    Pour le moment, le dialogue demande à l’utilisateur d’entrer une ville. Vous devez maintenant implémenter la logique pour récupérer les informations météorologiques relative à la ville indiquée.

6. Dans le canevas de création, directement sous l’action **Entrée utilisateur** pour l’entrée de la ville, sélectionnez le symbole **+** pour ajouter une nouvelle action.
7. Dans la liste des actions, sélectionnez **Accéder aux ressources externes**, puis **Envoyer une requête HTTP**.
8. Définissez les propriétés de la **Requête HTTP** comme suit, en remplaçant **YOUR_API_KEY** par votre clé API [OpenWeather](https://openweathermap.org/price) :
    - **Méthode HTTP** : GET
    - **Url** : `http://api.openweathermap.org/data/2.5/weather?units=metric&q=${dialog.city}&appid=YOUR_API_KEY`
    - **Propriété du résultat** : `dialog.api_response`

    Le résultat peut inclure l’une des quatre propriétés suivantes de la réponse HTTP :

    - **statusCode**. Accessible dans **dialog.api_response.statusCode**.
    - **reasonPhrase**. Accessible dans **dialog.api_response.reasonPhrase**.
    - **content**. Accessible dans **dialog.api_response.content**.
    - **headers**. Accessible dans **dialog.api_response.headers**.

    En outre, si le type de réponse est JSON, il s’agit d’un objet désérialisé disponible dans la propriété **dialog.api_response.content**. Pour plus d’informations sur l’API OpenWeather et la réponse retournée, consultez la [documentation de l’API OpenWeather](https://openweathermap.org/current).

    À présent, vous devez ajouter une logique au flux de dialogue qui gère la réponse, pouvant indiquer la réussite ou l’échec de la requête HTTP.

9. Dans le canevas de création, sous l’action **Envoyer une requête HTTP** que vous avez créée, ajoutez une action **Créer une condition** > **Branche : if/else**. Cette action définit une branche dans le flux de dialogue avec les chemins **True** et **False**.
10. Sous **Propriétés**, pour l’action de la branche, entrez l’expression suivante pour le champ **Condition** :

    ```
    =dialog.api_response.statusCode == 200
    ```

11. Si l’appel a réussi, vous devez stocker la réponse dans une variable. Dans le canevas de création, dans la branche **True**, ajoutez une action **Gérer les propriétés** > **Définir les propriétés**. Ensuite, dans le volet des propriétés, ajoutez les attributions de propriétés suivantes :

    | Propriété | Valeur |
    | -- | -- |
    | `dialog.weather` | `=dialog.api_response.content.weather[0].description` |
    | `dialog.temp` | `=round(dialog.api_response.content.main.temp)` |
    | `dialog.icon` | `=dialog.api_response.content.weather[0].icon` |

12. Toujours dans la branche **True**, ajoutez une action **Envoyer une réponse** sous l’action **Définir une propriété** et définissez son texte sur :

    ```
    The weather in ${dialog.city} is ${dialog.weather} and the temperature is ${dialog.temp}&deg;.
    ```

    ***Remarque** : Ce message utilise les propriétés **dialog.city**, **dialog.weather** et **dialog.temp** que vous définissez dans les actions précédentes. Vous serez aussi amené à utiliser la propriété **dialog.icon** ultérieurement.*

13. Vous devez également prévoir que le service météo ne retournera peut-être pas une réponse 200. Ainsi, dans la branche **False**, ajoutez une action **Envoyer une réponse** et définissez son texte sur `I got an error: ${dialog.api_response.content.message}.`.

    Le flux de dialogue doit maintenant se présenter ainsi :

    ![Flux de dialogue avec branche pour les résultats de réponse HTTP](./images/getWeather-dialog-2.png)

### <a name="add-a-trigger-for-the-dialog"></a>Ajouter un déclencheur pour le dialogue

À présent, vous avez besoin d’un moyen de lancer le nouveau dialogue à partir du dialogue d’accueil existant.

1. Dans le volet de navigation, sélectionnez le dialogue **BotMétéo** qui contient **AccueilUtilisateurs** (sous le nœud de bot de niveau supérieur du même nom).

    ![Workflow BotMétéo sélectionné](./images/select-workflow.png)

2. Dans le volet de propriétés du dialogue **BotMétéo** sélectionné, dans la section **Compréhension du langage**, définissez le **Type de module de reconnaissance** sur **Module de reconnaissance d’expression régulière**.

    > Le type de module de reconnaissance par défaut utilise le service Language Understanding pour produire l’intention de l’utilisateur à l’aide d’un modèle de compréhension du langage naturel. Nous utilisons un module de reconnaissance d’expression régulière pour simplifier cet exercice. Dans une application réelle, vous devez envisager d’utiliser Language Understanding pour permettre une reconnaissance d’intention plus sophistiquée.

3. Dans le menu **...** du dialogue **BotMétéo**, sélectionnez **Ajouter un déclencheur**.

    ![Menu Ajouter un déclencheur](./images/add-trigger.png)

    Créez ensuite un déclencheur avec les paramètres suivants :

    - **Quel est le type de ce déclencheur ?**  : Intention reconnue
    - **Quel est le nom de ce déclencheur (RegEx)** :  `WeatherRequested`
    - **Entrez un modèle regex** : `weather`

    > Le texte entré dans la zone de texte du modèle regex est un simple modèle d’expression régulière qui indiquera au bot de rechercher le mot *météo* dans tout message entrant.  Si « météo » est présent, le message devient une **intention reconnue** et le déclencheur est lancé.

4. Le déclencheur étant créé, vous devez configurer l’action correspondante. Dans le canevas de création du déclencheur, sélectionnez le symbole **+** sous votre nouveau nœud de déclencheur **MétéoDemandée**. Ensuite, dans la liste des actions, sélectionnez **Gestion de dialogue**, puis **Commencer un nouveau dialogue**.
5. Après avoir sélectionné l’action **Commencer un nouveau dialogue**, dans le volet de propriétés, sélectionnez le dialogue **ObtenirMétéo** dans la liste déroulante **Nom du dialogue** pour commencer le dialogue **ObtenirMétéo** que vous avez défini précédemment, quand le déclencheur **MétéoDemandée** est reconnu.

    Le flux d’activité **MétéoDemandée** doit ressembler à ceci :

    ![Un déclencheur regex commence le dialogue ObtenirMétéo](./images/weather-regex.png)

6. Redémarrez le bot et ouvrez le volet de chat web. Redémarrez ensuite la conversation, puis, après avoir entré votre nom, entrez `What is the weather like?`. Ensuite, lorsque vous y êtes invité, entrez une ville, par exemple `Seattle`. Le bot contacte le service et doit répondre avec un bref relevé météo.
7. Une fois le test terminé, fermez le volet de chat web et arrêtez le bot.

## <a name="handle-interruptions"></a>Gestion des interruptions

Un bot bien conçu doit permettre aux utilisateurs de modifier le flux de la conversation, par exemple en annulant une demande.

1. Dans le volet de navigation de Bot Composer, utilisez le menu **...** du dialogue **BotMétéo** pour ajouter un nouveau déclencheur (en plus des **déclencheurs AccueilUtilisateurs** et **MétéoDemandée** existants). Le nouveau déclencheur doit avoir les paramètres suivants :

    - **Quel est le type de ce déclencheur ?**  : Intention reconnue
    - **Quel est le nom de ce déclencheur (RegEx)** :  `CancelRequest`
    - **Entrez un modèle regex** : `cancel`

    > Le texte entré dans la zone de texte du modèle regex est un simple modèle d’expression régulière qui indiquera au bot de rechercher le mot *annuler* dans tout message entrant.

2. Dans le canevas de création du déclencheur, ajoutez une action **Envoyer une réponse** et définissez sa réponse texte sur `OK. Whenever you're ready, you can ask me about the weather.`.
3. Sous l’action **Envoyer une réponse**, ajoutez une nouvelle action pour mettre fin au dialogue en sélectionnant **Gestion des dialogues** et **Mettre fin à ce dialogue**.

    Le flux d’activité **CancelRequest** doit se présenter comme suit :

    ![Un déclencheur CancelRequest avec les actions Envoyer une réponse et Mettre fin au dialogue](./images/cancel-dialog.png)

    Maintenant que vous disposez d’un déclencheur pour répondre à la demande d’annulation d’un utilisateur, vous devez autoriser les interruptions aux flux de dialogue dans les cas où l’utilisateur peut vouloir effectuer une telle demande, par exemple lorsqu’il est invité à entrer un code postal après avoir demandé des informations météorologiques.

4. Dans le volet de navigation, sélectionnez **BeginDialog** sous le dialogue **ObtenirMétéo**.
5. Sélectionnez l’action **Demander du texte** qui demande à l’utilisateur d’entrer sa ville.
6. Dans les propriétés de l’action, sous l’onglet **Autre**, développez la section **Configurations d’invite** et affectez la valeur **true** à la propriété **Autoriser les interruptions**.
7. Redémarrez le bot et ouvrez le volet de chat web. Redémarrez la conversation et, après avoir indiqué votre nom, entrez `What is the weather like?`. Ensuite, à l’invite, entrez `cancel` et confirmez que la demande est annulée.
8. Après avoir annulé la demande, entrez `What's the weather like?`. Vous noterez que le déclencheur approprié démarre une nouvelle instance du dialogue **ObtenirMétéo**, vous invitant une fois de plus à entrer une ville.
9. Une fois le test terminé, fermez le volet de chat web et arrêtez le bot.

## <a name="enhance-the-user-experience"></a>Améliorer l’expérience utilisateur

Jusqu’à présent, les interactions avec le bot météo s’effectuaient par texte.  Les utilisateurs entrent du texte pour exprimer leurs intentions et le bot répond avec du texte. Bien que le texte soit souvent un moyen approprié de communiquer, vous pouvez améliorer l’expérience à l’aide d’autres formes d’élément de l’interface utilisateur.  Par exemple, vous pouvez utiliser des boutons pour lancer des actions recommandées ou afficher une *carte* pour présenter des informations visuellement.

### <a name="add-a-button"></a>Ajouter un bouton

1. Dans le volet de navigation de Bot Framework Composer, sous l’action **ObtenirMétéo**, sélectionnez **BeginDialog**.
2. Dans le canevas de création, sélectionnez l’action **Demander du texte** qui contient l’invite pour la ville.
3. Dans le volet des propriétés, sélectionnez **Afficher le code**, puis remplacez le code existant par le code suivant.

```
[Activity    
    Text = Enter your city.
    SuggestedActions = Cancel
]
```

Cette activité invite l’utilisateur à entrer sa ville comme précédemment, mais affiche également un bouton **Annuler**.

### <a name="add-a-card"></a>Ajouter une carte

1. Dans le dialogue **ObtenirMétéo**, sous le chemin **True**, vérifiez la réponse du service météo HTTP, puis sélectionnez l’action **Envoyer une réponse** qui affiche le relevé météo.
2. Dans le volet des propriétés, sélectionnez **Afficher le code** et remplacez le code existant par le code suivant.

```
[ThumbnailCard
    title = Weather for ${dialog.city}
    text = ${dialog.weather} (${dialog.temp}&deg;)
    image = http://openweathermap.org/img/w/${dialog.icon}.png
]
```

Ce modèle utilise les mêmes variables qu’auparavant pour les conditions météorologiques, mais il ajoute également un titre à la carte qui s’affichera avec une image pour les conditions météorologiques.

### <a name="test-the-new-user-interface"></a>Tester la nouvelle interface utilisateur

1. Redémarrez le bot et ouvrez le volet de chat web. Redémarrez la conversation puis, après avoir indiqué votre nom, entrez `What is the weather like?`. À l’invite, cliquez ensuite sur le bouton **Annuler** pour annuler la demande.
2. Après l’annulation, entrez `Tell me about the weather` et, lorsque vous y êtes invité, entrez une ville (par exemple `London`). Le bot contacte le service et doit répondre avec une carte indiquant les conditions météo.
3. Une fois le test terminé, fermez l’émulateur et arrêtez le bot.

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur Bot Framework Composer, consultez la [documentation de Bot Framework Composer](https://docs.microsoft.com/composer/introduction).
