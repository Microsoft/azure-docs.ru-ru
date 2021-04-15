---
title: Примеры PowerShell для управления приложениями в Azure Active Directory
description: Эти примеры PowerShell используются для приложений, которыми вы управляете в своем арендаторе Azure Active Directory. Их можно использовать для поиска данных об истечении срока действия секретов и сертификатов.
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 02/18/2021
ms.author: iangithinji
ms.reviewer: mifarca
ms.openlocfilehash: 6b6314935bfafc2fe6288c30619e1d01242a991d
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378828"
---
# <a name="azure-active-directory-powershell-examples-for-application-management"></a>Пример PowerShell для управления приложениями в Azure Active Directory

В следующей таблице содержатся ссылки на примеры скриптов PowerShell для управления приложениями в Azure AD. Для этих примеров требуется следующее:
- [модуль AzureAD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2); или
- [предварительная версия модуля AzureAD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (если не указано иное).

Дополнительные сведения о командлетах, используемых в этих примера, см. в разделе [Приложения](/powershell/module/azuread/#applications).

| Ссылка | Описание |
|---|---|
|**Скрипты для управления приложениями**||
| [Экспорт секретов и сертификатов (регистрация приложений)](scripts/powershell-export-all-app-registrations-secrets-and-certs.md) | Экспорт секретов и сертификатов для регистраций приложений в арендатор Azure Active Directory. |
| [Экспорт секретов и сертификатов (корпоративные приложения)](scripts/powershell-export-all-enterprise-apps-secrets-and-certs.md) | Экспорт секретов и сертификатов для корпоративных приложений в арендаторе Azure Active Directory. |
| [Экспорт секретов и сертификатов с истекающим сроком действия](scripts/powershell-export-apps-with-expriring-secrets.md) | Экспорт регистраций приложений с секретами и сертификатами, для которых истекает срок действия, а также их владельцев в арендаторе Azure Active Directory. |
| [Экспорт секретов и сертификатов с истекшим сроком действия](scripts/powershell-export-apps-with-secrets-beyond-required.md) | Экспорт регистраций приложений с секретами и сертификатами, для которых истекает срок действия после требуемой даты, в арендаторе Azure Active Directory. В этом случае используется неинтерактивный поток OAuth Client_Credentials. |
