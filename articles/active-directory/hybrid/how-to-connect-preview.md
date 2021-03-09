---
title: 'Azure AD Connect выполняет следующие функции: предварительные версии компонентов | Документация Майкрософт'
description: В этой статье более подробно описаны функции Azure AD Connect, находящиеся на этапе предварительной версии.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: c75cd8cf-3eff-4619-bbca-66276757cc07
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/15/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1d48cd8f95ecaf3e537a5221d766af150a51aa31
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102517690"
---
# <a name="more-details-about-features-in-preview"></a>Дополнительная информация о функциях в предварительной версии
В этой статье описывается, как использовать функции, которые сейчас доступны в предварительной версии.

## <a name="azure-ad-connect-sync-v2-endpoint-api"></a>API конечной точки версии 2 для синхронизации Azure AD Connect

Мы развернули новую конечную точку (API) для Azure AD Connect, которая повышает производительность операций службы синхронизации с Azure Active Directory. Используя новую конечную точку версии 2, вы получите заметное улучшение показателей при экспорте и импорте в Azure AD. Эта новая конечная точка также поддерживает синхронизацию групп, насчитывающих до 250 000 участников. Использование этой конечной точки также позволяет выполнять обратную запись Microsoft 365 Объединенных групп без ограничения членства в локальной Active Directory, если включена обратная запись группы. Дополнительные сведения см. в разделе [Azure AD Connect синхронизация API конечной точки Sync v2](how-to-connect-sync-endpoint-api-v2.md).

## <a name="user-writeback"></a>Обратная запись пользователей
> [!IMPORTANT]
> Предварительная версия функции обратной записи пользователей была удалена в августовском обновлении (2015 г.) для Azure AD Connect. Если вы включили ее, следует отключить эту функцию.
>
>

## <a name="next-steps"></a>Дальнейшие действия
Продолжите [выборочную установку Azure AD Connect](how-to-connect-install-custom.md).

Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
