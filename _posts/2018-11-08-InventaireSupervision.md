---
layout: post
title:  "Etat de la synchronisation entre l'inventaire et la supervision"
summary: "Avoir un état de la synchronisation entre son inventaire et sa supervision"
author: laurent
date: '2018-11-08 14:35:23 +0530'
category: ['PowerShell', 'Niv100']
thumbnail: /assets/img/header/Coding_1436x500.png
keywords: PowerShell, MySql, Niv100
permalink: /blog/InventaireSupervision/
usemathjax: true
---

# Avoir un état de la synchronisation entre son inventaire et sa supervision

## Besoin Initial

J'utilise tous les jours les outils suivants :

* Centreon pour la supervision
* Glpi pour mon inventaire

Malheureusement je ne suis pas le seul à gérer et créer des serveurs par contre je suis le seul à gérer la supervision.
Naturellement comme toujours dans ce genre de cas il arrive souvent que des serveurs soient créés sans que je n'en sois informé et du coup une dé-corrélation entre la supervision et l'inventaire apparaît.

L’agent d'inventaire Fusion Inventory est poussé par GPO sur tous les serveurs mis dans le domaine donc, normalement, mon inventaire doit être à peu près à jour.

Le but était donc de faire un outil qui récupère les informations serveurs présents dans mon GLPI et de les comparer avec mes objets présents dans la supervision.

Les étapes sont les suivantes :

* Se connecter aux bases Mysql
* Exporter les informations des 2 bases
* Comparer les informations avec un rendu lisible pour l'utilisateur

## Étape 1 : La connexion à une base MySql

Pour pouvoir se connecter à une base de données MySql il est nécessaire d'installer au préalable le connecteur .net suivant : [Connector/NET](https://dev.mysql.com/downloads/connector/net/5.2.html "Link to Connector .Net")

Il faut ensuite, dans un premier temps, charger ce connecteur pour pouvoir l'utiliser en ajoutant la commande suivante au début de notre script.

```powershell
[void] [system.reflection.Assembly]::LoadWithPartialName("MySql.Data")
```

Comme d'habitude, on positionne quelques variables pour simplifier l'utilisation du script

```powershell
$ServGLPI = "srv-glpi01" #Adresse du serveur GLPI
$UserGlpi = "root"  #Nom de l'utilisateur utilisé pour se connecter
$PasswordGlpi = "sdfghjkl" #Mot de passe de l'utilisateur
$DbGlpi = "glpidb" #Nom de la base de donnée MySql

$ServCentreon = "srv-centreon01" #Adresse du serveur Centreon
$UserCentreon = "root"  #Nom de l'utilisateur utilisé pour se connecter
$PasswordCentreon = "sdfghjkl" #Mot de passe de l'utilisateur
$DbCentreon = "centreondb" #Nom de la base de donnée MySql

$port = "3306"  #Port d'écoute du serveur MySql
```

Il faut ensuite créer une chaîne de connexion vers nos serveurs Mysql

```powershell
$ConnInventaire = “server=$ServGlpi;uid=$UserGlpi;password=$PasswordGlpi;database=$DbGlpi;Port=$Port"
$ObjMysqlInventaire = New-Object MySql.Data.MySqlClient.MySqlConnection($ConnInventaire)
$ObjMysqlInventaire.Open()

$ConnSuperviser = “server=$ServerCentreon;uid=$UserCentreon;password=$PasswordCentreon;database=$DbCentreon;Port=$Port"
$ObjMysqlSuperviser = New-Object MySql.Data.MySqlClient.MySqlConnection($ConnSuperviser)
$ObjMysqlSuperviser.Open()
```

À partir de cet instant nous avons 2 connexions *$ConnInventaire et $ConnSuperviser* ouvertes respectivement vers les bases de données sur les serveurs SRV-GLPI01 et SRV-CENTREON01.

Nous allons pouvoir utiliser ces connexions pour passer à la seconde étape.

## Étape 2 : Requêter les bases de données

```powershell
$ReqInventaire = "SELECT `glpi_computers`.name AS `Name`, `glpi_states`.`completename` AS `State`, `glpi_computertypes`.name AS `Type`, `glpi_computermodels`.name AS `Modele`, `glpi_locations`.`completename` AS `Localisation`
FROM `glpi_computers`
LEFT JOIN `glpi_entities` ON (`glpi_computers`.`entities_id` = `glpi_entities`.`id` )
LEFT JOIN `glpi_states` ON (`glpi_computers`.`states_id` = `glpi_states`.`id` )
LEFT JOIN `glpi_manufacturers` ON (`glpi_computers`.`manufacturers_id` = `glpi_manufacturers`.`id` )
LEFT JOIN `glpi_computertypes` ON (`glpi_computers`.`computertypes_id` = `glpi_computertypes`.`id` )
LEFT JOIN `glpi_computermodels` ON (`glpi_computers`.`computermodels_id` = `glpi_computermodels`.`id` )
LEFT JOIN `glpi_operatingsystems` ON (`glpi_computers`.`operatingsystems_id` = `glpi_operatingsystems`.`id` )
LEFT JOIN `glpi_locations` ON (`glpi_computers`.`locations_id` = `glpi_locations`.`id` )
LEFT JOIN `glpi_items_deviceprocessors` ON (`glpi_computers`.`id` = `glpi_items_deviceprocessors`.`items_id` AND `glpi_items_deviceprocessors`.`itemtype` = 'Computer' )
LEFT JOIN `glpi_deviceprocessors` AS `glpi_deviceprocessors_7083fb7d2b7a8b8abd619678acc5b604` ON (`glpi_items_deviceprocessors`.`deviceprocessors_id` = `glpi_deviceprocessors_7083fb7d2b7a8b8abd619678acc5b604`.`id` )
LEFT JOIN `glpi_ipaddresses` AS `glpi_ipaddresses_0cc35feab42e5909929ff742b4834540` ON (`glpi_computers`.`id` = `glpi_ipaddresses_0cc35feab42e5909929ff742b4834540`.`mainitems_id` AND `glpi_ipaddresses_0cc35feab42e5909929ff742b4834540`.`mainitemtype` = 'Computer' AND `glpi_ipaddresses_0cc35feab42e5909929ff742b4834540`.`is_deleted` = 0 )
WHERE `glpi_computers`.`is_deleted` = '0' AND `glpi_computers`.`is_template` = '0' AND ( (`glpi_computertypes`.`id` = '22') AND (`glpi_states`.`id` = '1') ) GROUP BY `glpi_computers`.`id`
ORDER BY `Name` ASC "
```

Premièrement je crée une variable *$ReqInventaire* qui va contenir ma requête. Dans ce cas-là je récupère tous les machines de type serveur.

> Petite astuce : pour trouver cette requête je passe GLPI en mode debug ;-)

```powershell
$SQLCommand = New-Object MySql.Data.MySqlClient.MySqlCommand($ReqInventaire,$ObjMysqlInventaire)
$MySQLDataAdaptater = New-Object MySql.Data.MySqlClient.MySqlDataAdapter($SQLCommand)
$MySQLDataSet = New-Object System.Data.DataSet
$RecordCount = $MySQLDataAdaptater.Fill($MySQLDataSet)
$ServersInventorier = $MySQLDataSet.Tables
$ObjMysqlInventaire.close()
```

Ensuite je passe cette requête dans ma connexion $ObjMysqlInventaire, je l'exécute, récupère les données dans un DataSet et enfin de ce DataSet j'exporte les informations qui m'intéresse dans la variable $ServersInventorier

> Pour plus d'information sur cette partie, je vous laisse faire un tour sur [Google](http:///www.google.Fr)

Je fais de même pour la requête dans Centreon

```powershell
$ReqSuperviser = "SELECT nagios_hosts.display_name AS Name FROM nagios_hosts ORDER BY Name ASC "
$SQLCommand = New-Object MySql.Data.MySqlClient.MySqlCommand($ReqSuperviser,$ObjMysqlSuperviser)
$MySQLDataAdaptater = New-Object MySql.Data.MySqlClient.MySqlDataAdapter($SQLCommand)
$MySQLDataSet = New-Object System.Data.DataSet
$RecordCount = $MySQLDataAdaptater.Fill($MySQLDataSet)
$ServersSupeviser = $MySQLDataSet.Tables
$ObjMysqlSuperviser.close()
```

A ce stade, on se retrouve donc avec 2 variables $ServersInventorier et $ServersSupeviser contenant respectivement la liste des serveurs inventoriés dans GLPi et la liste des serveurs supervisés dans Centreon.

## Étape 3 : Comparer les 2 listes

Pour comparer ces 2 listes j'ai utilisé la commande Compare-Object (tout simplement ;-) )

> The Compare-Object cmdlet compares two sets of objects. One set of objects is the "reference set," and the other set is the "difference set."
The result of the comparison indicates whether a property value appeared only in the object from the reference set (indicated by the <= symbol), only in the object from the difference set (indicated by the => symbol) or, if the IncludeEqual parameter is specified, in both objects (indicated by the == symbol). [extrait de la doc en ligne](https://docs.microsoft.com/fr-fr/powershell/module/microsoft.powershell.utility/compare-object?view=powershell-6)

Donc comme l'extrait ci-dessus l'explique il nous faut :

* reference set = $ServersInventorier
* difference set = $ServersSupeviser
* IncludeEqual : car je veux quand même savoir si ma machine est inventoriée et supervisée

```powershell
$Servers = Compare-Object -ReferenceObject $ServersInventorier.name -DifferenceObject $ServersSupeviser.name -IncludeEqual
ForEach ($server in $Servers)
{
    Switch ($server.SideIndicator)
    {
        "==" {Write-Host $server.SideIndicator $server.InputObject "est Inventorié et Supervisé" }
        "<=" {Write-Host $server.SideIndicator $server.InputObject "est Inventorié mais PAS SUPERVISE"}
        "=>" {Write-Host $server.SideIndicator $server.InputObject "est Supervisé mais PAS INVENTORIE"}
    }
}
```

Pour ce faire je récupère le résultat de ma commande compare-object dans une variable $Servers.
Ensuite je boucle sur cette variable pour faire afficher une phrase claire en fonction du cas.

La sortie résultant de cette boucle est du genre

```powershell
== Serveur01 est Inventorié et Supervisé
=> Serveur10 est Supervisé mais PAS INVENTORIE
<= Serveur20 est Inventorié mais PAS SUPERVISE
```

Voilà avec çà il ne me reste plus qu'a mettre à jour mon inventaire et ma supervision selon le cas.
