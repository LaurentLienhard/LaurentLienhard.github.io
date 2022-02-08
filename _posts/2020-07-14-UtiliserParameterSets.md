---
layout: post
title: "Utiliser les Parameter Sets !"
summary: "Utiliser des jeux de paramètres pour simplifier les commandes PowerShell"
author: laurent
date: '2020-07-14 14:35:23 +0530'
category: ['PowerShell', 'Niv200']
thumbnail: /assets/img/header/Coding_1436x500.png
keywords: PowerShell, Niv200, Advanced Function
permalink: /blog/UtiliserParameterSets/
usemathjax: true
---

## Définition

Les jeux de paramètres permettent de définir un ensemble de paramètre qui doivent être utiliser par une fonction.

Cela permet par exemple de définir pour une même fonction 2 paramètres différents en entrée. Dans cette article nous allons voir comment, par exemple, faire une fonction New-MyADUser avec comme paramètre soit un ensemble de nom soit un fichier.

## Fonction avec utilisateur en entrée

Cette fonction est assez basique. Elle va prendre en paramètre d'entrée une liste de nom d'utilisateur a créer.

```powershell
function New-MyADUser {
    param (
        [System.String[]]$UserName
    )

    foreach($User in $UserName) {
        New-ADUser -Name $User -GivenName ($User.Split(".")[0]) -Surname ($User.Split(".")[1])
    }

}
```

Ensuite je peux lancer la création de mes utilisateurs

```powershell
New-MyADUser -UserName "Utilisateur1.Test","Utilisateur2.Test","Utilisateur3.Test"
```

## Fonction avec un fichier en entrée

Dans ce cas nous allons reprendre la même fonction mais avec en entrée la liste des noms dans un fichier csv.

```powershell
function New-MyADUser {
    param (
        [System.String]$FileName
    )

    $UserName = Import-Csv -Path $FileName

    foreach ($User in $UserName) {
        New-ADUser -name $User.UserName -GivenName ($User.UserName.Split(".")[0]) -Surname ($User.UserName.Split(".")[1])
    }

}
```

Ensuite je peux lancer la création de mes utilisateurs

```powershell
New-MyADUser -FileName '.\Utiliser les Parameter Sets\Utilisateurs.csv'
```

Donc maintenant nous avons 2 fonctions différentes mais qui font sensiblement la même chose : créer des utilisateurs dans l'Active Directory

C'est la que rentre en jeu les jeux de paramètres. Nous allons immédiatement voir ca :-)

## Fonction avec un fichier ou un utilisateur en entrée

### ParameterSetName

Le but donc est de combiner ces 2 fonctions en une seule mais acceptant les 2 types de paramètre en entrée.

```powershell
function New-MyADUser {
    [CmdletBinding(DefaultParameterSetName = "ByUserName")]
    param (
        [Parameter(ParameterSetName='ByUserName')]
        [System.String[]]$UserName,
        [Parameter(ParameterSetName = 'ByFileName')]
        [System.String]$FileName
    )
}
```

Première étape je combine les 2 paramètres en leur affectant un nom de jeux de paramètres (ParameterSetName).

Pour ce faire vous avez du remarquer l'ajout de ```CmdletBinding```. Pour pouvoir utiliser la commande ```ParameterSetName``` il faut passer par une fonction avancée

Naturellement il faut donner un nom différent au 2 jeux de paramètres pour pouvoir, plus tard les différencier.

Si je fais

```powershell
▶ Get-Command -Name New-MyADUser -Syntax

New-MyADUser [-UserName <string[]>] [<CommonParameters>]
New-MyADUser [-FileName <string>] [<CommonParameters>]
```

On peut voir que j'ai bien 2 façons d'appeler ma fonction avec le ```Username``` ou le ```FileName``` les 2 étant exclusif c'est a dire que je ne peux pas faire

```powershell
▶ New-MyADUser -UserName $UserName -FileName $FileName

New-MyADUser : Parameter set cannot be resolved using the specified named parameters.
At line:1 char:1
+ New-MyADUser -UserName $UserName -FileName $FileName
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (:) [New-MyADUser], ParameterBindingException
    + FullyQualifiedErrorId : AmbiguousParameterSet,New-MyADUser
```

La commande ne peut pas trouver quel jeu de paramètre elle doit executer.

Bon maintenant que je peux appeler ma fonction avec les 2 types de paramètre comment faire dans la fonction pour traiter chacun des paramètres.

### $PSCmdlet.ParameterSetName

Grâce a cette ligne de commande nous pouvons récupérer le nom du jeu de paramètre utilisé, ```ByUserName``` ou ```ByFileName``` dans notre cas, et grâce a cela faire un traitement différent selon le paramètre passé à la fonction.

Ajoutons cette ligne de commande a notre fonction

```powershell
function New-MyADUser {
    [CmdletBinding(DefaultParameterSetName = "ByUserName")]
    param (
        [Parameter(ParameterSetName='ByUserName')]
        [System.String[]]$UserName,
        [Parameter(ParameterSetName = 'ByFileName')]
        [System.String]$FileName
    )

    $PSCmdlet.ParameterSetName
}
```

Si je fait ```New-MyADUser -Username "Utilisateur.Test"``` le résultat sera

```powershell
▶ New-MyADUser -UserName "Utilisateur.test"

ByUserName
```

Si je fait ```New-MyADUser -FileName ".\Utiliser les Parameter Sets\Utilisateurs.csv"``` le résultat sera

```powershell
▶ New-MyADUser -FileName ".\Utiliser les Parameter Sets\Utilisateurs.csv"

ByFileName
```

Effectivement le paramètre ```UserName``` fait bien parti du jeu de paramètre ```ByUserName``` et ```FileName``` du jeu de paramètre ```ByFileName```

Ok donc maintenant nous savons différencier les 2 cas grâce à ```$PSCmdlet.ParameterSetName```. Il ne nous reste plus qu'a effectuer le traitement adéquat en fonction du cas.

Pour ce faire nous allons simplement ajouter une commande ```switch``` dans notre fonction et copier le contenu de nos 2 fonctions précédentes

```powershell
function New-MyADUser {
    [CmdletBinding(DefaultParameterSetName = "ByUserName")]
    param (
        [Parameter(ParameterSetName='ByUserName')]
        [System.String[]]$UserName,
        [Parameter(ParameterSetName = 'ByFileName')]
        [System.IO.DirectoryInfo]$FileName
    )

    switch ($PSCmdlet.ParameterSetName) {
        "ByUserName" {
            $PSCmdlet.ParameterSetName
            foreach ($User in $UserName) {
                New-ADUser -name $User -GivenName ($User.Split(".")[0]) -Surname ($User.Split(".")[1])
            }
         }
        "ByFileName" {
            $PSCmdlet.ParameterSetName
            $UserName = Import-Csv -Path $FileName
            foreach ($User in $UserName) {
                New-ADUser -name $User.UserName -GivenName ($User.UserName.Split(".")[0]) -Surname ($User.UserName.Split(".")[1])
            }
         }
    }
}
```

```text
Cette fonction pourrait encore être simplifiée mais ce n'est pas l'objet de cet article
```

Voilà comment faire pour l'utilisation des jeux de paramètres

Si vous avez des questions ou ...

Attendez ! je suis sur que vous vous en posez une dès à présent !

Si! Si!

Vous savez la fameuse question : Comment je fais si un paramètre doit faire parti des 2 jeux de paramètres en même temps ?

Nous allons tenter d'y répondre ;-)

### Un paramètre dans plusieurs jeux de paramètres

Bon la plus logique des approches si vous avez compris le principe serait de faire

```powershell
function New-MyADUser {
    [CmdletBinding(DefaultParameterSetName = "ByUserName")]
    param (
        [Parameter(ParameterSetName = 'ByUserName')]
        [System.String[]]$UserName,
        [Parameter(ParameterSetName = 'ByUserName')]
        [System.String]$Password,
        [Parameter(ParameterSetName = 'ByFileName')]
        [System.IO.DirectoryInfo]$FileName,
        [Parameter(ParameterSetName = 'ByFileName')]
        [System.String]$Password
    )
}
```

mais l'execution de ce code retourne l'erreur

```powershell
Duplicate parameter $Password in parameter list.
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : DuplicateFormalParameter
```

Donc on ne peut pas mettre 2 fois le même nom de paramètre !

Soit! Je ne peux pas définir 2 paramètres identiques dans 2 jeux de paramètre distinct, mais est-ce que je peux définir 1 paramètre qui sera disponible dans les 2 jeux ?

et si on essayait cela

```powershell
function New-MyADUser {
    [CmdletBinding(DefaultParameterSetName = "ByUserName")]
    param (
        [Parameter(ParameterSetName = 'ByUserName')]
        [System.String[]]$UserName,
        [Parameter(ParameterSetName = 'ByFileName')]
        [System.IO.DirectoryInfo]$FileName,

        [Parameter(ParameterSetName = 'ByUserName')]
        [Parameter(ParameterSetName = 'ByFileName')]
        [System.String]$Password
    )
}
```

Déjà l'execution de ce code ne retourne pas d'erreur : un bon point !

Regardons la syntaxe de la fonction maintenant

```powershell
▶  Get-Command -Name New-MyADUser -Syntax

New-MyADUser [-UserName <string[]>] [-Password <string>] [<CommonParameters>]
New-MyADUser [-FileName <DirectoryInfo>] [-Password <string>] [<CommonParameters>]
```

Le paramètre ```password``` se retrouve bien dans les 2 jeux de paramètres : VICTOIRE !

En extrapolant un peu, on peu s'imaginer avoir plusieurs jeux de paramètres pour une fonction. il semble évident que cela va être compliqué de mettre un paramètre dans tous les jeux de paramètres de cette façon.

Et si pour simplifier nous le mettions tous simplement dans aucun !

```powershell
function New-MyADUser {
    [CmdletBinding(DefaultParameterSetName = "ByUserName")]
    param (
        [Parameter(ParameterSetName = 'ByUserName')]
        [System.String[]]$UserName,
        [Parameter(ParameterSetName = 'ByFileName')]
        [System.IO.DirectoryInfo]$FileName,

        [System.String]$Password
    )
}
```

Le résultat est le suivant :

```powershell
▶ Get-Command -Name New-MyADUser -Syntax


New-MyADUser [-UserName <string[]>] [-Password <string>] [<CommonParameters>]

New-MyADUser [-FileName <DirectoryInfo>] [-Password <string>] [<CommonParameters>]
```

Le paramètre ```password```est toujours membre des 2 jeux de paramètre.

Il suffit donc de n'__affecter__ un paramètre __a AUCUN jeux__ de paramètres pour qu'il soit __commun à TOUS les jeux__ de paramètres.

## Conclusion

Nous avons donc vu dans cet article :

- Qu'est-ce que c'est qu'un jeu de paramètres
- A quoi sert un jeu de paramètres
- Comment affecter un paramètre à plusieurs jeux de paramètres
- Comment affecter un paramètre a tous les jeux de paramètres

les exemples de code sont disponible sur le [Github](https://github.com/HowIAutomatedThis/Exemples/tree/master/Utiliser%20les%20Parameter%20Sets)

Si vous avez des questions, si je me suis trompé sur un point ou simplement échanger avec moi n'hésitez pas à me contacter sur les réseaux sociaux.
