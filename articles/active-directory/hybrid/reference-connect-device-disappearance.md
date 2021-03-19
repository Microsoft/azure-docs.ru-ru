---
title: Основные сведения об Azure AD Connect 1.4. XX. x и расходе устройства | Документация Майкрософт
description: В этом документе описывается проблема, возникающая в версии 1.4. XX. x Azure AD Connect
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.openlocfilehash: bc159452c81a673ca4a7ed46aa7eff19fd9209eb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "73176023"
---
# <a name="understanding-azure-ad-connect-14xxx-and-device-disappearance"></a>Основные сведения об Azure AD Connect 1.4. XX. x и о выявление устройства
При использовании версии 1.4. XX. x Azure AD Connect некоторые клиенты могут увидеть некоторые или все их устройства с Windows, которые исчезают из Azure AD. Это не причина для беспокойства, так как эти удостоверения устройств не используются Azure AD во время авторизации условного доступа. Это изменение не приведет к удалению устройств Windows, которые были правильно зарегистрированы в Azure AD для гибридного присоединение к Azure AD.

Если вы видите, что удаление объектов устройств в Azure AD превышает пороговое значение удаления экспорта, рекомендуется, чтобы клиент разрешал удаление. [Как разрешить удаление в поток, если они превышают пороговое значение удаления](how-to-connect-sync-feature-prevent-accidental-deletes.md)

## <a name="background"></a>Историческая справка
Устройства Windows, зарегистрированные как гибридные присоединенные к Azure AD, представлены в Azure AD как объекты устройств. Эти объекты устройств можно использовать для условного доступа. Устройства Windows 10 синхронизируются с облаком с помощью Azure AD Connect, устройства Windows нижнего уровня регистрируются напрямую с помощью AD FS или простого единого входа.

## <a name="windows-10-devices"></a>Устройства под управлением Windows 10
Только устройства Windows 10 с конкретным значением атрибута userCertificate, настроенным с помощью гибридного подключения Azure AD, должны быть синхронизированы с облаком Azure AD Connect. В предыдущих версиях Azure AD Connect это требование не было строго применено, что привело к ненужным объектам устройств в Azure AD. Такие устройства в Azure AD всегда неизменности в состоянии "ожидание", так как эти устройства не предназначены для регистрации в Azure AD. 

Эта версия Azure AD Connect синхронизирует только устройства Windows 10, которые правильно настроены для гибридного присоединения к Azure AD. Объекты устройств Windows 10 без конкретных userCertificate присоединение к Azure AD будут удалены из Azure AD.

## <a name="down-level-windows-devices"></a>Down-Level устройств Windows
Azure AD Connect никогда не должны синхронизировать [устройства Windows нижнего уровня](../devices/hybrid-azuread-join-plan.md#windows-down-level-devices). Все устройства в Azure AD, которые ранее были синхронизированы неправильно, теперь будут удалены из Azure AD. Если Azure AD Connect пытается удалить [устройства Windows нижнего уровня](../devices/hybrid-azuread-join-plan.md#windows-down-level-devices), это значит, что устройство не создано в [Microsoft Workplace Join для MSI-компьютеров, отличных от Windows 10](https://www.microsoft.com/download/details.aspx?id=53554) , и не может использоваться какой-либо другой функцией Azure AD.

Некоторым клиентам может потребоваться перейти к следующему [плану: планирование реализации гибридного Azure Active Directory Join](../devices/hybrid-azuread-join-plan.md) , чтобы правильно зарегистрировать устройства Windows и убедиться, что такие устройства могут полностью участвовать в условном доступе на основе устройств. 

## <a name="how-can-i-verify-which-devices-are-deleted-with-this-update"></a>Как проверить, какие устройства удаляются с помощью этого обновления?

Чтобы проверить, какие устройства удаляются, можно использовать следующий сценарий PowerShell: https://gallery.technet.microsoft.com/scriptcenter/Export-Hybrid-Azure-AD-f8e51436

Этот скрипт создает отчет о сертификатах, хранящихся в Active Directory объектах компьютеров, в частности, о сертификатах, выданных гибридной функцией присоединение к Azure AD.
Он проверяет наличие сертификатов в свойстве UserCertificate объекта компьютера в AD, а также для каждого недействительного сертификата проверяет, был ли выдан сертификат для гибридной функции присоединение к Azure AD (т. е. имя субъекта совпадает с CN = {ObjectGUID}).
Ранее Azure AD Connect будет синхронизироваться с Azure AD на любом компьютере, который содержал по крайней мере один действительный сертификат, но начиная с Azure AD Connect версии 1,4, модуль синхронизации может выявление гибридных сертификатов присоединение к Azure AD и будет "клаудфилтер" объект компьютера синхронизироваться с Azure AD, если не существует допустимый гибридный сертификат присоединение к Azure AD.
Устройства Azure AD, которые уже синхронизированы с AD, но не имеют допустимого гибридного сертификата присоединение к Azure AD, будут удалены модулем синхронизации (CloudFiltered = TRUE).

## <a name="next-steps"></a>Next Steps
- [Журнал версий Azure AD Connect](reference-connect-version-history.md)
