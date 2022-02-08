---
layout: post
title:  "Credential : comment mieux les utiliser ?"
summary: "Le module BetterCredentials"
author: laurent
date: '2019-01-09 14:35:23 +0530'
category: ['PowerShell', 'Niv100', 'Lightning', 'FrPSUG']
thumbnail: /assets/img/header/mot-de-passe.png
keywords: PowerShell, Identifiant, Niv100, Lightning
permalink: /blog/BetterCredentials/
usemathjax: true
---

## Information

Cette article a été rédigé pour une présentation du [French Powershell UserGroup](https://frpsug.com).

Cette présentation peut-être vu sur Youtube sur la [chaîne du FRPSUG](https://www.youtube.com/watch?v=3OR143IPQ4o&t)

## Module : BetterCredentials

Lors de mes dernières recherches, je suis tombé, par hasard, sur le module [BetterCredentials](https://github.com/Jaykul/BetterCredentials) de [Jaykul](https://github.com/Jaykul)

___Alors attention le module n'est pas complètement sec :-)___

Comme il le dit lui-même si bien ;-)

L’objectif de BetterCredentials est de fournir une commande Get-Credential entièrement compatible avec les versions antérieures qui améliore le Get-Credential d'origine en ajoutant des fonctionnalités supplémentaires qui sont absentes de la commande intégrée. Plus précisément, stockage des informations d'identification pour l'automatisation, et fourniture d'invites plus complètes avec une explication de l'utilisation des informations d'identification

En résumé c'est un `Get-Credential` mais en mieux...

pour l'utilisation il suffit d'installer le module directement de la PSGallery

```powershell
Install-Module -Name BetterCredentials -scope CurrentUser -AllowClobber -force
```

Le `-AllowClobber` est important pour faire l'installation

Il faut ensuite, classiquement, importer le module

```powershell
Import-Module BetterCredentials

```

la liste des commandes disponibles est la suivante

```powershell
PS > Get-Command -Module BetterCredentials

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Find-Credential                                    4.5        BetterCredentials
Function        Get-Credential                                     4.5        BetterCredentials
Function        Remove-Credential                                  4.5        BetterCredentials
Function        Set-Credential                                     4.5        BetterCredentials
Function        Test-Credential                                    4.5        BetterCredentials
```

La première commande qui nous intéresse bien sur est `Get-Credential`.

Comme pour toute nouvelle commande commençons par regarder l'aide

```powershell
 Get-Help -Name BetterCredentials\Get-Credential -Detailed
```

Remarquer que j'ai préfixé la commande avec le nom du module afin que le système comprenne que je veux l'aide de Get-Credential du module et non pas celle de la commande par défaut.

A la lecture de l'aide une phrase m'a beaucoup intéressé

```xml
 It also supports storing and retrieving credentials in your Windows Credential Manager, but otherwise functions identically to the built-in command
```

Hum, Hum intéressant.

Plutôt que de stocker les informations (UserName et Password) dans un fichier comme j'en ai parlé sur le post [Credential : comment les utiliser ?](https://laurentlienhard.github.io/powershell/credential/CredentialUse/) ce module est capable de les stocker dans le Coffre-Fort Windows.

Voyons voir un peu cette fonction plus précisément.

```powershell
PS > Get-Credential -UserName MonUtilsiateur -Domain MonDomaine -Store

UserName                                      Password
--------                                      --------
MonDomaine\MonUtilsiateur System.Security.SecureString
```

Bon jusque là rien de surprenant puisque le résultat est un object Credential avec Username et Password.

ok, ok mais avant nous avons bien parlé d'un stockage dans le Coffre-Fort Windows.

Allons jeter une oeil dans le `Credential Manager` et effectivement on retrouve une entrée correspondant à notre utilisateur avec son mot de passe.

![Credential Manager](/assets/img/posts/20190109/CredentialManagerWithUser.png)

Afin de pouvoir récupérer ce compte il suffit de reprendre la commande

```powershell
$cred = Get-Credential -UserName MonUtilsiateur -Domain MonDomaine
```

Si l'utilisateur existe dans le coffre-fort il récupère ce compte sinon il fait un prompt pour demander le mot de passe

Dans notre cas l'utilisateur est dans le coffre-fort

```powershell
PS >  $cred = Get-Credential -UserName MonUtilsiateur -Domain MonDomaine
PS > $cred


Target        : MicrosoftPowerShell:user=MonDomaine\MonUtilsiateur
TargetAlias   :
Type          : Generic
Persistence   : Enterprise
Description   :
LastWriteTime : 2/10/2019 7:25:35 PM
UserName      : MonDomaine\MonUtilsiateur
Password      : System.Security.SecureString
```

Cette variable `$cred` peut-être utiliser par exemple dans la commande suivante

```powershell
Enter-PSSession -ComputerName MonComputer -Credential $cred
```

J'avoue ne pas trop savoir quoi penser de ce module.

L'idée me parait intéressante mais ce module demande encore à être développé ce qui ne semble pas être le cas. Autre point bloquant, étant donné qu'il se base sur le Credential Manager de Windows, ce module n'est naturellement pas Cross-Platform :-(

Mais c'est là, la force de la communauté donc avis à tous les amateurs qui voudraient participer à l'évolution de ce module :-)
