---
lab:
  title: Analyser Cognitive Services
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: e0e0042421a4f7150fc3b95cef80887c81a78f3a
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/12/2022
ms.locfileid: "145195553"
---
# <a name="monitor-cognitive-services"></a>Analyser Cognitive Services

Azure Cognitive Services peut s’avérer être un élément essentiel d’une infrastructure d’application globale. Il est important de pouvoir surveiller l’activité et d’obtenir des alertes concernant les problèmes qui nécessitent votre attention.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour cette formation

Si vous avez déjà cloné le référentiel de code **AI-102-AIEngineer** dans l’environnement dans lequel vous travaillez sur ce laboratoire, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

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
    - **Groupe de ressources** : *choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *entrez un nom unique.*
    - **Niveau tarifaire** : standard S0
3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Notez l’URI de point de terminaison. Vous en aurez besoin ultérieurement.

## <a name="configure-an-alert"></a>Configurer une alerte

Commençons la surveillance en définissant une règle d’alerte afin de détecter l’activité dans votre ressource Cognitive Services.

1. Dans le Portail Azure, accédez à votre ressource Cognitive Services et affichez sa page **Alertes** (dans la section **Surveillance**).
2. Sélectionnez **+ Nouvelle règle d’alerte**.
3. Dans la page **Créer une règle d’alerte**, sous **Étendue**, vérifiez que la ressource Cognitive Services est répertoriée.
4. Sous **Condition**, cliquez sur **Ajouter une condition** et affichez le volet **Sélectionner un signal** qui s’affiche à droite, dans lequel vous pouvez sélectionner un type de signal à surveiller.
5. Dans la liste des **types de signal**, sélectionnez **Journal d’activité**, puis dans la liste filtrée, sélectionnez **Répertorier les clés**.
6. Passez en revue l’activité des 6 dernières heures.
7. Sélectionnez l’onglet **Actions**. Notez que vous pouvez spécifier un *groupe d’actions*. Cela vous permet de configurer des actions automatisées lorsqu’une alerte est déclenchée, par exemple en envoyant une notification par e-mail. Nous ne le ferons pas dans cet exercice; mais il peut être utile de le faire dans un environnement de production.
8. Sous l’onglet **Détails**, définissez le **Nom de la règle d’alerte** sur **Alerte de liste de clés**.
9. Sélectionnez **Revoir + créer**. 
10. Passez en revue la configuration de l’alerte. Sélectionnez **Créer**, puis attendez la fin de la création de la règle d’alerte.
11. Dans Visual Studio Code, faites un clic droit sur le dossier **03-monitor** et ouvrez un terminal intégré. Ensuite, entrez la commande suivante pour vous connecter à votre abonnement Azure à l’aide d’Azure CLI.

    ```
    az login
    ```

    Un onglet de navigateur web s’ouvre et vous invite à vous connecter à Azure. Pour ce faire, fermez l’onglet du navigateur et revenez à Visual Studio Code.

    > **Conseil** : Si vous avez plusieurs abonnements, vous devez vous assurer que vous travaillez dans celui qui contient votre ressource Cognitive Services.  Utilisez cette commande pour déterminer votre abonnement actuel.
    >
    > ```
    > az account show
    > ```
    >
    > Si vous devez modifier l’abonnement, exécutez cette commande, en remplaçant *&lt;subscriptionName&gt;* par le nom d’abonnement approprié.
    >
    > ```
    > az account set --subscription <subscriptionName>
    > ```

10. Vous pouvez maintenant utiliser la commande suivante pour obtenir la liste des clés Cognitive Services, en remplaçant *&lt;resourceName&gt;* par le nom de votre ressource Cognitive Services et *&lt;resourceGroup&gt;* par le nom du groupe de ressources dans lequel vous l’avez créée.

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

La commande retourne une liste des clés de votre ressource Cognitive Services.

11. Revenez au navigateur contenant le Portail Azure et actualisez votre **page Alertes**. Vous devez voir une alerte **Sev 4** répertoriée dans la table (si ce n’est pas le cas, attendez cinq minutes et actualisez à nouveau).
12. Sélectionnez l’alerte pour afficher les détails la concernant.

## <a name="visualize-a-metric"></a>Visualiser une métrique

En plus de définir des alertes, vous pouvez afficher les métriques de votre ressource Cognitive Services pour surveiller son utilisation.

1. Dans le Portail Azure, dans la page de votre ressource Cognitive Services, sélectionnez **Métriques** (dans la section **Surveillance**).
2. S’il n’existe aucun graphique existant, sélectionnez **+ Nouveau graphique**. Ensuite, dans la liste **Métriques**, passez en revue les métriques possibles que vous pouvez visualiser et sélectionnez **Nombre total d’appels**.
3. Dans la liste **Agrégation**, sélectionnez **Nombre**.  Cela vous permettra de superviser le nombre total d’appels à votre ressource Cognitive Services, ce qui s’avère utile pour déterminer l’utilisation du service sur une période donnée.
4. Pour générer certaines demandes à votre service Cognitive Services, vous allez utiliser **curl**, un outil de ligne de commande pour les requêtes HTTP. Dans Visual Studio Code, dans le dossier **03-monitor**, ouvrez **rest-test.cmd** et modifiez la commande **curl** qu’elle contient (illustrée ci-dessous), en remplaçant *&lt;yourEndpoint&gt;* et *&lt;yourKey&gt;* par votre URI de point de terminaison et votre clé **Key1** pour utiliser l'API Analyse de texte dans votre ressource Cognitive Services.

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.1/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':           [{'id':1,'text':'hello'}]}"
    ```

5. Enregistrez vos modifications, puis dans le terminal intégré du dossier **03-monitor**, exécutez la commande suivante :

    ```
    rest-test
    ```

La commande retourne un document JSON contenant des informations sur la langue détectée dans les données d’entrée (qui doit être l’anglais).

6. Réexécutez la commande **rest-test** plusieurs fois pour générer une activité d’appel (vous pouvez utiliser la clé **^** pour parcourir les commandes précédentes).
7. Revenez à la page **Métriques** de le Portail Azure et actualisez le graphique **Nombre total d’appels**. Plusieurs minutes peuvent être nécessaires pour que les appels que vous avez effectués à l’aide de *curl* soient répercutés dans le graphique. Actualisez le graphique jusqu’à ce qu’il soit mis à jour pour les inclure.

## <a name="more-information"></a>Plus d’informations

L’une des options de surveillance de Cognitive Services consiste à utiliser la *journalisation des diagnostics*. Une fois activée, la journalisation des diagnostics capture des informations enrichies sur l’utilisation de votre ressource Cognitive Services et peut constituer un outil de supervision et de débogage utile. La génération d’informations peut prendre plus d’une heure après la configuration de la journalisation des diagnostics. C’est pourquoi nous n’avons pas exploré cet aspect dans cet exercice ; mais vous pouvez en savoir plus dans la [documentation Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/diagnostic-logging).
