---
title: Создание веб-перехватчиков на правилах Azure IoT Central | Документация Майкрософт
description: Создание веб-перехватчиков в Azure IoT Central для автоматического уведомления других приложений о срабатывании правил.
author: viv-liu
ms.author: viviali
ms.date: 04/03/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: corywink
ms.openlocfilehash: b2ac4bbf1457144d23a91c4e83b554b3ee806119
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87337234"
---
# <a name="create-webhook-actions-on-rules-in-azure-iot-central"></a>Создание действий веб-перехватчика на правилах Azure IoT Central

*Этот раздел предназначен для разработчиков и администраторов.*

Веб-перехватчики позволяют подключить приложение IoT Central к другим приложениям и службам для удаленного мониторинга и уведомления. Веб-перехватчики автоматически уведомляют другие приложения и службы, к которым вы подключаетесь, при каждой активации правила в приложении IoT Central. Приложение IoT Central отправляет запрос POST в конечную точку HTTP другого приложения при каждом запуске правила. Полезные данные содержат сведения об устройстве и сведения о триггере правила.

## <a name="set-up-the-webhook"></a>Настройка веб-перехватчика

В этом примере вы подключаетесь к RequestBin, чтобы получать уведомления при срабатывании правил с помощью веб-перехватчиков.

1. Откройте [RequestBin](https://requestbin.net/).

1. Создайте новый RequestBin и скопируйте **Bin URL** (URL-адрес Bin).

1. Создайте [правило телеметрии](tutorial-create-telemetry-rules.md). Сохраните правило и добавьте новое действие.

    ![Экран создания веб-перехватчика](media/howto-create-webhooks/webhookcreate.png)

1. Выберите действие веб-перехватчика, укажите отображаемое имя и вставьте URL-адрес Bin как URL-адрес обратного вызова.

1. Сохраните правило.

Теперь, когда правило будет активировано, вы увидите, что в RequestBin появится новый запрос.

## <a name="payload"></a>Payload

При активации правила запрос HTTP POST выполняется в URL-адрес обратного вызова, содержащий полезные данные JSON с данными телеметрии, устройством, правилом и приложением. Полезная нагрузка может выглядеть следующим образом:

```json
{
    "timestamp": "2020-04-06T00:20:15.06Z",
    "action": {
        "id": "<id>",
        "type": "WebhookAction",
        "rules": [
            "<rule_id>"
        ],
        "displayName": "Webhook 1",
        "url": "<callback_url>"
    },
    "application": {
        "id": "<application_id>",
        "displayName": "Contoso",
        "subdomain": "contoso",
        "host": "contoso.azureiotcentral.com"
    },
    "device": {
        "id": "<device_id>",
        "etag": "<etag>",
        "displayName": "MXChip IoT DevKit - 1yl6vvhax6c",
        "instanceOf": "<device_template_id>",
        "simulated": true,
        "provisioned": true,
        "approved": true,
        "cloudProperties": {
            "City": {
                "value": "Seattle"
            }
        },
        "properties": {
            "deviceinfo": {
                "firmwareVersion": {
                    "value": "1.0.0"
                }
            }
        },
        "telemetry": {
            "<interface_instance_name>": {
                "humidity": {
                    "value": 47.33228889360127
                }
            }
        }
    },
    "rule": {
        "id": "<rule_id>",
        "displayName": "Humidity monitor"
    }
}
```
Если правило отслеживает агрегированные данные телеметрии за определенный период времени, полезная нагрузка будет содержать другой раздел телеметрии.

```json
{
    "telemetry": {
        "<interface_instance_name>": {
            "Humidity": {
                "avg": 39.5
            }
        }
    }
}
```

## <a name="data-format-change-notice"></a>Уведомление об изменении формата данных

Если вы создали и сохранили один или несколько веб-перехватчиков до **3 апреля 2020**, необходимо удалить веб-перехватчик и создать новый веб-перехватчик. Это связано с тем, что старые веб-перехватчики используют устаревший формат полезных данных, который будет считаться устаревшим в будущем.

### <a name="webhook-payload-format-deprecated-as-of-3-april-2020"></a>Полезные данные веб-перехватчика (формат устарел до 3 апреля 2020)

```json
{
    "id": "<id>",
    "displayName": "Webhook 1",
    "timestamp": "2019-10-24T18:27:13.538Z",
    "rule": {
        "id": "<id>",
        "displayName": "High temp alert",
        "enabled": true
    },
    "device": {
        "id": "mx1",
        "displayName": "MXChip IoT DevKit - mx1",
        "instanceOf": "<device-template-id>",
        "simulated": true,
        "provisioned": true,
        "approved": true
    },
    "data": [{
        "@id": "<id>",
        "@type": ["Telemetry"],
        "name": "temperature",
        "displayName": "Temperature",
        "value": 66.27310467496761,
        "interfaceInstanceName": "sensors"
    }],
    "application": {
        "id": "<id>",
        "displayName": "x - Store Analytics Checkout",
        "subdomain": "<subdomain>",
        "host": "<host>"
    }
}
```

## <a name="known-limitations"></a>Известные ограничения

В настоящее время нет программного метода подписки или отмены подписки на эти веб-перехватчики через API.

Если у вас есть идеи по улучшению этой функции, опубликуйте свои предложения на [форуме пользовательских голосов](https://feedback.azure.com/forums/911455-azure-iot-central).

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как настраивать и использовать веб-перехватчики, предлагаем следующий шаг: изучение [настройки групп действий Azure Monitor](howto-use-action-groups.md).
