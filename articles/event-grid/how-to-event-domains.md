---
title: Публикация событий с помощью доменов событий с помощью службы "Сетка событий Azure"
description: Сведения об управлении большими наборами разделов в Сетке событий Azure и публикация событий в этих разделах с помощью доменов событий.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: e6861e89def10eec391bf302b1ddc726b38bb98c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97109901"
---
# <a name="manage-topics-and-publish-events-using-event-domains"></a>Управление разделами и публикация событий с помощью доменов событий

В этой статье показано, как сделать следующее:

* создать домен в Сетке событий;
* подписаться на разделы службы "Сетка событий";
* получение списка ключей;
* опубликовать события в домен.

Общие сведения о доменах событий см. в статье [Общие сведения о доменах событий, используемых для управления разделами службы "Сетка событий Azure"](event-domains.md).

## <a name="create-an-event-domain"></a>Создание Домена событий

Чтобы управлять большими наборами разделов, создайте домен событий.

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

```azurecli-interactive
az eventgrid domain create \
  -g <my-resource-group> \
  --name <my-domain-name> \
  -l <location>
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
```azurepowershell-interactive
New-AzEventGridDomain `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain-name> `
  -Location <location>
```
---

После успешного создания возвращаются следующие значения:

```json
{
  "endpoint": "https://<my-domain-name>.westus2-1.eventgrid.azure.net/api/events",
  "id": "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>",
  "inputSchema": "EventGridSchema",
  "inputSchemaMapping": null,
  "location": "westus2",
  "name": "<my-domain-name>",
  "provisioningState": "Succeeded",
  "resourceGroup": "<my-resource-group>",
  "tags": null,
  "type": "Microsoft.EventGrid/domains"
}
```

Запишите значения `endpoint` и `id`, так как они необходимы для управления доменом и публикации событий.

## <a name="manage-access-to-topics"></a>Управление доступом к разделам

Управление доступом к разделам выполняется через [назначение ролей](../role-based-access-control/role-assignments-cli.md). Назначение ролей использует управление доступом на основе ролей Azure для ограничения операций с ресурсами Azure для полномочных пользователей в определенной области.

Сетка событий имеет две встроенные роли, которые можно использовать для назначения определенным пользователям доступа к различным разделам в пределах домена. Это роль `EventGrid EventSubscription Contributor (Preview)`, которая позволяет создавать и удалять подписки, и роль `EventGrid EventSubscription Reader (Preview)`, которая позволяет только перечислять подписки на события.

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)
Следующая команда интерфейса командной строки Azure ограничивает `alice@contoso.com` созданием и удалением подписок на события только в разделе `demotopic1`:

```azurecli-interactive
az role assignment create \
  --assignee alice@contoso.com \
  --role "EventGrid EventSubscription Contributor (Preview)" \
  --scope /subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
Следующая команда PowerShell ограничивает `alice@contoso.com` созданием и удалением подписок на события только в разделе `demotopic1`:

```azurepowershell-interactive
New-AzRoleAssignment `
  -SignInName alice@contoso.com `
  -RoleDefinitionName "EventGrid EventSubscription Contributor (Preview)" `
  -Scope /subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1
```
---

Дополнительные сведения об управлении доступом для операций службы "Сетка событий" см. в статье [Сетка событий: безопасность и проверка подлинности](./security-authentication.md).

## <a name="create-topics-and-subscriptions"></a>Создание разделов и подписок

Служба "Сетка событий" автоматически создает соответствующий раздел в домене и управляет им на основе вызова для создания подписки на событие для раздела домена. Отдельного шага по созданию раздела в домене нет. Аналогичным образом, когда последняя подписка на событие для раздела удаляется, сам раздел также удаляется.

Подписка на раздел в домене совпадает с подпиской на любой другой ресурс Azure. Для идентификатора исходного ресурса укажите идентификатор домена событий, возвращенный при создании домена ранее. Чтобы указать раздел, на который вы хотите подписаться, добавьте `/topics/<my-topic>` в конец идентификатора исходного ресурса. Чтобы создать подписку на события в области домена, которая принимает все события в домене, укажите идентификатор домена событий без указания каких-либо разделов.

Как правило, пользователь, которому вы предоставили доступ в предыдущем разделе, создаст подписку. Для упрощения этой статьи создайте подписку. 

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

```azurecli-interactive
az eventgrid event-subscription create \
  --name <event-subscription> \
  --source-resource-id "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1" \
  --endpoint https://contoso.azurewebsites.net/api/updates
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurepowershell-interactive
New-AzEventGridSubscription `
  -ResourceId "/subscriptions/<sub-id>/resourceGroups/<my-resource-group>/providers/Microsoft.EventGrid/domains/<my-domain-name>/topics/demotopic1" `
  -EventSubscriptionName <event-subscription> `
  -Endpoint https://contoso.azurewebsites.net/api/updates
```

---

Если вам нужна тестовая конечная точка, куда будут отправляться события, вы всегда можете развернуть [предварительно готовое веб-приложение](https://github.com/Azure-Samples/azure-event-grid-viewer), отображающее входящие события. Вы можете отправлять события на веб-сайт тестирования по адресу `https://<your-site-name>.azurewebsites.net/api/updates`.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"  alt="Button to Deploy to Aquent." /></a>

Разрешения, заданные для раздела, хранятся в Azure Active Directory и должны быть явно удалены. Удаление подписки на событие не отменяет доступ пользователей к созданию подписок на события, если у них есть доступ на запись в разделе.


## <a name="publish-events-to-an-event-grid-domain"></a>Публикация событий в Домен службы "Сетка событий"

Публикация событий в домене аналогична публикации [в пользовательском разделе](./post-to-custom-topic.md). Тем не менее вместо публикации в пользовательском разделе публикуйте все события на конечную точку домена. В данных события JSON укажите раздел, к которому вы хотите добавить события. Следующий массив событий приведет к публикации события с идентификатором `"id": "1111"` в раздел `demotopic1`, а событие с идентификатором `"id": "2222"` будет отправлено в раздел `demotopic2`:

```json
[{
  "topic": "demotopic1",
  "id": "1111",
  "eventType": "maintenanceRequested",
  "subject": "myapp/vehicles/diggers",
  "eventTime": "2018-10-30T21:03:07+00:00",
  "data": {
    "make": "Contoso",
    "model": "Small Digger"
  },
  "dataVersion": "1.0"
},
{
  "topic": "demotopic2",
  "id": "2222",
  "eventType": "maintenanceCompleted",
  "subject": "myapp/vehicles/tractors",
  "eventTime": "2018-10-30T21:04:12+00:00",
  "data": {
    "make": "Contoso",
    "model": "Big Tractor"
  },
  "dataVersion": "1.0"
}]
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)
Чтобы получить конечную точку домена с помощью Azure CLI, выполните следующую команду:

```azurecli-interactive
az eventgrid domain show \
  -g <my-resource-group> \
  -n <my-domain>
```

Чтобы получить ключи, которые будет использовать домен, выполните команду ниже:

```azurecli-interactive
az eventgrid domain key list \
  -g <my-resource-group> \
  -n <my-domain>
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)
Чтобы получить конечную точку домена с помощью PowerShell, выполните следующую команду:

```azurepowershell-interactive
Get-AzEventGridDomain `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain>
```

Чтобы получить ключи, которые будет использовать домен, выполните команду ниже:

```azurepowershell-interactive
Get-AzEventGridDomainKey `
  -ResourceGroupName <my-resource-group> `
  -Name <my-domain>
```
---

Затем используйте предпочтительный метод создания запроса HTTP POST для публикации событий в домен Сетки событий.

## <a name="search-lists-of-topics-or-subscriptions"></a>Поиск в списках разделов или подписок

Для поиска и управления большим количеством разделов или подписок API-интерфейсы сетки событий поддерживают списки и разбивку на страницы.

### <a name="using-cli"></a>Использование синтаксиса командной строки
Например, следующая команда выводит список разделов с именем `mytopic` , содержащим. 

```azurecli-interactive
az eventgrid topic list --odata-query "contains(name, 'mytopic')"
```

Дополнительные сведения об этой команде см. в разделе [`az eventgrid topic list`](/cli/azure/eventgrid/topic?#az_eventgrid_topic_list) . 


## <a name="next-steps"></a>Дальнейшие действия

* Общие понятия, касающиеся доменов событий, и сведения об их эффективности см. в [этой статье](event-domains.md).
