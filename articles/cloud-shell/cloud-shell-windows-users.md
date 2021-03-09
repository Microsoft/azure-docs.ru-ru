---
title: Azure Cloud Shell для пользователей Windows | Документация Майкрософт
description: Руководство для пользователей, не знакомых с системами Linux
services: azure
documentationcenter: ''
author: maertendMSFT
manager: hemantm
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/03/2018
ms.author: damaerte
ms.openlocfilehash: 8cc1934cc97ecf821de8644dda45d867b3ca3e85
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102508952"
---
# <a name="powershell-in-azure-cloud-shell-for-windows-users"></a>PowerShell в Azure Cloud Shell для пользователей Windows

В мае 2018 года были [объявлены](https://azure.microsoft.com/blog/pscloudshellrefresh/) изменения для PowerShell в Azure Cloud Shell.
Теперь работа с PowerShell в Azure Cloud Shell выполняется на базе [PowerShell Core 6](https://github.com/powershell/powershell) в среде Linux.
В связи с этим изменением могут возникать некоторые отличия в работе с PowerShell в Cloud Shell, по сравнению с ожидаемыми данными на Windows PowerShell.

## <a name="file-system-case-sensitivity"></a>Учет регистра файловой системы

Регистр файловой системы Windows не учитывается, тогда как в Linux регистр учитывается.
`file.txt` и `FILE.txt` ранее считались одним и тем же файлом, но теперь будут разными.
Необходимо использовать правильный регистр для завершения `tab-completing` в файловой системе.
Та часть, которая обрабатывается PowerShell, например имена командлетов `tab-completing`, параметры и значения, используется без учета регистра.

## <a name="windows-powershell-aliases-vs-linux-utilities"></a>Псевдонимы Windows PowerShell и служебные программы Linux

Некоторые существующие псевдонимы PowerShell имеют те же имена, что и встроенные команды Linux, такие как `cat` ,, `ls` `sort` , `sleep` и т. д. В PowerShell Core 6 псевдонимы, конфликтующие с встроенными командами Linux, были удалены.
Ниже приведены распространенные псевдонимы, которые были удалены, а также их эквивалентные команды:  

|Удаленный псевдоним   |Эквивалентная команда   |
|---|---|
|`cat`    | `Get-Content` |
|`curl`   | `Invoke-WebRequest` |
|`diff`   | `Compare-Object` |
|`ls`     | `dir` <br> `Get-ChildItem` |
|`mv`     | `Move-Item`   |
|`rm`     | `Remove-Item` |
|`sleep`  | `Start-Sleep` |
|`sort`   | `Sort-Object` |
|`wget`   | `Invoke-WebRequest` |

## <a name="persisting-home"></a>Сохранение $HOME

Пользователи могут заранее сохранить сценарии и другие файлы только на своем облачном диске.
Теперь пользовательский каталог $HOME также сохраняется между сеансами.

## <a name="powershell-profile"></a>Профиль PowerShell

Профиль PowerShell не создается по умолчанию.
Чтобы создать профиль, создайте каталог `PowerShell` в `$HOME/.config`.

```azurepowershell-interactive
mkdir (Split-Path $profile.CurrentUserAllHosts)
```

В `$HOME/.config/PowerShell` вы сможете создать свой профиль: `profile.ps1` и/или `Microsoft.PowerShell_profile.ps1`.

## <a name="whats-new-in-powershell-core-6"></a>Новые возможности в PowerShell Core 6

Дополнительные сведения о новинках в PowerShell Core 6 можно получить в [документах PowerShell](/powershell/scripting/whats-new/what-s-new-in-powershell-70) и в записи блога [Getting Started with PowerShell Core on Windows, Mac, and Linux](https://blogs.msdn.microsoft.com/powershell/2017/06/09/getting-started-with-powershell-core-on-windows-mac-and-linux/) (приступая к работе с PowerShell Core в Windows, Mac и Linux).