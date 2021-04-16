---
title: Пример PowerShell. Замена сертификатов в приложениях Application Proxy
description: Пример PowerShell, который выполняет массовую замену сертификатов в приложениях Azure Active Directory (Azure AD) Application Proxy.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 12/05/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: a2775ad9de9fa422f32342ea4f9e7bdba1a50d5f
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107376993"
---
# <a name="get-all-application-proxy-applications-published-with-the-identical-certificate-and-replace-it"></a>Получение сведений обо всех приложениях Application Proxy с одинаковыми сертификатами и их замена

Этот пример скрипта PowerShell позволяет выполнить массовую замену сертификатов для всех приложений Azure Active Directory (Azure AD) Application Proxy, опубликованных с одним и тем же сертификатом.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Для работы с этим примером требуется [модуль Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2) (AzureAD) или [предварительная версия модуля Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (AzureADPreview).

## <a name="sample-script"></a>Пример скрипта

[!code-azurepowershell[main](~/powershell_scripts/application-proxy/get-custom-domain-replace-cert.ps1 "Get all Application Proxy applications published with the identical certificate and replace it")]

## <a name="script-explanation"></a>Описание скрипта

| Get-Help | Примечания |
|---|---|
|[Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal) | Получает субъект-службу. |
|[Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication) | Получает приложение Azure AD. |
|[Get-AzureADApplicationProxyApplication](/powershell/module/azuread/get-azureadapplicationproxyapplication) | Извлекает приложение, настроенное для Application Proxy в Azure AD. |
|[Set-AzureADApplicationProxyApplicationCustomDomainCertificate](/powershell/module/azuread/set-azureadapplicationproxyapplicationcustomdomaincertificate) | Назначает сертификат приложению, настроенному для Application Proxy в Azure AD. Эта команда отправляет сертификат и позволяет приложению использовать личные домены. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure AD PowerShell см. в этом [обзоре](/powershell/azure/active-directory/overview).

См. другие [примеры Azure AD PowerShell для Azure AD Application Proxy](../application-proxy-powershell-samples.md).
