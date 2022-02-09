---
layout: post
title: "Code Debugging"
summary: "Utilisation basique du debug dans VScode"
author: laurent
date: '2022-02-09 14:35:23 +0530'
category: ['PowerShell', 'Niv100']
thumbnail: /assets/img/header/Coding_1436x500.png
keywords: PowerShell, Niv100, Debugging, VSCode
permalink: /blog/CodeDebug/
usemathjax: true
---

## _C'est quoi le debugging ?_

Le débogage est le processus de détection et de suppression des erreurs existantes et potentielles (également appelées «bogues») dans un code logiciel qui peuvent provoquer un comportement inattendu ou un plantage.

Pour déboguer un programme, l'utilisateur doit c

Les outils de débogage (appelés débogueurs) sont utilisés pour identifier les erreurs de codage à différentes étapes de développement. Ils sont utilisés pour reproduire les conditions dans lesquelles l'erreur s'est produite, puis examiner l'état du programme à ce moment et localiser la cause de l'erreur.

Les programmeurs peuvent suivre l'exécution du programme étape par étape en évaluant la valeur des variables et arrêter l'exécution si nécessaire pour obtenir la valeur des variables ou réinitialiser les variables du programme

Dans cet article je vais m'attarder au débogage dans le langage PowerShell dans Visual Studio Code.

Si tu ne connais pas Visual Studio Code je t'invite à l'installer immédiatement en allant sur le [site](https://code.visualstudio.com/).

C'est à mon avis le meilleur éditeur du moment pour coder

## _Debugging dans VSCode_

### __Les points d'arrêt__

Le point d'arrêt, comme son non l'indique, est un point que l'on va poser dans notre code pour stopper l'execution du code a un endroit.

Dans VSCode il y a plusieurs façons de définir un point d'arrêt. Le point d'arrêt se visualise par un point rouge apparaissant dans la barre de défilement du code.

* la touche F9

Se placer sur la ligne de code et faire F9. (un nouvel appui sur F9 supprime le point d'arrêt)
![Point d'Arrêt F9](/assets/img/posts/20220209/pointarretF9.png "Point d'Arrêt F9")

* la palette de commande

Se placer sur la ligne de code et faire F1 puis chercher ```Toggle Breakpoint```
![Point d'Arrêt F9](/assets/img/posts/20220209/pointarretpalette.png "Point d'Arrêt F9")

* le menu ```Run```

Se placer sur la ligne de code et cliquer sur ```Run``` puis sur ```Toggle Breakpoint```

![Point d'Arrêt menu](/assets/img/posts/20220209/pointarretmenu.png "Point d'Arrêt Menu")

```text
On peut voir dans ce menu qu'il y a un autre sous-menu New Breakpoint qui propose d'autre type de points d'arrêt. 

Cela sera l'objet d'un autre article
```

## _mode Debug_

une fois le (ou les points d'arrêt) placé(s), on peut démarrer le débogage

* En appuyant sur F5

* En cliquant sur le menu ```Run``` puis ```Start Debugging```

L'execution du script va se faire et il va se mettre en pause sur le premier point d'arrêt qu'il rencontre

Plusieurs éléments apparaissent dans VSCode en mode Debug

* Une barre de menu sur le haut de l'écran

![Debug Bar](/assets/img/posts/20220209/debugbar.png "Debug Bar")

* une série de pallette sur le coté (droit ou gauche selon votre configuration)

![Palette](/assets/img/posts/20220209/palette.png "Palette")

### __la barre de debug__

Sur la barre de debug, on peut voir les informations suivantes :

![Continue](/assets/img/posts/20220209/BarDebugRestart.png) Continue/Pause [F5] => Cette commande reprend le programme et continue de l'exécuter normalement, sans s'arrêter à moins qu'il n'atteigne un autre point d'arrêt.

![Step Over](/assets/img/posts/20220209/BarDebugStepOver.png) Step Over [F10] => La commande Step Over prend une seule étape. Il exécute la ligne actuellement en surbrillance, puis s'arrête à nouveau

![Step Into](/assets/img/posts/20220209/BarDebugStepInto.png) Step Into [F11] => La commande Step Into fonctionne de la même manière que Step Over, sauf lorsqu'elle touche une fonction. Sur une fonction, Step Into entre dans cette fonction et s'arrête sur la première ligne à l'intérieur

![Step Out](/assets/img/posts/20220209/BarDebugStepOut.png) Step Out [Shift+F11] => La commande Step Out exécute tout le code de la fonction en cours, puis s'arrête à l'instruction suivante (s'il y en a une). En d'autres termes, cela vous permet de sortir de la fonction actuelle en un seul grand saut

![Restart](/assets/img/posts/20220209/BarDebugRestart.png) Restart [Ctrl+Shift+F5] => relance le debug du script

![Stop](/assets/img/posts/20220209/BarDebugStop.png) Stop [Shift+F5] => stop le mode debug

### __Comment récupérer la valeur d'une variable ?__

Une  fois le point d'arrêt définis et le mode debug lancé, le script stop au point d'arrêt.

A partir d'ici on peut vérifier la valeur d'une variable par exemple dans la console

![Variable dans la console](/assets/img/posts/20220209/VariableDansLaConsole.png)

Dans cette exemple, la variable ```$files``` ne retourne rien car la ligne 9 ne c'est pas encore exécutée. Pour executer la ligne 9 il suffit de cliquer sur F10 pour passer à la ligne de code suivante.

![Variable dans la console](/assets/img/posts/20220209/VariableDansLaConsole1.png)

L'exécution de la ligne 9 se fait et le debugger stop à la ligne suivante. Maintenant la variables ```$files``` contient bien les fichiers contenus dans le dossier

On peut également retrouver toutes les variables liées au script dans la palette ```VARIABLES``` sur le coté

![Variable dans la palette](/assets/img/posts/20220209/VariableDansLaPalette.png)

Et enfin on peut utiliser le palette ```WATCH``` pour voir les modifications de la variable.

![Variable dans la palette](/assets/img/posts/20220209/VariableDansLaPalette1.png)

Cliquer sur le signe + et mettre le nom de la variable que vous voulez suivre (ici je veux suivre la valeur de ```$x```)

Utiliser la touche F10 pour continuer le debug et voir la valeur de la variable ```$x``` changer dans la palette ```WATCH```

![Variable dans la palette](/assets/img/posts/20220209/VariableDansLaPalette2.png)

![Variable dans la palette](/assets/img/posts/20220209/VariableDansLaPalette3.png)

A chaque passage dans la boucle ```ForEach``` la valeur de la variable ```$x``` change dans la palette ```WATCH```
