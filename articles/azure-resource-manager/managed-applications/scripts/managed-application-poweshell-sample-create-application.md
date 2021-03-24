---
title: Пример скрипта Azure PowerShell. Развертывание управляемого приложения
description: Здесь представлен пример скрипта Azure PowerShell, с помощью которого можно развернуть определение управляемого приложения в подписке.
author: tfitzmac
ms.devlang: powershell
ms.topic: sample
ms.date: 10/27/2017
ms.author: tomfitz
ms.openlocfilehash: a2687e9c943df8454ff42a17f44866dcdb7f4730
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86055891"
---
# <a name="deploy-a-managed-application-for-a-service-catalog-with-powershell"></a>Развертывание управляемого приложения для каталога служб с помощью Azure PowerShell

С помощью этого скрипта определение управляемого приложения развертывается из каталога служб.


[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../../powershell_scripts/managed-applications/create-application/create-application.ps1 "Create application")]


## <a name="script-explanation"></a>Описание скрипта

В этом скрипте используется следующая команда для развертывания управляемого приложения. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzManagedApplication](/powershell/module/az.resources/new-azmanagedapplication) | Позволяет создать управляемое приложение. Укажите идентификатор определения и параметры для шаблона. |


## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения об управляемых приложениях Azure см. в [этой статье](../overview.md).
* Дополнительные сведения см. в [документации по Azure PowerShell](/powershell/azure/get-started-azureps).
