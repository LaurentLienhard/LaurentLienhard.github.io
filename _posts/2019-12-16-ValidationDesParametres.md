---
layout: post
title: "La validation des paramètres"
summary: "La validation des paramètres dans une fonction Powershell"
author: laurent
date: '2019-12-16 14:35:23 +0530'
category: ['PowerShell', 'Niv100']
thumbnail: /assets/img/header/parameter_validation.png
keywords: PowerShell, Regex, Parameter, Niv100, Advanced Function
permalink: /blog/ValidationDesParametres/
usemathjax: true
---

## La validation des paramètres

Dans cette article nous allons parler d'une option disponible dans Powershell : la validation des paramètres

Cette option a pour but, comme son nom l'indique, de valider qu'un paramètre passé à une fonction est bien celui que l'on attends.

Tout au long de cette article nous garderons le script suivant comme fil conducteur.

```powershell
function Do-SomeThing {
    [CmdletBinding()]
    param (
        [String[]]$ComputerName,
        [Int]$Age
    )

    begin {
        #La valeur de computername ne doit pas être nul ou vide
        if (($null -eq $ComputerName) -or ($ComputerName -eq "")) {
            throw "La valeur de ComputerName ne peut pas être vide ou null"
        }
        #Test pour vérifier que le nom du computer ne dépasse pas 13 caractères
        If ($ComputerName.Length -gt 13) {
            Throw "Le nom du Computer ne doit pas dépasser 13 caractères"
        }

        #L'age de l'ordinateur doit-être compris entre 5 et 10 ans
        if (!(($Age -gt 5) -and ($Age -lt 10))) {
            throw "l'age doit-être compris entre 5 et 10 ans"
        }

    }

    process {
    }

    end {
    }
}

```

## [ValidateNotNullOrEmpty()] and [ValidateNotNull()]

```ValidateNotNull``` : vérifie uniquement si la valeur transmise au paramètre est une valeur null. Cela fonctionnera toujours s'il est passé une chaîne vide.

```ValidateNotNullOrEmpty``` : vérifie également si la valeur transmise est une valeur null et s'il s'agit d'une chaîne ou d'une collection vide

Dans notre exemple nous pouvons remplacer le test suivant

```powershell
if (($null -eq $ComputerName) -or ($ComputerName -eq "")) {
    throw "La valeur de ComputerName ne peut pas être vide ou null"
}
```

en modifiant simplement la déclaration de notre variable $ComputerName

```powershell
Param (
    [ValidateNotNullOrEmpty()]
    [String[]]$ComputerName
)
```

## [ValidateLength()]

Cette validation permet de définir une taille minimale et maximale attendue pour un paramètre.

Dans notre exemple nous voulons que le paramètre ```ComputerName``` fasse minimum 1 caractère et au maximum 13 caractères.

Nous pouvons donc remplacer le test :

```powershell
    If ($ComputerName.Length -gt 13) {
        Throw "Le nom du Computer ne doit pas dépasser 13 caractère"
    }
```

par la déclaration de paramètre suivant :

```powershell
Param (
    [ValidateLength(1,13)]
    [String[]]$ComputerName
)
```

Mais attendez ! Nous avions déjà modifié la déclaration de ce paramètre précédemment pour ajouter ```[ValidateNotNullOrEmpty()]``` ?

Effectivement les validations des paramètres peuvent être cumulées.
Dans notre cas nous déclarons le paramètre de cette façon :

```powershell
    param (
        [ValidateNotNullOrEmpty()]
        [ValidateLength(1,13)]
        [String[]]$ComputerName
    )
```

```ComputerName``` ne doit-être ni Null, ni vide et doit faire entre 1 et 13 caractères.

## [ValidateRange()]

C'est en quelque sorte le pendant de [ValidateLength()] mais pour les chiffres.
Comme pour [ValidateLength()], il permet de définir une valeur minimale et maximale que peut prendre un paramètre.

Dans notre exemple l'age du PC doit-être compris entre 5 et 10 ans.
Nous avions donc le test suivant :

```powershell
    if (!(($Age -gt 5) -and ($Age -lt 10))) {
        throw "l'age doit-être compris entre 5 et 10 ans"
    }
```

nous pouvons donc remplacer ce test par :

```powershell
param (
    [ValidateRange(5,10)]
    [int]$Age
    )
```

## [ValidateCount()]

Comme pour ValidateRange() il permet de définir le nombre minimal et le nombre maximal d'objet que peux prendre une collection passée en paramètre.

```powershell
    param (
        [ValidateNotNullOrEmpty()]
        [ValidateLength(1,13)]
        [ValidateCount(1,3)]
        [String[]]$ComputerName
    )
```

Dans notre exemple le paramètre ```ComputerName``` ne doit-être ni Null, ni vide, doit faire entre 1 et 13 caractères et nous ne pouvons passer que 1 à 3 objects dans la collection.

## [ValidateSet()]

Cette validation permet de fournir une liste de réponse possible pour la valeur du paramètre.

Cette validation peut-être rendu Case Sensitive, si à la fin de la déclaration, la valeur ignorecase est définie à $False

le test de validation suivant

```powershell
        #Être sur que la valeur est l'une de celle que je veux
        if("SIEGE","AGENCE","STOCK" -notcontains $Site) {
            throw "Le site doit-être SIEGE,AGENCE ou STOCK"
        }
```

peut-être remplacer par la déclaration de la variable ```Site``` suivante

```powershell
    param (
        [ValidateSet("SIEGE","AGENCE","STOCK",ignorecase=$true)]
        [String]$Site
    )
```

Dans notre exemple, la paramètre ne peut prendre qu'une des 3 valeurs définies.

Nous avons également demandé à ce que le valeur du paramètre ne soit pas Case
Sensitive donc SIEGE, Siege, sieGE sont toutes des valeurs corrects.

## [ValidatePattern()]

Permet d'utiliser une expression régulière (REGEX) avant de tester la valeur d'un paramètre.

Dans notre script, nous avins le test suivant

```powershell
        #Test pour savoir si le ComputerName correspond la nomenclature de l'entreprise
        if ($ComputerName -notmatch "^SRV-\D{6}\d{3}$") {
            Throw "Le ComputerName doit-être sous la forme SRV-LETTRE(6)CHIFFRE(3)"
        }
```

Nous voulons donc être sur que le paramètre ```ComputerName``` soit toujours sous la forme d'un string commençant par SRV- suivi de 6 lettres et de 3 chiffres.

Nous pouvons remplacer ce test par la validation

```powershell
    param (
        [ValidateNotNullOrEmpty()]
        [ValidateLength(1,13)]
        [ValidateCount(1,3)]
        [ValidatePattern('^SRV-\D{6}\d{3}$')]
        [String[]]$ComputerName
    )
```

## [ValidateScript()]

Cette validation permet de définir un script qui va être executer pour valider la valeur du paramètre.

Notre test pour valider l'existence du chemin

```powershell
        #Savoir si le chemin passé en paramètre existe et est un répertoire
        IF (!(Test-Path $Path -PathType ‘Container’))
        {
            Throw "$($Path) n'est pas un dossier valide"
        }
```

peut-être remplacer par la validation suivante

```powershell
    param (
        [ValidateScript(
            {IF (!(Test-Path -Path $_)) {
                Throw "$($Path) n'est pas un dossier valide"
            } else {
                $true
            }
            }
        )]
        [String]$Path
    )
```

Comme vous pouvez l'imaginer, le script peut-être quelque chose de plus compliqué qu'un simple test comme ici.

## Conclusion

Pour conclure nous pouvons comparer simplement nos 2 versions de script.

Le script d'origine

```powershell
function Do-SomeThing {
    [CmdletBinding()]
    param (
        [String[]]$ComputerName,
        [Int]$Age
    )

    begin {
        #La valeur de computername ne doit pas etre nul ou vide
        if (($null -eq $ComputerName) -or ($ComputerName -eq "")) {
            throw "La valeur de ComputerName ne peut pas être vide ou null"
        }
        #Test pour vérifier que le nom du computer ne dépasse pas 13 caractères
        If ($ComputerName.Length -gt 13) {
            Throw "Le nom du Computer ne doit pas dépasser 13 caractères"
        }

        #L'age de l'ordinateur doit-être compris entre 5 et 10 ans
        if (!(($Age -gt 5) -and ($Age -lt 10))) {
            throw "l'age doit-être compris entre 5 et 10 ans"
        }

    }

    process {
    }

    end {
    }
}
```

Le script final

```powershell
function Do-SomeThing {
    [CmdletBinding()]
    param (
        [ValidateNotNullOrEmpty()]
        [ValidateLength(1,13)]
        [ValidateCount(1,3)]
        [ValidatePattern('^SRV-\D{6}\d{3}$')]
        [String[]]$ComputerName,
        [ValidateScript({IF (!(Test-Path -Path $_)) {
            Throw "$($Path) n'est pas un dossier valide"
        } else {
            $true
        }
        })]
        [String]$Path,
        [ValidateSet("SIEGE","AGENCE","STOCK",ignorecase=$true)]
        [String]$Site,
        [ValidateRange(5,10)]
        [int]$Age
    )

    begin {
    }

    process {
    }

    end {
    }
}
```

Nous constatons assez clairement que nous avons diminué les lignes de code nécessaire pour valider les paramètres et de ce fait simplifier la compréhension du script.
