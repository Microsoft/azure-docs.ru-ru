---
title: Получение управляемой группы ресурсов и изменение размера виртуальных машин в Azure PowerShell
description: Здесь представлен пример скрипта Azure PowerShell, с помощью которого можно получить управляемую группу ресурсов для управляемого приложения Azure. Скрипт позволяет изменять размеры виртуальных машин.
author: tfitzmac
ms.devlang: powershell
ms.topic: sample
ms.date: 10/27/2017
ms.author: tomfitz
ms.openlocfilehash: 602aaeb67ca081ebac71ca1d6d24a2de3c020603
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "86055993"
---
# <a name="get-resources-in-a-managed-resource-group-and-resize-vms-with-powershell"></a>Получение ресурсов из управляемой группы ресурсов и изменение размера виртуальных машин с помощью Azure PowerShell

С помощью этого скрипта можно получить ресурсы из управляемой группы ресурсов и изменить размер виртуальных машин в этой группе.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../../powershell_scripts/managed-applications/get-application/get-application.ps1 "Get application")]


## <a name="script-explanation"></a>Описание скрипта

В этом скрипте используются следующие команды для развертывания управляемого приложения. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [Get-AzManagedApplication](/powershell/module/az.resources/get-azmanagedapplication) | Перечисление управляемых приложений Azure. Укажите имя группы ресурсов, чтобы отфильтровать результаты. |
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Перечисление ресурсов. Укажите группу и тип ресурсов, чтобы отфильтровать результаты. |
| [Update-AzVM](/powershell/module/az.compute/update-azvm) | Обновление размера виртуальной машины. |


## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения об управляемых приложениях Azure см. в [этой статье](../overview.md).
* Дополнительные сведения см. в [документации по Azure PowerShell](/powershell/azure/get-started-azureps).
