---
title: Пример PowerShell. Экспорт секретов и сертификатов для регистраций приложений в арендаторе Azure Active Directory.
description: Пример скрипта PowerShell, который позволяет экспортировать все секреты и сертификаты для указанных регистраций приложений в арендаторе Azure Active Directory.
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
ms.openlocfilehash: b5cbb6b3843e81d9265405dcea24a092e57bf65e
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107377051"
---
# <a name="export-secrets-and-certificates-for-app-registrations"></a>Экспорт секретов и сертификатов для регистраций приложений

Этот пример скрипта PowerShell экспортирует все секреты и сертификаты для указанных регистраций приложений из вашего каталога в CSV-файл.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

Для работы с этим примером требуется [модуль Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2) (AzureAD) или [предварительная версия модуля Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (AzureADPreview).

## <a name="sample-script"></a>Пример скрипта

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-all-app-registrations-secrets-and-certs.ps1 "Exports all secrets and certificates for the specified app registrations in your directory.")]

## <a name="script-explanation"></a>Описание скрипта

С помощью команды Add-Member создаются столбцы в CSV-файле.
Вы можете изменить переменную $Path непосредственно в PowerShell, указав путь к CSV-файлу (если вы предпочитаете неинтерактивный экспорт).

| Get-Help | Примечания |
|---|---|
| [Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication) | Извлекает приложение из каталога. |
| [Get-AzureADApplicationOwner](/powershell/module/azuread/Get-AzureADApplicationOwner) | Извлекает владельцев приложения из каталога. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure AD PowerShell см. в этом [обзоре](/powershell/azure/active-directory/overview).

Дополнительные сведения см. в статье [Пример PowerShell для управления приложениями в Azure Active Directory](../app-management-powershell-samples.md).
