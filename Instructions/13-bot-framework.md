---
lab:
  title: Créer un bot à l’aide du kit de développement logiciel (SDK) Bot Framework
  module: Module 7 - Conversational AI and the Azure Bot Service
ms.openlocfilehash: ab51a1f11f4eee99838634d83b5d9261a8c20ecb
ms.sourcegitcommit: 480c5898009ea964025fdecb57900aefeeac81fd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/07/2022
ms.locfileid: "147019882"
---
# <a name="create-a-bot-with-the-bot-framework-sdk"></a>Créer un bot à l’aide du kit de développement logiciel (SDK) Bot Framework

Les *bots* sont des agents logiciels qui peuvent participer à des dialogues conversationnels avec des utilisateurs humains. Le Microsoft Bot Framework fournit une plateforme complète pour la création de bots qui peuvent être fournis en tant que services cloud par le biais d’Azure Bot Service.

Dans cet exercice, vous allez utiliser le kit de développement logiciel (SDK) Microsoft Bot Framework pour créer et déployer un bot.

## <a name="before-you-start"></a>Avant de commencer

Commençons par préparer l’environnement pour le développement de bots.

### <a name="update-the-bot-framework-emulator"></a>Démarrez le Bot Framework Emulator

Vous allez utiliser le kit de développement logiciel (SDK) Bot Framework pour créer votre bot et le Bot Framework Emulator pour le tester. Le Bot Framework Emulator est mis à jour régulièrement. Nous allons donc vérifier que la dernière version est installée.

> **Remarque** : Les mises à jour peuvent inclure des modifications apportées à l’interface utilisateur qui affectent les instructions de cet exercice.

1. Démarrez le **Bot Framework Emulator** et, si vous êtes invité à installer une mise à jour, faites-le pour l’utilisateur actuellement connecté. Si vous n’êtes pas invité automatiquement, utilisez l’option **Vérifier la mise à jour** dans le menu **Aide** pour rechercher les mises à jour.
2. Après avoir installé une mise à jour disponible, fermez le Bot Framework Emulator jusqu’à ce que vous en ayez besoin ultérieurement.

> **Important** : Il peut arriver que le téléchargement de la mise à jour échoue et qu’une investigation soit en cours. Si vous ne voyez aucun progrès après quelques minutes à essayer de faire la mise à jour, vous pouvez abandonner le téléchargement et utiliser la version actuellement installée de l’émulateur.

### <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

Si vous n’avez pas déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement où vous travaillez sur ce laboratoire, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/AI-102-AIEngineer` vers un dossier local (peu importe quel dossier).
3. Une fois le référentiel cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## <a name="create-a-bot"></a>Créer un bot

Vous pouvez utiliser le kit de développement logiciel (SDK) Bot Framework pour créer un bot basé sur un modèle, puis personnaliser le code pour répondre à vos besoins spécifiques.

> **Remarque** : Vous pouvez choisir d’utiliser **C#** ou **Python** dans cet exercice. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **13-bot-framework** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier de votre langage choisi et ouvrez un terminal intégré.
3. Dans le terminal, exécutez les commandes suivantes pour installer les modèles et packages de bot dont vous avez besoin :

**C#**

```C#
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

**Python**

```Python
pip install botbuilder-core
pip install asyncio
pip install aiohttp
pip install cookiecutter==1.7.0
```

4. Une fois les modèles et les packages installés, exécutez la commande suivante pour créer un bot basé sur le modèle *EchoBot* :

**C#**

```C#
dotnet new echobot -n TimeBot
```

**Python**

```Python
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

Si vous utilisez Python, lorsque vous y êtes invité par cookiecutter, entrez les détails suivants :
- **bot_name** : TimeBot
- **bot_description** : Un bot pour notre époque
    
5. Dans le volet terminal, entrez les commandes suivantes pour remplacer le répertoire actif par le dossier **TimeBot** et listez les fichiers de code qui ont été générés pour votre bot :

    ```Code
    cd TimeBot
    dir
    ```

## <a name="test-the-bot-in-the-bot-framework-emulator"></a>Tester le bot dans Bot Framework Emulator

Vous avez créé un bot basé sur le modèle *EchoBot*. Vous pouvez maintenant l’exécuter localement et le tester à l’aide de Bot Framework Emulator (qui doit être installé sur votre système).

1. Dans le volet terminal, vérifiez que le répertoire actif est le dossier **TimeBot** contenant vos fichiers de code bot, puis entrez la commande suivante pour démarrer votre bot en cours d’exécution localement.

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```
    
Lorsque le bot démarre, notez que le point de terminaison sur lequel il est en cours d’exécution s’affiche. Cela doit ressembler à **http://localhost:3978** .

2. Démarrez le Bot Framework Emulator et ouvrez votre bot en spécifiant le point de terminaison avec le chemin **/api/messages** ajouté, comme suit :

    `http://localhost:3978/api/messages`

3. Une fois la conversation ouverte dans un volet de **Conversation en direct**, attendez le message *Bonjour et bienvenue*.
4. Entrez un message tel que *Bonjour* et affichez la réponse du bot, qui doit renvoyer le message que vous avez entré.
5. Fermez le Bot Framework Emulator et revenez à Visual Studio Code, puis dans la fenêtre de terminal, entrez **Ctrl+C** pour arrêter le bot.

## <a name="modify-the-bot-code"></a>Modifier le code du bot

Vous avez créé un bot qui renvoie l’entrée de l’utilisateur. Il n’est pas particulièrement utile, mais sert à illustrer le flux de base d’un dialogue conversationnel. Une conversation avec un bot se compose d’une séquence d’*activités*, dans laquelle le texte, les graphiques ou les *cartes* de l’interface utilisateur sont utilisés pour échanger des informations. Le bot commence la conversation avec un message d’accueil, qui est le résultat d’une activité de *mise à jour de conversation* déclenchée lorsqu’un utilisateur initialise une session de conversation avec le bot. Ensuite, la conversation se compose d’une séquence d’activités supplémentaires dans lesquelles l’utilisateur et le bot le prennent à son tour pour envoyer des *messages*.

1. Dans Visual Studio Code, ouvrez le fichier de code suivant pour votre bot :
    - **C#** : TimeBot/Bots/EchoBot.cs
    - **Python** : TimeBot/bot.py

    Notez que le code de ce fichier se compose de fonctions de *gestionnaire d’activités* ; l’une pour l’activité de mise à jour de conversation *Membre ajouté* (lorsqu’une personne rejoint la session de conversation) et une autre pour l’activité *Message* (lorsqu’un message est reçu). La conversation est basée sur le concept de *tours*, dans lequel chaque tour représente une interaction dans laquelle le bot reçoit, traite et répond à une activité. Le *contexte de tour* est utilisé pour suivre les informations sur l’activité en cours de traitement au tour actuel.

2. En haut du fichier de code, ajoutez l’instruction import d’espace de noms suivante :

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. Modifiez la fonction de gestionnaire d’activités de l’activité *Message* pour qu’elle corresponde au code suivant :

**C#**

```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string inputMessage = turnContext.Activity.Text;
    string responseMessage = "Ask me what the time is.";
    if (inputMessage.ToLower().StartsWith("what") && inputMessage.ToLower().Contains("time"))
    {
        var now = DateTime.Now;
        responseMessage = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");
    }
    await turnContext.SendActivityAsync(MessageFactory.Text(responseMessage, responseMessage), cancellationToken);
}
```

**Python**

```Python
async def on_message_activity(self, turn_context: TurnContext):
    input_message = turn_context.activity.text
    response_message = 'Ask me what the time is.'
    if (input_message.lower().startswith('what') and 'time' in input_message.lower()):
        now = datetime.now()
        response_message = 'The time is {}:{:02d}.'.format(now.hour,now.minute)
    await turn_context.send_activity(response_message)
```
    
4. Enregistrez vos modifications puis, dans le volet terminal, vérifiez que le répertoire actif est le dossier **TimeBot** contenant vos fichiers de code bot, puis entrez la commande suivante pour démarrer votre bot en cours d’exécution localement.

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```

Lorsque le bot démarre, notez que le point de terminaison sur lequel il est en cours d’exécution s’affiche.

5. Démarrez le Bot Framework Emulator et ouvrez votre bot en spécifiant le point de terminaison avec le chemin **/api/messages** ajouté, comme suit :

    `http://localhost:3978/api/messages`

6. Une fois la conversation ouverte dans un volet de **Conversation en direct**, attendez le message *Bonjour et bienvenue*.
7. Entrez un message tel que *Bonjour* et affichez la réponse du bot, qui doit me demander *Quelle heure est-il*.
8. Entrez *Quelle heure est-il ?* et affichez la réponse.

    Le bot répond désormais à la requête « Quelle heure est-il ? » en affichant l’heure locale où le bot est en cours d’exécution. Pour toute autre requête, il invite l’utilisateur à lui demander l’heure. Il s’agit d’un bot très limité, qui peut être amélioré grâce à l’intégration au service Language Understanding et au code personnalisé supplémentaire, mais il sert d’exemple fonctionnel de la façon dont vous pouvez créer une solution avec le kit de développement logiciel (SDK) Bot Framework en étendant un bot créé à partir d’un modèle.

9. Fermez le Bot Framework Emulator et revenez à Visual Studio Code, puis dans la fenêtre de terminal, entrez **Ctrl+C** pour arrêter le bot.

## <a name="more-information"></a>Plus d’informations

Pour en savoir plus sur Bot Framework, consultez l'article [Documentation du kit SDK Bot Framework](https://docs.microsoft.com/azure/bot-service/index-bf-sdk).
