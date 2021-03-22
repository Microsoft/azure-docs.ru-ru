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
ms.openlocfilehash: e63931f62398e1344d001bedb27cc8d0d776bef1
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102193307"
---
# <a name="azure-active-directory-powershell-examples-for-application-management"></a>Пример PowerShell для управления приложениями в Azure Active Directory

В следующей таблице содержатся ссылки на примеры скриптов PowerShell для управления приложениями в Azure AD. Для этих примеров требуется следующее:
- [модуль AzureAD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2); или
- [предварительная версия модуля AzureAD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (если не указано иное).

Дополнительные сведения о командлетах, используемых в этих примера, см. в разделе [Приложения](/powershell/module/azuread/?view=azureadps-2.0#applications&preserve-view=true).

| Ссылка | Описание |
|---|---|
|**Скрипты для управления приложениями**||
| [Экспорт всех регистраций, секретов и сертификатов приложений](scripts/powershell-export-all-app-registrations-secrets-and-certs.md) | Экспортирует все регистрации, секреты и сертификаты приложений для указанных приложений в каталоге. |
