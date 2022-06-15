---
lab:
  title: Utiliser un conteneur Cognitive Services
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 244ab1ef3754e668d64996dece9711682651691d
ms.sourcegitcommit: 29a684646784fe4f7370343b6c005728a953770d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2022
ms.locfileid: "145195560"
---
# <a name="use-a-cognitive-services-container"></a>Utiliser un conteneur Cognitive Services

L’utilisation de services Cognitive Services hébergés dans Azure permet aux développeurs d’applications de se concentrer sur l’infrastructure de leur propre code tout en tirant parti des services évolutifs gérés par Microsoft. Toutefois, dans de nombreux scénarios, les organisations nécessitent davantage de contrôle sur leur infrastructure de service et les données transmises entre les services.

La plupart des API Cognitive Services peuvent être empaquetées et déployées dans un *conteneur*, ce qui permet aux organisations d’héberger Cognitive Services dans leur propre infrastructure; par exemple, dans des serveurs Docker locaux, des Azure Container Instances ou des clusters Azure Kubernetes Services. Les services Cognitive Services en conteneur doivent communiquer avec un compte Cognitive Services basé sur Azure pour prendre en charge la facturation ; mais les données d’application ne sont pas transmises au service principal, et les organisations ont un meilleur contrôle sur la configuration de déploiement de leurs conteneurs, ce qui permet des solutions personnalisées pour l’authentification, l’extensibilité et d’autres considérations.

## <a name="clone-the-repository-for-this-course"></a>Cloner le référentiel pour ce cours

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
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S0
3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## <a name="deploy-and-run-a-text-analytics-container"></a>Déployer et exécuter un conteneur Analyse de texte

De nombreuses API Cognitive Services couramment utilisées sont disponibles dans des images conteneur. Pour obtenir une liste complète, consultez la [documentation Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#container-availability-in-azure-cognitive-services). Dans cet exercice, vous allez utiliser l’image conteneur pour l’API de *détection de langue* Analyse de texte; mais les principes sont les mêmes pour toutes les images disponibles.

1. Dans le Portail Azure, dans la page **Accueil**, sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *container instances* et créez une ressource **Container Instances** avec les paramètres suivants :

    - **Paramètres de base**:
        - **Abonnement** : *votre abonnement Azure*
        - **Groupe de ressources** : *Choisissez le groupe de ressources comprenant votre ressource Cognitive Services*
        - **Nom du conteneur** : *Entrez un nom unique*
        - **Région** : *choisissez n’importe quelle région disponible*
        - **Source d’image** : Autre registre
        - **Type d’image** : Cloud
        - **Image** : `mcr.microsoft.com/azure-cognitive-services/textanalytics/language:1.1.013570001-amd64`
        - **Type de système d’exploitation** : Linux
        - **Size** : 1 processeur virtuel, 4 Go de mémoire
    - **Mise en réseau** :
        - **Type de mise en réseau** : Cloud
        - **Étiquette du nom DNS** : *Entrez un nom unique pour le point de terminaison du conteneur*
        - **Ports** : *Modifier le port TCP de 80 à **5000***
    - **Avancé** :
        - **Stratégie de redémarrage** : Si échec
        - **Variables d’environnement** :

            | Marquer comme sécurisé | Clé | Valeur |
            | -------------- | --- | ----- |
            | Oui | `ApiKey` | *Une des clés de votre ressource Cognitive Services* |
            | Yes | `Billing` | *L’URI de point de terminaison pour votre ressource Cognitive Services* |
            | No | `Eula` | `accept` |

        - **Remplacement de commande** : [ ]
    - **Étiquettes** :
        - *N’ajoutez aucune étiquette*

2. Attendez la fin du déploiement, puis accédez à la ressource déployée.
3. Observez les propriétés suivantes de votre ressource d’instance de conteneur dans sa page **Vue d’ensemble** :
    - **État** : Il doit être *En cours d’exécution*.
    - **Adresse IP** : Il s’agit de l’adresse IP publique que vous pouvez utiliser pour accéder à vos instances de conteneur.
    - **FQDN** : Il s’agit du *nom de domaine complet* de la ressource d’instances de conteneur. Vous pouvez l’utiliser pour accéder aux instances de conteneur au lieu de l’adresse IP.

    > **Remarque** : Dans cet exercice, vous avez déployé l’image conteneur Cognitive Services pour la traduction de texte vers une ressource Azure Container Instances (ACI). Vous pouvez utiliser une approche similaire pour la déployer sur un hôte *[Docker](https://www.docker.com/products/docker-desktop)* sur votre propre ordinateur ou réseau en exécutant la commande suivante (sur une seule ligne) afin de déployer le conteneur de détection de langue sur votre instance Docker locale, en remplaçant *&lt;yourEndpoint&gt;* et *&lt;yourKey&gt;* par votre URI de point de terminaison et l’une des clés de votre ressource Cognitive Services.
    >
    > ```
    > docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 mcr.microsoft.com/azure-cognitive-services/textanalytics/language Eula=accept Billing=<yourEndpoint> ApiKey=<yourKey>
    > ```
    >
    > La commande recherche l’image sur votre ordinateur local et, si elle ne la trouve pas, elle l’extrait du registre d’images *mcr.microsoft.com* et la déploie sur votre instance Docker. Une fois le déploiement terminé, le conteneur démarre et écoute les demandes entrantes sur le port 5000.

## <a name="use-the-container"></a>Utiliser le conteneur

1. Dans Visual Studio Code, dans le dossier **04-containers**, ouvrez **rest-test.cmd** et modifiez la commande **curl** qu’elle contient (illustrée ci-dessous), en remplaçant *&lt;your_ACI_IP_address_or_FQDN&gt;* par l’adresse IP ou le nom de domaine complet de votre conteneur.

    ```
    curl -X POST "http://<your_ACI_IP_address_or_FQDN>:5000/text/analytics/v3.0/languages?" -H "Content-Type: application/json" --data-ascii "{'documents':[{'id':1,'text':'Hello world.'},{'id':2,'text':'Salut tout le monde.'}]}"
    ```

2. Enregistrez les modifications apportées au script. Notez que vous n’avez pas besoin de spécifier le point de terminaison ou la clé Cognitive Services : la requête est traitée par le service conteneurisé. Le conteneur communique régulièrement avec le service dans Azure pour signaler l’utilisation de la facturation, mais n’envoie pas de données de demande.
3. Cliquez avec le bouton droit de la souris sur le dossier **04-containers** et ouvrez un terminal intégré. Puis entrez la commande suivante pour exécuter le script :

    ```
    rest-test
    ```

4. Vérifiez que la commande retourne un document JSON contenant des informations sur les langues détectées dans les deux documents d’entrée (qui doivent être l’anglais et le français).

## <a name="clean-up"></a>Nettoyer

Si vous avez terminé les tests de votre instance de conteneur, vous devez la supprimer.

1. Dans le Portail Azure, ouvrez le groupe de ressources dans lequel vous avez créé vos ressources pour cet exercice.
2. Sélectionnez la ressource d’instance de conteneur et supprimez-la.

## <a name="more-information"></a>Plus d’informations

Pour plus d’informations sur la conteneurisation de Cognitive Services, consultez la [documentation sur les conteneurs Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/containers/).
