---
lab:
  title: Configuration d'un environnement de labo
  module: Setup
ms.openlocfilehash: 57363ce9fec94cb3a1a6ef7622a10bca48a6f055
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195607"
---
# <a name="lab-environment-setup"></a>Configuration d'un environnement de labo

Ces exercices sont conçus pour être effectués dans un environnement de labo hébergé. Si vous souhaitez les terminer sur votre propre ordinateur, vous pouvez le faire en installant le logiciel suivant. Vous pouvez rencontrer des dialogues et un comportement inattendus lors de l’utilisation de votre propre environnement. En raison de la large gamme de configurations locales possibles, l’équipe de formation ne peut pas prendre en charge les problèmes que vous pouvez rencontrer dans votre propre environnement.

> **Remarque** : les instructions ci-dessous s’appliquent aux ordinateurs Windows 10. Vous pouvez également utiliser Linux ou MacOS. Vous devrez peut-être adapter les instructions du labo pour votre système d’exploitation choisi.

### <a name="base-operating-system-windows-10"></a>Système d'exploitation de base (Windows 10)

#### <a name="windows-10"></a>Windows 10

Installez Windows 10 et appliquez toutes les mises à jour.

#### <a name="edge"></a>Edge

Installez [Edge (Chromium)](https://microsoft.com/edge)

### <a name="net-core-sdk"></a>SDK .NET Core

1. Accédez à la page https://dotnet.microsoft.com/download pour procéder au téléchargement et à l'installation. Téléchargez le kit de développementlogiciel (SDK) .NET Core, pas seulement le runtime)

### <a name="c-redistributable"></a>C++ Redistributable

1. Téléchargez et installez le Visual C++ Redistributable (x64) à partir de https://aka.ms/vs/16/release/vc_redist.x64.exe.

### <a name="nodejs"></a>Node.JS

1. Téléchargez la dernière version de LTS à partir de https://nodejs.org/en/download/ 
2. Installez la version à l’aide des options par défaut

### <a name="python-and-required-packages"></a>Python (et packages requis)

1. Téléchargez la version 3.8 à partir de https://docs.conda.io/en/latest/miniconda.html 
2. Exécutez le programme d'installation **Important**: Sélectionnez les options pour ajouter Miniconda à la variable PATH et pour l'enregistrer comme environnement Python par défaut.
3. Après l’installation, ouvrez l’invite Anaconda et entrez les commandes suivantes pour installer les packages : 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### <a name="azure-cli"></a>Azure CLI

1. Téléchargez depuis https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest 
2. Installez la version à l’aide des options par défaut

### <a name="git"></a>Git

1. Télécharger et installer à partir de https://git-scm.com/download.html, à l’aide des options par défaut


### <a name="visual-studio-code-and-extensions"></a>Visual Studio Code (et extensions)

1. Téléchargez depuis https://code.visualstudio.com/Download 
2. Installez la version à l’aide des options par défaut 
3. Après l'installation, démarrez Visual Studio Code et sous l'onglet **Extensions** (CTRL+Maj+X), recherchez et installez les extensions suivantes depuis Microsoft :
    - Python
    - C#
    - Azure Functions
    - PowerShell


### <a name="bot-framework-emulator"></a>Bot Framework Emulator

Suivez les instructions présentées à l’adresse https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md pour télécharger et installer la dernière version stable du Bot Framework Emulator pour votre système d’exploitation.

### <a name="bot-framework-composer"></a>Bot Framework Composer

Installer à partir de https://docs.microsoft.com/en-us/composer/install-composer.
