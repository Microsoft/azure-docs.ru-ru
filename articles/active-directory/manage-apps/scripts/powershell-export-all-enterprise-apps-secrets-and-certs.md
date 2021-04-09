---
title: Пример PowerShell. Экспорт секретов и сертификатов для корпоративных приложений в клиенте Azure Active Directory.
description: Пример скрипта PowerShell, который позволяет экспортировать все секреты и сертификаты для указанных корпоративных приложений в клиенте Azure Active Directory.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 03/09/2021
ms.author: kenwith
ms.reviewer: mifarca
ms.openlocfilehash: 20caefe74a7c047fb8690bb1d9e6f4eb9da7e9b7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102635201"
---
# <a name="export-secrets-and-certificates-for-enterprise-apps"></a>Экспорт секретов и сертификатов для корпоративных приложений
Этот пример скрипта PowerShell позволяет экспортировать все секреты, сертификаты и сведения о владельцах для указанных корпоративных приложений из вашего каталога в CSV-файл.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

Для работы с этим примером требуется [модуль Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2) (AzureAD) или [предварительная версия модуля Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (AzureADPreview).

## <a name="sample-script"></a>Пример скрипта

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-all-enterprise-apps-secrets-and-certs.ps1 "Exports all secrets and certificates for the specified enterprise apps in your directory.")]

## <a name="script-explanation"></a>Описание скрипта

С помощью команды Add-Member создаются столбцы в CSV-файле.
Вы можете изменить переменную $Path непосредственно в PowerShell, указав путь к CSV-файлу (если вы предпочитаете неинтерактивный экспорт).

| Get-Help | Примечания |
|---|---|
| [Get-AzureADServicePrincipal](/powershell/module/azuread/Get-azureADServicePrincipal?view=azureadps-2.0&preserve-view=true) | Извлечение сведений о корпоративном приложении из вашего каталога. |
| [Get-AzureADServicePrincipalOwner](/powershell/module/azuread/Get-AzureADServicePrincipalOwner?view=azureadps-2.0&preserve-view=true) | Извлечение сведений о владельцах корпоративного приложения из вашего каталога. |


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure AD PowerShell см. в этом [обзоре](/powershell/azure/active-directory/overview).

Дополнительные сведения см. в статье [Пример PowerShell для управления приложениями в Azure Active Directory](../app-management-powershell-samples.md).
