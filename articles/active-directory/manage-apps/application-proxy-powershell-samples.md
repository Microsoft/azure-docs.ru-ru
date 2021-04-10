---
title: Примеры PowerShell для прокси приложения Azure AD
description: Эти примеры PowerShell для прокси приложения Azure AD позволяют получить сведения о приложениях и соединителях прокси приложения, доступных в каталоге, назначить приложениям пользователей и группы, а также получить сведения о сертификатах.
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
ms.collection: M365-identity-device-management
ms.openlocfilehash: e5189df623325578bb3b579f5c4c3456067986ff
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105709486"
---
# <a name="azure-ad-powershell-examples-for-azure-ad-application-proxy"></a>Примеры Azure AD PowerShell для прокси приложения Azure AD

В следующей таблице содержатся ссылки на примеры скриптов PowerShell для прокси приложения Azure AD. Для этих примеров требуется [модуль Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2) или [предварительная версия модуля Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true), если не указано иное.


Дополнительные сведения о командлетах, используемых в этих примерах, см. в разделах [Управление приложением прокси приложения](/powershell/module/azuread/#application_proxy_application_management) и [Управление соединителем прокси приложения](/powershell/module/azuread/#application_proxy_connector_management).

| Ссылка | Описание |
|---|---|
|**Приложения прокси приложений**||
| [Вывод основных сведений обо всех приложениях прокси приложения](scripts/powershell-get-all-app-proxy-apps-basic.md) | Выводит список основных сведений (AppId, DisplayName, ObjId) по всем приложениям прокси приложения в каталоге. |
| [Вывод подробных сведений обо всех приложениях прокси приложения](scripts/powershell-get-all-app-proxy-apps-extended.md) | Выводит список подробных сведений (AppId, DisplayName, ExternalUrl, InternalUrl, ExternalAuthenticationType) обо всех приложениях прокси приложения в каталоге.  |
| [Получение списка приложений прокси приложения по группам соединителей](scripts/powershell-get-all-app-proxy-apps-by-connector-group.md) | Выводит сведения обо всех приложениях прокси приложения в каталоге с информацией о том, каким группам соединителей назначены эти приложения. |
| [Получение всех приложений прокси приложения с политикой срока действия маркеров](scripts/powershell-get-all-app-proxy-apps-with-policy.md) | Выводит список всех приложений прокси приложения в каталоге, для которых применяется политика срока действия маркера, и сведения об этой политике. Для работы с этим примером требуется [предварительная версия модуля Azure AD PowerShell (версии 2) для Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true). |
|**Группы соединителей**||
| [Получение всех групп соединителей и соединителей в каталоге](scripts/powershell-get-all-connectors.md) | Выводит список всех групп соединителей и соединителей в каталоге. |
| [Перемещение всех приложений, назначенных группе соединителей, в другую группу соединителей](scripts/powershell-move-all-apps-to-connector-group.md) | Перемещает все приложения, в этот момент назначенные некоторой группе соединителей, в другую группу соединителей. |
|**Назначенные пользователи и группы**||
| [Отображение пользователей и групп, назначенных приложению прокси приложения](scripts/powershell-display-users-group-of-app.md) | Выводит список пользователей и групп, назначенных конкретному приложению прокси приложения. |
| [Назначение пользователя приложению](scripts/powershell-assign-user-to-app.md) | Назначает приложению конкретного пользователя. |
| [Назначение группы приложению](scripts/powershell-assign-group-to-app.md) | Назначает приложению конкретную группу. |
|**Конфигурация внешних URL-адресов**||
| [Получение всех приложений прокси приложения для доменов по умолчанию (.msappproxy.net)](scripts/powershell-get-all-default-domain-apps.md)  | Выводит список всех приложений прокси приложения для доменов по умолчанию (.msappproxy.net) |
| [Получение всех приложений прокси приложения, которые используют публикацию с подстановочными знаками](scripts/powershell-get-all-wildcard-apps.md) | Выводит список всех приложений прокси приложения, которые используют публикацию с подстановочными знаками. |
|**Конфигурация личного домена**||
| [Получение всех приложений прокси приложения, которые используют личные домены, и сведений о сертификатах](scripts/powershell-get-all-custom-domains-and-certs.md) | Выводит список всех приложений прокси приложения, которые используют личные домены, и сведения о сертификатах, связанные с этими личными доменами. |
| [Получение всех опубликованных приложений прокси приложений Azure AD, для которых не отправлены сертификаты](scripts/powershell-get-all-custom-domain-no-cert.md) | Выводит список всех приложений прокси приложения, которые используют личные домены, но не имеют отправленных и действительных сертификатов TLS/SSL. |
| [Получение всех приложений прокси приложения Azure AD, опубликованных с одинаковыми сертификатами](scripts/powershell-get-custom-domain-identical-cert.md) | Выводит список всех приложений прокси приложений Azure AD, которые опубликованы с одинаковыми сертификатами. |
| [Получение всех приложений прокси приложений Azure AD, опубликованных с одинаковыми сертификатами, и замена сертификата](scripts/powershell-get-custom-domain-replace-cert.md) | Для тех приложений прокси приложения Azure AD, которые опубликованы с одинаковыми сертификатами, вы можете заменить все сертификаты одновременно. |
