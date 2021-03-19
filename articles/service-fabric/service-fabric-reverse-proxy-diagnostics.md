---
title: Диагностика обратных прокси-серверов Azure Service Fabric
description: Узнайте, как отслеживать и диагностировать обработку запросов на обратных прокси-сервере для приложения Azure Service Fabric.
author: kavyako
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: kavyako
ms.openlocfilehash: bbc1fe5a76ecb5720bc49e0a082d5e9151b403d8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75645469"
---
# <a name="monitor-and-diagnose-request-processing-at-the-reverse-proxy"></a>Мониторинг и диагностика обработки запросов на обратном прокси-сервере

Начиная с выпуска Service Fabric 5.7 для сбора доступны события обратного прокси-сервера. Эти события доступны в двух каналах. Один предоставляет только события ошибок, связанные со сбоями обработки запросов на обратном прокси-сервере, а второй содержит подробные события с записями для успешных и неудачных запросов.

Ознакомьтесь с разделом [Сбор событий обратного прокси-сервера](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations), чтобы включить сбор событий из этих каналов в локальном кластере и кластере Azure Service Fabric.

## <a name="troubleshoot-using-diagnostics-logs"></a>Устранение неполадок с помощью журналов диагностики
Ниже приведены примеры того, как интерпретировать распространенное содержимое журналов сбоев, с которым вы можете столкнуться.

1. Обратный прокси-сервер возвращает код состояния ответа 504 (превышено время ожидания).

    Одной из причин может быть то, что службе не удается ответить в течение времени ожидания запроса.
   Для первого события ниже в журнал записаны сведения о запросе, полученном на обратном прокси-сервере. 
   Второе событие указывает на сбой запроса во время переадресации к службе ("internal error = ERROR_WINHTTP_TIMEOUT"). 

    Ниже приведены полезные данные.

   * **traceId**: этот идентификатор GUID можно использовать, чтобы сопоставить все события, соответствующие одному запросу. В приведенных ниже двух событиях traceId = **2f87b722-e254-4ac2-a802-fd315c1a0271**, то есть они относятся к одному и тому же запросу.
   * **requestUrl**: URL-адрес (URL-адрес обратного прокси-сервера), на который был отправлен запрос.
   * **verb**: HTTP-команда.
   * **remoteAddress**: адрес клиента, отправившего запрос.
   * **resolvedServiceUrl**: URL-адрес конечной точки службы, на которую был разрешен входящий запрос. 
   * **errorDetails**: дополнительные сведения об ошибке.

     ```
     {
     "Timestamp": "2017-07-20T15:57:59.9871163-07:00",
     "ProviderName": "Microsoft-ServiceFabric",
     "Id": 51477,
     "Message": "2f87b722-e254-4ac2-a802-fd315c1a0271 Request url = https://localhost:19081/LocationApp/LocationFEService?zipcode=98052, verb = GET, remote (client) address = ::1, resolved service url = Https://localhost:8491/LocationApp/?zipcode=98052, request processing start time =     15:58:00.074114 (745,608.196 MSec) ",
     "ProcessId": 57696,
     "Level": "Informational",
     "Keywords": "0x1000000000000021",
     "EventName": "ReverseProxy",
     "ActivityID": null,
     "RelatedActivityID": null,
     "Payload": {
      "traceId": "2f87b722-e254-4ac2-a802-fd315c1a0271",
      "requestUrl": "https://localhost:19081/LocationApp/LocationFEService?zipcode=98052",
      "verb": "GET",
      "remoteAddress": "::1",
      "resolvedServiceUrl": "Https://localhost:8491/LocationApp/?zipcode=98052",
      "requestStartTime": "2017-07-20T15:58:00.0741142-07:00"
     }
     }

     {
     "Timestamp": "2017-07-20T16:00:01.3173605-07:00",
     ...
     "Message": "2f87b722-e254-4ac2-a802-fd315c1a0271 Error while forwarding request to service: response status code = 504, description = Reverse proxy Timeout, phase = FinishSendRequest, internal error = ERROR_WINHTTP_TIMEOUT ",
     ...
     "Payload": {
      "traceId": "2f87b722-e254-4ac2-a802-fd315c1a0271",
      "statusCode": 504,
      "description": "Reverse Proxy Timeout",
      "sendRequestPhase": "FinishSendRequest",
      "errorDetails": "internal error = ERROR_WINHTTP_TIMEOUT"
     }
     }
     ```

2. Обратный прокси-сервер возвращает код состояния ответа 404 (не найдено). 
    
    Ниже приведен пример события, в котором обратный прокси-сервер возвращает код 404, так как ему не удалось найти соответствующую конечную точку службы.
    Ниже приведены интересующие нас полезные данные операции.
   * **процессрекуестфасе**: указывает этап во время обработки запроса в момент возникновения сбоя, **_трижетендпоинт_** , т. е. Это произошло при попытке получить конечную точку службы для переадресации. 
   * **errorDetails**: отображает условия поиска конечной точки. Здесь можно увидеть, что указано значение listenerName = **FrontEndListener**, в то время как список конечных точек реплик содержит только прослушиватель с именем **OldListener**.
    
     ```
     {
     ...
     "Message": "c1cca3b7-f85d-4fef-a162-88af23604343 Error while processing request, cannot forward to service: request url = https://localhost:19081/LocationApp/LocationFEService?ListenerName=FrontEndListener&zipcode=98052, verb = GET, remote (client) address = ::1, request processing start time = 16:43:02.686271 (3,448,220.353 MSec), error = FABRIC_E_ENDPOINT_NOT_FOUND, message = , phase = TryGetEndoint, SecureOnlyMode = false, gateway protocol = https, listenerName = FrontEndListener, replica endpoint = {\"Endpoints\":{\"\":\"Https:\/\/localhost:8491\/LocationApp\/\"}} ",
     "ProcessId": 57696,
     "Level": "Warning",
     "EventName": "ReverseProxy",
     "Payload": {
      "traceId": "c1cca3b7-f85d-4fef-a162-88af23604343",
      "requestUrl": "https://localhost:19081/LocationApp/LocationFEService?ListenerName=NewListener&zipcode=98052",
      ...
      "processRequestPhase": "TryGetEndoint",
      "errorDetails": "SecureOnlyMode = false, gateway protocol = https, listenerName = FrontEndListener, replica endpoint = {\"Endpoints\":{\"OldListener\":\"Https:\/\/localhost:8491\/LocationApp\/\"}}"
     }
     }
     ```
     Другой пример ситуации, в которой обратный прокси-сервер может вернуть код 404 (не найдено): параметру конфигурации ApplicationGateway\Http **SecureOnlyMode** присвоено значение true, а обратный прокси-сервер ожидает передачи данных по протоколу **HTTPS**, однако все конечные точки реплики являются небезопасными (ожидают передачи данных по протоколу HTTP).
     Обратный прокси-сервер возвращает код 404, так как ему не удалось найти конечную точку, ожидающую передачу данных по протоколу HTTPS для переадресации запросов. Анализ параметров в полезных данных события помогает сузить область поиска проблемы.
    
     ```
      "errorDetails": "SecureOnlyMode = true, gateway protocol = https, listenerName = NewListener, replica endpoint = {\"Endpoints\":{\"OldListener\":\"Http:\/\/localhost:8491\/LocationApp\/\", \"NewListener\":\"Http:\/\/localhost:8492\/LocationApp\/\"}}"
     ```

3. Запрос к обратному прокси-серверу приводит к ошибке времени ожидания. 
    Журналы событий содержат событие с подробными сведениями о полученном запросе (здесь не показано).
    Следующее событие показывает, что служба ответила кодом состояния 404, после чего обратный прокси-сервер инициировал повторное разрешение. 

    ```
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Request to service returned: status code = 404, status description = , Reresolving ",
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "statusCode": 404,
        "statusDescription": ""
      }
    }
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Re-resolved service url = Https://localhost:8491/LocationApp/?zipcode=98052 ",
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "requestUrl": "Https://localhost:8491/LocationApp/?zipcode=98052"
      }
    }
    ```
    Если включен сбор всех событий, то вы сможете просмотреть вереницу событий, показывающую все попытки разрешения и переадресации.
    Последнее событие в цикле показывает, что сбой обработки запроса произошел из-за превышения времени ожидания. Оно также содержит количество успешных попыток разрешения.
    
    > [!NOTE]
    > Рекомендуется оставить сбор данных из канала подробных событий отключенным по умолчанию и включать его для устранения неполадок по мере необходимости.

    ```
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Error while processing request: number of successful resolve attempts = 12, error = FABRIC_E_TIMEOUT, message = , phase = ResolveServicePartition,  ",
      "EventName": "ReverseProxy",
      ...
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "resolveCount": 12,
        "errorval": -2147017729,
        "errorMessage": "",
        "processRequestPhase": "ResolveServicePartition",
        "errorDetails": ""
      }
    }
    ```
    
    Если включен сбор только критических событий и событий ошибок, то в журнале вы увидите одно событие со сведениями о времени ожидания и количестве попыток разрешения. 
    
    Если служба собирается отправить код состояния 404 пользователю, в ответе нужно добавить заголовок X-ServiceFabric. После добавления заголовка в ответ обратный прокси-сервер отправляет код состояния клиенту.  

4. Случаи, когда клиент отключил запрос.

    Следующее событие записывается, когда обратный прокси-сервер переадресовывает ответ клиенту, но тот отключается.

    ```
    {
      ...
      "Message": "6e2571a3-14a8-4fc7-93bb-c202c23b50b8 Unable to send response to client: phase = SendResponseHeaders, error = -805306367, internal error = ERROR_SUCCESS ",
      "ProcessId": 57696,
      "Level": "Warning",
      ...
      "EventName": "ReverseProxy",
      "Payload": {
        "traceId": "6e2571a3-14a8-4fc7-93bb-c202c23b50b8",
        "sendResponsePhase": "SendResponseHeaders",
        "errorval": -805306367,
        "winHttpError": "ERROR_SUCCESS"
      }
    }
    ```
5. Обратный прокси-сервер возвращает код состояния 404 FABRIC_E_SERVICE_DOES_NOT_EXIST

    Ошибка FABRIC_E_SERVICE_DOES_NOT_EXIST возвращается, если в манифесте службы не указана схема универсального кода ресурса (URI) для конечной точки службы.

    ```
    <Endpoint Name="ServiceEndpointHttp" Port="80" Protocol="http" Type="Input"/>
    ```

    Чтобы устранить эту проблему, укажите схему универсального кода ресурса (URI) в манифесте.
    ```
    <Endpoint Name="ServiceEndpointHttp" UriScheme="http" Port="80" Protocol="http" Type="Input"/>
    ```

> [!NOTE]
> В настоящее время не регистрируются события, относящиеся к обработке запросов WebSocket. Эта возможность будет добавлена в следующем выпуске.

## <a name="next-steps"></a>Дальнейшие действия
* [Статистическая обработка и сбор событий с помощью Диагностики Azure для Windows](service-fabric-diagnostics-event-aggregation-wad.md) для обеспечения сбора журналов в кластерах Azure.
* Чтобы просмотреть события Service Fabric в Visual Studio, ознакомьтесь с разделом [Мониторинг и диагностика состояния служб в локальной среде разработки](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md).
* Примеры шаблонов Azure Resource Manager для настройки разных параметров проверки сертификата службы для защищенного обратного прокси-сервера см. в разделе [Configure reverse proxy to connect to secure services](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample#configure-reverse-proxy-to-connect-to-secure-services).
* Чтобы узнать больше, прочитайте раздел [Обратный прокси-сервер в Service Fabric](service-fabric-reverseproxy.md).
