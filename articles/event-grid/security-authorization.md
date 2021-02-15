---
title: 'Сетка событий Azure: безопасность и проверка подлинности'
description: В статье описываются служба "Сетка событий Azure" и ее основные понятия.
ms.topic: conceptual
ms.date: 02/12/2021
ms.openlocfilehash: 326fa00645302eb4b9c9bc59f17c1ca153bdb0b7
ms.sourcegitcommit: d4734bc680ea221ea80fdea67859d6d32241aefc
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/14/2021
ms.locfileid: "100371726"
---
# <a name="authorizing-access-to-event-grid-resources"></a>Авторизация доступа к ресурсам сетки событий
Служба "Сетка событий Azure" позволяет управлять уровнем доступа, предоставляемого разным пользователям, для выполнения различных **операций управления** , таких как список подписок на события, создание новых и создание ключей. Сетка событий использует управление доступом на основе ролей Azure (Azure RBAC).

> [!NOTE]
> EventGrid не поддерживает Azure RBAC для публикации событий в разделах или доменах сетки событий. Используйте ключ или токен подписанного URL-адрес (SAS) для проверки подлинности клиентов, публикующих события. Дополнительные сведения см. в разделе [Проверка подлинности клиентов публикации](security-authenticate-publishing-clients.md). 

## <a name="operation-types"></a>Типы операций
Чтобы получить список операций, поддерживаемых службой "Сетка событий Azure", выполните следующую команду Azure CLI: 

```azurecli-interactive
az provider operation show --namespace Microsoft.EventGrid
```

Следующие операции возвращают потенциально секретную информацию, которая отфильтровывается из обычных операций чтения. Доступ к этим операциям рекомендуется ограничить. 

* Microsoft.EventGrid/eventSubscriptions/getFullUrl/action
* Microsoft.EventGrid/topics/listKeys/action
* Microsoft.EventGrid/topics/regenerateKey/action


## <a name="built-in-roles"></a>Встроенные роли

"Сетка событий" предоставляет две встроенные роли для управления подписками на события. Они важны при реализации [доменов событий](event-domains.md) , поскольку они предоставляют пользователям разрешения, необходимые для подписки на разделы в домене событий. Эти роли предназначены для подписки на события и не предоставляют доступа к действиям, например для создания тем.

Вы можете [назначить эти роли для пользователя или группы](../role-based-access-control/quickstart-assign-role-user-portal.md).

**Участник EventGrid подписки**: управление операциями подписки в сетке событий

```json
[
  {
    "Description": "Lets you manage EventGrid event subscription operations.",
    "IsBuiltIn": true,
    "Id": "428e0ff05e574d9ca2212c70d0e0a443",
    "Name": "EventGrid EventSubscription Contributor",
    "IsServiceRole": false,
    "Permissions": [
      {
        "Actions": [
          "Microsoft.Authorization/*/read",
          "Microsoft.EventGrid/eventSubscriptions/*",
          "Microsoft.EventGrid/systemtopics/eventsubscriptions/*",
          "Microsoft.EventGrid/partnertopics/eventsubscriptions/*",
          "Microsoft.EventGrid/topicTypes/eventSubscriptions/read",
          "Microsoft.EventGrid/locations/eventSubscriptions/read",
          "Microsoft.EventGrid/locations/topicTypes/eventSubscriptions/read",
          "Microsoft.Insights/alertRules/*",
          "Microsoft.Resources/deployments/*",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Support/*"
        ],
        "NotActions": [],
        "DataActions": [],
        "NotDataActions": [],
        "Condition": null
      }
    ],
    "Scopes": [
      "/"
    ]
  }
]
```

**Читатель EventGrid подписки**: чтение подписок на сетку событий

```json
[
  {
    "Description": "Lets you read EventGrid event subscriptions.",
    "IsBuiltIn": true,
    "Id": "2414bbcf64974faf8c65045460748405",
    "Name": "EventGrid EventSubscription Reader",
    "IsServiceRole": false,
    "Permissions": [
      {
        "Actions": [
          "Microsoft.Authorization/*/read",
          "Microsoft.EventGrid/eventSubscriptions/read",
          "Microsoft.EventGrid/topicTypes/eventSubscriptions/read",
          "Microsoft.EventGrid/locations/eventSubscriptions/read",
          "Microsoft.EventGrid/locations/topicTypes/eventSubscriptions/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read"
        ],
        "NotActions": [],
        "DataActions": [],
        "NotDataActions": []
       }
    ],
    "Scopes": [
      "/"
    ]
  }
]
```

## <a name="custom-roles"></a>Пользовательские роли

Если вам нужно указать разрешения, которые отличаются от встроенных ролей, можно создать пользовательские роли.

Далее приведены примеры определений ролей Сетки событий, позволяющих пользователям выполнять различные действия. Эти пользовательские роли отличаются от встроенных, потому что они предоставляют более широкий доступ, чем просто подписка на события.

**EventGridReadOnlyRole.json**: разрешено только для операций чтения.

```json
{
  "Name": "Event grid read only role",
  "Id": "7C0B6B59-A278-4B62-BA19-411B70753856",
  "IsCustom": true,
  "Description": "Event grid read only role",
  "Actions": [
    "Microsoft.EventGrid/*/read"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<Subscription Id>"
  ]
}
```

**EventGridNoDeleteListKeysRole.json**: разрешен ограниченный набор действий по публикации и запрещены действия удаления.

```json
{
  "Name": "Event grid No Delete Listkeys role",
  "Id": "B9170838-5F9D-4103-A1DE-60496F7C9174",
  "IsCustom": true,
  "Description": "Event grid No Delete Listkeys role",
  "Actions": [
    "Microsoft.EventGrid/*/write",
    "Microsoft.EventGrid/eventSubscriptions/getFullUrl/action"
    "Microsoft.EventGrid/topics/listkeys/action",
    "Microsoft.EventGrid/topics/regenerateKey/action"
  ],
  "NotActions": [
    "Microsoft.EventGrid/*/delete"
  ],
  "AssignableScopes": [
    "/subscriptions/<Subscription id>"
  ]
}
```

**EventGridContributorRole.json**: разрешено выполнять все действия сетки событий.

```json
{
  "Name": "Event grid contributor role",
  "Id": "4BA6FB33-2955-491B-A74F-53C9126C9514",
  "IsCustom": true,
  "Description": "Event grid contributor role",
  "Actions": [
    "Microsoft.EventGrid/*/write",
    "Microsoft.EventGrid/*/delete",
    "Microsoft.EventGrid/topics/listkeys/action",
    "Microsoft.EventGrid/topics/regenerateKey/action",
    "Microsoft.EventGrid/eventSubscriptions/getFullUrl/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/<Subscription id>"
  ]
}
```

Пользовательские роли можно создавать с помощью [PowerShell](../role-based-access-control/custom-roles-powershell.md), [Azure CLI](../role-based-access-control/custom-roles-cli.md) и [REST](../role-based-access-control/custom-roles-rest.md).



### <a name="encryption-at-rest"></a>Шифрование при хранении

События и данные, записываемые на диск службой "Сетка событий", шифруются при хранении с помощью ключа, которым управляет корпорация Майкрософт. Кроме того, максимальный период, в течение которого хранятся события или данные, составляет 24 часа согласно [политике повтора службы "Сетка событий"](delivery-and-retry.md). Служба "Сетка событий" автоматически удалит все события или данные через 24 часа или по прошествии срока жизни события в зависимости того, что наступит раньше.

## <a name="permissions-for-event-subscriptions"></a>Разрешения для подписок на события
При использовании обработчика событий, который не является веб-перехватчиком (например, концентратор событий или хранилище очередей), у пользователя возникает необходимость в доступе на запись к данному ресурсу. Данная проверка разрешений предотвращает попытки неавторизованных пользователей отправить события в ресурс.

Чтобы использовать ресурс, который является источником события, необходимо иметь разрешение **Microsoft.EventGrid/EventSubscriptions/Write**. Это разрешение необходимо, так как вы записываете новую подписку в области действия ресурса. Требуемый ресурс зависит от того, на какой раздел оформляется подписка: системный или пользовательский. В этом разделе описываются оба типа подписки.

### <a name="system-topics-azure-service-publishers"></a>Системные разделы (издатели служб Azure)
Для системных разделов, если вы не являетесь владельцем или участником исходного ресурса, вам нужно разрешение на запись новой подписки на события в области действия ресурса, публикующего событие. Ресурс имеет следующий формат: `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{resource-provider}/{resource-type}/{resource-name}`

Например, чтобы подписаться на событие в учетной записи хранения с именем **myacct**, требуется разрешение Microsoft.EventGrid/EventSubscriptions/Write для следующего ресурса: `/subscriptions/####/resourceGroups/testrg/providers/Microsoft.Storage/storageAccounts/myacct`

### <a name="custom-topics"></a>Пользовательские разделы
Для пользовательских разделов необходимо разрешение на запись новой подписки на события в области действия для сетки событий. Ресурс имеет следующий формат: `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.EventGrid/topics/{topic-name}`

Например, чтобы подписаться на пользовательский раздел с именем **mytopic**, требуется разрешение Microsoft.EventGrid/EventSubscriptions/Write для следующего ресурса: `/subscriptions/####/resourceGroups/testrg/providers/Microsoft.EventGrid/topics/mytopic`



## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о сетке событий см. в статье [Сведения о сетке событий](overview.md)
