---
title: Доступ ко встроенным метрикам — Azure IoT Edge
description: Удаленный доступ к встроенным метрикам из компонентов среды выполнения IoT Edge
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 10/05/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 1a78db821c0fab01ad5d6752216a8f7682fb2c46
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103200493"
---
# <a name="access-built-in-metrics"></a>Доступ к встроенным метрикам

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Компоненты среды выполнения IoT Edge, центр IoT Edge и агент IoT Edge создают встроенные метрики в [формате демонстрации Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/). Удаленный доступ к этим метрикам для отслеживания работоспособности IoT Edge устройства.

Начиная с выпуска 1.0.10, метрики автоматически предоставляются по умолчанию через **порт 9600** модулей **edgeHub** и **edgeAgent** ( `http://edgeHub:9600/metrics` и `http://edgeAgent:9600/metrics` ). По умолчанию они не сопоставляются с узлом.

Получите доступ к метрикам с узла, предоставив и сопоставляя порт метрик из модуля `createOptions` . Приведенный ниже пример сопоставляет порт метрик по умолчанию с портом 9601 на узле:

```
{
  "ExposedPorts": {
    "9600/tcp": {}
  },
  "HostConfig": {
    "PortBindings": {
      "9600/tcp": [
        {
          "HostPort": "9601"
        }
      ]
    }
  }
}
```

При сопоставлении конечных точек метрик edgeHub и edgeAgent выбирайте разные и уникальные номера портов узлов.

> [!NOTE]
> Переменная среды `httpSettings__enabled` не должна иметь значение, чтобы `false` встроенные метрики были доступны для коллекции.
>
> Переменные среды, которые можно использовать для отключения метрик, перечислены в документации по [репозиторию Azure/iotedge](https://github.com/Azure/iotedge/blob/master/doc/EnvironmentVariables.md).

## <a name="available-metrics"></a>Доступные метрики

Метрики содержат теги, помогающие определить природу собираемой метрики. Все метрики содержат следующие Теги:

| Тег | Описание |
|-|-|
| iothub | Концентратор, с которым взаимодействуют устройства |
| edge_device | Идентификатор текущего устройства |
| instance_number | Идентификатор GUID, представляющий текущую среду выполнения. При перезапуске все метрики будут сброшены. Этот идентификатор GUID упрощает выверку перезапусков. |

В формате Prometheus демонстрации существует четыре основных типа метрик: Счетчик, датчик, гистограмма и сводка. Дополнительные сведения о различных типах метрик см. в документации по [типам метрик Prometheus](https://prometheus.io/docs/concepts/metric_types/).

Квантилей, предоставляемый для встроенной гистограммы и сводных метрик, — 0,1, 0,5, 0,9 и 0,99.

Модуль **edgeHub** создает следующие метрики:

| Название | Измерения | Описание |
|-|-|-|
| `edgehub_gettwin_total` | `source` (источник операции)<br> `id` (идентификатор модуля) | Тип: Счетчик<br> Общее число вызовов Жеттвин |
| `edgehub_messages_received_total` | `route_output` (выходные данные отправленного сообщения)<br> `id` | Тип: Счетчик<br> Общее число сообщений, полученных от клиентов |
| `edgehub_messages_sent_total` | `from` (Источник сообщения)<br> `to` (назначение сообщения)<br>`from_route_output`<br> `to_route_input` (вход назначения сообщения)<br> `priority` (приоритет сообщения к назначению) | Тип: Счетчик<br> Общее число сообщений, отправленных клиентам или вышестоящим<br> `to_route_input` пусто `to` , если имеет $upstream |
| `edgehub_reported_properties_total` | `target`(цель обновления)<br> `id` | Тип: Счетчик<br> Общее число обращений о обновлениях свойств |
| `edgehub_message_size_bytes` | `id`<br> | Тип: сводка<br> Размер сообщения от клиентов<br> Значения могут сообщаться, как если бы новые измерения не выдавались в `NaN` течение определенного периода времени (в настоящее время 10 минут); для `summary` типа `_count` `_sum` будут выдаваться соответствующие счетчики и. |
| `edgehub_gettwin_duration_seconds` | `source` <br> `id` | Тип: сводка<br> Время, затраченное на операции получения двойника |
| `edgehub_message_send_duration_seconds` | `from`<br> `to`<br> `from_route_output`<br> `to_route_input` | Тип: сводка<br> Время, затраченное на отправку сообщения |
| `edgehub_message_process_duration_seconds` | `from` <br> `to` <br> `priority` | Тип: сводка<br> Время, затраченное на обработку сообщения из очереди |
| `edgehub_reported_properties_update_duration_seconds` | `target`<br> `id` | Тип: сводка<br> Время, затраченное на обновление сообщаемых свойств |
| `edgehub_direct_method_duration_seconds` | `from` Оператор<br> `to` получателю | Тип: сводка<br> Время, затраченное на разрешение прямого сообщения |
| `edgehub_direct_methods_total` | `from`<br> `to` | Тип: Счетчик<br> Общее число отправленных прямых сообщений |
| `edgehub_queue_length` | `endpoint` (Источник сообщения)<br> `priority` (приоритет очереди) | Тип: датчик<br> Текущая длина очереди edgeHub для заданного приоритета |
| `edgehub_messages_dropped_total` | `reason` (no_route, ttl_expiry)<br> `from` <br> `from_route_output` | Тип: Счетчик<br> Общее количество сообщений, удаленных по причине |
| `edgehub_messages_unack_total` | `reason` (storage_failure)<br> `from`<br> `from_route_output` | Тип: Счетчик<br> Общее число неподтвержденных сообщений из-за сбоя хранилища |
| `edgehub_offline_count_total` | `id` | Тип: Счетчик<br> Общее количество раз, когда edgeHub перешел в режим "вне сети" |
| `edgehub_offline_duration_seconds`| `id` | Тип: сводка<br> Концентратор временных пограничных устройств был отключен |
| `edgehub_operation_retry_total` | `id`<br> `operation` (имя операции) | Тип: Счетчик<br> Общее число повторных попыток выполнения операций edgeHub |
| `edgehub_client_connect_failed_total` | `id` <br> `reason` (не прошло проверку подлинности)<br> | Тип: Счетчик<br> Общее число раз, когда клиенты не удалось подключиться к edgeHub |

Модуль **edgeAgent** создает следующие метрики:

| Название | Измерения | Описание |
|-|-|-|
| `edgeAgent_total_time_running_correctly_seconds` | `module_name` | Тип: датчик<br> Время, в течение которого модуль был указан в развертывании и был в состоянии выполнения |
| `edgeAgent_total_time_expected_running_seconds` | `module_name` | Тип: датчик<br> Время, в течение которого модуль был указан в развертывании |
| `edgeAgent_module_start_total` | `module_name`, `module_version` | Тип: Счетчик<br> Сколько раз edgeAgent запрашиваемый DOCKER для запуска модуля |
| `edgeAgent_module_stop_total` | `module_name`, `module_version` | Тип: Счетчик<br> Сколько раз edgeAgent запросила DOCKER для завершения работы модуля |
| `edgeAgent_command_latency_seconds` | `command` | Тип: датчик<br> Время, затраченное на выполнение данной команды DOCKER. Возможные команды: создание, обновление, удаление, запуск, завершение, перезапуск |
| `edgeAgent_iothub_syncs_total` | | Тип: Счетчик<br> Количество попыток edgeAgent синхронизировать двойника с iotHub, как успешно, так и неудачно. Это число включает агент, запрашивающий двойника, и уведомление концентратора об обновлении двойника |
| `edgeAgent_unsuccessful_iothub_syncs_total` | | Тип: Счетчик<br> Количество раз, когда edgeAgent не удалось синхронизировать свой двойника с iotHub. |
| `edgeAgent_deployment_time_seconds` | | Тип: Счетчик<br> Время, затраченное на завершение нового развертывания после получения изменений. |
| `edgeagent_direct_method_invocations_count` | `method_name` | Тип: Счетчик<br> Количество раз, когда вызывается встроенный прямой метод edgeAgent, например ping или restart. |
| `edgeAgent_host_uptime_seconds` | | Тип: датчик<br> Как долго будет находиться узел |
| `edgeAgent_iotedged_uptime_seconds` | | Тип: датчик<br> Как долго работает иотеджед |
| `edgeAgent_available_disk_space_bytes` | `disk_name`, `disk_filesystem`, `disk_filetype` | Тип: датчик<br> Объем свободного места на диске |
| `edgeAgent_total_disk_space_bytes` | `disk_name`, `disk_filesystem`, `disk_filetype`| Тип: датчик<br> Размер диска |
| `edgeAgent_used_memory_bytes` | `module_name` | Тип: датчик<br> Объем ОЗУ, используемый всеми процессами |
| `edgeAgent_total_memory_bytes` | `module_name` | Тип: датчик<br> Доступная память |
| `edgeAgent_used_cpu_percent` | `module_name` | Тип: гистограмма<br> Процент ЦП, используемый всеми процессами |
| `edgeAgent_created_pids_total` | `module_name` | Тип: датчик<br> Число процессов или потоков, созданных контейнером |
| `edgeAgent_total_network_in_bytes` | `module_name` | Тип: датчик<br> Число байтов, полученных из сети |
| `edgeAgent_total_network_out_bytes` | `module_name` | Тип: датчик<br> Число байтов, отправленных в сеть |
| `edgeAgent_total_disk_read_bytes` | `module_name` | Тип: датчик<br> Число байтов, считанных с диска |
| `edgeAgent_total_disk_write_bytes` | `module_name` | Тип: датчик<br> Число байтов, записанных на диск |
| `edgeAgent_metadata` | `edge_agent_version`, `experimental_features`, `host_information` | Тип: датчик<br> Общие метаданные об устройстве. Значение всегда равно 0, сведения кодируются в тегах. Обратите внимание `experimental_features` и `host_information` являются объектами JSON. `host_information` выглядит так ```{"OperatingSystemType": "linux", "Architecture": "x86_64", "Version": "1.0.10~dev20200803.4", "Provisioning": {"Type": "dps.tpm", "DynamicReprovisioning": false, "AlwaysReprovisionOnStartup": true}, "ServerVersion": "19.03.6", "KernelVersion": "5.0.0-25-generic", "OperatingSystem": "Ubuntu 18.04.4 LTS", "NumCpus": 6, "Virtualized": "yes"}``` :. Примечание `ServerVersion` . это версия DOCKER, а `Version` — версия управляющей программы безопасности IOT Edge. |

## <a name="next-steps"></a>Дальнейшие действия

* [Общие сведения о среде выполнения Azure IoT Edge и ее архитектуре](iot-edge-runtime.md)
* [Свойства двойников модулей агента IoT Edge и центра IoT Edge](module-edgeagent-edgehub.md)
