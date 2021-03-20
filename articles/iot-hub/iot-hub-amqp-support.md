---
title: Общие сведения о поддержке AMQP в центре Интернета вещей Azure | Документация Майкрософт
description: Руководством для разработчиков. Поддержка устройств, подключающихся к устройствам центра Интернета вещей и подключенных к службе конечных точек с помощью протокола AMQP. Содержит сведения о встроенной поддержке AMQP в пакетах SDK для устройств Azure IoT.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/30/2019
ms.author: robinsh
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 7b3dcfc51df7f0fe4291e9c5babccc1444ad32e9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81730754"
---
# <a name="communicate-with-your-iot-hub-by-using-the-amqp-protocol"></a>Обмен данными с центром Интернета вещей с помощью протокола AMQP

Центр Интернета вещей Azure поддерживает [OASIS расширенный протокол управления очередью сообщений (AMQP) версии 1,0](https://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf) для предоставления разнообразных функциональных возможностей с помощью конечных точек, ориентированных на устройства и службы. В этом документе описывается использование клиентов AMQP для подключения к центру Интернета вещей для использования функций центра Интернета вещей.

## <a name="service-client"></a>Клиент службы

### <a name="connect-and-authenticate-to-an-iot-hub-service-client"></a>Подключение и аутентификация в центре Интернета вещей (клиент службы)

Для подключения к центру Интернета вещей с помощью AMQP клиент может использовать проверку подлинности [на основе утверждений (CBS)](https://www.oasis-open.org/committees/download.php/60412/amqp-cbs-v1.0-wd03.doc) или [простую проверку подлинности и уровень безопасности (SASL)](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer).

Для клиента службы требуются следующие сведения:

| Сведения | Значение |
|-------------|--------------|
| Имя узла центра Интернета вещей | `<iot-hub-name>.azure-devices.net` |
| Имя раздела | `service` |
| Ключ доступа | Первичный или вторичный ключ, связанный со службой |
| Подписанный URL-адрес | Кратковременная подпись общего доступа в следующем формате: `SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}` . Чтобы получить код для создания этой подписи, см. раздел [Управление доступом к центру Интернета вещей](./iot-hub-devguide-security.md#security-token-structure).

В следующем фрагменте кода используется [Библиотека uAMQP в Python](https://github.com/Azure/azure-uamqp-python) для подключения к центру Интернета вещей через ссылку отправителя.

```python
import uamqp
import urllib
import time

# Use generate_sas_token implementation available here:
# https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
policy_name = 'service'
access_key = '<primary-or-secondary-key>'
operation = '<operation-link-name>'  # example: '/messages/devicebound'

username = '{policy_name}@sas.root.{iot_hub_name}'.format(
    iot_hub_name=iot_hub_name, policy_name=policy_name)
sas_token = generate_sas_token(hostname, access_key, policy_name)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

# Create a send or receive client
send_client = uamqp.SendClient(uri, debug=True)
receive_client = uamqp.ReceiveClient(uri, debug=True)
```

### <a name="invoke-cloud-to-device-messages-service-client"></a>Вызов сообщений из облака на устройство (клиент службы)

Сведения о обмене сообщениями из облака на устройство между службой и центром Интернета вещей, а также между устройством и центром Интернета вещей см. в статье [Отправка сообщений из облака на устройство из центра Интернета](iot-hub-devguide-messages-c2d.md)вещей. Клиент службы использует две ссылки для отправки сообщений и получения отзывов о ранее отправленных сообщениях с устройств, как описано в следующей таблице.

| Кем создано | Тип связи | Путь ссылки | Описание |
|------------|-----------|-----------|-------------|
| Служба | Ссылка для отправителя | `/messages/devicebound` | Сообщения, отправляемые из облака на устройство, отправляются по этой ссылке службой. Для сообщений, отправляемых по этой ссылке `To` , свойству присваивается путь к каналу связи приемника целевого устройства `/devices/<deviceID>/messages/devicebound` . |
| Служба | Ссылка на приемник | `/messages/serviceBound/feedback` | Сообщения о завершении, отклонении и отзыве, полученные с устройств, полученных по этой ссылке службой. Дополнительные сведения о сообщениях с отзывами см. в статье [Отправка сообщений из облака на устройство из центра Интернета вещей](./iot-hub-devguide-messages-c2d.md#message-feedback). |

В следующем фрагменте кода показано, как создать сообщение из облака на устройство и отправить его на устройство с помощью [библиотеки uAMQP в Python](https://github.com/Azure/azure-uamqp-python).

```python
import uuid
# Create a message and set message property 'To' to the device-bound link on device
msg_id = str(uuid.uuid4())
msg_content = b"Message content goes here!"
device_id = '<device-id>'
to = '/devices/{device_id}/messages/devicebound'.format(device_id=device_id)
ack = 'full'  # Alternative values are 'positive', 'negative', and 'none'
app_props = {'iothub-ack': ack}
msg_props = uamqp.message.MessageProperties(message_id=msg_id, to=to)
msg = uamqp.Message(msg_content, properties=msg_props,
                    application_properties=app_props)

# Send the message by using the send client that you created and connected to the IoT hub earlier
send_client.queue_message(msg)
results = send_client.send_all_messages()

# Close the client if it's not needed
send_client.close()
```

Чтобы получить отзыв, клиент службы создает ссылку получателя. В следующем фрагменте кода показано, как создать ссылку с помощью [библиотеки uAMQP в Python](https://github.com/Azure/azure-uamqp-python):

```python
import json

operation = '/messages/serviceBound/feedback'

# ...
# Re-create the URI by using the preceding feedback path and authenticate it
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
batch = receive_client.receive_message_batch(max_batch_size=10)
for msg in batch:
    print('received a message')
    # Check content_type in message property to identify feedback messages coming from device
    if msg.properties.content_type == 'application/vnd.microsoft.iothub.feedback.json':
        msg_body_raw = msg.get_data()
        msg_body_str = ''.join(msg_body_raw)
        msg_body = json.loads(msg_body_str)
        print(json.dumps(msg_body, indent=2))
        print('******************')
        for feedback in msg_body:
            print('feedback received')
            print('\tstatusCode: ' + str(feedback['statusCode']))
            print('\toriginalMessageId: ' + str(feedback['originalMessageId']))
            print('\tdeviceId: ' + str(feedback['deviceId']))
            print
    else:
        print('unknown message:', msg.properties.content_type)
```

Как показано в приведенном выше коде, в сообщении с сообщением о типе содержимого " *приложение/vnd.microsoft.iothub.feedback.js*" в качестве типа контента. Чтобы определить состояние доставки исходного сообщения, можно использовать свойства в тексте JSON сообщения:

* Ключ `statusCode` в тексте отзыва имеет одно из следующих значений: *Success*, *Expires*, *деливерикаунтексцеедед*, *отклонено* или *очищено*.

* Ключ `deviceId` в тексте обратной связи содержит идентификатор целевого устройства.

* Ключ `originalMessageId` в тексте обратной связи содержит идентификатор исходного сообщения, отправленного из облака на устройство, которое было послано службой. Это состояние доставки можно использовать для корреляции отзывов о сообщениях, отправляемых из облака на устройство.

### <a name="receive-telemetry-messages-service-client"></a>Получение сообщений телеметрии (клиент службы)

По умолчанию центр Интернета вещей хранит полученные сообщения телеметрии устройств в встроенном концентраторе событий. Клиент службы может использовать протокол AMQP для получения хранимых событий.

Для этой цели клиент службы сначала должен подключиться к конечной точке центра Интернета вещей и получить адрес перенаправления для встроенных концентраторов событий. Затем клиент службы использует указанный адрес для подключения к встроенному концентратору событий.

На каждом шаге клиент должен предоставить следующие сведения:

* Допустимые учетные данные службы (маркер подписанного общего доступа к службе).

* Правильно отформатированный путь к секции группы потребителей, из которой он намеревается получать сообщения. Для заданной группы потребителей и идентификатора секции путь имеет следующий формат: `/messages/events/ConsumerGroups/<consumer_group>/Partitions/<partition_id>` (группа потребителей по умолчанию — `$Default` ).

* Необязательный предикат фильтрации для обозначения начальной точки в секции. Этот предикат может иметь формат порядкового номера, смещения или пометки времени в очереди.

В следующем фрагменте кода для демонстрации предыдущих шагов используется [Библиотека uAMQP в Python](https://github.com/Azure/azure-uamqp-python) .

```python
import json
import uamqp
import urllib
import time

# Use the generate_sas_token implementation that's available here: https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
policy_name = 'service'
access_key = '<primary-or-secondary-key>'
operation = '/messages/events/ConsumerGroups/{consumer_group}/Partitions/{p_id}'.format(
    consumer_group='$Default', p_id=0)

username = '{policy_name}@sas.root.{iot_hub_name}'.format(
    policy_name=policy_name, iot_hub_name=iot_hub_name)
sas_token = generate_sas_token(hostname, access_key, policy_name)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

# Optional filtering predicates can be specified by using endpoint_filter
# Valid predicates include:
# - amqp.annotation.x-opt-sequence-number
# - amqp.annotation.x-opt-offset
# - amqp.annotation.x-opt-enqueued-time
# Set endpoint_filter variable to None if no filter is needed
endpoint_filter = b'amqp.annotation.x-opt-sequence-number > 2995'

# Helper function to set the filtering predicate on the source URI


def set_endpoint_filter(uri, endpoint_filter=''):
    source_uri = uamqp.address.Source(uri)
    source_uri.set_filter(endpoint_filter)
    return source_uri


receive_client = uamqp.ReceiveClient(
    set_endpoint_filter(uri, endpoint_filter), debug=True)
try:
    batch = receive_client.receive_message_batch(max_batch_size=5)
except uamqp.errors.LinkRedirect as redirect:
    # Once a redirect error is received, close the original client and recreate a new one to the re-directed address
    receive_client.close()

    sas_auth = uamqp.authentication.SASTokenAuth.from_shared_access_key(
        redirect.address, policy_name, access_key)
    receive_client = uamqp.ReceiveClient(set_endpoint_filter(
        redirect.address, endpoint_filter), auth=sas_auth, debug=True)

# Start receiving messages in batches
batch = receive_client.receive_message_batch(max_batch_size=5)
for msg in batch:
    print('*** received a message ***')
    print(''.join(msg.get_data()))
    print('\t: ' + str(msg.annotations['x-opt-sequence-number']))
    print('\t: ' + str(msg.annotations['x-opt-offset']))
    print('\t: ' + str(msg.annotations['x-opt-enqueued-time']))
```

Для конкретного идентификатора устройства центр Интернета вещей использует хэш идентификатора устройства, чтобы определить, в какой секции хранить свои сообщения. В предыдущем фрагменте кода показано, как события поступают из одной такой секции. Однако обратите внимание, что обычному приложению часто требуется получать события, которые хранятся во всех секциях концентратора событий.

## <a name="device-client"></a>Клиент устройства

### <a name="connect-and-authenticate-to-an-iot-hub-device-client"></a>Подключение и аутентификация в центре Интернета вещей (клиент устройства)

Для подключения к центру Интернета вещей с помощью AMQP устройство может использовать проверку подлинности [на основе утверждений (CBS)](https://www.oasis-open.org/committees/download.php/60412/amqp-cbs-v1.0-wd03.doc) или [простую проверку подлинности и уровень безопасности (SASL)](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) .

Для клиента устройства требуются следующие сведения:

| Сведения | Значение |
|-------------|--------------|
| Имя узла центра Интернета вещей | `<iot-hub-name>.azure-devices.net` |
| Ключ доступа | Первичный или вторичный ключ, связанный с устройством |
| Подписанный URL-адрес | Кратковременная подпись общего доступа в следующем формате: `SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}` . Чтобы получить код для создания этой подписи, см. раздел [Управление доступом к центру Интернета вещей](./iot-hub-devguide-security.md#security-token-structure).

В следующем фрагменте кода используется [Библиотека uAMQP в Python](https://github.com/Azure/azure-uamqp-python) для подключения к центру Интернета вещей через ссылку отправителя.

```python
import uamqp
import urllib
import uuid

# Use generate_sas_token implementation available here:
# https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
device_id = '<device-id>'
access_key = '<primary-or-secondary-key>'
username = '{device_id}@sas.{iot_hub_name}'.format(
    device_id=device_id, iot_hub_name=iot_hub_name)
sas_token = generate_sas_token('{hostname}/devices/{device_id}'.format(
    hostname=hostname, device_id=device_id), access_key, None)

# e.g., '/devices/{device_id}/messages/devicebound'
operation = '<operation-link-name>'
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
send_client = uamqp.SendClient(uri, debug=True)
```

В качестве операций с устройствами поддерживаются следующие пути ссылок:

| Кем создано | Тип связи | Путь ссылки | Описание |
|------------|-----------|-----------|-------------|
| Устройства | Ссылка на приемник | `/devices/<deviceID>/messages/devicebound` | Сообщения из облака на устройства, предназначенные для устройств, поступают по этой ссылке на каждое целевое устройство. |
| Устройства | Ссылка для отправителя | `/devices/<deviceID>/messages/events` | Сообщения, отправляемые с устройства в облако, отправляются по этой ссылке. |
| Устройства | Ссылка для отправителя | `/messages/serviceBound/feedback` | Отзывы сообщений, отправляемых из облака на устройство, передаются в службу по этой ссылке устройствами. |

### <a name="receive-cloud-to-device-commands-device-client"></a>Получение команд из облака на устройство (клиент устройства)

Команды из облака на устройство, отправляемые на устройства, поступают по `/devices/<deviceID>/messages/devicebound` ссылке. Устройства могут получить эти сообщения в пакетах и при необходимости использовать полезные данные сообщения, свойства сообщений, заметки или свойства приложения в сообщении.

В следующем фрагменте кода используется [Библиотека uAMQP в Python](https://github.com/Azure/azure-uamqp-python)для получения сообщений из облака на устройство.

```python
# ...
# Create a receive client for the cloud-to-device receive link on the device
operation = '/devices/{device_id}/messages/devicebound'.format(
    device_id=device_id)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
while True:
    batch = receive_client.receive_message_batch(max_batch_size=5)
    for msg in batch:
        print('*** received a message ***')
        print(''.join(msg.get_data()))

        # Property 'to' is set to: '/devices/device1/messages/devicebound',
        print('\tto:                     ' + str(msg.properties.to))

        # Property 'message_id' is set to value provided by the service
        print('\tmessage_id:             ' + str(msg.properties.message_id))

        # Other properties are present if they were provided by the service
        print('\tcreation_time:          ' + str(msg.properties.creation_time))
        print('\tcorrelation_id:         ' +
              str(msg.properties.correlation_id))
        print('\tcontent_type:           ' + str(msg.properties.content_type))
        print('\treply_to_group_id:      ' +
              str(msg.properties.reply_to_group_id))
        print('\tsubject:                ' + str(msg.properties.subject))
        print('\tuser_id:                ' + str(msg.properties.user_id))
        print('\tgroup_sequence:         ' +
              str(msg.properties.group_sequence))
        print('\tcontent_encoding:       ' +
              str(msg.properties.content_encoding))
        print('\treply_to:               ' + str(msg.properties.reply_to))
        print('\tabsolute_expiry_time:   ' +
              str(msg.properties.absolute_expiry_time))
        print('\tgroup_id:               ' + str(msg.properties.group_id))

        # Message sequence number in the built-in event hub
        print('\tx-opt-sequence-number:  ' +
              str(msg.annotations['x-opt-sequence-number']))
```

### <a name="send-telemetry-messages-device-client"></a>Отправка сообщений телеметрии (клиент устройства)

Вы также можете отправить сообщения телеметрии с устройства с помощью AMQP. При необходимости на устройстве можно указать словарь свойств приложения или различные свойства сообщения, такие как идентификатор сообщения.

В следующем фрагменте кода используется [Библиотека uAMQP в Python](https://github.com/Azure/azure-uamqp-python) для отправки сообщений с устройства в облако с устройства.

```python
# ...
# Create a send client for the device-to-cloud send link on the device
operation = '/devices/{device_id}/messages/events'.format(device_id=device_id)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username), urllib.quote_plus(sas_token), hostname, operation)

send_client = uamqp.SendClient(uri, debug=True)

# Set any of the applicable message properties
msg_props = uamqp.message.MessageProperties()
msg_props.message_id = str(uuid.uuid4())
msg_props.creation_time = None
msg_props.correlation_id = None
msg_props.content_type = None
msg_props.reply_to_group_id = None
msg_props.subject = None
msg_props.user_id = None
msg_props.group_sequence = None
msg_props.to = None
msg_props.content_encoding = None
msg_props.reply_to = None
msg_props.absolute_expiry_time = None
msg_props.group_id = None

# Application properties in the message (if any)
application_properties = { "app_property_key": "app_property_value" }

# Create message
msg_data = b"Your message payload goes here"
message = uamqp.Message(msg_data, properties=msg_props, application_properties=application_properties)

send_client.queue_message(message)
results = send_client.send_all_messages()

for result in results:
    if result == uamqp.constants.MessageState.SendFailed:
        print result
```

## <a name="additional-notes"></a>Дополнительные сведения

* Подключения AMQP могут быть разорваны из-за сбоя сети или истечения срока действия маркера проверки подлинности (созданного в коде). Клиент службы должен справиться с этими обстоятельствами и при необходимости восстановить соединение и ссылки. Если срок действия маркера проверки подлинности истек, клиент может избежать отбрасывания подключения посредством упреждающего продления маркера до истечения срока его действия.

* Иногда клиент должен иметь возможность правильно обрабатывать перенаправления ссылок. Чтобы понять такую операцию, см. документацию по клиенту AMQP.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о протоколе AMQP см. в [спецификации AMQP v 1.0](https://www.amqp.org/sites/amqp.org/files/amqp.pdf).

Дополнительные сведения об обмене сообщениями в центре Интернета вещей см. в следующих статьях:

* [Получение сообщений из облака на устройство](./iot-hub-devguide-messages-c2d.md)
* [Поддержка дополнительных протоколов](iot-hub-protocol-gateway.md)
* [Поддержка протокола передачи данных телеметрии очереди сообщений (MQTT)](./iot-hub-mqtt-support.md)
