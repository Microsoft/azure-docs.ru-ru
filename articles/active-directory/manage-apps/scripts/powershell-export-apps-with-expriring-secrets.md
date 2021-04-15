---
title: Пример PowerShell. Экспорт приложений с секретами и сертификатами с истекающим сроком действия в клиенте Azure Active Directory.
description: Пример PowerShell, который экспортирует все приложения с секретами и сертификатами с истекающим сроком действия для указанных приложений в клиенте Azure Active Directory.
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 03/09/2021
ms.author: iangithinji
ms.reviewer: mifarca
ms.openlocfilehash: 7f129e06904497b43eff8a3f0221fb57565ac112
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107375412"
---
# <a name="export-apps-with-expiring-secrets-and-certificates"></a>Экспорт приложений с секретами и сертификатами с истекающим сроком действия

Этот пример скрипта PowerShell экспортирует все регистрации приложений с секретами, сертификатами и владельцами с истекающим сроком действия для указанных приложений из каталога в CSV-файл.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

Для работы с этим примером требуется [модуль Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2) (AzureAD) или [предварительная версия модуля Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (AzureADPreview).

## <a name="sample-script"></a>Пример скрипта

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-apps-with-expiring-secrets.ps1 "Exports all apps with expiring secrets and certificates for the specified apps in your directory.")]

## <a name="script-explanation"></a>Описание скрипта

Скрипт можно использовать напрямую без изменений. Администратору предложат ввести дату окончания срока действия и указать, должны ли отображаться секреты или сертификаты с уже истекшим сроком действия или нет.

С помощью команды Add-Member создаются столбцы в CSV-файле.
С помощью команды New-Object создается объект, который будет использоваться для столбцов при экспорте CSV-файла.
Вы можете изменить переменную $Path непосредственно в PowerShell, указав путь к CSV-файлу (если вы предпочитаете неинтерактивный экспорт).

| Get-Help | Примечания |
|---|---|
| [Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication?view=azureadps-2.0&preserve-view=true) | Извлекает приложение из каталога. |
| [Get-AzureADApplicationOwner](/powershell/module/azuread/Get-AzureADApplicationOwner?view=azureadps-2.0&preserve-view=true) | Извлекает владельцев приложения из каталога. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure AD PowerShell см. в этом [обзоре](/powershell/azure/active-directory/overview).

Дополнительные сведения см. в статье [Пример PowerShell для управления приложениями в Azure Active Directory](../app-management-powershell-samples.md).
