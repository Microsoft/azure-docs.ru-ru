---
title: Примеры PowerShell для управления приложениями в Azure Active Directory
description: Эти примеры PowerShell используются для приложений, которыми вы управляете в своем арендаторе Azure Active Directory. Их можно использовать для поиска данных об истечении срока действия секретов и сертификатов.
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
ms.openlocfilehash: f5f7ec8245a43440a400b9ca6b55bf1093eb62cc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102636194"
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
