---
title: Получение журналов IoT Edge — Azure IoT Edge
description: IoT Edge извлечение и отправку журнала модуля в хранилище BLOB-объектов Azure.
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 11/12/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
ms.custom: devx-track-azurecli
services: iot-edge
ms.openlocfilehash: c06120d1a2e8aa6aa0c006c6f40fed6fab44c5b7
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103200694"
---
# <a name="retrieve-logs-from-iot-edge-deployments"></a>Получение журналов из IoT Edge развертываний

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Получение журналов из IoT Edge развертываний без необходимости физического или SSH-доступа к устройству с помощью прямых методов, входящих в модуль агента IoT Edge. Прямые методы реализуются на устройстве, а затем могут быть вызваны из облака. Агент IoT Edge содержит прямые методы, помогающие удаленно отслеживать IoT Edge устройства и управлять ими. Прямые методы, описанные в этой статье, общедоступны в выпуске 1.0.10.

Дополнительные сведения о прямых методах, способах их использования и их реализации в собственных модулях см. в разделе [изучение и вызов прямых методов из центра Интернета вещей](../iot-hub/iot-hub-devguide-direct-methods.md).

Имена этих прямых методов обрабатываются с учетом регистра.

## <a name="recommended-logging-format"></a>Рекомендуемый формат ведения журнала

Хотя это и не является обязательным, для обеспечения лучшей совместимости с этой функцией рекомендуется использовать следующий формат ведения журнала:

```
<{Log Level}> {Timestamp} {Message Text}
```

`{Timestamp}` следует отформатировать как `yyyy-MM-dd hh:mm:ss.fff zzz` и `{Log Level}` следовать таблице ниже, которая извлекает уровни серьезности из [кода серьезности в стандарте системного журнала](https://wikipedia.org/wiki/Syslog#Severity_level).

| Значение | Severity |
|-|-|
| 0 | Аварийный режим |
| 1 | Оповещение |
| 2 | Critical |
| 3 | Error |
| 4 | Предупреждение |
| 5 | Уведомление |
| 6 | Informational |
| 7 | Отладка |

[Класс Logger в IOT Edge](https://github.com/Azure/iotedge/blob/master/edge-util/src/Microsoft.Azure.Devices.Edge.Util/Logger.cs) выступает в качестве канонической реализации.

## <a name="retrieve-module-logs"></a>Получение журналов модулей

Используйте прямой метод **жетмодулелогс** для получения журналов модуля IOT Edge.

Этот метод принимает полезные данные JSON со следующей схемой:

```json
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "regex string",
             "filter": {
                "tail": "int",
                "since": "string",
                "until": "string",
                "loglevel": "int",
                "regex": "regex string"
             }
          }
       ],
       "encoding": "gzip/none",
       "contentType": "json/text" 
    }
```

| Название | Тип | Описание |
|-|-|-|
| schemaVersion | строка | Установите значение `1.0` |
| items | Массив JSON | Массив с `id` `filter` кортежами и. |
| ID | строка | Регулярное выражение, предоставляющее имя модуля. Он может сопоставлять несколько модулей на пограничном устройстве. Ожидается формат [регулярных выражений .NET](/dotnet/standard/base-types/regular-expressions) . |
| фильтр | Раздел JSON | Фильтры журналов, применяемые к модулям, соответствующим `id` регулярному выражению в кортеже. |
| односторонне | Целое число | Число строк журнала в прошлом для получения, начиная с последнего. OPTIONAL. |
| since | строка | Возвращать только журналы, начиная с этого времени, как продолжительность (1 d, 90 м, 2 дня 3 часа, 2 минуты), rfc3339 отметка времени или отметка времени UNIX.  Если `tail` указаны и `since` , и, журналы извлекаются с использованием `since` значения First. Затем `tail` значение применяется к результату, и возвращается окончательный результат. OPTIONAL. |
| until | строка | Возвращать журналы только до указанного времени, в виде метки времени rfc3339, метки времени UNIX или длительности (1 d, 90 м, 2 дня 3 часа 2 минуты). OPTIONAL. |
| уровень ведения журнала | Целое число | Фильтровать строки журнала, которые меньше или равны указанному уровню ведения журнала. Строки журнала должны следовать рекомендуемому формату ведения журнала и использовать стандарт [уровня серьезности syslog](https://en.wikipedia.org/wiki/Syslog#Severity_level) . OPTIONAL. |
| regex | строка | Фильтрация строк журнала, имеющих содержимое, совпадающее с указанным регулярным выражением, с использованием формата [регулярных выражений .NET](/dotnet/standard/base-types/regular-expressions) . OPTIONAL. |
| encoding | строка | `gzip` или `none`. По умолчанию — `none`. |
| сontentType | строка | `json` или `text`. По умолчанию — `text`. |

> [!NOTE]
> Если содержимое журнала превышает предельное значение размера ответа для прямых методов, которое в настоящее время 128 КБ, ответ возвращает ошибку.

Успешный выбор журналов возвращает **"Status": 200** , за которыми следуют полезные данные, содержащие журналы, полученные из модуля, отфильтрованные по параметрам, указанным в запросе.

Например:

```azurecli
az iot hub invoke-module-method --method-name 'GetModuleLogs' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
'
```

В портал Azure вызовите метод с именем метода `GetModuleLogs` и следующими полезными данными JSON:

```json
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
```

![Вызов прямого метода "Жетмодулелогс" в портал Azure](./media/how-to-retrieve-iot-edge-logs/invoke-get-module-logs.png)

Вы также можете передать выходные данные CLI в служебные программы Linux, например [gzip](https://en.wikipedia.org/wiki/Gzip), для обработки сжатого ответа. Например:

```azurecli
az iot hub invoke-module-method \
  --method-name 'GetModuleLogs' \
  -n <hub name> \
  -d <device id> \
  -m '$edgeAgent' \
  --method-payload '{"contentType": "text","schemaVersion": "1.0","encoding": "gzip","items": [{"id": "edgeHub","filter": {"since": "2d","tail": 1000}}],}' \
  -o tsv --query 'payload[0].payloadBytes' \
  | base64 --decode \
  | gzip -d
```

## <a name="upload-module-logs"></a>Отправка журналов модулей

Используйте прямой метод **уплоадмодулелогс** , чтобы отправить запрошенные журналы в указанный контейнер хранилища BLOB-объектов Azure.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

> [!NOTE]
> Если вы хотите передать журналы с устройства, расположенного за устройством шлюза, вам потребуется настроить [прокси API и модули хранилища BLOB-объектов](how-to-configure-api-proxy-module.md) , настроенные на устройстве верхнего уровня. Эти модули направляют журналы с устройства более низкого уровня через устройство шлюза в хранилище в облаке.

::: moniker-end

Этот метод принимает полезные данные JSON, аналогичные **жетмодулелогс**, с добавлением ключа "sasUrl":

```json
    {
       "schemaVersion": "1.0",
       "sasUrl": "Full path to SAS URL",
       "items": [
          {
             "id": "regex string",
             "filter": {
                "tail": "int",
                "since": "string",
                "until": "string",
                "loglevel": "int",
                "regex": "regex string"
             }
          }
       ],
       "encoding": "gzip/none",
       "contentType": "json/text" 
    }
```

| Название | Тип | Описание |
|-|-|-|
| sasURL | строка (URI) | [URL-адрес подписи общего доступа с доступом на запись к контейнеру хранилища BLOB-объектов Azure](/archive/blogs/jpsanders/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer). |

Успешный запрос на отправку журналов возвращает **"Status": 200** , за которым следуют полезные данные со следующей схемой:

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Название | Тип | Описание |
|-|-|-|
| status | строка | Один из `NotStarted` ,,, `Running` `Completed` `Failed` или `Unknown` . |
| message | строка | Сообщение, если ошибка, пустая строка в противном случае. |
| correlationId | строка   | Идентификатор для запроса состояния запроса на отправку. |

Например:

Следующий вызов передает последние строки журнала 100 из всех модулей в сжатом формате JSON:

```azurecli
az iot hub invoke-module-method --method-name UploadModuleLogs -n <hub name> -d <device id> -m \$edgeAgent --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "<sasUrl>",
        "items": [
            {
                "id": ".*",
                "filter": {
                    "tail": 100
                }
            }
        ],
        "encoding": "gzip",
        "contentType": "json"
    }
'
```

Следующий вызов передает последние 100 строк журнала из edgeAgent и edgeHub с последними 1000 строками из модуля tempSensor в несжатом текстовом формате:

```azurecli
az iot hub invoke-module-method --method-name UploadModuleLogs -n <hub name> -d <device id> -m \$edgeAgent --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "<sasUrl>",
        "items": [
            {
                "id": "edge",
                "filter": {
                    "tail": 100
                }
            },
            {
                "id": "tempSensor",
                "filter": {
                    "tail": 1000
                }
            }
        ],
        "encoding": "none",
        "contentType": "text"
    }
'
```

В портал Azure вызовите метод с именем метода `UploadModuleLogs` и следующими полезными данными JSON после заполнения sasURL данными:

```json
    {
       "schemaVersion": "1.0",
       "sasUrl": "<sasUrl>",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
```

![Вызов прямого метода "Уплоадмодулелогс" в портал Azure](./media/how-to-retrieve-iot-edge-logs/invoke-upload-module-logs.png)

## <a name="upload-support-bundle-diagnostics"></a>Отправка диагностики пакета поддержки

Используйте прямой метод **уплоадсуппортбундле** для объединения и передачи ZIP-файла IOT Edge журналов модулей в доступный контейнер хранилища BLOB-объектов Azure. Этот прямой метод запускает [`iotedge support-bundle`](./troubleshoot.md#gather-debug-information-with-support-bundle-command) команду на устройстве IOT Edge для получения журналов.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

> [!NOTE]
> Если вы хотите передать журналы с устройства, расположенного за устройством шлюза, вам потребуется настроить [прокси API и модули хранилища BLOB-объектов](how-to-configure-api-proxy-module.md) , настроенные на устройстве верхнего уровня. Эти модули направляют журналы с устройства более низкого уровня через устройство шлюза в хранилище в облаке.

::: moniker-end

Этот метод принимает полезные данные JSON со следующей схемой:

```json
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
```

| Название | Тип | Описание |
|-|-|-|
| schemaVersion | строка | Установите значение `1.0` |
| sasURL | строка (URI) | [URL-адрес подписи общего доступа с доступом на запись к контейнеру хранилища BLOB-объектов Azure](/archive/blogs/jpsanders/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer) |
| since | строка | Возвращать только журналы, начиная с этого времени, как продолжительность (1 d, 90 м, 2 дня 3 часа, 2 минуты), rfc3339 отметка времени или отметка времени UNIX. OPTIONAL. |
| until | строка | Возвращать журналы только до указанного времени, в виде метки времени rfc3339, метки времени UNIX или длительности (1 d, 90 м, 2 дня 3 часа 2 минуты). OPTIONAL. |
| еджерунтимеонли | Логическое | Если значение — true, возвращаются только журналы из агента ребра, пограничной концентратора и управляющей программы пограничной системы безопасности. По умолчанию: false.  OPTIONAL. |

> [!IMPORTANT]
> Пакет поддержки IoT Edge может содержать персональные данные.

Успешный запрос на отправку журналов возвращает **"Status": 200** , за которым следуют полезные данные с той же схемой, что и ответ **уплоадмодулелогс** :

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Название | Тип | Описание |
|-|-|-|
| status | строка | Один из `NotStarted` ,,, `Running` `Completed` `Failed` или `Unknown` . |
| message | строка | Сообщение, если ошибка, пустая строка в противном случае. |
| correlationId | строка   | Идентификатор для запроса состояния запроса на отправку. |

Например:

```azurecli
az iot hub invoke-module-method --method-name 'UploadSupportBundle' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
'
```

В портал Azure вызовите метод с именем метода `UploadSupportBundle` и следующими полезными данными JSON после заполнения sasURL данными:

```json
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
```

![Вызов прямого метода "Уплоадсуппортбундле" в портал Azure](./media/how-to-retrieve-iot-edge-logs/invoke-upload-support-bundle.png)

## <a name="get-upload-request-status"></a>Получение состояния запроса на отправку

Используйте прямой метод **жеттаскстатус** для запроса состояния запроса на отправку журналов. Полезные данные запроса **жеттаскстатус** используют объект `correlationId` из запроса на отправку журналов для получения состояния задачи. `correlationId`Возвращается в ответ на вызов прямого метода **уплоадмодулелогс** .

Этот метод принимает полезные данные JSON со следующей схемой:

```json
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
```

Успешный запрос на отправку журналов возвращает **"Status": 200** , за которым следуют полезные данные с той же схемой, что и ответ **уплоадмодулелогс** :

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Название | Тип | Описание |
|-|-|-|
| status | строка | Один из `NotStarted` ,,, `Running` `Completed` `Failed` или `Unknown` . |
| message | строка | Сообщение, если ошибка, пустая строка в противном случае. |
| correlationId | строка   | Идентификатор для запроса состояния запроса на отправку. |

Например:

```azurecli
az iot hub invoke-module-method --method-name 'GetTaskStatus' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
'
```

В портал Azure вызовите метод с именем метода `GetTaskStatus` и следующими полезными данными JSON после заполнения идентификатора GUID сведениями:

```json
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
```

![Вызов прямого метода "Жеттаскстатус" в портал Azure](./media/how-to-retrieve-iot-edge-logs/invoke-get-task-status.png)

## <a name="next-steps"></a>Дальнейшие действия

[Свойства двойников модулей агента IoT Edge и центра IoT Edge](module-edgeagent-edgehub.md)
