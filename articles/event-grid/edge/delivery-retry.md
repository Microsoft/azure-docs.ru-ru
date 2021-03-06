---
title: Доставка и повторная отправка — служба "Сетка событий Azure" IoT Edge | Документация Майкрософт
description: Доставка и повторная попытка в сетке событий на IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: aa0b3a05fb26f6be951b697145d7b22e03b7792d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86171624"
---
# <a name="delivery-and-retry"></a>Доставка и повторные попытки доставки

Сетка событий обеспечивает надежную доставку. Он пытается доставить каждое сообщение по крайней мере один раз для каждой подходящей подписки немедленно. Если конечная точка подписчика не подтверждает получение события или произошла ошибка, сетка событий повторяет попытку доставки на основе предопределенного **расписания повторных** попыток и **политики повтора**.  По умолчанию модуль сетки событий доставляет подписчику одно событие за раз. В этом случае полезные данные представляют собой массив с одним событием. Можно предоставить модулю более одного события за раз, включив функцию пакетной обработки данных. Дополнительные сведения об этой функции см. в разделе [Пакетная обработка выходных данных](delivery-output-batching.md).  

> [!IMPORTANT]
>Нет поддержки сохраняемости для данных событий. Это означает, что повторное развертывание или перезапуск модуля сетки событий приведет к потере всех событий, которые еще не были доставлены.

## <a name="retry-schedule"></a>Расписание повторных попыток

Сетка событий ожидает ответа до 60 секунд после доставки сообщения. Если конечная точка подписчика не подтверждает ответ, то сообщение будет поставлено в очередь в одной из наших очередей для последующих повторных попыток.

Существует два предварительно настроенных очереди, которые определяют расписание, в котором будет предпринята попытка повторной попытки. Их можно сформулировать следующим образом.

| Расписание | Описание |
| ---------| ------------ |
| 1 минута | Сообщения, которые в конце концов, попытаются каждую минуту.
| 10 минут. | Сообщения, которые в конце концов, попытаются каждую десятую минуту.

### <a name="how-it-works"></a>Принцип работы

1. Сообщение поступает в модуль сетки событий. Предпринята попытка немедленной доставки.
1. В случае сбоя доставки сообщение помещается в очередь в 1 минуту и повторяется через минуту.
1. Если доставка не будет выполнена, сообщение помещается в очередь в течение 10 минут и повторяется каждые 10 минут.
1. Попытки доставки выполняются до тех пор, пока не будут достигнуты ограничения политики "успех" или "Повтор".

## <a name="retry-policy-limits"></a>Ограничения политики повтора

Существует две конфигурации, определяющие политику повторных попыток. Их можно сформулировать следующим образом.

* Максимальное число попыток
* Срок жизни события (TTL)

Событие будет удалено, если достигнуто какое-либо из ограничений политики повтора. Само расписание повторов было описано в разделе Расписание повторных попыток. Настройку этих ограничений можно выполнить для всех подписчиков или для каждой подписки. В следующем разделе каждая из них описана более подробно.

## <a name="configuring-defaults-for-all-subscribers"></a>Настройка значений по умолчанию для всех подписчиков

Существует два свойства: `brokers__defaultMaxDeliveryAttempts` и `broker__defaultEventTimeToLiveInSeconds` , которые можно настроить как часть развертывания сетки событий, которая управляет политиками повтора по умолчанию для всех подписчиков.

| Имя свойства | Описание |
| ---------------- | ------------ |
| `broker__defaultMaxDeliveryAttempts` | Максимальное число попыток доставки события. Значение по умолчанию: 30.
| `broker__defaultEventTimeToLiveInSeconds` | Срок жизни события (в секундах), по истечении которого событие будет удалено, если оно не доставлено. Значение по умолчанию: **7200** с

## <a name="configuring-defaults-per-subscriber"></a>Настройка значений по умолчанию на подписчик

Также можно указать ограничения для политики повтора для каждой подписки.
Сведения о настройке значений по умолчанию для каждого подписчика см. в [документации по API](api.md) . Значения по умолчанию на уровне подписки переопределяют конфигурации уровня модуля.

## <a name="examples"></a>Примеры

В следующем примере настраивается политика повтора в модуле сетки событий с Макснумберофаттемптс = 3 и сроком жизни события 30 минут.

```json
{
  "Env": [
    "broker__defaultMaxDeliveryAttempts=3",
    "broker__defaultEventTimeToLiveInSeconds=1800"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ]
    }
  }
}
```

В следующем примере настраивается подписка веб-перехватчика с Макснумберофаттемптс = 3 и сроком жизни события в 30 минут.

```json
{
 "properties": {
  "destination": {
   "endpointType": "WebHook",
   "properties": {
    "endpointUrl": "<your_webhook_url>",
    "eventDeliverySchema": "eventgridschema"
   }
  },
  "retryPolicy": {
   "eventExpiryInMinutes": 30,
   "maxDeliveryAttempts": 3
  }
 }
}
```
