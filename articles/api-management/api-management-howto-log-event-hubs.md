---
title: Как регистрировать события в Центрах событий Azure при использовании службы управления API Azure | Документация Майкрософт
description: Узнайте, как регистрировать события в Центрах событий Azure при использовании службы управления API Azure. Концентраторы событий — это служба передачи данных с высокой степенью масштабируемости.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 88f6507d-7460-4eb2-bffd-76025b73f8c4
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/29/2018
ms.author: apimpm
ms.openlocfilehash: 5066169951409fa86aa75a64e8fc6d4189947f27
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92072413"
---
# <a name="how-to-log-events-to-azure-event-hubs-in-azure-api-management"></a>Как регистрировать события в Центрах событий Azure при использовании службы управления API Azure
Центры событий Azure — это высокомасштабируемая служба приема данных, которая может обрабатывать миллионы событий в секунду, позволяя вам обрабатывать и анализировать огромное количество данных, создаваемых подключенными устройствами и приложениями. Центры событий выступают в качестве "главного входа"для событий конвейера. После того как данные поступили в концентратор событий, они могут быть преобразованы и сохранены с использованием любого поставщика аналитики в режиме реального времени или адаптера пакетной обработки или хранения. Центры событий отделяют создание потока событий от потребления этих событий, чтобы потребители событий могли обращаться к событиям по собственному расписанию.

Эта статья сопровождает видеоролик [Интеграция службы управления API Azure с Центрами событий](https://azure.microsoft.com/documentation/videos/integrate-azure-api-management-with-event-hubs/). В ней рассматривается регистрация событий управления API с помощью Центров событий Azure.

## <a name="create-an-azure-event-hub"></a>Создание концентратора событий Azure

Подробные инструкции о том, как создать концентратор событий и получить строки подключения, необходимые для отправки событий в концентратор и получения событий из него, см. в статье [Создание пространства имен Центров событий и концентратора событий с помощью портала Azure](../event-hubs/event-hubs-create.md).

## <a name="create-an-api-management-logger"></a>Создание средства ведения журнала для управления API
Теперь, когда концентратор событий создан, нужно настроить [средство ведения журнала](/rest/api/apimanagement/2019-12-01/logger) в службе управления API, которое сможет регистрировать события в концентраторе событий.

Средства ведения журнала службы управления API настраиваются с помощью [REST API службы управления API](/rest/api/apimanagement/ApiManagementREST/API-Management-REST). Подробные примеры запросов см. [в разделе Создание средств ведения журнала](/rest/api/apimanagement/2019-12-01/logger/createorupdate).

## <a name="configure-log-to-eventhub-policies"></a>Настройка политик входа в eventhub

После настройки средства ведения журнала в оснастке управления API можно настроить политику log to eventhub для записи в журнал нужных событий. Политику log to eventhub можно использовать в разделе Политика входящих подключений или в разделе Политика исходящего трафика.

1. Перейдите к экземпляру службы управления API Azure.
2. Выберите вкладку API.
3. Выберите API-интерфейс, для которого требуется добавить политику. В этом примере мы добавляем политику **Echo API** в продукт **Unlimited**.
4. Выберите **Все операции**.
5. В верхней части экрана выберите вкладку "Конструктор".
6. В окне обработки входящих или исходящих запросов щелкните треугольник (рядом со значком карандаша).
7. Выберите редактор кода. Дополнительные сведения см. в статье [Настройка или изменение политик службы управления API Azure](set-edit-policies.md).
8. Наведите указатель мыши на раздел политики `inbound` или `outbound`.
9. В окне справа выберите **Расширенные политики**  >  **Журнал для EventHub**. В результате будет вставлен шаблон инструкции политики `log-to-eventhub`.

```xml
<log-to-eventhub logger-id="logger-id">
    @{
        return new JObject(
            new JProperty("EventTime", DateTime.UtcNow.ToString()),
            new JProperty("ServiceName", context.Deployment.ServiceName),
            new JProperty("RequestId", context.RequestId),
            new JProperty("RequestIp", context.Request.IpAddress),
            new JProperty("OperationName", context.Operation.Name)
        ).ToString();
    }
</log-to-eventhub>
```
Замените `logger-id` значением, которое вы использовали для `{loggerId}` URL-адреса запроса, чтобы создать средство ведения журнала на предыдущем шаге.

Вы можете использовать любое выражение, которое возвращает строку в качестве значения для элемента `log-to-eventhub` . В этом примере в журнал заносится строка в формате JSON, содержащая дату и время, имя службы, идентификатор запроса, IP-адрес запроса и имя операции.

Нажмите кнопку **Сохранить** , чтобы сохранить конфигурацию обновленной политики. Сразу же после сохранения политика становится активной, а события регистрируются в указанном концентраторе событий.

> [!NOTE]
> Максимальный поддерживаемый размер сообщения, который может быть отправлен в концентратор событий из этой политики управления API, составляет 200 килобайт (КБ). Если сообщение, отправляемое в концентратор событий, превышает 200 КБ, оно будет автоматически обрезано, а усеченное сообщение будет передано в концентраторы событий.

## <a name="preview-the-log-in-event-hubs-by-using-azure-stream-analytics"></a>Предварительный просмотр журнала в концентраторах событий с помощью Azure Stream Analytics

Вы можете просмотреть журнал в концентраторах событий с помощью [запросов Azure Stream Analytics](../event-hubs/process-data-azure-stream-analytics.md). 

1. В портал Azure перейдите к концентратору событий, в который средство ведения журнала отправляет события. 
2. В разделе " **компоненты**" перейдите на вкладку **обработка данных** .
3. На карточке **Включение аналитики в реальном времени из событий** выберите **проводник**.
4. Вы можете просмотреть журнал на вкладке " **Просмотр ввода** ". Если отображаемые данные не являются текущими, выберите **Обновить** , чтобы просмотреть последние события.

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительная информация о Центрах событий Azure
  * [Приступая к работе с Центрами событий Azure](../event-hubs/event-hubs-c-getstarted-send.md)
  * [Прием сообщений через EventProcessorHost](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md)
  * [Руководство по программированию Центров событий](../event-hubs/event-hubs-programming-guide.md)
* Дополнительные сведения об интеграции службы управления API и Центров событий
  * [Справочник по сущности "Средство ведения журнала"](/rest/api/apimanagement/2019-12-01/logger)
  * [Справочник по политике регистрации в концентраторе событий](./api-management-advanced-policies.md#log-to-eventhub)
  * [Мониторинг API-интерфейсов с помощью Управления API Azure, Центров событий и Moesif](api-management-log-to-eventhub-sample.md)  
* Узнайте больше об интеграции с [Azure Application Insights](api-management-howto-app-insights.md).

[publisher-portal]: ./media/api-management-howto-log-event-hubs/publisher-portal.png
[create-event-hub]: ./media/api-management-howto-log-event-hubs/create-event-hub.png
[event-hub-connection-string]: ./media/api-management-howto-log-event-hubs/event-hub-connection-string.png
[event-hub-dashboard]: ./media/api-management-howto-log-event-hubs/event-hub-dashboard.png
[receiving-policy]: ./media/api-management-howto-log-event-hubs/receiving-policy.png
[sending-policy]: ./media/api-management-howto-log-event-hubs/sending-policy.png
[event-hub-policy]: ./media/api-management-howto-log-event-hubs/event-hub-policy.png
[add-policy]: ./media/api-management-howto-log-event-hubs/add-policy.png