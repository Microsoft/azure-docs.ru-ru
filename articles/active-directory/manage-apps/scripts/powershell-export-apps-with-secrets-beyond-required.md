---
title: Пример PowerShell. Экспорт приложений с секретами и сертификатами со сроком действия, который истекает после требуемой даты, в арендатор Azure Active Directory.
description: Пример PowerShell, позволяющий экспортировать все приложения с секретами и сертификатами, срок действия которых истекает после требуемой даты, для указанных приложений в арендаторе Azure Active Directory.
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
ms.openlocfilehash: daeea48758a9f08e7eedbfcaddcde3815f5c1e16
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105729105"
---
# <a name="export-apps-with-secrets-and-certificates-expiring-beyond-the-required-date"></a>Экспорт приложений с секретами и сертификатами, срок действия которых истекает после требуемой даты

Этот пример скрипта PowerShell экспортирует (не в интерактивном режиме) все секреты и сертификаты регистрации, срок действия которых истекает после требуемой даты, для указанных приложений из каталога в CSV-файле.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-apps-with-secrets-beyond-required.ps1 "Exports all apps with secrets and certificates expiring beyond the required date for the specified apps in your directory.")]

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт работает не в интерактивном режиме. Администратору потребуется изменить значения в разделе #PARAMETERS TO CHANGE, указав идентификатор своего приложения, секрет приложения, имя арендатора, срок действия учетных данных приложения и путь, по которому будет экспортирован CSV-файл.
Этот скрипт использует [поток OAuth Client_Credential](../../develop/v2-oauth2-client-creds-grant-flow.md). Функция RefreshToken создаст маркер доступа на основе значений параметров, измененных администратором.

С помощью команды Add-Member создаются столбцы в CSV-файле.

| Get-Help | Примечания |
|---|---|
| [Invoke-WebRequest](/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.1&preserve-view=true) | Отправляет HTTP- и HTTPS-запросы к веб-странице или веб-службе. Анализирует ответ и возвращает коллекции ссылок, изображений и других значимых HTML-элементов. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure AD PowerShell см. в этом [обзоре](/powershell/azure/active-directory/overview).

Дополнительные сведения см. в статье [Пример PowerShell для управления приложениями в Azure Active Directory](../app-management-powershell-samples.md).
