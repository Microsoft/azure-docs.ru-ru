---
title: Устранение неполадок подключения устройств к Azure IoT Central | Документация Майкрософт
description: Устранение неполадок, связанных с тем, что вы не видите данные с устройств в IoT Central
services: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 08/13/2020
ms.topic: troubleshooting
ms.service: iot-central
ms.custom: device-developer, devx-track-azurecli
ms.openlocfilehash: d1a7c94152b611ea0dbea249156add617178d7ca
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98673240"
---
# <a name="troubleshoot-why-data-from-your-devices-isnt-showing-up-in-azure-iot-central"></a>Устранение неполадок, при которых данные с устройств не отображаются в Azure IoT Central

Этот документ помогает разработчикам устройств узнать, почему данные, отправляемые их устройствами, IoT Central могут не отображаться в приложении.

Исследование состоит из двух основных областей:

- Проблемы с подключением устройств
  - Проблемы проверки подлинности, такие как устройство, имеет недопустимые учетные данные
  - Проблемы, связанные с подключением к сети
  - Устройство не утверждено или заблокировано
- Проблемы с фигурами полезных данных устройства

Это руководство по устранению неполадок посвящено проблемам подключения устройств и проблемам с фигурами полезных данных устройства.

## <a name="device-connectivity-issues"></a>Проблемы с подключением устройств

Этот раздел поможет определить, достигли ли ваши данные IoT Central.

Установите средство и расширение, если вы еще этого не сделали `az cli` `azure-iot` .

Сведения об установке см. в `az cli` разделе [Установка Azure CLI](/cli/azure/install-azure-cli).

Чтобы [установить](/cli/azure/azure-cli-reference-for-IoT#extension-reference-installation) `azure-iot` расширение, выполните следующую команду:

```azurecli
az extension add --name azure-iot
```

> [!NOTE]
> При первом запуске команды расширения может быть предложено установить `uamqp` библиотеку.

После установки `azure-iot` расширения запустите устройство, чтобы убедиться, что отправляемые им сообщения допускают свой способ IOT Central.

Используйте следующие команды для входа в подписку, в которой имеется приложение IoT Central:

```azurecli
az login
az set account --subscription <your-subscription-id>
```

Чтобы отслеживать данные телеметрии, отправляемые устройством, используйте следующую команду:

```azurecli
az iot central diagnostics monitor-events --app-id <app-id> --device-id <device-name>
```

Если устройство успешно подключено к IoT Central, вы увидите результат, аналогичный приведенному ниже:

```output
Monitoring telemetry.
Filtering on device: device-001
{
    "event": {
        "origin": "device-001",
        "module": "",
        "interface": "",
        "component": "",
        "payload": {
            "temp": 65.57910343679293,
            "humid": 36.16224660107426
        }
    }
}
```

Для мониторинга обновлений свойств устройство обменивается данными с IoT Central используйте следующую команду предварительной версии:

```azurecli
az iot central diagnostics monitor-properties --app-id <app-id> --device-id <device-name>
```

Если устройство успешно отправляет обновления свойств, вы увидите результат, аналогичный приведенному ниже:

```output
Changes in reported properties:
version : 32
{'state': 'true', 'name': {'value': {'value': 'Contoso'}, 'status': 'completed', 'desiredVersion': 7, 'ad': 'completed', 'av': 7, 'ac
': 200}, 'brightness': {'value': {'value': 2}, 'status': 'completed', 'desiredVersion': 7, 'ad': 'completed', 'av': 7, 'ac': 200}, 'p
rocessorArchitecture': 'ARM', 'swVersion': '1.0.0'}
```

Если в окне терминала отображаются данные, то данные будут переводиться в приложение IoT Central.

Если через несколько минут данные не отображаются, попробуйте нажать клавишу `Enter` или `return` на клавиатуре, если выходные данные будут зависнуть.

Если данные в терминале по-прежнему не отображаются, скорее всего, устройство испытывает проблемы с сетевым подключением или неправильно отправляет данные для IoT Central.

### <a name="check-the-provisioning-status-of-your-device"></a>Проверка состояния подготовки устройства

Если данные не отображаются на мониторе, проверьте состояние подготовки устройства, выполнив следующую команду:

```azurecli
az iot central device registration-info --app-id <app-id> --device-id <device-name>
```

В следующих выходных данных показан пример устройства, для которого заблокировано подключение:

```json
{
  "@device_id": "v22upeoqx6",
  "device_registration_info": {
    "device_status": "blocked",
    "display_name": "Environmental Sensor - v22upeoqx6",
    "id": "v22upeoqx6",
    "instance_of": "urn:krhsi_k0u:modelDefinition:w53jukkazs",
    "simulated": false
  },
  "dps_state": {
    "error": "Device is blocked from connecting to IoT Central application. Unblock the device in IoT Central and retry. Learn more:
https://aka.ms/iotcentral-docs-dps-SAS",
    "status": null
  }
}
```

| Состояние подготовки устройства | Описание | Возможное устранение рисков |
| - | - | - |
| Подготовлено | Не удается немедленно распознать ошибку. | Н/Д |
| Зарегистрировано | Устройство еще не подключено к IoT Central. | Проверьте журналы устройств на наличие проблем с подключением. |
| Блокировано | Устройству запрещено подключаться к IoT Central. | Устройству запрещено подключаться к IoT Central приложению. Разблокируйте устройство в IoT Central и повторите попытку. Дополнительные сведения см. в разделе [блочные устройства](concepts-get-connected.md#device-status-values). |
| Неутвержденные | Устройство не утверждено. | Устройство не утверждено для подключения к IoT Central приложению. Утвердите устройство в IoT Central и повторите попытку. Дополнительные сведения см. в разделе [утверждение устройств](concepts-get-connected.md#device-registration) . |
| Несвязанные | Устройство не связано с шаблоном устройства. | Свяжите устройство с шаблоном устройства, чтобы IoT Central знала, как анализировать данные. |

Дополнительные сведения о [кодах состояния устройств](concepts-get-connected.md#device-status-values).

### <a name="error-codes"></a>Коды ошибок

Если вы по-прежнему не можете диагностировать, почему данные не отображаются в `monitor-events` , следующий шаг заключается в поиске кодов ошибок, сообщаемых устройством.

Запустите сеанс отладки на устройстве или собирайте журналы с устройства. Проверьте коды ошибок, которые сообщает устройство.

В следующих таблицах показаны распространенные коды ошибок и возможные действия по их устранению.

Если вы видите проблемы, связанные с потоком проверки подлинности, выполните следующие действия.

| Код ошибки | Описание | Возможное устранение рисков |
| - | - | - |
| 400 | Недопустимый текст запроса. Например, он не может быть проанализирован или объект не может быть проверен. | Убедитесь, что вы отправляете правильный текст запроса как часть потока аттестации или используете пакет SDK для устройства. |
| 401 | Не удается проверить маркер авторизации. Например, срок действия истек или не применяется к URI запроса. Этот код ошибки также возвращается на устройства как часть потока аттестации доверенного платформенного модуля. | Убедитесь, что устройство имеет правильные учетные данные. |
| 404 | Экземпляр службы подготовки устройств или ресурс, например регистрация, не существует. | Отправьте запрос в службу [поддержки клиентов](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview). |
| 412 | `ETag`В запросе не совпадает с `ETag` именем существующего ресурса согласно rfc7232). | Отправьте запрос в службу [поддержки клиентов](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview). |
| 429 | Операции регулируется службой. Конкретные ограничения службы см. в разделе [ограничения службы подготовки устройств](../../azure-resource-manager/management/azure-subscription-service-limits.md#iot-hub-device-provisioning-service-limits)к добавлению в центр Интернета вещей. | Уменьшение частоты сообщений, разделение обязанностей между несколькими устройствами. |
| 500 | Внутренняя ошибка. | Отправьте запрос в службу [поддержки клиентов](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) , чтобы узнать, могут ли они помочь вам. |

## <a name="payload-shape-issues"></a>Проблемы с фигурами полезных данных

После того как устройство отправляет данные в IoT Central, необходимо убедиться, что устройство отправляет данные в допустимом формате.

Существует две основные категории распространенных проблем, которые приводят к тому, что данные устройства не отображаются в IoT Central:

- Несоответствие шаблона устройства с данными устройства:
  - Несовпадение имен, таких как опечатки или проблемы с сопоставлением вариантов.
  - Несмоделированные свойства, в которых схема не определена в шаблоне устройства.
  - Несоответствие схем, например тип, определенный в шаблоне `boolean` , в виде, но данные являются строкой.
  - Одно и то же имя телеметрии определено в нескольких интерфейсах, но устройство не соответствует IoT самонастраивающийся.
- Фигура данных является недопустимым JSON. Дополнительные сведения см. в разделе [данные телеметрии, свойства и команды](concepts-telemetry-properties-commands.md).

Чтобы определить категории, в которых находится ваша ошибка, выполните наиболее подходящую команду для своего сценария:

- Чтобы проверить данные телеметрии, используйте команду Предварительный просмотр:

    ```azurecli
    az iot central diagnostics validate-messages --app-id <app-id> --device-id <device-name>
    ```

- Чтобы проверить обновления свойств, используйте команду Предварительный просмотр

    ```azurecli
    az iot central diagnostics validate-properties --app-id <app-id> --device-id <device-name>
    ```

При первом запуске команды может появиться запрос на установку `uamqp` библиотеки `validate` .

В следующих выходных данных показаны примеры сообщений об ошибках и предупреждений из команды Validate:

```output
Validating telemetry.
Filtering on device: v22upeoqx6.
Exiting after 300 second(s), or 10 message(s) have been parsed (whichever happens first).
[WARNING] [DeviceId: v22upeoqx6] No encoding found. Expected encoding 'utf-8' to be present in message header.

[WARNING] [DeviceId: v22upeoqx6] Content type '' is not supported. Expected Content type is 'application/json'.

[ERROR] [DeviceId: v22upeoqx6] [TemplateId: urn:krhsi_k0u:modelDefinition:w53jukkazs] Datatype of field 'humid' does not match the da
tatype 'double'. Data '56'. All dates/times/datetimes/durations must be ISO 8601 compliant.
```

Если вы предпочитаете использовать графический интерфейс пользователя, используйте IoT Central представление **необработанных данных** , чтобы определить, не моделируется ли что-либо. Представление " **необработанные данные** " не обнаруживает, отправляет ли устройство неверно сформированный код JSON.

:::image type="content" source="media/troubleshoot-connection/raw-data-view.png" alt-text="Снимок экрана представления &quot;необработанные данные&quot;":::

После обнаружения проблемы может потребоваться обновить встроенное по устройства или создать новый шаблон устройства, моделирующий ранее несмоделированные данные.

Если вы решили создать новый шаблон, который моделирует данные правильно, перенесите устройства из старого шаблона в новый шаблон. Дополнительные сведения см. в статье [Управление устройствами в приложении IOT Central Azure](howto-manage-devices.md).

## <a name="next-steps"></a>Дальнейшие действия

Если вам нужна дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack overflow](https://azure.microsoft.com/support/community/). Кроме того, вы можете отправить запрос в [службу поддержки Azure](https://portal.azure.com/#create/Microsoft.Support).

Дополнительные сведения см. в статье [Поддержка Azure IOT и параметры справки](../../iot-fundamentals/iot-support-help.md).
