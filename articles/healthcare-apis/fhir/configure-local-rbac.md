---
title: Настройка локального управления доступом на основе ролей (Local RBAC) для Azure API для FHIR
description: В этой статье описывается, как настроить API Azure для FHIR на использование внешнего клиента Azure AD для плоскости данных.
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 03/15/2020
ms.author: matjazl
ms.openlocfilehash: 096e4e3ecbcedaec674e074a2baccbb336e03c94
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020524"
---
# <a name="configure-local-rbac-for-fhir"></a>Настройка локального RBAC для FHIR 

В этой статье объясняется, как настроить API Azure для FHIR на использование внешнего дополнительного клиента Azure Active Directory для управления доступом к плоскости данных. Используйте этот режим, только если вы не можете использовать клиент Azure Active Directory, связанный с подпиской.

> [!NOTE]
> Если плоскость данных службы FHIR настроена для использования основного клиента Azure Active Directory, связанного с вашей подпиской, [Используйте Azure RBAC для назначения ролей плоскости данных](configure-azure-rbac.md).

## <a name="add-service-principal"></a>Добавление субъекта-службы

Локальный RBAC позволяет использовать внешний клиент Azure Active Directory с сервером FHIR. Чтобы разрешить локальной системе RBAC проверять принадлежность к группам в этом клиенте, API Azure для FHIR должен иметь субъект-службу в клиенте. Этот субъект-служба будет создан автоматически в клиентах, привязанных к подпискам, которые развернули API Azure для FHIR, но если к клиенту не привязана подписка, администратору клиента потребуется создать этот субъект-службу с помощью одной из следующих команд:

С помощью `Az` модуля PowerShell:

```azurepowershell-interactive
New-AzADServicePrincipal -ApplicationId 3274406e-4e0a-4852-ba4f-d7226630abb7
```

также можно использовать `AzureAd` модуль PowerShell:

```azurepowershell-interactive
New-AzureADServicePrincipal -AppId 3274406e-4e0a-4852-ba4f-d7226630abb7
```

также можно использовать Azure CLI:

```azurecli-interactive
az ad sp create --id 3274406e-4e0a-4852-ba4f-d7226630abb7
```

## <a name="configure-local-rbac"></a>Настройка локального RBAC

Вы можете настроить API Azure для FHIR на использование внешнего или дополнительного клиента Azure Active Directory в колонке **Аутентификация** :

![Локальные назначения RBAC](media/rbac/local-rbac-guids.png).

В поле Authority (центр) введите допустимый клиент Azure Active Directory. После проверки клиента в поле **Разрешенные идентификаторы объектов** должны быть активированы и можно ввести список идентификаторов объектов идентификаторов. Эти идентификаторы могут быть идентификаторами объектов идентификаторов:

* Пользователь Azure Active Directory.
* Субъект-служба Azure Active Directory.
* Группа безопасности Azure Active Directory.

Дополнительные сведения см. в статье о [поиске идентификаторов объектов идентификаторов](find-identity-object-ids.md) .

После ввода требуемых идентификаторов объектов нажмите кнопку **сохранить** и дождитесь сохранения изменений, прежде чем пытаться получить доступ к плоскости данных с помощью назначенных пользователей, субъектов-служб или групп.

## <a name="caching-behavior"></a>Поведение кэширования

API Azure для FHIR будет кэшировать решения в течение 5 минут. Если вы предоставите пользователю доступ к серверу FHIR, добавив его в список разрешенных идентификаторов объектов или удалите их из списка, вы должны пройти до пяти минут на изменение разрешений для распространения.

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как назначить FHIR доступ к плоскости данных с помощью внешнего (дополнительного) клиента Azure Active Directory. Далее вы узнаете о дополнительных параметрах для Azure API для FHIR:
 
>[!div class="nextstepaction"]
>[Дополнительные параметры API Azure для FHIR](azure-api-for-fhir-additional-settings.md)
