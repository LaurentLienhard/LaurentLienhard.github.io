---
layout: post
title: "Affectation de licence Office 365"
summary: "Affectation automatique de licences Office 365"
author: laurent
date: '2020-02-01 14:35:23 +0530'
category: ['PowerShell', 'Niv200']
thumbnail: /assets/img/header/office-365.jpg
keywords: PowerShell, AzureAD, Niv200, Office365
permalink: /blog/AffectationAutoLicenceO365/
usemathjax: true
---

Depuis longtemps j'ai pris l'habitude d'affecter les licences Office 365 avec une petite fonction powershell tout ce qu'il ya de plus simple

```powershell
$bal = $global:username + "@" + $global:domaine
Set-MsolUser -UserPrincipalName $bal -UsageLocation "FR"
Set-MsolUserLicense -UserPrincipalName $bal -AddLicenses "XXXX:ENTERPRISEPACK"
[string]$Consumedlicence = Get-MsolAccountSku | Where-Object AccountSkuId -EQ "XXXX:ENTERPRISEPACK" | select ConsumedUnits
[string]$ActiveLicence = Get-MsolAccountSku | Where-Object AccountSkuId -EQ "XXXX:ENTERPRISEPACK" |  select ActiveUnits
Write-Host $Consumedlicence.split("=")[1].split("}")[0] "licences sur" $ActiveLicence.split("=")[1].split("}")[0] "sont affectées"
```

Malheureusement tous le monde n'étant pas familier avec le PowerShell, on m'a demandé de trouver une façon plus simple de dispatcher rapidement les licences Office 365 aux différents utilisateurs

Après quelques recherches j'ai trouvé une solution proposée dans Azure AD qui permet d'affecter les licences Office 365 en fonction de l'appartenance a des groupes.

Dans l'environment dans lequel j'évolue au moment de l'écriture de cette article nous sommes en configuration hybride c'est a dire que nous avons un Active Directory en local qui se synchronise avec notre tenant AzureAD via Azure AD Connect.

Je vais donc effectuer une partie de la configuration (la création des groupes par exemple) dans mon Active Directory local mais pour les personnes en full azureAD rien n'empêche de le faire en mode cloud

## Principe

Le principe est assez simple en fait.

* Création de groupe dans l'active Directory (ou dans AzureAD)
* Dans AzureAD configuré l'affectation des licences Office 365 sur les groupes crées
* dispatcher les utilisateurs dans les groupes

## Création des groupes

Tout pourrais se faire via la mmc de Windows mais comme nous sommes quand même sur un blog qui parle un peu de Powershell on va le faire en ligne de commande ;-)

```powershell
$HashArguments = @{
  Name = "XX-Licences O365-E3"
  SamAccountName = "XX-Licences O365-E3"
  GroupCategory = "Security"
  GroupScope = "Global"
  DisplayName = "Utilisateurs M365E3"
  Path = "CN=Users,DC=Fabrikam,DC=Com"
  Description = "Mettre ici les user XX pour leur affecter une licence E3 "
}

New-ADGroup @HashArguments
```

> XX correspond au nom de l'entité pour laquelle j'ai mis en place cette solution
> Pour information cette façon d'écrire les arguments se nomme le [Splatting](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-6 "Lien vers la doc")

Dans mon cas, je créé autant de groupe que de licence à affecter

![Groupe AD](/assets/img/posts/20200201/Liste_Groupe_AD.png "Groupe AD")

Une fois la synchronisation forcé a partir de mon serveur Azure AD Connect

![Synchro AAD](/assets/img/posts/20200201/Synchro-AAD.png "Synchro AAD")

je peux vérifier sur AzureAD que mes groupes sont bien présent

```powershell
PS>  Connect-AzureAD -Credential $CredAdminO365
Account                                              Environment TenantId                             TenantDomain
-------                                              ----------- --------                             ------------
administrateur@XXXXXXXXXXXXXXXXXXXXXX.onmicrosoft.com AzureCloud  XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX XXXXXXXXXXXX...


PS> Get-AzureADGroup -SearchString "XX-Licences O365-E3"
ObjectId                             DisplayName         Description
--------                             -----------         -----------
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx XX-Licences O365-E3 Mettre ici les user XX pour leur affecter une licence E3
```

A cette étape nous avons donc créé tous les groupes nécessaire pour dispatcher nos différentes licences à nos utilisateurs
Nous allons voir maintenant comment attribuer automatiquement des licences aux membres des différents groupes

## Affectation des licences

Comme nous l'avons dit plus haut, les différentes licences vont être appliqués sur les groupes AzureAD et non pas directement sur chaque utilisateurs.

>Il est important de noter cette différence. Nos verrons plus loin que les licences affectées via un groupe sont dites ```héritée``` alors que les autres sont dites ```directe```

Première étape je vérifie déjà qu'il n'y ai pas de licence affectée à mon groupe

```powershell
PS> Connect-MsolService -Credential $CredAdminO365
PS> (Get-MsolGroup -ObjectId xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx).Licenses | Select-Object SkuPartNumber
```

> Pour l'affectation des licences aux groupes je n'ai pas trouvé de solution en powershell je vais donc le faire en mode graphique

Se connecter a son tenant AzureAD et aller dans le ```Azure Active Directory``` puis dans ```Groupes```

Rechercher le groupe précédemment créé et cliquer sur ```Licences```
![Licences](/assets/img/posts/20200201/Menu_licenses.png "Licences")

cliquer ensuite sur ```Affectations``` puis sélectionner les licences qui devront être affectés aux utilisateurs membre de ce groupe
![selection Licences](/assets/img/posts/20200201/Selection_licences.png "Selection Licences")
puis cliquer sur ```Enregistrer```

Après quelques secondes (et rafraîchissement de la page) la liste des licences apparaît
![liste Licences](/assets/img/posts/20200201/liste_licences.png "liste Licences")

## test

Pour tester, je vais ajouter un utilisateur à mon groupe dans l'Active Directory
![Ajout groupe AD](/assets/img/posts/20200201/ajout_groupe.png "Ajout groupe AD")

Après quelques instant, pour effectuer la synchronisation Azure AD Connect, l'utilisateur apparaît dans mon groupe AzureAD

```powershell
PS> Get-AzureADGroupMember -ObjectId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
ObjectId                             DisplayName UserPrincipalName     UserType
--------                             ----------- -----------------     --------
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx test.XX     test.XX@XXXXXXXXXX.fr Member
```

Maintenant vérifions les licences que notre utilisateur Test à récupéré

```powershell
PS> Get-AzureADUser -SearchString "test.XX"
ObjectId                             DisplayName UserPrincipalName     UserType
--------                             ----------- -----------------     --------
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx test.XX     test.XX@XXXXXXXXXX.fr Member

PS>  Get-AzureADUserLicenseDetail -ObjectId xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Select-Object SkuPartNumber
SkuPartNumber
-------------
EMS
ENTERPRISEPACK
```

Notre utilisateur Test a bien récupérer les licences ```Enterprise Mobility + Security E3``` (EMS) et ```Office 365 E3``` (ENTERPRISEPACK)

Nous pouvons également vérifier cela directement dans la console Azure Active Directory en allant sur le compte de l'utilisateur
![Detail utilisateur](/assets/img/posts/20200201/detail_licence_utilisateur.png "Detail utilisateur")

En regardant les licences de plus près, on se rend compte qu'en fait 2 licences EMS et 2 licences Office 365 E3 sont affectées. Une Directe et une héritée !
![Detail utilisateur](/assets/img/posts/20200201/detail_licence.png "Detail licence")
Effectivement, mes utilisateurs, avaient au préalable, déja des licences affectées directement via la console d'adminstration Office 365 ou par le script du début de l'article.

Dans le cas ou l'on ré-affecte la même licence à l'utilisateur (comme dans cette exemple) ce n'est pas trop gênant, en effet cela affiche 2 licences mais cela n'en décompte qu'une seule au niveau de la facturation.

Par contre si l'utilisateur a déjà une licence Office 365 E3 et que l'on veut le passer sur une licence Office 365 F1 c'est plus problématique car la nouvelle licence F1, héritée du groupe, ne sera active qu'a partir du moment ou la licence E3 directe sera supprimée.

## Nettoyage des licences

Cette partie fera l'objet d'un nouvel article a venir ;-)

Teasing de dingue :-)
