---
layout: post
title: "Credential : comment les utiliser ?"
summary: "Les différentes façons d'utiliser les credentials"
author: laurent
date: '2019-01-10 14:35:23 +0530'
category: ['PowerShell', 'Niv100', 'Lightning', 'FrPSUG']
thumbnail: /assets/img/header/mot-de-passe.png
keywords: PowerShell, Identifiant, Niv100, Lightning
permalink: /blog/CredentialUse/
usemathjax: true
---

## Information

Cette article a été rédigé pour une présentation du [French Powershell UserGroup](https://frpsug.com).

Cette présentation peut-être vu sur Youtube sur la [chaîne du FRPSUG](https://www.youtube.com/watch?v=3OR143IPQ4o&t)

## Les différentes façons d'utiliser les credentials…

## Demande Initiale

Dès mes débuts sur powershell, je me suis très vite posé la question de la gestion des credentials dans mes scripts

Du besoin simple qui est peut-être géré de façon basique à l'utilisation des credentials dans des scripts automatiques j'ai longtemps cherché le meilleur moyen de faire.

## Traitement de la demande

___1. Get-Credential___

La façon la plus simple d'utiliser les credentials et d'utiliser la commande de base

```powershell
$cred = Get-Credential -Message "Message affiché dans la popup" -UserName MonUtilisateur
```

Le résultat est le suivant

```powershell
PS > $cred

UserName                           Password
--------                           --------
MonUtilisateur System.Security.SecureString
```

Cette variable `$cred` peut-être utiliser par exemple dans la commande suivante

```powershell
Enter-PSSession -ComputerName MonComputer -Credential $cred
```

___2. ConvertFrom-SecureString : Stockage sur le disque___

Une autre solution, un peu plus avancée, est de stocker le mot de passe dans un fichier sur le PC.
Naturellement ce stockage doit se faire de façon sécurisée.
Comme précédemment il faut d'abord créer l'objet `$Cred`

```powershell
$cred = Get-Credential -Message "Message affiché dans la popup" -UserName MonUtilisateur
```

Dans un second temps nous allons stocker le mot de passe de façon crypté sur le disque dur

```powershell
$Cred.Password | ConvertFrom-SecureString | Out-File C:\temp\password.txt
```

Dans le fichier c:\temp\password.txt le mot de passe se trouve sous cette forme

```xml
01000000d08c9ddf0115d1118c7a00c04fc297eb0100000057670149ac674a41ad9d185a8a82724b0000000002000000000010660000000100002000000093aaaf1ed598a69bbfb4cc77e81dfeb2786f26db6184538833af18054ef1a8a3000000000e800000000200002000000098c97f4f344d0159f337966d55060ad3297cae7515938457a713ddd9eaac5cdf200000003d986891fb27cb3983f798082083ac734d97d6235a186d3cc43db26f63bd44684000000018620d4739c0a26a6261e8c9867e94605cd35c61090c982999d5bb09fb54ec7d9a3499ebeb304c67720edfa37a34fe7fd4bce8fd8468dbee5081f56c81f4ce46
```

Pour pouvoir utiliser ce mot de passe stocké de façon sécurisé, il faut d'abord le décrypter. Pour ce faire nous allons procéder de la façon suivante

```powershell
$Username = "MonUtilisateur"
$SecurePassword = Get-Content c:\temp\password.txt | ConvertTo-SecureString
$Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $Username,$SecurePassword
```

```powershell
PS > $cred

UserName                           Password
--------                           --------
MonUtilisateur System.Security.SecureString
```

Comme dans le point 1 on se retrouve avec une variable `$Cred` utilisable dans la commande

```powershell
Enter-PSSession -ComputerName MonComputer -Credential $cred
```

___3. Export-Clixml : Stockage sur le disque___

L'avantage de cette méthodologie est que vous pouvez exploiter la polyvalence de PowerShell pour vous assurer que les données ne sont pas seulement exportées, mais également stockées de manière sécurisée à l'aide de chaînes sécurisées. Il convient de noter que ces fichiers d’informations d’identification créés ne peuvent être ouverts que par le même utilisateur sur le même système.

Pour créer le fichier d'export il faut procéder de la façon suivante

```powershell
get-credential -message "mot de passe utilisateur ?" -UserName MonUtilisateur | Export-Clixml -Path "c:\temp\user.xml"
```

Le fichier c:\temp\user.xml contient les informations suivantes

```xml
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">MonUtilisateur</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb0100000057670149ac674a41ad9d185a8a82724b00000000020000000000106600000001000020000000dadd8864c9b930a2eb07a6745ac4fb5711912c318c401f7e35bb91d4d1cc180b000000000e8000000002000020000000b5a862ba266c236357445b773ca38d73ed124cf82d863ac4c11e2b48d57fca4b2000000054180930ba9fd53a6c4bdd9d7f69c044c88072b0d411486bccc1ca3cca417bf440000000d6197eafe8a133235bd1b44e376c3ff02e94da9f39b7d24b9a68ef5dbd629e44180ce15c3e67830d758fa1296f60a98cb2371ef915990c921e728f44c72c4cbd</SS>
    </Props>
  </Obj>
</Objs>
```

Pour récupérer les informations il faut utiliser la commande

```powershell
 $Cred = Import-Clixml -Path "c:\temp\user.xml"
```

encore une fois on récupère bien notre variable `$Cred`

```powershell
PS > $cred

UserName                           Password
--------                           --------
MonUtilisateur System.Security.SecureString
```

Aujourd'hui, personnellement, j'utilise la méthode 3.
