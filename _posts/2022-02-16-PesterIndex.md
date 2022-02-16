---
layout: post
title: "Index Pester"
summary: "Indexation des différents test pester"
author: laurent
date: '2022-02-16 06:00:00 +0530'
category: ['Powershell', 'Niv100', 'Niv200', 'Niv300']
thumbnail: /assets/img/header/Coding_1436x500.png
keywords: Pester, Niv100, Niv200, Niv300, Pester
permalink: /blog/PesterIndex/
usemathjax: true
---

## Index des différents test pester que j'ai essayé

Exemple de code :

```powershell
function MyFunction {
    [CmdletBinding()]
    param (
        [System.String[]]$MyStringParameter = "ThisIsMyString",
        [System.Int32[]]$MyIntParameter = 32,
        [System.Array]$Filter = @()
    )
}
```

* Valider la présence d'un paramètre dans une fonction

```powershell
It 'Parameter MyStringParamter should be present and be of type String' {
    Get-Command MyFunction | Should -HaveParameter 'MyStringParameter' -Type [System.String[]]
    }
It 'Parameter MyIntParamter should be present and be of type Int32' {
    Get-Command MyFunction | Should -HaveParameter 'MyIntParameter' -Type [System.Int32[]]
    }
It 'Parameter Filter should be present and be of type Array' {
    Get-Command MyFunction | Should -HaveParameter 'Filter' -Type [System.Array]
    }
```

* Valider la valeur par défaut d'un paramètre dans une fonction

```powershell
It 'Parameter MyStringParamter should have ThisIsMyString by default' {
    Get-Command MyFunction | Should -HaveParameter 'MyStringParameter' -DefaultValue 'ThisIsMyString'
    }
It 'Parameter MyIntParamter should have 32 by default' {
    Get-Command MyFunction | Should -HaveParameter 'MyIntParameter' -DefaultValue 32
    }
It 'Parameter Filtre should be empty by default' {
    Get-Command MyFunction | Should -HaveParameter 'Filter' -DefaultValue '@()'
    }
```
