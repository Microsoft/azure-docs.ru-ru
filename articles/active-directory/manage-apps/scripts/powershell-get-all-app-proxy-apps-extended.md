---
title: Пример PowerShell. Вывод расширенных сведений о приложениях Application Proxy
description: Пример скрипта PowerShell, который выводит список всех приложений Azure Active Directory (Azure AD) Application Proxy, а также сведения об идентификаторах приложений (AppId), именах (DisplayName), внешних URL-адресах (ExternalUrl), внутренних URL-адресах (InternalUrl) и типе аутентификации (ExternalAuthenticationType).
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
ms.openlocfilehash: ccd0c7be7fd0dd533028faa0dc2bbdad30d74c79
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99258716"
---
# <a name="get-all-application-proxy-apps-and-list-extended-information"></a>Получение сведений о приложениях Application Proxy и вывод расширенной информации

Этот пример скрипта PowerShell позволяет вывести сведения обо всех приложениях Azure Active Directory (Azure AD) Application Proxy, в том числе данные об идентификаторах приложений (AppId), именах (DisplayName), внешних URL-адресах (ExternalUrl), внутренних URL-адресах (InternalUrl), типе аутентификации (ExternalAuthenticationType), режиме единого входа и других параметрах.

Изменение значения переменной ssoMode позволяет фильтровать выходные данные в режиме единого входа. Дополнительные сведения задокументированы в скрипте.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Для работы с этим примером требуется [модуль Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2) (Azure AD).

## <a name="sample-script"></a>Пример скрипта

[!code-azurepowershell[main](~/powershell_scripts/application-proxy/get-all-appproxy-apps-extended.ps1 "Get all Application Proxy apps")]

## <a name="script-explanation"></a>Описание скрипта

| Get-Help | Примечания |
|---|---|
|[Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal) | Получает субъект-службу. |
|[Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication) | Получает приложение Azure AD. |
|[Get-AzureADApplicationProxyApplication](/powershell/module/azuread/get-azureadapplicationproxyapplication) | Извлекает приложение, настроенное для Application Proxy в Azure AD. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure AD PowerShell см. в этом [обзоре](/powershell/azure/active-directory/overview).

См. другие [примеры Azure AD PowerShell для Azure AD Application Proxy](../application-proxy-powershell-samples.md).
