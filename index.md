---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
ms.openlocfilehash: ba3d2a5154e5e9b08e750f7d9ced8b62048dd66d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195581"
---
# <a name="content-directory"></a>Répertoire de contenu

Ce référentiel contient les exercices pratiques de labo pour le cours Microsoft [AI-102 Conception et implémentation d’une solution Microsoft Azure AI](https://docs.microsoft.com/learn/certifications/courses/ai-102t00) et les [modules auto-rythmés sur Microsoft Learn](https://aka.ms/AzureLearn_AIEngineer) équivalents. Les exercices sont conçus pour accompagner les supports de cours d'apprentissage et vous permettre de vous exercer à utiliser les technologies qu'ils décrivent.

Pour accomplir ces exercices, vous devez disposer d’un abonnement Microsoft Azure. Si votre instructeur ne vous en a pas fourni un, vous pouvez vous inscrire pour un essai gratuit sur [https://azure.microsoft.com](https://azure.microsoft.com).

## <a name="labs"></a>Laboratoires

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| Module | Laboratoire |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

