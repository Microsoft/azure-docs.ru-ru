---
title: Управляемые приложения с уведомлениями
description: Настройка управляемых приложений с помощью конечных точек веб-перехватчика для получения уведомлений о создании, обновлениях, удалениях и ошибках в экземплярах управляемых приложений.
ms.topic: conceptual
ms.author: ilahat
author: ilahat
ms.date: 11/01/2019
ms.openlocfilehash: 2a2e9d429d494c35c49a5b0a3e10b291fd8f24a6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100633943"
---
# <a name="azure-managed-applications-with-notifications"></a>Управляемые приложения Azure с уведомлениями

Уведомления управляемых приложений Azure позволяют издателям автоматизировать действия на основе событий жизненного цикла экземпляров управляемых приложений. Издатели могут указывать пользовательские конечные точки веб-перехватчика уведомлений для получения уведомлений о событиях новых и существующих экземпляров управляемых приложений. Издатели могут настраивать пользовательские рабочие процессы во время подготовки, обновления и удаления приложений.

## <a name="getting-started"></a>Начало работы
Чтобы начать получать управляемые приложения, запустите общедоступную конечную точку HTTPS и укажите ее при публикации определения приложения каталога услуг или предложения Azure Marketplace.

Ниже приведены рекомендуемые шаги для быстрого начала работы.
1. Запустите общедоступную конечную точку HTTPS, которая регистрирует входящие запросы POST и возвращает `200 OK` .
2. Добавьте конечную точку в определение приложения каталога услуг или в предложение Azure Marketplace, как описано далее в этой статье.
3. Создайте экземпляр управляемого приложения, который ссылается на определение приложения или предложение Azure Marketplace.
4. Проверьте, что уведомления получены.
5. Включите авторизацию, как описано в разделе **Проверка подлинности конечной точки** этой статьи.
6. Следуйте инструкциям в разделе **Схема уведомления** этой статьи, чтобы проанализировать запросы уведомлений и реализовать бизнес-логику на основе уведомления.

## <a name="add-service-catalog-application-definition-notifications"></a>Добавление уведомлений об определении приложения из каталога услуг
#### <a name="azure-portal"></a>Портал Azure
Чтобы приступить к работе, см. статью [Публикация приложения каталога услуг с помощью портал Azure](./publish-portal.md).

![Уведомления об определении приложения каталога услуг в портал Azure](./media/publish-notifications/service-catalog-notifications.png)

#### <a name="rest-api"></a>REST API

> [!NOTE]
> В настоящее время в свойствах определения приложения можно указать только одну конечную точку `notificationEndpoints` .

``` JSON
    {
      "properties": {
        "isEnabled": true,
        "lockLevel": "ReadOnly",
        "displayName": "Sample Application Definition",
        "description": "Notification-enabled application definition.",
        "notificationPolicy": {
            "notificationEndpoints": [
                {
                    "uri": "https://isv.azurewebsites.net:1214?sig=unique_token"
                }
            ]
        },
        "authorizations": [
          {
            "principalId": "d6b7fbd3-4d99-43fe-8a7a-f13aef11dc18",
            "roleDefinitionId": "8e3af657-a8ff-443c-a75c-2fe8c4bcb635"
          },
        ...

```
## <a name="add-azure-marketplace-managed-application-notifications"></a>Добавление уведомлений управляемого приложения Azure Marketplace
Дополнительные сведения см. в разделе [Создание предложения приложения Azure](../../marketplace/create-new-azure-apps-offer.md).

![Уведомления об управляемых приложениях Azure Marketplace в портал Azure](./media/publish-notifications/marketplace-notifications.png)
## <a name="event-triggers"></a>Триггеры событий
В следующей таблице описаны все возможные сочетания EventType и ProvisioningState и их триггеров.

EventType | ProvisioningState | Триггер для уведомления
---|---|---
PUT | Принято | Управляемая группа ресурсов создана и спроектирована успешно после размещения приложения (до начала развертывания в управляемой группе ресурсов).
PUT | Выполнено | Полная подготовка управляемого приложения прошла удачно после размещения.
PUT | Сбой | Сбой помещения подготовки экземпляра приложения в любой момент.
PATCH | Выполнено | После успешного исправления на экземпляре управляемого приложения для обновления тегов, политики JIT-доступа или управляемого удостоверения.
DELETE | Удаление | Как только пользователь инициирует удаление экземпляра управляемого приложения.
DELETE | Удаленная | После полного и успешного удаления управляемого приложения.
DELETE | Сбой | После любой ошибки в процессе отмены инициализации, блокирующей удаление.
## <a name="notification-schema"></a>Схема уведомления
При перезапуске конечной точки веб-перехватчика для обработки уведомлений необходимо проанализировать полезные данные, чтобы получить важные свойства, которые затем будут действовать при уведомлении. Служба "Каталог услуг" и уведомления управляемого приложения Azure Marketplace предоставляют множество одинаковых свойств. В таблице ниже приведены два небольших различия.

#### <a name="service-catalog-application-notification-schema"></a>Схема уведомления приложения каталога услуг
Ниже приведен пример уведомления каталога услуг после успешной подготовки экземпляра управляемого приложения.
``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Succeeded",
    "applicationDefinitionId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applicationDefinitions/<appDefName>"    
}

```

Если подготовка завершается неудачно, в указанную конечную точку будет отправлено уведомление с подробными сведениями об ошибке.

``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Failed",
    "applicationDefinitionId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applicationDefinitions/<appDefName>",
    "error": {
        "code": "ErrorCode",
        "message": "error message",
        "details": [
            {
                "code": "DetailedErrorCode",
                "message": "error message"
            }
        ]
    }
}

```

#### <a name="azure-marketplace-application-notification-schema"></a>Схема уведомлений приложений Azure Marketplace

Ниже приведен пример уведомления каталога услуг после успешной подготовки экземпляра управляемого приложения.
``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Succeeded",
    "billingDetails": {
        "resourceUsageId":"<resourceUsageId>"
    },
    "plan": {
        "publisher": "publisherId",
        "product": "offer",
        "name": "skuName",
        "version": "1.0.1"
    }
}

```

Если подготовка завершается неудачно, в указанную конечную точку будет отправлено уведомление с подробными сведениями об ошибке.

``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Failed",
    "billingDetails": {
        "resourceUsageId":"<resourceUsageId>"
    },
    "plan": {
        "publisher": "publisherId",
        "product": "offer",
        "name": "skuName",
        "version": "1.0.1"
    },
    "error": {
        "code": "ErrorCode",
        "message": "error message",
        "details": [
            {
                "code": "DetailedErrorCode",
                "message": "error message"
            }
        ]
    }
}

```

Параметр | Описание
---|---
eventType | Тип события, вызвавшего уведомление. (Например, размещение, исправление, удаление.)
applicationId | Полный идентификатор ресурса управляемого приложения, для которого было активировано уведомление.
eventTime | Метка времени события, вызвавшего уведомление. (Дата и время в формате UTC 8601.)
provisioningState | Состояние подготовки экземпляра управляемого приложения. (Например, успешно, сбой, удаление, удаление.)
Ошибка | *Указывается только при сбое provisioningState*. Содержит код ошибки, сообщение и сведения о неполадке, которая привела к сбою.
аппликатиондефинитионид | *Задается только для управляемых приложений каталога услуг*. Представляет полный идентификатор ресурса определения приложения, для которого была подготовлена инициализация экземпляра управляемого приложения.
План | *Задается только для управляемых приложений Azure Marketplace*. Представляет издателя, предложение, номер SKU и версию управляемого экземпляра приложения.
биллингдетаилс | *Задается только для управляемых приложений Azure Marketplace.* Сведения о выставлении счетов для экземпляра управляемого приложения. Содержит Ресаурцеусажеид, которые можно использовать для запроса сведений об использовании в Azure Marketplace.

## <a name="endpoint-authentication"></a>Проверка подлинности конечной точки
Чтобы защитить конечную точку веб-перехватчика и убедиться в подлинности уведомления, сделайте следующее:
1. Укажите параметр запроса поверх URI веб-перехватчика: HTTPS \: //йоур-ендпоинт.ком? SIG = GUID. При каждом уведомлении убедитесь, что параметр запроса `sig` имеет ожидаемое значение `Guid` .
2. Выдайте GET в экземпляре управляемого приложения с помощью applicationId. Проверьте, что provisioningState соответствует provisioningState уведомления, чтобы обеспечить согласованность.

## <a name="notification-retries"></a>Повторы уведомлений

Служба уведомлений управляемого приложения принимает на `200 OK` уведомление ответ от конечной точки веб-перехватчика. Служба уведомлений повторит попытку, если конечная точка веб-перехватчика возвращает код ошибки HTTP, который больше или равен 500, если он возвращает код ошибки 429 или если конечная точка временно недоступна. Если конечная точка веб-перехватчика не станет доступной в течение 10 часов, сообщение уведомления будет удалено, а повторные попытки будут прекращены.
