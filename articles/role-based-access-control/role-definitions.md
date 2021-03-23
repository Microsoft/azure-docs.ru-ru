---
title: Общие сведения об определениях ролей Azure в Azure RBAC
description: Узнайте о определениях ролей Azure в контроле доступа на основе ролей Azure (Azure RBAC) для точного управления доступом к ресурсам Azure.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: conceptual
ms.workload: identity
ms.date: 03/22/2021
ms.author: rolyon
ms.custom: ''
ms.openlocfilehash: 5b2ec3289d187997763ee0d9280a777d4fa1f396
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104801763"
---
# <a name="understand-azure-role-definitions"></a>Общие сведения о определениях ролей Azure

Если вы пытаетесь понять, как работает роль Azure, или вы создаете собственную [настраиваемую роль Azure](custom-roles.md), то полезно понимать, как определяются роли. В этой статье подробно описаны определения ролей и приведены примеры.

## <a name="role-definition"></a>Определение роли

*Определение роли* представляет собой коллекцию разрешений. Иногда это просто называется *ролью*. В определении роли перечисляются операции, которые можно выполнить, например чтение, запись и удаление. Он также может выводить список операций, исключаемых из разрешенных операций или операций, связанных с базовыми данными.

Ниже приведен пример свойств в определении роли при отображении с помощью Azure PowerShell.

```
Name
Id
IsCustom
Description
Actions []
NotActions []
DataActions []
NotDataActions []
AssignableScopes []
```

Ниже приведен пример свойств в определении роли при отображении с использованием портал Azure, Azure CLI или REST API:

```
roleName
name
type
description
actions []
notActions []
dataActions []
notDataActions []
assignableScopes []
```

В следующей таблице описано, что означают свойства роли.

| Свойство | Описание |
| --- | --- |
| `Name`</br>`roleName` | Отображаемое имя роли. |
| `Id`</br>`name` | Уникальный идентификатор роли. Встроенные роли имеют одинаковый идентификатор роли в облаках. |
| `IsCustom`</br>`roleType` | Указывает, является ли эта роль настраиваемой. Задайте для параметра значение `true` или `CustomRole` для пользовательских ролей. Задайте значение `false` или `BuiltInRole` для встроенных ролей. |
| `Description`</br>`description` | Описание роли. |
| `Actions`</br>`actions` | Массив строк, описывающий операции управления, которые роль разрешает выполнять. |
| `NotActions`</br>`notActions` | Массив строк, указывающий операции управления, которые исключаются из разрешенных `Actions`. |
| `DataActions`</br>`dataActions` | Массив строк, указывающий операции с данными, которые роль разрешает выполнять с вашими данными внутри этого объекта. |
| `NotDataActions`</br>`notDataActions` | Массив строк, указывающий операции с данными, которые исключаются из разрешенных `DataActions`. |
| `AssignableScopes`</br>`assignableScopes` | Массив строк, указывающий области, доступные для назначения роли. |

### <a name="operations-format"></a>Формат операций

Операции указываются с помощью строк в следующем формате:

- `{Company}.{ProviderName}/{resourceType}/{action}`

Часть `{action}` строки операции указывает тип операций, которые можно выполнять с ресурсом того или иного типа. Например, в `{action}` могут содержаться следующие подстроки:

| Подстрока действия    | Описание         |
| ------------------- | ------------------- |
| `*` | Подстановочный знак предоставляет доступ ко всем операциям, которые соответствуют строке. |
| `read` | Разрешает операции чтения (GET). |
| `write` | Включает операции записи (размещение или исправление). |
| `action` | Включает настраиваемые операции, такие как перезапуск виртуальных машин (POST). |
| `delete` | Разрешает операции удаления (DELETE). |

### <a name="role-definition-example"></a>Пример определения роли

Вот определение роли [участника](built-in-roles.md#contributor) , отображаемое в Azure PowerShell и Azure CLI. Операция подстановочного знака (`*`) в разделе `Actions` указывает, что субъект, которому назначена эта роль, может выполнять все действия или, иными словами, осуществлять полное управление. Сюда входят и действия, определяемые в будущем (по мере добавления новых типов ресурсов в Azure). Операции, указанные в разделе `NotActions`, вычитаются из раздела `Actions`. В случае роли [участника](built-in-roles.md#contributor) `NotActions` удаляет возможность этой роли управлять доступом к ресурсам, а также управлять назначениями Azure Blueprint.

Роль участника, отображаемая в Azure PowerShell:

```json
{
  "Name": "Contributor",
  "Id": "b24988ac-6180-42a0-ab88-20f7382dd24c",
  "IsCustom": false,
  "Description": "Lets you manage everything except access to resources.",
  "Actions": [
    "*"
  ],
  "NotActions": [
    "Microsoft.Authorization/*/Delete",
    "Microsoft.Authorization/*/Write",
    "Microsoft.Authorization/elevateAccess/Action",
    "Microsoft.Blueprint/blueprintAssignments/write",
    "Microsoft.Blueprint/blueprintAssignments/delete"
  ],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/"
  ]
}
```

Роль участника, отображаемая в Azure CLI:

```json
{
  "assignableScopes": [
    "/"
  ],
  "description": "Lets you manage everything except access to resources.",
  "id": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
  "name": "b24988ac-6180-42a0-ab88-20f7382dd24c",
  "permissions": [
    {
      "actions": [
        "*"
      ],
      "notActions": [
        "Microsoft.Authorization/*/Delete",
        "Microsoft.Authorization/*/Write",
        "Microsoft.Authorization/elevateAccess/Action",
        "Microsoft.Blueprint/blueprintAssignments/write",
        "Microsoft.Blueprint/blueprintAssignments/delete"
      ],
      "dataActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "Contributor",
  "roleType": "BuiltInRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
```

## <a name="management-and-data-operations"></a>Операции управления и работы с данными

Управление доступом на основе ролей для операций управления указывается в свойствах `Actions` и `NotActions` определения роли. Ниже перечислены некоторые примеры операций управления в Azure:

- управление доступом к учетной записи хранения;
- создание, обновление или удаление контейнера больших двоичных объектов;
- удаление группы ресурсов со всеми ее ресурсами.

Доступ к управлению не наследуется на ваши данные, если для метода проверки подлинности контейнера задано значение "учетная запись пользователя Azure AD", а не "ключ доступа". Такое разделение предотвращает неограниченный доступ ролей с подстановочными знаками (`*`) к вашим данным. Например, если у пользователя в подписке есть роль [Читатель](built-in-roles.md#reader), он может просматривать учетную запись хранения, но по умолчанию не имеет возможности просматривать базовые данные.

Ранее управление доступом на основе ролей не использовалось для операций с данными. Авторизация для операций с данными отличается в зависимости от поставщиков ресурсов. Одна и та же модель авторизации управления доступом на основе ролей, используемая для операций управления, была расширена для операций с данными.

Для поддержки операций с данными новые свойства данных были добавлены в определение роли. Операции с данными указаны в свойствах `DataActions` и `NotDataActions`. За счет добавления этих свойств данных, поддерживается разделение управления и данных. Это предотвращает случайный доступ текущих назначений ролей с подстановочными знаками (`*`) к данным. Ниже приведены некоторые операции с данными, которые можно указать в `DataActions` и `NotDataActions`:

- Чтение списка BLOB-объектов в контейнере.
- Запись BLOB-объекта хранилища в контейнер.
- Удаление сообщения из очереди.

Ниже приведено определение роли [читателя данных BLOB-объекта хранилища](built-in-roles.md#storage-blob-data-reader) , которое включает операции в `Actions` `DataActions` свойствах и. Эта роль позволяет считывать контейнер больших двоичных объектов, а также базовые данные большого двоичного объекта.

Роль модуля чтения данных BLOB-объекта хранилища, отображаемая в Azure PowerShell:

```json
{
  "Name": "Storage Blob Data Reader",
  "Id": "2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
  "IsCustom": false,
  "Description": "Allows for read access to Azure Storage blob containers and data",
  "Actions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/read",
    "Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action"
  ],
  "NotActions": [],
  "DataActions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "NotDataActions": [],
  "AssignableScopes": [
    "/"
  ]
}
```

Роль модуля чтения данных BLOB-объекта хранилища, отображаемая в Azure CLI:

```json
{
  "assignableScopes": [
    "/"
  ],
  "description": "Allows for read access to Azure Storage blob containers and data",
  "id": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
  "name": "2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
  "permissions": [
    {
      "actions": [
        "Microsoft.Storage/storageAccounts/blobServices/containers/read",
        "Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action"
      ],
      "notActions": [],
      "dataActions": [
        "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
      ],
      "notDataActions": []
    }
  ],
  "roleName": "Storage Blob Data Reader",
  "roleType": "BuiltInRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
```

В свойства `DataActions` и `NotDataActions` можно добавить только операции с данными. Поставщики ресурсов определяют операции, которые являются операциями с данными, задавая для свойства `isDataAction` значение `true`. Список операций, где свойство `isDataAction` имеет значение `true`, см. в статье [Операции поставщиков ресурсов Azure Resource Manager](resource-provider-operations.md). В определениях ролей, в которых нет операций с данными, свойства `DataActions` и `NotDataActions` не являются обязательными.

Авторизацию всех вызовов API для операций управления выполняет Azure Resource Manager. Авторизацию вызовов API для операций с данными выполняет поставщик ресурсов или Azure Resource Manager.

### <a name="data-operations-example"></a>Пример операций с данными

Чтобы лучше понять, как работают операции управления и операции с данными, рассмотрим конкретный пример. Алисе назначена роль [Владелец](built-in-roles.md#owner) на уровне подписки. Бобу назначена роль [участника данных BLOB-объекта хранилища](built-in-roles.md#storage-blob-data-contributor) в области учетной записи хранения. Этот пример показан на схеме ниже.

![Управление доступом на основе ролей расширено для поддержки операций управления и операций с данными](./media/role-definitions/rbac-management-data.png)

Роль [владельца](built-in-roles.md#owner) для Алисы и роль [участника BLOB-данных хранилища](built-in-roles.md#storage-blob-data-contributor) для Боба имеют следующие действия.

Владелец

&nbsp;&nbsp;&nbsp;&nbsp;Действия<br>
&nbsp;&nbsp;&nbsp;&nbsp;`*`

Участник данных BLOB-объектов хранилища

&nbsp;&nbsp;&nbsp;&nbsp;Действия<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/delete`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/read`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/write`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action`<br>
&nbsp;&nbsp;&nbsp;&nbsp;Действия с данными<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/move/action`<br>
&nbsp;&nbsp;&nbsp;&nbsp;`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write`

Так как Алиса имеет действие с подстановочным знаком ( `*` ) в области действия подписки, его разрешения наследуются, чтобы позволить им выполнять все действия по управлению. Алиса может выполнять чтение, запись и удаление контейнеров. Однако она не может производить операции с данными без выполнения дополнительных шагов. Например, по умолчанию Алиса не может считывать большие двоичные объекты в контейнере. Чтобы выполнять чтение больших двоичных объектов, Алиса должна извлечь ключи доступа к хранилищу и использовать их для получения доступа к большим двоичным объектам.

Разрешения Боба ограничиваются только `Actions` и `DataActions` задаются в роли [участника данных BLOB-объекта хранилища](built-in-roles.md#storage-blob-data-contributor) . На основе роли Боб может выполнять операции управления и операции с данными. Например, Боб может читать, записывать и удалять контейнеры в указанной учетной записи хранения, а также выполнять чтение, запись и удаление больших двоичных объектов.

Дополнительные сведения об управлении и безопасности плоскости данных для хранилища см. в [руководстве по безопасности службы хранилища Azure](../storage/blobs/security-recommendations.md).

### <a name="what-tools-support-using-azure-roles-for-data-operations"></a>Какие средства поддерживают использование ролей Azure для операций с данными?

Для просмотра операций с данными и работы с ними необходимо иметь правильные версии средств или пакетов SDK:

| Средство  | Версия  |
|---------|---------|
| [Azure PowerShell](/powershell/azure/install-az-ps) | 1.1.0 или более поздней версии |
| [Azure CLI](/cli/azure/install-azure-cli) | 2.0.30 или более поздней версии |
| [Azure для .NET](/dotnet/azure/) | 2.8.0-preview или более поздней версии |
| [Пакет Azure SDK для Go](/azure/go/azure-sdk-go-install) | 15.0.0 или более поздней версии |
| [Azure для Java](/java/azure/) | 1.9.0 или более поздней версии |
| [Azure для Python](/azure/python/) | 0.40.0 или более поздней версии |
| [Пакет SDK Azure для Ruby](https://rubygems.org/gems/azure_sdk) | 0.17.1 или более поздней версии |

Чтобы просмотреть и использовать операции с данными в REST API, необходимо установить для параметра **api-version** следующую версию или более позднюю:

- 2018-07-01

## <a name="actions"></a>Actions

В разрешении `Actions` описаны операции управления, которые разрешает выполнять роль. Это коллекция строк операций, которые определяют защищенные действия поставщиков ресурсов Azure. Ниже приведены некоторые примеры операций управления, которые можно использовать в `Actions`.

> [!div class="mx-tableFixed"]
> | Строка операции    | Описание         |
> | ------------------- | ------------------- |
> | `*/read` | Предоставляет доступ к операциям чтения для всех типов ресурсов всех поставщиков ресурсов Azure.|
> | `Microsoft.Compute/*` | Предоставляет доступ ко всем операциям для всех типов ресурсов в поставщике ресурсов Microsoft.Compute.|
> | `Microsoft.Network/*/read` | Предоставляет доступ к операциям чтения для всех типов ресурсов в поставщике ресурсов Microsoft.Network.|
> | `Microsoft.Compute/virtualMachines/*` | Предоставляет доступ ко всем операциям виртуальных машин и их дочерних типов ресурсов.|
> | `microsoft.web/sites/restart/Action` | Предоставляет доступ к перезапуску веб-приложения.|

## <a name="notactions"></a>NotActions

`NotActions`Разрешение определяет операции управления, которые вычитаются или исключаются из разрешенных с помощью `Actions` подстановочных знаков ( `*` ). Используйте `NotActions` разрешение, если набор операций, которые требуется разрешить, проще определить путем вычитания из `Actions` , имеющего подстановочный знак ( `*` ). Доступ, предоставляемый роли (набор действующих разрешений), вычисляется путем вычитания операций `NotActions` из операций `Actions`.

`Actions - NotActions = Effective management permissions`

В следующей таблице показаны два примера действующих разрешений для операции с подстановочными знаками [Microsoft. костманажемент](resource-provider-operations.md#microsoftcostmanagement) :

> [!div class="mx-tableFixed"]
> | Actions | NotActions | Действующие разрешения на управление |
> | --- | --- | --- |
> | `Microsoft.CostManagement/exports/*` | *Нет* | `Microsoft.CostManagement/exports/action`</br>`Microsoft.CostManagement/exports/read`</br>`Microsoft.CostManagement/exports/write`</br>`Microsoft.CostManagement/exports/delete`</br>`Microsoft.CostManagement/exports/run/action` |
> | `Microsoft.CostManagement/exports/*` | `Microsoft.CostManagement/exports/delete` | `Microsoft.CostManagement/exports/action`</br>`Microsoft.CostManagement/exports/read`</br>`Microsoft.CostManagement/exports/write`</br>`Microsoft.CostManagement/exports/run/action` |

> [!NOTE]
> Если пользователю назначена роль, которая исключает определенную операцию в `NotActions`, а также другая роль, которая предоставляет доступ к той же операции, то пользователю будет разрешено выполнять эту операцию. `NotActions` не является запрещающим правилом. Это просто удобный способ создания набора допустимых операций путем исключения некоторых операций.
>

### <a name="differences-between-notactions-and-deny-assignments"></a>Различия между неизменностью и запретом назначений

`NotActions` и отказ от назначения не одинаковы и служат разным целям. `NotActions` — Это удобный способ вычитания определенных действий из операции с подстановочными знаками ( `*` ).

Запрет назначений блокирует выполнение определенных действий пользователями, даже если назначение роли предоставляет им доступ. Дополнительные сведения о запретах назначений Azure см. в [этой статье](deny-assignments.md).

## <a name="dataactions"></a>Действия с данными

В разрешении `DataActions` указаны операции с данными, которые роль разрешает выполнять с вашими данными внутри этого объекта. Например, если у пользователя есть доступ на чтение большого двоичного объекта для учетной записи хранения, это позволит считывать большие двоичные объекты в этой учетной записи хранения. Ниже приведены некоторые примеры операций с данными, которые можно использовать в `DataActions`.

> [!div class="mx-tableFixed"]
> | Строка операции    | Описание         |
> | ------------------- | ------------------- |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` | Возвращает большой двоичный объект или список больших двоичных объектов. |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write` | Возвращает результат записи большого двоичного объекта. |
> | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/read` | Возвращает сообщение. |
> | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/*` | Возвращает сообщение или результат записи или удаления сообщения. |

## <a name="notdataactions"></a>NotDataActions

`NotDataActions`Разрешение указывает операции с данными, которые вычитаются или исключаются из разрешенных с помощью `DataActions` подстановочных знаков ( `*` ). Используйте `NotDataActions` разрешение, если набор операций, которые требуется разрешить, проще определить путем вычитания из `DataActions` , имеющего подстановочный знак ( `*` ). Доступ, предоставляемый роли (набор действующих разрешений), вычисляется путем вычитания операций `NotDataActions` из операций `DataActions`. Каждый поставщик ресурсов предоставляет соответствующий набор API-интерфейсов для выполнения операций с данными.

`DataActions - NotDataActions = Effective data permissions`

В следующей таблице показаны два примера действующих разрешений для операции с подстановочными знаками [Microsoft. Storage](resource-provider-operations.md#microsoftstorage) :

> [!div class="mx-tableFixed"]
> | Действия с данными | NotDataActions | Действующие разрешения на доступ к данным |
> | --- | --- | --- |
> | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/*` | *Нет* | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/read`</br>`Microsoft.Storage/storageAccounts/queueServices/queues/messages/write`</br>`Microsoft.Storage/storageAccounts/queueServices/queues/messages/delete`</br>`Microsoft.Storage/storageAccounts/queueServices/queues/messages/add/action`</br>`Microsoft.Storage/storageAccounts/queueServices/queues/messages/process/action` |
> | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/*` | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/delete`</br> | `Microsoft.Storage/storageAccounts/queueServices/queues/messages/read`</br>`Microsoft.Storage/storageAccounts/queueServices/queues/messages/write`</br>`Microsoft.Storage/storageAccounts/queueServices/queues/messages/add/action`</br>`Microsoft.Storage/storageAccounts/queueServices/queues/messages/process/action` |

> [!NOTE]
> Если пользователю назначена роль, которая исключает определенную операцию с данными в `NotDataActions`, а также другая роль, которая предоставляет доступ к той же операции, то пользователю будет разрешено выполнять эту операцию. `NotDataActions` не является запрещающим правилом. Это просто удобный способ создания набора допустимых операций с данными путем исключения некоторых операций.
>

## <a name="assignablescopes"></a>AssignableScopes

`AssignableScopes`Свойство определяет области (группы управления, подписки или группы ресурсов), для которых доступно определение этой роли. Роль можно сделать доступной для назначения только в группах управления, подписках или группах ресурсов, которым он необходим. Необходимо использовать по крайней мере одну группу управления, подписку или группу ресурсов.

Для встроенных ролей в качестве `AssignableScopes` задана корневая область (`"/"`). Корневая область указывает, что роль доступна для назначения во всех областях. Примеры допустимых назначаемых областей:

> [!div class="mx-tableFixed"]
> | Роль доступна для назначения | Пример |
> |----------|---------|
> | Одна подписка | `"/subscriptions/{subscriptionId1}"` |
> | Две подписки | `"/subscriptions/{subscriptionId1}", "/subscriptions/{subscriptionId2}"` |
> | Группа сетевых ресурсов | `"/subscriptions/{subscriptionId1}/resourceGroups/Network"` |
> | Одна группа управления | `"/providers/Microsoft.Management/managementGroups/{groupId1}"` |
> | Группа управления и подписка | `"/providers/Microsoft.Management/managementGroups/{groupId1}", /subscriptions/{subscriptionId1}",` |
> | Все области (применимо только к встроенным ролям) | `"/"` |

Дополнительные сведения о `AssignableScopes` пользовательских ролях см. в статье [пользовательские роли Azure](custom-roles.md).

## <a name="next-steps"></a>Следующие шаги

* [Встроенные роли Azure](built-in-roles.md)
* [Настраиваемые роли Azure](custom-roles.md)
* [Операции с поставщиками ресурсов Azure](resource-provider-operations.md)
