---
layout: post
title: "Montre moi ton profile !"
summary: "Présentation des différents profiles PowerShell"
author: laurent
date: '2020-01-02 14:35:23 +0530'
category: ['PowerShell', 'Niv100', 'Lightning', 'FrPSUG']
thumbnail: /assets/img/header/Coding_1436x500.png
keywords: PowerShell, Profile, Niv100, Lightning
permalink: /blog/MontreMoiTonProfile/
usemathjax: true
---

## Information

Cette article a été rédigé pour une présentation du [French Powershell UserGroup](https://frpsug.com).

Cette présentation peut-être vu sur Youtube sur la [chaîne du FRPSUG](https://youtu.be/kRwxrg7c94o)

## C'est quoi un profile ?

Évidement c'est la première question que l'on peut se poser ;-)

Un profile PowerShell est simplement un script :

* avec un nom spéciale
* qui se trouve dans un répertoire spécial
* qui s'execute au démarrage de PowerShell

Le (ou plutôt les) profile sert à customiser votre environnement :

* ajouter des alias
* ajouter des fonctions
* connecter des drive
* ...

## Les différents types de profile

Dans le point précédent, je fais allusion à plusieurs profiles, effectivement il existe un profile pour chaque hôte :

* Windows Powershell
* Powershell (core)

mais il y a également une profile qui s'applique à tous les hôtes

Comme pour les profiles d'hôte, Il existe un profile pour tous les utilisateurs d'une machine ou pour l'utilisateur courant.

Donc en résumé un profile s'applique :

* à l'hôte actuellement utilisé (CurrentHost) ou à tous les hôtes (AllHosts)
* à l'utilisateur actuellement connecté (CurrentHost) ou à tous les utilisateurs (AllUsers)

Les combinaisons possibles sont donc :

* AllUsers, AllHosts
* AllUsers, CurrentHost
* CurrentUser, AllHosts
* CurrentUser, CurrentHost

L'ordre ci-dessus n'est pas anodin, il s'agit de l'ordre dans lequel les profiles sont chargés au démarrage d'une console

Pour adresser ces différents profiles on utilise la variable ```$PROFILE``` de la façon suivante :

```text
* AllUsers, AllHosts       => $PROFILE.AllUsersAllHosts
* AllUsers, CurrentHost    => $PROFILE.AllUsersCurrentHost
* CurrentUser, AllHosts    => $PROFILE.CurrentUserAllHosts
* CurrentUser, CurrentHost => $PROFILE.CurrentUserCurrentHost
```

Par défaut quand on parle de la variable ```$PROFILE``` on parle de la variable ```$PROFILE.CurrentUserCurrentHost```

Chaque combinaisons est stocké dans un répertoire particulier (cas de Windows PowerShell et PowerShell Core) :

* $PROFILE.AllUsersAllHosts       => $PSHOME\Profile.ps1
* $PROFILE.AllUsersCurrentHost    => $PSHOME\Microsoft.PowerShell_profile.ps1
* $PROFILE.CurrentUserAllHosts    => $Home\[My]Documents\PowerShell\Profile.ps1
* $PROFILE.CurrentUserCurrentHost => $Home\[My]Documents\PowerShell\Microsoft.PowerShell_profile.ps1

> $PSHOME fait allusion au repertoire d'installation de PowerShell : C:\Windows\System32\WindowsPowerShell\v1.0
pour Windows PowerShell C:\Program Files\PowerShell\7-preview pour PowerShell Core

Dans le cas des autres environnements (ISE, VSCode) les chemins sont identiques mais les noms de fichiers peuvent être différent

## Comment créer un profile ?

La première étape est de vérifier si un fichier profile n'est pas déja existant

```powershell
Test-Path -Path $PROFILE
False
```

Si le résultat est ```False```, il suffit de créer le fichier

```powershell
New-Item -ItemType File -path $PROFILE -force
```

Ensuite il suffit de l'éditer avec n'importe quel éditeur de texte (notepad, notepad++, VSCode)

```powershell
notepad $PROFILE
```

## Mais on mets quoi dans un profile ?

Comme nous l'avons vu le profile PowerShell est un script comme les autres. Nous pouvons donc y mettre tous ce qui peut se trouver dans un script "normal"

Par exemple dans le profile ```$PROFILE.AllUsersAllHosts``` sur ma machine j'ai cette fonction

```powershell
#region <Admin or Not>
function IsAdmin
{
    $CurrentUser =
    [System.Security.Principal.WindowsIdentity]::GetCurrent()
    $principal = New-Object System.Security.principal.windowsprincipal($CurrentUser)
    $principal.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
}

# Personalized greeting
#
if (isAdmin)
{
    $ElevationMode = 'Privilegie'
    $host.UI.RawUI.BackGroundColor = 'DarkRed'
    Clear-Host
}
else
{
    $ElevationMode = 'Non Privilegie'
    #$host.UI.RawUI.BackGroundColor = 'DarkMagenta'
    Clear-Host
}

$CurrentUser =
[System.Security.Principal.WindowsIdentity]::GetCurrent()

Write-Host '+---------------------------------------------------+'
Write-Host ("+- Bonjour {0} " -f ($CurrentUser.Name).split('\')[1])
Write-Host '+---------------------------------------------------+'
#endregion <Admin or Not>
```

Cette fonction permet d'afficher dans le titre de la fenêtre quand je lance la console en administrateur

![Administrator Mode](/assets/images/post/2020-01-02-Montre_moi_ton_profile/Console_Powershell_Admin.png "Administrator Mode")

ou en utilisateur normal

![Normal Mode](/assets/images/post/2020-01-02-Montre_moi_ton_profile/Console_Powershell_NormalMode.png "Normal Mode")

j'ai également une fonction d'installation de module

```powershell
$private:AutoLoad = @{
    Modules = @(
        'Plaster',
        'Psake',
        'PSDepend',
        'PSDeploy'
    )
}

Write-Host "=> loading modules from AllUsersAllHosts"
Foreach ($private:M in $AutoLoad.Modules)
{
    if (!(Get-Module -ListAvailable -Name $m))
    {
        Write-Warning "$($m) not found => installation launch "
        Install-Module -Name $m -AllowClobber -Force -ErrorAction SilentlyContinue -Scope CurrentUser
    }
}
```

Il s'agit de module que j'utilise pour la compilation de mes builds lorsque je code.

Pour plus d'exemple sur les profiles vous pouvez faire une recherche google.
