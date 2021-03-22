---
title: Пример PowerShell. Экспорт регистраций, секретов и сертификатов приложений в клиенте Azure Active Directory
description: Пример скрипта PowerShell, который позволяет экспортировать все регистрации, секреты и сертификаты приложений для указанных приложений в клиенте Azure Active Directory.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 02/18/2021
ms.author: kenwith
ms.reviewer: mifarca
ms.openlocfilehash: 768f2f3241144085acb7a218b60034cdfa9e45b9
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102193275"
---
# <a name="export-app-registrations-secrets-and-certificates"></a>Экспорт регистраций, секретов и сертификатов приложений

Этот пример скрипта PowerShell экспортирует все регистрации, секреты и сертификаты для указанных приложений в каталоге.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

Для работы с этим примером требуется [модуль Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2) (AzureAD) или [предварительная версия модуля Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (AzureADPreview).

## <a name="sample-script"></a>Пример скрипта

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-all-app-registrations-secrets-and-certs.ps1 "Exports all app registrations, secrets, and certificates for the specified apps in your directory.")]

## <a name="script-explanation"></a>Описание скрипта

| Get-Help | Примечания |
|---|---|
| [Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication?view=azureadps-2.0&preserve-view=true) | Экспортирует все регистрации, секреты и сертификаты приложений для указанных приложений в каталоге. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure AD PowerShell см. в этом [обзоре](/powershell/azure/active-directory/overview).

См. другие [примеры Azure AD PowerShell для управления приложениями](../app-management-powershell-samples.md).
