---
title: Поиск идентификаторов объектов удостоверений для проверки подлинности с помощью API Azure для FHIR
description: В этой статье объясняется, как определить идентификаторы объектов удостоверений, необходимые для настройки проверки подлинности для Azure API для FHIR.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 706d7e081743f2bab1f593e00dc792f218a000ea
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103019598"
---
# <a name="find-identity-object-ids-for-authentication-configuration"></a>Поиск идентификаторов объектов удостоверений для конфигурации проверки подлинности

В этой статье вы узнаете, как найти идентификаторы объектов удостоверений, необходимые при настройке API Azure для FHIR, чтобы [использовать внешний или вторичный клиент Active Directory](configure-local-rbac.md) для плоскости данных.

## <a name="find-user-object-id"></a>Найти идентификатор объекта пользователя

Если у вас есть пользователь с именем пользователя `myuser@contoso.com` , вы можете выбрать его `ObjectId` с помощью следующей команды PowerShell:

```azurepowershell-interactive
$(Get-AzureADUser -Filter "UserPrincipalName eq 'myuser@contoso.com'").ObjectId
```

также можно использовать Azure CLI:

```azurecli-interactive
az ad user show --id myuser@contoso.com --query objectId --out tsv
```

## <a name="find-service-principal-object-id"></a>Поиск идентификатора объекта субъекта-службы

Предположим, вы зарегистрировали [клиентское приложение службы](register-service-azure-ad-client-app.md) и хотите разрешить этому клиенту службы доступ к API Azure для FHIR. идентификатор объекта для субъекта-службы клиента можно найти с помощью следующей команды PowerShell:

```azurepowershell-interactive
$(Get-AzureADServicePrincipal -Filter "AppId eq 'XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX'").ObjectId
```

где `XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX` — идентификатор клиентского приложения службы. Кроме того, можно использовать `DisplayName` клиент службы:

```azurepowershell-interactive
$(Get-AzureADServicePrincipal -Filter "DisplayName eq 'testapp'").ObjectId
```

При использовании Azure CLI можно использовать:

```azurecli-interactive
az ad sp show --id XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX --query objectId --out tsv
```

## <a name="find-a-security-group-object-id"></a>Поиск идентификатора объекта группы безопасности

Если вы хотите указать идентификатор объекта группы безопасности, можно использовать следующую команду PowerShell:

```azurepowershell-interactive
$(Get-AzureADGroup -Filter "DisplayName eq 'mygroup'").ObjectId
```
Где `mygroup` — имя интересующей вас группы.

При использовании Azure CLI можно использовать:

```azurecli-interactive
az ad group show --group "mygroup" --query objectId --out tsv
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как найти идентификаторы объектов удостоверений, необходимые для настройки API Azure для FHIR на использование внешнего или дополнительного клиента Azure Active Directory. Далее прочитайте об использовании идентификаторов объектов для настройки локальных параметров RBAC.
 
>[!div class="nextstepaction"]
>[Настройка локальных параметров RBAC](configure-local-rbac.md)
