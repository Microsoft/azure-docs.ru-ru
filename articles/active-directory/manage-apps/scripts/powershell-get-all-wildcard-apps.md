---
title: Пример PowerShell. Вывод сведений о приложениях Application Proxy, использующих подстановочные знаки
description: Пример PowerShell, который выводит данные обо всех приложениях Azure Active Directory (Azure AD) Application Proxy, использующих подстановочные знаки.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 12/05/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 944fe9606963c4d4fb9ec41e4d1a87ab62d1bc6b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102548986"
---
# <a name="get-all-application-proxy-apps-using-wildcard-publishing"></a>Получение всех приложений прокси приложения, которые используют публикацию с подстановочными знаками

Этот пример скрипта PowerShell выводит данные обо всех приложениях Azure Active Directory (Azure AD) Application Proxy, использующих при публикации подстановочные знаки.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Для работы с этим примером требуется [модуль AzureAD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2) (AzureAD) или [предварительная версия модуля AzureAD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (AzureADPreview).

## <a name="sample-script"></a>Пример скрипта

[!code-azurepowershell[main](~/powershell_scripts/application-proxy/get-all-wildcard-apps.ps1 "Get all Application Proxy apps using wildcards")]

## <a name="script-explanation"></a>Описание скрипта

| Get-Help | Примечания |
|---|---|
|[Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal) | Получает субъект-службу. |
|[Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication) | Получает приложение Azure AD. |
|[Get-AzureADApplicationProxyApplication](/powershell/module/azuread/get-azureadapplicationproxyapplication) | Извлекает приложение, настроенное для Application Proxy в Azure AD. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure AD PowerShell см. в этом [обзоре](/powershell/azure/active-directory/overview).

См. другие [примеры Azure AD PowerShell для Azure AD Application Proxy](../application-proxy-powershell-samples.md).
