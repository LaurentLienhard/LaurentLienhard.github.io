---
layout: post
title: "Remote - Containers"
summary: "Exploration de la possibilité de coder sur des environnements distants"
author: laurent
date: '2020-07-27 14:35:23 +0530'
category: ['PowerShell', 'Niv200']
thumbnail: /assets/img/header/Coding_1436x500.png
keywords: PowerShell, Niv200, Code, Remote
permalink: /blog/RemoteContainers/
usemathjax: true
---

## Introduction

Si comme moi vous testez des tas de choses sur votre machine on se retrouve rapidement avec une machine qui n'a plus vraiment un environnement de travail maîtrisé ;-)

Vous vous êtes sûrement poser la question

```text
Comment être sur de travailler sur un environnement de développement maîtrisé ?
```

Au début je tournais avec des machines virtuelles HyperV sur mon portable mais je me suis très vite retrouvé limité par les ressources de ma machine.

Je faisais tourner une VM windows 10 compléte pour faire du développement Powershell par exemple ou pour tester d'autre langage comme GO ou Javascript.

Je cherchais une solution qui soit :

* légère a deployer
* maîtrisée - invariable dans le temps
* facilement réutilisable

Après quelques recherches sur Internet, je suis tombé par hasard sur une vidéo présentant l'extension Visual Studio Code ```Remote Development```

```Remote Development``` est en fait un pack contenant les extensions :
* Remote - SSH
* Remote - WSL
* Remote - Containers

la partie qui va nous intéresser aujourd'hui est ```Remote - Containers```

## Remote - Containers

### Installation

```text
Je ne vais pas expliquer dans cette article ce qu'est docker ou les containers ce n'est pas le sujet de l'article
```

Ma machine de travail étant sous Windows 10 il faut installer en pré-requis  [DOCKER](https://www.docker.com/products/docker-desktop)

Une fois l'installation de Docker effectuée, il faut installer l'extension ```Remote - Development``` dans Visual Studio Code :

![Installation Extension](/assets/img/posts/20200727/InstallationExtension.png "Installation Extension")

Une fois installé, on retrouve bien les 3 extensions contenues dans le pack

![Liste Extension](/assets/img/posts/20200727/ListeExtension.png "Liste Extension")

### Test

L'extension possède un mode de test que nous allons utiliser tout de suite pour vérifier l'installation.

Pour tester il suffit de cliquer sur l'icône apparue en bas a gauche dans VSCode

![Icône Extension](/assets/img/posts/20200727/IconeExtension.png "Icône Extension")

De choisir dans le menu ```Try a sample```

![Test Extension](/assets/img/posts/20200727/TryExtension.png "Test Extension")

On va prendre l'environnement ```Node``` pour tester

![Test Node](/assets/img/posts/20200727/NodeEnv.png "Test Node")

Une fois l'environnement choisi, VSCode va se rouvrir et démarrer automatiquement un container avec toute la configuration nécessaire

![Start Node](/assets/img/posts/20200727/StartNodeEnv.png "Start Node")

Dans le terminal on peut vérifier que nous sommes bien dans une environnement Node

![Node Version](/assets/img/posts/20200727/NodeVersion.png "Node Version")

Pour quitter cette environnement de travail il faut de nouveau cliquer sur l'icone en bas a gauche de VSCode et chossir ```Close remote Connection``` dans le menu

![Close Node](/assets/img/posts/20200727/CloseNodeEnv.png "Close Node")

### Powershell

Ok maintenant que l'on a vu comment installer l'extension et qu'on a vérifié que tout fonctionne nous allons voir comment configurer un environnement de développement pour Powershell

![Test Extension](/assets/img/posts/20200727/TryExtension.png "Test Extension")

Dans cette capture d'écran nous voyons 3 possibilité qui sont assez claire :

* Open folder (ouvrir un dossier existant dans un container)
* Open Workspace (ouvrir un workspace VSCode dans un container)
* Open repository (ouvrir un dépôt GIT dans un container)

Je vais utiliser la première dans cette article. Pour se faire je créé un répertoire dans mon arborescence qui va me servir de test

![PS DEMO](/assets/img/posts/20200727/PSDemo.png "PS Demo")

Comme pour les tests, je clique sur l'icone en bas a gauche dans VSCode et je choisis ```Open folder in container``` et choisir le dossier précédemment crée.

Une fenêtre de selection d'environnement un peu plus étoffée que la précédente apparaît

![Selection Env](/assets/img/posts/20200727/SelectionEnv.png "Selection Env")

Dans cette fenêtre rechercher ```Powershell``` et le lancer

Première remarque : nous sommes dans une machine sous linux

```powershell
root@5056e6336fe2:/workspaces/POWERSHELL# uname
Linux
```

ce qui m'amène a la seconde remarque : nous sommes sur la dernière version de PS CORE ;-)

```powershell
root@5056e6336fe2:/workspaces/POWERSHELL# pwsh
PowerShell 7.0.3
Copyright (c) Microsoft Corporation. All rights reserved.

https://aka.ms/powershell
Type 'help' to get help.
```

2 fichiers sont crées :

* devcontainer.json (contient la configuration du conteneur de VS Code) : est utilisé pour lancer votre conteneur de développement. Vous pouvez également spécifier les extensions à installer une fois que le conteneur est en cours d'exécution ou post-créer des commandes pour préparer l'environnement
* Dockerfile (contient la définition de la machine docker)

En regardant dans le fichier ```devcontainer.json``` on remarque une partie extension. Effectivement l'avantage de cette solution est que justement comme nous sommes dans un container nos environnements locaux et ```"containeurisés"``` sont distincts.

0n le voit bien en regardant dans la liste des extensions installées, on a bien des extensions locales et des extensions dans le container. Ici dans le container je n'ai que Powershell puisque c'est la seule extension que je demande a installer dans le fichier ```devcontainer.json```

![Extension Container](/assets/img/posts/20200727/ListeExtensionContainer.png "Extension Container")

je vais ajouter une extension Code Spell Checker et le pack de langue francais.

Pour ce faire je modifie le fichier en ajoutant :

```powershell
"extensions": [
    "ms-vscode.powershell",
    "streetsidesoftware.code-spell-checker",
    "streetsidesoftware.code-spell-checker-french"
    ],
```

En cliquant sur l'icone en bas a gauche on peut choisir le menu ```Rebuild Container```

![Rebuild Container](/assets/img/posts/20200727/RebuildContainer.png "Rebuild Container")

En revérifiant la liste des extensions on voit bien que les nouvelles extensions sont installées dans le container mais pas sur ma machine locale

![Extension Container](/assets/img/posts/20200727/ListeExtensionContainer2.png "Extension Container")

PARFAIT ! je peux donc configurer mon VSCode différemment pour chaque environnement de développement.

Après les extensions dans notre environnement de développement on peut également configurer tous les modules nécessaires.

Pour se faire dans le fichier ```devcontainer.json``` j'ajoute la commande suivante :

```powershell
"postStartCommand": "pwsh -c 'Install-module PSHTML,PSAKE,PLASTER -scope CurrentUser -force'"
```

en faisant ça, je demande à mon container d'executer la commande après son démarrage et d'installer les modules PSAKE, PLASTER et [PSHTML](https://github.com/Stephanevg/PSHTML) (de mon ami Stéphane Van Gullick)

RE-PARFAIT ! Je peux configurer mon environnement de développement comme je le souhaites.

## Conclusion

Si nous reprenons les critères que nous avions définis au début de l'article

* légère a deployer
* maîtrisée - invariable dans le temps
* facilement réutilisable

On peut dire, je pense, que l'extension ```Remote - Container``` y répond

## PS : Apprendre un nouveau langage (GO, Node.js, ...)

Personnellement je pense utiliser l'extension ```Remote -Container``` dans le cadre de mon apprentissage de nouveau langage de développement.

Je m'intéresse, par exemple, a GO et Node.js et je peux donc utilise cette extension pour monter des environnements de test sans impacter ma machine personnelle ;-)

Autre avantage et pas des moindres, si vous choisissez Node.js comme environment tout est installé, il ne reste plus qu'a coder !

![Node Container](/assets/img/posts/20200727/NodeContainer.png "Node Container")

```powershell
node@07d7f88a44d2:/workspaces/vscode-remote-try-node$ node --version
v12.18.1
```

et voila un environnement node.js tous propre ;-)
