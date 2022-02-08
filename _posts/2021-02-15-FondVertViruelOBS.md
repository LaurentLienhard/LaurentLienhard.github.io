---
layout: post
title: "Fond vert virtuel sur OBS"
summary: "Comment faire un fond vert sur OBS sans utiliser de fond vert ;-)"
author: laurent
date: '2021-02-15 14:35:23 +0530'
category: ['Streaming', 'Niv100']
thumbnail: /assets/img/header/OBS-Logo.png
keywords: Streaming, Niv100, OBS
permalink: /blog/FondVertViruelOBS/
usemathjax: true
---

## Introduction

Streamer n'est pas l'une de mes activités, mais je fais beaucoup de Meetup avec Microsoft Teams.

Je cherchais un moyen de rendre mes Meetup et ceux du [French Powershell UserGroup](https://frpsug.com) un peu plus sexy

Comme je le disais plus haut je ne fait pas de vrai stream et en plus je n'ai pas de bureau dédié chez moi pour poser mon ordinateur. L'achat d'un fond vert n'étais donc pas la meilleur solution dans mon cas

J'ai donc cherché une solution pour faire un fond vert ... mais sans fond vert ;-)

Attention cette solution s'approche plus de la bidouille mais reste utilisable quand même

Produits utilisés :

* Skype
* OBS

## Première étape : Ajout d'un fond vert

Pour ça je passe par les Settings de Skype en affichant la preview de la webcam.

![Preview Skype](/assets/img/posts/20210215/Preview-Skype.png "Preview Skype")

De là, j'ajoute en background un fond vert que j'ai tout simplement chercher sur internet

Il est important de noter que cette fenêtre doit toujours rester a l'écran. Il ne faut pas la réduire dans la barre des tâches ou encore pire la fermer.

## Seconde étape : ajouter skype dans OBS

Au niveau d'OBS, dans les sources, ajouter une source Windows Capture

![Windows Capture](/assets/img/posts/20210215/OBS-Add-WindowsDeviceCapture.png "Windows Capture")

Donner un nom a la source ici : Skype WebCam

![Windows Capture](/assets/img/posts/20210215/OBS-Add-WindowsDeviceCapture-2.png "Windows Capture")

Valider pour ajouter la fenêtre Skype dans vos sources

![Windows Capture](/assets/img/posts/20210215/OBS-Add-WindowsDeviceCapture-3.png "Windows Capture")

## Troisième étape : appliquer les filtres

### Crop/Pad

Faite un clique droit sur la source que vous venez d'ajouter et choisir le menu Filtres

![Windows Capture Filter](/assets/img/posts/20210215/OBS-Add-WindowsDeviceCapture-Filter.png "Windows Capture filter")

Choisir le filtre Crop/Pad

![Windows Capture Filter](/assets/img/posts/20210215/OBS-Add-WindowsDeviceCapture-FilterCrop.png "Windows Capture filter")

C'est la qu'il va falloir y aller un peu au pifomètre ;-)

Dans les différentes ligne il faut jouer sur les valeurs jusqu'a ce que vous obteniez un cadrage correcte.

![Windows Capture Filter](/assets/img/posts/20210215/OBS-Add-WindowsDeviceCapture-FilterCrop-2.png "Windows Capture filter")

### Chroma Key

Ensuite nous allons a nouveau ajouter un filtre mais cette fois le filtre : Chroma Key

![Windows Capture Filter](/assets/img/posts/20210215/OBS-Add-WindowsDeviceCapture-FilterChroma.png "Windows Capture filter")

Il suffit de tous laisser par défaut et de valider.

![Windows Capture Filter](/assets/img/posts/20210215/OBS-Add-WindowsDeviceCapture-FilterChroma-2.png "Windows Capture filter")

## Résultat

Si tout c'est bien passé, dans OBS, vous devez obtenir un truc comme ca.

![Resultat](/assets/img/posts/20210215/resultat.png "Resultat")

Nous avons donc un fond vert ... sans fond vert ! Pari rempli !

## What's next ?

Je voudrais pouvoir envoyer dans Teams lors de mon meetup ce qui sort d'OBS pour faire de belle présentation avec toute la puissance D'OBS.