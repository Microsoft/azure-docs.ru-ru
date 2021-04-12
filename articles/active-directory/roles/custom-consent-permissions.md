---
title: Разрешения для предоставления согласия для приложений для пользовательских ролей в Azure Active Directory | Документация Майкрософт
description: Обзорные сведения о настройке разрешений для предоставления согласия для приложений для пользовательских ролей Azure AD с помощью портала Azure, PowerShell или API Graph.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: overview
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: psignoret
ms.custom: it-pro
ms.openlocfilehash: f9c2c15bbfcf9a9271e629ef26c11ecc4cbaaa6f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98740114"
---
# <a name="app-consent-permissions-for-custom-roles-in-azure-active-directory"></a>Разрешения для предоставления согласия для приложений для пользовательских ролей в Azure Active Directory

Эта статья содержит описание всех существующих на этот момент разрешений для предоставления согласия для приложений для определений пользовательских ролей в Azure Active Directory (Azure AD). В этой статье вы найдете разрешения, необходимые для некоторых распространенных сценариев, связанных с предоставлением согласия и разрешениями для приложений.

## <a name="required-license-plan"></a>Требуемый план лицензирования

Для использования этой функции вашей организации Azure AD требуется лицензия Azure AD Premium P1. Чтобы найти подходящую лицензию, ознакомьтесь с разделом [Сравнение общедоступных функций выпусков Free, Basic и Premium](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="app-consent-permissions"></a>Разрешения на предоставление согласия для приложения

Используйте разрешения, перечисленные в этой статье, для управления политиками согласия приложений, а также разрешением на предоставление согласия для приложений.

> [!NOTE]
> Портал администрирования Azure AD пока не поддерживает добавление разрешений, перечисленных в этой статье, в пользовательское определение роли каталога. Для создания пользовательской роли каталога с разрешениями, перечисленными в этой статье, необходимо [использовать Azure AD PowerShell](custom-create.md#create-a-role-using-powershell).

### <a name="granting-delegated-permissions-to-apps-on-behalf-of-self-user-consent"></a>Предоставление делегированных разрешений приложениям от собственного имени (согласие пользователя)

Чтобы разрешить пользователям предоставлять согласие для приложений от собственного имени (согласие пользователя), необходимо следовать политике согласий для приложений.

- microsoft.directory/servicePrincipals/managePermissionGrantsForSelf.{id}

Где `{id}` заменяется идентификатором [политики согласия для приложений](../manage-apps/manage-app-consent-policies.md), задающей условия, которые должны быть выполнены, чтобы это разрешение было активным.

Например, чтобы разрешить пользователям предоставлять согласие от своего имени в соответствии со встроенной политикой согласия для приложения с идентификатором `microsoft-user-default-low`, следует использовать разрешение `...managePermissionGrantsForSelf.microsoft-user-default-low`.

### <a name="granting-permissions-to-apps-on-behalf-of-all-admin-consent"></a>Предоставление разрешений для приложений от имени всех (согласие администратора)

Чтобы делегировать приложениям согласие администратора на уровне клиента для делегированных разрешений и разрешений приложения (роли приложений):

- microsoft.directory/servicePrincipals/managePermissionGrantsForAll.{id}

Где `{id}` заменяется идентификатором [политики согласия для приложений](../manage-apps/manage-app-consent-policies.md), задающей условия, которые должны быть выполнены, чтобы это разрешение действовало.

Например, чтобы разрешить исполнителям роли предоставлять приложениям согласие администратора на уровне клиента в соответствии с пользовательской [политикой согласия для приложения](../manage-apps/manage-app-consent-policies.md) с идентификатором `low-risk-any-app`, следует использовать разрешение `microsoft.directory/servicePrincipals/managePermissionGrantsForAll.low-risk-any-app`.

### <a name="managing-app-consent-policies"></a>Управление политиками согласия для приложений

Чтобы делегировать создание, обновление и удаление политик согласия для [приложений](../manage-apps/manage-app-consent-policies.md).

- microsoft.directory/permissionGrantPolicies/create
- microsoft.directory/permissionGrantPolicies/standard/read
- microsoft.directory/permissionGrantPolicies/basic/update
- microsoft.directory/permissionGrantPolicies/delete

## <a name="full-list-of-permissions"></a>Полный список разрешений

Разрешение | Описание
---------- | -----------
microsoft.directory/servicePrincipals/managePermissionGrantsForSelf.{id} | Предоставляет возможность давать согласие для приложения от своего имени (согласия пользователя) в соответствии с политикой согласий для приложения `{id}`.
microsoft.directory/servicePrincipals/managePermissionGrantsForAll.{id} | Предоставляет разрешение на предоставление согласия для приложений от имени всех (согласия администратора на уровне клиента) в соответствии с политикой согласий для приложений `{id}`.
microsoft.directory/permissionGrantPolicies/standard/read | Предоставляет возможность читать политики согласия для приложений.
microsoft.directory/permissionGrantPolicies/basic/update | Предоставляет возможность обновлять основные свойства существующих политик согласия для приложений.
microsoft.directory/permissionGrantPolicies/create | Предоставляет возможность создавать политики согласия для приложений.
microsoft.directory/permissionGrantPolicies/delete | Предоставляет возможность удалять политики согласия для приложений.

## <a name="next-steps"></a>Дальнейшие действия

- [Создание настраиваемых ролей с помощью портала Azure, Azure AD PowerShell или API Graph](custom-create.md)
- [Просмотрите назначения для настраиваемой роли](../roles/view-assignments.md).
