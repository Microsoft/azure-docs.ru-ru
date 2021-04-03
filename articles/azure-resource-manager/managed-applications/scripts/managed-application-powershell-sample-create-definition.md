---
title: Создание определения управляемого приложения с помощью Azure PowerShell
description: Здесь представлен пример скрипта Azure PowerShell, с помощью которого можно создать определение управляемого приложения в подписке Azure.
author: tfitzmac
ms.devlang: powershell
ms.topic: sample
ms.date: 10/27/2017
ms.author: tomfitz
ms.openlocfilehash: b605d63db09b81bfe73eca8f4abd7a38d2325004
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "86056027"
---
# <a name="create-a-managed-application-definition-with-powershell"></a>Создание определения управляемого приложения с помощью Azure PowerShell

С помощью этого скрипта определение управляемого приложения публикуется в каталоге службы.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../../powershell_scripts/managed-applications/create-definition/create-definition.ps1 "Create definition")]


## <a name="script-explanation"></a>Описание скрипта

В этом скрипте используется приведенная ниже команда для создания определения управляемого приложения. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzManagedApplicationDefinition](/powershell/module/az.resources/new-azmanagedapplicationdefinition) | Позволяет создать определение управляемого приложения. Укажите пакет с необходимыми файлами. |


## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения об управляемых приложениях Azure см. в [этой статье](../overview.md).
* Дополнительные сведения см. в [документации по Azure PowerShell](/powershell/azure/get-started-azureps).
