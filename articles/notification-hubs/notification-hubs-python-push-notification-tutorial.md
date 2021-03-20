---
title: Использование концентраторов уведомлений с Python
description: Узнайте, как использовать центры уведомлений Azure из приложения Python.
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 5640dd4a-a91e-4aa0-a833-93615bde49b4
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: python
ms.devlang: php
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.custom: devx-track-python
ms.openlocfilehash: f81614005a1b0374dc249187c4ff3c920b7c97e9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92424839"
---
# <a name="how-to-use-notification-hubs-from-python"></a>Использование концентраторов уведомлений с Python

[!INCLUDE [notification-hubs-backend-how-to-selector](../../includes/notification-hubs-backend-how-to-selector.md)]

Можно обращаться ко всем функциям Центров уведомлений из серверной части Java, PHP, Python или Ruby, используя интерфейс REST Центра уведомлений в соответствии с описанием в статье MSDN [Интерфейсы API REST концентраторов уведомлений](/previous-versions/azure/reference/dn223264(v=azure.100)).

> [!NOTE]
> Это образец эталонной реализации для отправки уведомлений в Python, который не является официально поддерживаемым пакетом SDK концентраторов уведомлений Python. Этот образец был создан с помощью Python 3.4.

В этой статье показано, как:

- Построение клиента REST для функций концентраторов уведомлений на языке Python.
- Отправка уведомлений с помощью интерфейса API REST концентратора уведомления Python.
- Получение дампа запросов и ответов HTTP REST для отладки и образовательных целей.

Можно воспользоваться указаниями в разделе [Приступая к работе с центрами уведомлений](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md) для выбранной мобильной платформы, которая реализует серверную часть на языке Python.

> [!NOTE]
> Область действия образца ограничена отправкой уведомлений. Образец не выполняет функции управления регистрацией.

## <a name="client-interface"></a>Интерфейс клиента

Интерфейс основного клиента предоставляет те же методы, что и [пакет SDK центров уведомлений для .NET](https://msdn.microsoft.com/library/jj933431.aspx). Этот интерфейс дает возможность напрямую переводить все руководства и примеры, доступные на этом сайте в настоящий момент и пополняемые интернет-сообществом.

Полный код доступен в [примере оболочки REST Python].

Например, чтобы создать клиента, необходимо выполнить следующие действия.

```python
isDebug = True
hub = NotificationHub("myConnectionString", "myNotificationHubName", isDebug)
```

Чтобы отправить всплывающее уведомление Windows:

```python
wns_payload = """<toast><visual><binding template=\"ToastText01\"><text id=\"1\">Hello world!</text></binding></visual></toast>"""
hub.send_windows_notification(wns_payload)
```

## <a name="implementation"></a>Реализация

Если вы еще этого не делали, выполните шаги, описанные в [Учебника по началу работы], до последнего раздела, в котором вам нужно реализовать серверную часть.

Подробные сведения о реализации полноценной оболочки REST можно найти в [MSDN](/previous-versions/azure/reference/dn530746(v=azure.100)). В этом разделе описана реализация основных действий на языке Python, необходимых для доступа к конечным точкам REST службы "Центры уведомлений" и отправки уведомлений.

1. Проанализируйте строку подключения
2. Создайте маркер проверки подлинности
3. Отправка уведомления с помощью HTTP REST API

### <a name="parse-the-connection-string"></a>Проанализируйте строку подключения

Ниже показан основной класс, реализующий клиента, конструктор которого выполняет анализ строки подключения:

```python
class NotificationHub:
    API_VERSION = "?api-version=2013-10"
    DEBUG_SEND = "&test"

    def __init__(self, connection_string=None, hub_name=None, debug=0):
        self.HubName = hub_name
        self.Debug = debug

        # Parse connection string
        parts = connection_string.split(';')
        if len(parts) != 3:
            raise Exception("Invalid ConnectionString.")

        for part in parts:
            if part.startswith('Endpoint'):
                self.Endpoint = 'https' + part[11:]
            if part.startswith('SharedAccessKeyName'):
                self.SasKeyName = part[20:]
            if part.startswith('SharedAccessKey'):
                self.SasKeyValue = part[16:]
```

### <a name="create-security-token"></a>Создание маркера безопасности

Подробные сведения о создании токенов безопасности можно найти [здесь](/previous-versions/azure/reference/dn495627(v=azure.100)).
Для создания маркера на основе универсального кода ресурса (URI) текущего запроса и учетных данных, извлеченных из строки подключения, добавьте следующие методы в класс `NotificationHub`.

```python
@staticmethod
def get_expiry():
    # By default returns an expiration of 5 minutes (=300 seconds) from now
    return int(round(time.time() + 300))


@staticmethod
def encode_base64(data):
    return base64.b64encode(data)


def sign_string(self, to_sign):
    key = self.SasKeyValue.encode('utf-8')
    to_sign = to_sign.encode('utf-8')
    signed_hmac_sha256 = hmac.HMAC(key, to_sign, hashlib.sha256)
    digest = signed_hmac_sha256.digest()
    encoded_digest = self.encode_base64(digest)
    return encoded_digest


def generate_sas_token(self):
    target_uri = self.Endpoint + self.HubName
    my_uri = urllib.parse.quote(target_uri, '').lower()
    expiry = str(self.get_expiry())
    to_sign = my_uri + '\n' + expiry
    signature = urllib.parse.quote(self.sign_string(to_sign))
    auth_format = 'SharedAccessSignature sig={0}&se={1}&skn={2}&sr={3}'
    sas_token = auth_format.format(signature, expiry, self.SasKeyName, my_uri)
    return sas_token
```

### <a name="send-a-notification-using-http-rest-api"></a>Отправка уведомления с помощью HTTP REST API

Сначала следует определить класс, представляющий уведомление.

```python
class Notification:
    def __init__(self, notification_format=None, payload=None, debug=0):
        valid_formats = ['template', 'apple', 'fcm',
                         'windows', 'windowsphone', "adm", "baidu"]
        if not any(x in notification_format for x in valid_formats):
            raise Exception(
                "Invalid Notification format. " +
                "Must be one of the following - 'template', 'apple', 'fcm', 'windows', 'windowsphone', 'adm', 'baidu'")

        self.format = notification_format
        self.payload = payload

        # array with keynames for headers
        # Note: Some headers are mandatory: Windows: X-WNS-Type, WindowsPhone: X-NotificationType
        # Note: For Apple you can set Expiry with header: ServiceBusNotification-ApnsExpiry
        # in W3C DTF, YYYY-MM-DDThh:mmTZD (for example, 1997-07-16T19:20+01:00).
        self.headers = None
```

Этот класс представляет собой контейнер для собственного текста уведомления либо набор свойств шаблонного уведомления, а также набор заголовков, содержащих свойства формата (собственная платформа или шаблон) и специальные свойства платформы (например, свойство срока действия Apple и заголовки WNS).

Обратитесь к [документации по REST API для службы "Центры уведомлений"](/previous-versions/azure/reference/dn495827(v=azure.100)) и изучите форматы специализированных платформ уведомлений, чтобы узнать обо всех доступных параметрах.

Имея в распоряжении этот класс, запишите методы отправки уведомлений внутри класса `NotificationHub`.

```python
def make_http_request(self, url, payload, headers):
    parsed_url = urllib.parse.urlparse(url)
    connection = http.client.HTTPSConnection(
        parsed_url.hostname, parsed_url.port)

    if self.Debug > 0:
        connection.set_debuglevel(self.Debug)
        # adding this querystring parameter gets detailed information about the PNS send notification outcome
        url += self.DEBUG_SEND
        print("--- REQUEST ---")
        print("URI: " + url)
        print("Headers: " + json.dumps(headers, sort_keys=True,
                                       indent=4, separators=(' ', ': ')))
        print("--- END REQUEST ---\n")

    connection.request('POST', url, payload, headers)
    response = connection.getresponse()

    if self.Debug > 0:
        # print out detailed response information for debugging purpose
        print("\n\n--- RESPONSE ---")
        print(str(response.status) + " " + response.reason)
        print(response.msg)
        print(response.read())
        print("--- END RESPONSE ---")

    elif response.status != 201:
        # Successful outcome of send message is HTTP 201 - Created
        raise Exception(
            "Error sending notification. Received HTTP code " + str(response.status) + " " + response.reason)

    connection.close()


def send_notification(self, notification, tag_or_tag_expression=None):
    url = self.Endpoint + self.HubName + '/messages' + self.API_VERSION

    json_platforms = ['template', 'apple', 'fcm', 'adm', 'baidu']

    if any(x in notification.format for x in json_platforms):
        content_type = "application/json"
        payload_to_send = json.dumps(notification.payload)
    else:
        content_type = "application/xml"
        payload_to_send = notification.payload

    headers = {
        'Content-type': content_type,
        'Authorization': self.generate_sas_token(),
        'ServiceBusNotification-Format': notification.format
    }

    if isinstance(tag_or_tag_expression, set):
        tag_list = ' || '.join(tag_or_tag_expression)
    else:
        tag_list = tag_or_tag_expression

    # add the tags/tag expressions to the headers collection
    if tag_list != "":
        headers.update({'ServiceBusNotification-Tags': tag_list})

    # add any custom headers to the headers collection that the user may have added
    if notification.headers is not None:
        headers.update(notification.headers)

    self.make_http_request(url, payload_to_send, headers)


def send_apple_notification(self, payload, tags=""):
    nh = Notification("apple", payload)
    self.send_notification(nh, tags)


def send_fcm_notification(self, payload, tags=""):
    nh = Notification("fcm", payload)
    self.send_notification(nh, tags)


def send_adm_notification(self, payload, tags=""):
    nh = Notification("adm", payload)
    self.send_notification(nh, tags)


def send_baidu_notification(self, payload, tags=""):
    nh = Notification("baidu", payload)
    self.send_notification(nh, tags)


def send_mpns_notification(self, payload, tags=""):
    nh = Notification("windowsphone", payload)

    if "<wp:Toast>" in payload:
        nh.headers = {'X-WindowsPhone-Target': 'toast',
                      'X-NotificationClass': '2'}
    elif "<wp:Tile>" in payload:
        nh.headers = {'X-WindowsPhone-Target': 'tile',
                      'X-NotificationClass': '1'}

    self.send_notification(nh, tags)


def send_windows_notification(self, payload, tags=""):
    nh = Notification("windows", payload)

    if "<toast>" in payload:
        nh.headers = {'X-WNS-Type': 'wns/toast'}
    elif "<tile>" in payload:
        nh.headers = {'X-WNS-Type': 'wns/tile'}
    elif "<badge>" in payload:
        nh.headers = {'X-WNS-Type': 'wns/badge'}

    self.send_notification(nh, tags)


def send_template_notification(self, properties, tags=""):
    nh = Notification("template", properties)
    self.send_notification(nh, tags)
```

Указанные выше методы отправляют запрос HTTP POST к конечной точке "/messages" центра уведомлений с соответствующим текстом и заголовками для отправки уведомления.

### <a name="using-debug-property-to-enable-detailed-logging"></a>Включение подробного ведения журнала с помощью свойства отладки

Если включить свойство отладки при инициализации центра уведомлений, в журнал будут записаны подробные сведения о дампе запроса и ответа HTTP, а также детальные выходные данные об отправке уведомлений.
[Свойство TestSend службы "Центры уведомлений"](/previous-versions/azure/reference/dn495827(v=azure.100)) возвращает подробные сведения о результатах отправки уведомлений.
Чтобы его использовать, выполните инициализацию с помощью приведенного кода.

```python
hub = NotificationHub("myConnectionString", "myNotificationHubName", isDebug)
```

В результате к URL-адресу типа HTTP запроса на отправку для концентратора уведомлений прикрепляется строка запроса "test".

## <a name="complete-the-tutorial"></a><a name="complete-tutorial"></a>Завершение работы с учебником

Теперь вы можете завершить работу с учебником по началу работы, отправив уведомление из серверной части Python.

Инициализируйте клиент концентратора уведомлений (замените строку подключения и имя концентратора в соответствии с инструкциями в [Учебника по началу работы]):

```python
hub = NotificationHub("myConnectionString", "myNotificationHubName")
```

Затем добавьте код отправки, определяемый целевой мобильной платформой. В этом примере добавлены методы более высокого уровня, чтобы включить отправку уведомлений для определенной платформы, например send_windows_notification (для Windows), send_apple_notification (для Apple) и т. д.

### <a name="windows-store-and-windows-phone-81-non-silverlight"></a>Магазин Windows и Windows Phone 8.1 (без Silverlight)

```python
wns_payload = """<toast><visual><binding template=\"ToastText01\"><text id=\"1\">Test</text></binding></visual></toast>"""
hub.send_windows_notification(wns_payload)
```

### <a name="windows-phone-80-and-81-silverlight"></a>Windows Phone 8.0 и 8.1 Silverlight

```python
hub.send_mpns_notification(toast)
```

### <a name="ios"></a>iOS

```python
alert_payload = {
    'data':
        {
            'msg': 'Hello!'
        }
}
hub.send_apple_notification(alert_payload)
```

### <a name="android"></a>Android

```python
fcm_payload = {
    'data':
        {
            'msg': 'Hello!'
        }
}
hub.send_fcm_notification(fcm_payload)
```

### <a name="kindle-fire"></a>Kindle Fire

```python
adm_payload = {
    'data':
        {
            'msg': 'Hello!'
        }
}
hub.send_adm_notification(adm_payload)
```

### <a name="baidu"></a>Baidu

```python
baidu_payload = {
    'data':
        {
            'msg': 'Hello!'
        }
}
hub.send_baidu_notification(baidu_payload)
```

После выполнения кода Python на целевом устройстве должно отобразиться уведомление.

## <a name="examples"></a>Примеры

### <a name="enabling-the-debug-property"></a>Включение свойства `debug`

Если включить флаг отладки во время инициализации центра уведомлений (NotificationHub), вы увидите подробный дамп HTTP-запроса и ответа HTTP наряду с данными NotificationOutcome, которые указывают, какие именно заголовки HTTP переданы в запросе и какой ответ HTTP был получен от центра уведомлений.

![Снимок экрана консоли с подробными сведениями о дампе запросов и ответов H T T P и сообщениях о результатах уведомлений, отмеченных красным цветом.][1]

Вы увидите подробный результат центра уведомлений, например сведения о том,

- когда сообщение успешно отправлено в службу push-уведомлений.
    ```xml
    <Outcome>The Notification was successfully sent to the Push Notification System</Outcome>
    ```
- Если какое-либо push-уведомление не достигло адресата, отобразятся следующие выходные данные (указывающие, что, вероятнее всего, регистрации для доставки уведомлений не были найдены из-за несоответствующих тегов).
    ```xml
    '<NotificationOutcome xmlns="http://schemas.microsoft.com/netservices/2010/10/servicebus/connect" xmlns:i="https://www.w3.org/2001/XMLSchema-instance"><Success>0</Success><Failure>0</Failure><Results i:nil="true"/></NotificationOutcome>'
    ```

### <a name="broadcast-toast-notification-to-windows"></a>Рассылка всплывающих уведомлений для Windows

Обратите внимание на заголовки, которые отправляются при рассылке всплывающих уведомлений клиенту Windows.

```python
hub.send_windows_notification(wns_payload)
```

![Снимок экрана консоли с подробными сведениями о запросе H T T и форматом уведомлений служебной шины и значениями X W N S, выделенными красным цветом.][2]

### <a name="send-notification-specifying-a-tag-or-tag-expression"></a>Отправка уведомления с указанием тега (или регулярного выражения)

Обратите внимание на заголовок HTTP Tags, который будет добавлен к HTTP-запросу (в приведенном ниже примере уведомление отправляется только для регистраций с полезными данными "sports").

```python
hub.send_windows_notification(wns_payload, "sports")
```

![Снимок экрана консоли с подробными сведениями о запросе H T T и форматом уведомлений служебной шины, тегом уведомления служебной шины и значениями X W N S, выделенными красным цветом.][3]

### <a name="send-notification-specifying-multiple-tags"></a>Отправка уведомления с указанием нескольких тегов

Обратите внимание на изменения заголовка Tags HTTP при отправке нескольких тегов.

```python
tags = {'sports', 'politics'}
hub.send_windows_notification(wns_payload, tags)
```

![Снимок экрана консоли с подробными сведениями о запросе H T T и форматом уведомлений служебной шины, нескольких тегах уведомлений служебной шины и значениями X W N S, выделенными красным цветом.][4]

### <a name="templated-notification"></a>Шаблон уведомления

Обратите внимание, что происходит изменение заголовка Format HTTP и текст полезных данных отправляется в составе запроса HTTP.

**Сторона клиента — зарегистрированный шаблон:**

```python
var template = @"<toast><visual><binding template=""ToastText01""><text id=""1"">$(greeting_en)</text></binding></visual></toast>";
```

**Серверная сторона — отправка полезных данных:**

```python
template_payload = {'greeting_en': 'Hello', 'greeting_fr': 'Salut'}
hub.send_template_notification(template_payload)
```

![Снимок экрана консоли с подробными сведениями о запросе H T T, типом содержимого и значениями формата уведомлений служебной шины, выделенными красным цветом.][5]

## <a name="next-steps"></a>Дальнейшие действия

В этой статье рассмотрено создание простого клиента REST Python для службы "Центры уведомлений". Здесь можно выполнять следующие действия:

- Скачать полный [пример программы-оболочки REST Python], содержащий весь код из этой статьи.
- Продолжение изучения функции добавления тегов для концентраторов уведомлений в [учебнике "экстренные новости] "
- Продолжить изучение функции шаблонов центров уведомлений в учебнике [по передаче локализованных экстренных новостей]

<!-- URLs -->
[примере оболочки REST Python]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/notificationhubs-rest-python
[Руководство по началу работы]: ./notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[учебника по передаче экстренных новостей]: ./notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[по передаче локализованных экстренных новостей]: ./notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md

<!-- Images. -->
[1]: ./media/notification-hubs-python-backend-how-to/DetailedLoggingInfo.png
[2]: ./media/notification-hubs-python-backend-how-to/BroadcastScenario.png
[3]: ./media/notification-hubs-python-backend-how-to/SendWithOneTag.png
[4]: ./media/notification-hubs-python-backend-how-to/SendWithMultipleTags.png
[5]: ./media/notification-hubs-python-backend-how-to/TemplatedNotification.png
