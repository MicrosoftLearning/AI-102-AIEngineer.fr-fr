---
lab:
  title: Activer des fournisseurs de ressources
  module: Setup
ms.openlocfilehash: 2cd48d0d9f67b9bab3e64452bf6199e1f438492a
ms.sourcegitcommit: e06fbeaa3bc15e4dbe99f72216dfeeb27f58babd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/15/2022
ms.locfileid: "145195606"
---
# <a name="enable-resource-providers"></a>Activer des fournisseurs de ressources

Certains fournisseurs de ressources doivent être inscrits dans votre abonnement Azure. Suivez ces étapes pour vous assurer qu’ils sont inscrits.

1. Connectez-vous au portail Azure à l’adresse `https://portal.azure.com` avec les informations d’identification Microsoft associées à votre abonnement.
2. Dans la page **Accueil**, sélectionnez **Abonnements** (ou développez le menu **&#8801;** , sélectionnez **Tous les services**, puis, dans la catégorie **Général**, sélectionnez **Abonnements**).
3. Sélectionnez votre abonnement Azure (si vous avez plusieurs abonnements, sélectionnez celui que vous avez créé en utilisant votre Pass Azure).
4. Dans le panneau de votre abonnement, dans le volet de gauche, dans la section **Paramètres**, sélectionnez **Fournisseurs de ressources**.
5. Dans la liste des fournisseurs de ressources, vérifiez que les fournisseurs suivants sont inscrits (si ce n’est pas le cas, sélectionnez-les et cliquez sur **Inscrire**) :
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - microsoft.insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance
