---
title: Обратимое удаление управления API Azure (Предварительная версия) | Документация Майкрософт
description: Обратимое удаление позволяет восстанавливать удаленные экземпляры управления API.
ms.service: api-management
ms.topic: conceptual
author: vladvino
ms.author: apimpm
ms.date: 11/27/2020
ms.openlocfilehash: e2842f3e428abb4f0eb628dbb8e446f2714d5d89
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101652391"
---
# <a name="api-management-soft-delete-preview"></a>Обратимое удаление управления API (Предварительная версия)

С помощью функции обратимого удаления (Предварительная версия) в службе управления API можно восстановить и восстановить недавно удаленные экземпляры управления API (APIM).

> [!IMPORTANT]
> Только экземпляры управления API, удаленные с помощью `2020-06-01-preview` и более поздних версий API, будут обратимо удалены и восстановлены с помощью действий, описанных в этой статье. Экземпляры APIM, удаленные с помощью предыдущих версий API, по-прежнему будут жестко удалены. В настоящее время Azure PowerShell и Azure CLI не используют эту `2020-06-01-preview` версию, и это также приведет к поведению жесткого удаления.

## <a name="supporting-interfaces"></a>Поддержка интерфейсов

Функция обратимого удаления доступна через [REST API](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/restore).

> [!TIP]
> Советы и средства для вызова интерфейсов API-интерфейса Azure см. в [справочнике по REST API Azure](/rest/api/azure/) .

| Операция | Описание | Пространство имен управления API | Минимальная версия API |
|--|--|--|--|
| [Создать или обновить](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/createorupdate) | Создает или обновляет службу управления API.  | Служба управления API | Любой |
| [Создать или изменить](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/createorupdate) `restore` свойство с установленным на **true** | Отменяет удаление службы управления API, если она была ранее обратима удалена. Если `restore` указан параметр и для него задано значение `true` все остальные свойства, будут игнорироваться.  | Служба управления API |  2020-06-01 — предварительная версия |
| [Удалить](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/delete) | Удаляет существующую службу управления API. | Служба управления API | 2020-06-01 — предварительная версия|
| [Получить по имени](/rest/api/apimanagement/2020-06-01-preview/deletedservices/getbyname) | Получите обратимо удаленную службу управления API по имени. | Удаленные службы | 2020-06-01 — предварительная версия |
| [Список по подписке](/rest/api/apimanagement/2020-06-01-preview/deletedservices/listbysubscription) | Список всех обратимо удаленных служб, доступных для отмены удаления для данной подписки. | Удаленные службы | 2020-06-01 — предварительная версия
| [Purge](/rest/api/apimanagement/2020-06-01-preview/deletedservices/purge) | Удаляет службу управления API (без возможности отмены удаления). | Удаленные службы | 2020-06-01 — предварительная версия

## <a name="soft-delete-behavior"></a>Поведение обратимого удаления

Для создания экземпляра управления API можно использовать любую версию API, однако для `2020-06-01-preview` обратимого удаления экземпляра APIM необходимо использовать или более поздние версии (а также восстановить его).

После удаления экземпляра службы управления API служба будет находиться в удаленном состоянии, что сделает ее недоступной для любых операций APIM. В этом состоянии экземпляр APIM может быть перечислен, восстановлен или очищен (окончательно удален).

В то же время Azure запланирует удаление базовых данных, соответствующих экземпляру APIM, для выполнения после предопределенного интервала хранения (48 часа). Запись DNS, соответствующая экземпляру, также сохраняется в течение интервала хранения. Нельзя повторно использовать имя экземпляра управления API, которое было обратимо удалено, пока не будет пройден срок хранения.

Если экземпляр APIM не восстанавливается в течение 48 часов, он будет окончательно удален (невосстанавливаемый). Вы также можете [очистить](#purge-a-soft-deleted-apim-instance) (окончательно удалить) экземпляр APIM, форгоинг срок хранения обратимого удаления.

## <a name="list-deleted-apim-instances"></a>Список удаленных экземпляров APIM

Вы можете проверить, доступен ли Обратимо удаленный экземпляр APIM для восстановления (отмены удаления), используя операции по [подписке](/rest/api/apimanagement/2020-06-01-preview/deletedservices/listbysubscription) , [получаемые по имени](/rest/api/apimanagement/2020-06-01-preview/deletedservices/getbyname) или списку.

### <a name="get-a-soft-deleted-instance-by-name"></a>Получить Обратимо удаленный экземпляр по имени

Используйте операцию [Get по имени](/rest/api/apimanagement/2020-06-01-preview/deletedservices/getbyname) для управления API, подставив `{subscriptionId}` , `{location}` и `{serviceName}` вместе с подпиской Azure, расположением ресурса и именем экземпляра службы управления API:

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.ApiManagement/locations/{location}/deletedservices/{serviceName}?api-version=2020-06-01-preview
```

Если доступно для отмены удаления, Azure возвратит запись экземпляра APIM, в которой отображаются его `deletionDate` и `scheduledPurgeDate` , например:

```json
{
    "id": "subscriptions/########-####-####-####-############/providers/Microsoft.ApiManagement/locations/southcentralus/deletedservices/apimtest",
    "name": "apimtest",
    "type": "Microsoft.ApiManagement/deletedservices",
    "location": "South Central US",
    "properties": {
        "serviceId": "/subscriptions/########-####-####-####-############/resourceGroups/apimtestgroup/providers/Microsoft.ApiManagement/service/apimtest",
        "scheduledPurgeDate": "2020-11-26T19:40:26.3596893Z",
        "deletionDate": "2020-11-24T19:40:50.1013572Z"
    }
}
```

### <a name="list-all-soft-deleted-instances-for-a-given-subscription"></a>Список всех обратимо удаленных экземпляров для данной подписки

Используйте список управления API [с помощью операции подписки](/rest/api/apimanagement/2020-06-01-preview/deletedservices/listbysubscription) , подставив `{subscriptionId}` идентификатор подписки:

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.ApiManagement/deletedservices?api-version=2020-06-01-preview
```

В результате будет возвращен список всех обратимо удаленных служб, доступных для отмены удаления в данной подписке, в `deletionDate` которых показаны и `scheduledPurgeDate` для каждого.

## <a name="recover-a-deleted-apim-instance"></a>Восстановление удаленного экземпляра APIM

Используйте операции [создания или обновления](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/createorupdate) управления API, подставив `{subscriptionId}` , `{resourceGroup}` и `{apimServiceName}` с помощью подписки Azure, имени группы ресурсов и имени службы управления API:

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ApiManagement/service/{apimServiceName}?api-version=2020-06-01-preview
```

. . . и задайте `restore` для свойства значение `true` в тексте запроса. (Если этот флаг задан и имеет значение *true*, все остальные свойства будут игнорироваться.) Например:

```json
{
  "properties": {
    "publisherEmail": "help@contoso.com",
    "publisherName": "Contoso",
    "restore": true
  },
  "sku": {
    "name": "Developer",
    "capacity": 1
  },
  "location": "South Central US"
}
```

## <a name="purge-a-soft-deleted-apim-instance"></a>Очистка обратимо удаленного экземпляра APIM

Используйте операцию [очистки](/rest/api/apimanagement/2020-06-01-preview/deletedservices/purge) управления API, подставив `{subscriptionId}` , `{location}` и `{serviceName}` вместе с подпиской Azure, расположением ресурса и именем службы управления API:

```rest
DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.ApiManagement/locations/{location}/deletedservices/{serviceName}?api-version=2020-06-01-preview
```

Экземпляр службы управления API будет удален из Azure без возможности восстановления.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о долгосрочном резервном копировании и восстановлении управления API:

- [Реализация аварийного восстановления с помощью функций резервного копирования и восстановления службы в Azure API Management](api-management-howto-disaster-recovery-backup-restore.md)