---
layout: post
title:  "Word : decoupage d'un fichier Word"
summary: "Découper un fichier et l'enregistrer en plusieurs documents"
author: IronTUX
date: '2018-10-217 14:35:23 +0530'
category: ['PowerShell', 'Niv200']
thumbnail: /assets/img/posts/code.jpg
keywords: PowerShell, Regex, Office, Niv200
permalink: /blog/Split-Docx2Pdf/
usemathjax: true
---

# Découper un fichier Word en plusieurs PDF…

## Demande Initiale

La demande initiale qui m'a amené à réfléchir sur ce sujet provient d'un collègue.

Mon collègue réceptionne un gros document Word de son ERP contenant sur chaque page une lettre pour un destinataire différent.

Afin de pouvoir l'intégrer dans la GED (Gestion électronique de document) il avait les besoins suivant :

* Découper le gros fichier en plusieurs fichiers (1 page = 1 fichier)
* Récupérer une donnée dans les documents pour nommer le PDF
* Sauvegarder les nouveaux documents en PDF

Ne connaissant pas grand-chose (c'est un euphémisme :-) ) dans la manipulation de Word via PowerShell j'ai découpé, comme d'habitude, ma réflexion en petites étapes.

## Traitement de la demande

### L'ouverture et le découpage du fichier

La première étape est donc d'ouvrir le document Word en question. Pour cela il faut instancier un objet Word et ouvrir le document

```powershell
$word = New-Object -ComObject word.application
$word.Visible = $True
$doc = $word.Documents.Open($inputFile)
```

Je laisse "$word.Visible" à True le temps du debug ensuite je le mettrais en False pour que le traitement soit totalement transparent.

$ImputFile est une variable qui contient le chemin vers mon fichier Word à traiter.

Une fois le document ouvert on entre dans le vif du sujet : Comment découper ce document de X pages en X documents d'une page?

J'ai cherché pas mal sur le net avant de trouver une solution qui, n'est peut-être pas la plus élégante, mais qui a le mérite de fonctionner ;-).

Le principe est assez simple en fait : on prend une page après l'autre du document et on fait un copier/coller dans un nouveau document que l'on enregistre.

```powershell
$pages = $doc.ComputeStatistics([Microsoft.Office.Interop.Word.WdStatistic]::wdStatisticPages)
$rngPage = $doc.Range()

for ($i = 1; $i -le $pages; $i += $pageLength)
{

    [Void]$word.Selection.GoTo([Microsoft.Office.Interop.Word.WdGoToItem]::wdGoToPage,

        [Microsoft.Office.Interop.Word.WdGoToDirection]::wdGoToAbsolute,

        $i #Starting Page

    )

    $rngPage.Start = $word.Selection.Start


    [Void]$word.Selection.GoTo([Microsoft.Office.Interop.Word.WdGoToItem]::wdGoToPage,

        [Microsoft.Office.Interop.Word.WdGoToDirection]::wdGoToAbsolute,

        $i + $pageLength #Next page Number

    )

    $rngPage.End = $word.Selection.Start

    $marginTop = $word.Selection.PageSetup.TopMargin
    $marginBottom = $word.Selection.PageSetup.BottomMargin
    $marginLeft = $word.Selection.PageSetup.LeftMargin
    $marginRight = $word.Selection.PageSetup.RightMargin

    $rngPage.Copy()
    $newDoc = $word.Documents.Add()

    $word.Selection.PageSetup.TopMargin = $marginTop
    $word.Selection.PageSetup.BottomMargin = $marginBottom
    $word.Selection.PageSetup.LeftMargin = $marginLeft
    $word.Selection.PageSetup.RightMargin = $marginRight

    $word.Selection.Paste() # Now we have our new page on a new doc
    $word.Selection.EndKey(6, 0) #Move to the end of the file
    $word.Selection.TypeBackspace() #Seems to grab an extra section/page break
    $word.Selection.Delete() #Now we have our doc down to size
}
```

Je ne rentrerais pas dans le détail mais le code est assez compréhensible. J’ai récupéré ce bout de code et je l'ai un peu adapté à mon besoin. J'avoue que je ne comprends pas forcément tout ce qu'il fait (mais il le fait :-) )

Nous verrons l'enregistrement un peu plus loin dans la suite de l'article

### Récupérer le nom du fichier dans le document Word

Chaque nouveau document Word créé correspond à une lettre pour un destinataire.
S'agissant de lettre pour des locataires, le numéro unique que je dois récupérer dans le document Word et le numéro de bail qui se présente sous la forme d'une suite de 10 nombres commençant toujours par 0.

Pour rechercher ce nombre j'ai utilisé les Regex.

```powershell
$FileNamePattern = ".*de bail.*(0\d{9})"
$regex = [Regex]::Match($rngPage.Text, $fileNamePattern)

if ($regex.Success) {
    $id = $regex.Groups[1].Value
}
else {
    $id = "patternNotFound" + $i
}
```

Je définis mon pattern qui recherche n’importe où dans le document la chaîne "de bail" suivi de n'importe quel caractère puis suivi d'un groupe de 9 nombres précédé par un 0.

J'applique mon pattern et je récupère le résultat dans la variable $regex.

Si tout se passe bien je récupère le numéro de bail dans la variable $id avec laquelle je formerais mon nom de document.

### Enregistrer chaque document en PDF

```powershell
$path = $outputPath + $id + ".pdf"
$newDoc.saveas([ref] $path, 17)
$newDoc.close([ref]$False)
```

$OutputPath est une variable qui contient le chemin vers le répertoire de destination de mes fichiers PDF.

Voici la fonction dans son intégralité

```powershell
function Convert-Docx2Pdf {

[CmdletBinding()]
param (
    [Parameter(Mandatory = $False)][string]$FileNamePattern = ".*de bail.*(0\d{9})",
    [Parameter(Mandatory = $False)][string]$pageLength = 1,
    [Parameter(Mandatory = $true)][string]$InputFile ,
    [Parameter(Mandatory = $False)][string]$outputPath = $env:temp + "\Outputdir\"
)

BEGIN {
if (Test-Path $outputPath) {
    Remove-Item -Path $outputPath -Recurse -Force -Confirm:$false
}
New-Item -Path $outputPath -ItemType Directory -Force -Confirm:$false
}

PROCESS {
$word = New-Object -ComObject word.application

$word.Visible = $False



$doc = $word.Documents.Open($inputFile)

$pages = $doc.ComputeStatistics([Microsoft.Office.Interop.Word.WdStatistic]::wdStatisticPages)



$rngPage = $doc.Range()



for ($i = 1; $i -le $pages; $i += $pageLength)
{

    [Void]$word.Selection.GoTo([Microsoft.Office.Interop.Word.WdGoToItem]::wdGoToPage,

        [Microsoft.Office.Interop.Word.WdGoToDirection]::wdGoToAbsolute,

        $i #Starting Page

    )

    $rngPage.Start = $word.Selection.Start



    [Void]$word.Selection.GoTo([Microsoft.Office.Interop.Word.WdGoToItem]::wdGoToPage,

        [Microsoft.Office.Interop.Word.WdGoToDirection]::wdGoToAbsolute,

        $i + $pageLength #Next page Number

    )

    $rngPage.End = $word.Selection.Start



    $marginTop = $word.Selection.PageSetup.TopMargin

    $marginBottom = $word.Selection.PageSetup.BottomMargin

    $marginLeft = $word.Selection.PageSetup.LeftMargin

    $marginRight = $word.Selection.PageSetup.RightMargin


    $rngPage.Copy()

    $newDoc = $word.Documents.Add()


    $word.Selection.PageSetup.TopMargin = $marginTop

    $word.Selection.PageSetup.BottomMargin = $marginBottom

    $word.Selection.PageSetup.LeftMargin = $marginLeft

    $word.Selection.PageSetup.RightMargin = $marginRight



    $word.Selection.Paste() # Now we have our new page on a new doc

    $word.Selection.EndKey(6, 0) #Move to the end of the file

    $word.Selection.TypeBackspace() #Seems to grab an extra section/page break

    $word.Selection.Delete() #Now we have our doc down to size



    #Get Name

    $regex = [Regex]::Match($rngPage.Text, $fileNamePattern)

    if ($regex.Success) {
        $id = $regex.Groups[1].Value
    }
    else {
        $id = "patternNotFound" + $i
    }


    $path = $outputPath + $id + ".pdf"

    $newDoc.saveas([ref] $path, 17)

    $newDoc.close([ref]$False)
}
}

END {
[gc]::collect()
[gc]::WaitForPendingFinalizers()
}
}
```