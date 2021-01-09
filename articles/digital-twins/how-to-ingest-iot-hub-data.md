---
title: Прием данных телеметрии из Центра Интернета вещей
titleSuffix: Azure Digital Twins
description: Узнайте, как получать сообщения телеметрии устройства из центра Интернета вещей.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 9/15/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 9ecc14aa9591d6e62dccd9899a80de44411928a1
ms.sourcegitcommit: 8dd8d2caeb38236f79fe5bfc6909cb1a8b609f4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/08/2021
ms.locfileid: "98051094"
---
# <a name="ingest-iot-hub-telemetry-into-azure-digital-twins"></a>Прием данных телеметрии центра Интернета вещей в Azure Digital двойников

Azure Digital двойников управляет данными из устройств IoT и других источников. Общий источник данных устройства для использования в цифровом двойников Azure — это [центр Интернета вещей](../iot-hub/about-iot-hub.md).

Процесс приема данных в Azure Digital двойников заключается в настройке внешнего ресурса вычислений, например функции, выполняемой с помощью [функций Azure](../azure-functions/functions-overview.md). Функция получает данные и использует [API дигиталтвинс](/rest/api/digital-twins/dataplane/twins) для задания свойств или запуска событий телеметрии в [цифровом двойников](concepts-twins-graph.md) соответственно. 

В этом документе описывается процесс создания функции, которая может принимать данные телеметрии из центра Интернета вещей.

## <a name="prerequisites"></a>Предварительные условия

Прежде чем продолжить работу с этим примером, необходимо настроить следующие ресурсы в качестве необходимых компонентов:
* **Центр Интернета вещей**. Инструкции см. в разделе " *Создание центра Интернета вещей* " [этого руководства центра Интернета вещей](../iot-hub/quickstart-send-telemetry-cli.md).
* **Функция** с правильными разрешениями для вызова вашего экземпляра Digital двойника. Инструкции см. в разделе [*инструкции. Настройка функции в Azure для обработки данных*](how-to-create-azure-function.md). 
* **Экземпляр цифрового двойников Azure** , который будет принимать данные телеметрии устройства. Инструкции см. [*в статье Настройка экземпляра и проверки подлинности Azure Digital двойников*](./how-to-set-up-instance-portal.md).

### <a name="example-telemetry-scenario"></a>Пример сценария телеметрии

В этой инструкции показано, как отправить сообщения из центра Интернета вещей в Azure Digital двойников с помощью функции в Azure. Существует множество возможных конфигураций и стратегий сопоставления, которые можно использовать для отправки сообщений, но пример для этой статьи содержит следующие компоненты.
* Устройство термометра в центре Интернета вещей с известным ИДЕНТИФИКАТОРом устройства
* Цифровой двойника для представления устройства с совпадающим ИДЕНТИФИКАТОРом

> [!NOTE]
> В этом примере используется понятный идентификатор, совпадающий с идентификатором устройства и соответствующим ИДЕНТИФИКАТОРом цифрового двойника, но можно предоставить более сложные сопоставления между устройством и его двойника (например, с таблицей сопоставлений).

Каждый раз, когда устройство термостата отправляет событие телеметрии температуры, функция обрабатывает данные телеметрии и свойство *температуры* цифрового двойника должно обновляться. Этот сценарий описан в схеме ниже:

:::image type="content" source="media/how-to-ingest-iot-hub-data/events.png" alt-text="Схема, показывающая блок-диаграмму. На диаграмме устройство центра Интернета вещей отправляет данные телеметрии температуры через центр Интернета вещей в функцию в Azure, которая обновляет свойство температуры двойника в Azure Digital двойников." border="false":::

## <a name="add-a-model-and-twin"></a>Добавление модели и двойника

Вы можете добавить или передать модель с помощью приведенной ниже команды CLI, а затем создать двойника с помощью этой модели, которая будет обновлена информацией из центра Интернета вещей.

Модель выглядит следующим образом:
:::code language="json" source="~/digital-twins-docs-samples/models/Thermostat.json":::

Чтобы **передать эту модель в экземпляр двойников**, откройте Azure CLI и выполните следующую команду:

```azurecli-interactive
az dt model create --models '{  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",  "@type": "Interface",  "@context": "dtmi:dtdl:context;2",  "contents": [    {      "@type": "Property",      "name": "Temperature",      "schema": "double"    }  ]}' -n {digital_twins_instance_name}
```

Затем необходимо **создать один двойника с помощью этой модели**. Используйте следующую команду, чтобы создать двойника и установить 0,0 в качестве начального значения температуры.

```azurecli-interactive
az dt twin create --dtmi "dtmi:contosocom:DigitalTwins:Thermostat;1" --twin-id thermostat67 --properties '{"Temperature": 0.0,}' --dt-name {digital_twins_instance_name}
```

Выходные данные успешной команды двойника Create должны выглядеть следующим образом:
```json
{
  "$dtId": "thermostat67",
  "$etag": "W/\"0000000-9735-4f41-98d5-90d68e673e15\"",
  "$metadata": {
    "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
    "Temperature": {
      "ackCode": 200,
      "ackDescription": "Auto-Sync",
      "ackVersion": 1,
      "desiredValue": 0.0,
      "desiredVersion": 1
    }
  },
  "Temperature": 0.0
}
```

## <a name="create-a-function"></a>Создание функции

В этом разделе используются те же шаги запуска Visual Studio и схема функций из [*руководства: Настройка функции для обработки данных*](how-to-create-azure-function.md). Скелет обрабатывает аутентификацию и создает клиент службы, готовый к обработке данных и вызов интерфейсов API цифровых двойников Azure в ответе. 

В следующих шагах вы добавите в него Специальный код для обработки событий телеметрии IoT из центра Интернета вещей.  

### <a name="add-telemetry-processing"></a>Добавление обработки телеметрии
    
События телеметрии поступают в виде сообщений с устройства. Первым шагом при добавлении кода обработки телеметрии является извлечение соответствующей части сообщения об устройстве из события сетки событий. 

Различные устройства могут структурировать свои сообщения по-разному, поэтому код для **этого шага зависит от подключенного устройства.** 

В следующем коде показан пример для простого устройства, отправляющего данные телеметрии в формате JSON. Этот пример полностью рассмотрен в руководстве по [*подключению комплексного решения*](./tutorial-end-to-end.md). Следующий код находит идентификатор устройства, отправившего сообщение, а также значение температуры.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/IoTHubToTwins.cs" id="Find_device_ID_and_temperature":::

Следующий пример кода принимает значение ID и температуру и использует их для установки исправления (внесения обновлений) в двойника.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/IoTHubToTwins.cs" id="Update_twin_with_device_temperature":::

### <a name="update-your-function-code"></a>Обновление кода функции

Теперь, когда вы понимаете код из предыдущих примеров, откройте функцию из раздела [*Предварительные требования*](#prerequisites) в Visual Studio. (Если у вас нет функции, созданной в Azure, перейдите по ссылке в предварительных требованиях, чтобы создать ее сейчас).

Замените код функции на этот пример кода.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/IoTHubToTwins.cs":::

Сохраните код функции и опубликуйте приложение функции в Azure. Дополнительные сведения см. в разделе [*Публикация приложения-функции*](./how-to-create-azure-function.md#publish-the-function-app-to-azure) в статье [*как настроить функцию в Azure для обработки данных*](how-to-create-azure-function.md).

После успешной публикации вы увидите выходные данные в окне командной строки Visual Studio, как показано ниже:

```cmd
1>------ Build started: Project: adtIngestFunctionSample, Configuration: Release Any CPU ------
1>adtIngestFunctionSample -> C:\Users\source\repos\Others\adtIngestFunctionSample\adtIngestFunctionSample\bin\Release\netcoreapp3.1\bin\adtIngestFunctionSample.dll
2>------ Publish started: Project: adtIngestFunctionSample, Configuration: Release Any CPU ------
2>adtIngestFunctionSample -> C:\Users\source\repos\Others\adtIngestFunctionSample\adtIngestFunctionSample\bin\Release\netcoreapp3.1\bin\adtIngestFunctionSample.dll
2>adtIngestFunctionSample -> C:\Users\source\repos\Others\adtIngestFunctionSample\adtIngestFunctionSample\obj\Release\netcoreapp3.1\PubTmp\Out\
2>Publishing C:\Users\source\repos\Others\adtIngestFunctionSample\adtIngestFunctionSample\obj\Release\netcoreapp3.1\PubTmp\adtIngestFunctionSample - 20200911112545669.zip to https://adtingestfunctionsample20200818134346.scm.azurewebsites.net/api/zipdeploy...
========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
========== Publish: 1 succeeded, 0 failed, 0 skipped ==========
```
Вы также можете проверить состояние процесса публикации в [портал Azure](https://portal.azure.com/). Найдите _группу ресурсов_ и перейдите к _журналу действий_ и найдите в списке пункт _получить профиль публикации веб-приложения_ и убедитесь, что состояние прошло.

:::image type="content" source="media/how-to-ingest-iot-hub-data/azure-function-publish-activity-log.png" alt-text="Снимок экрана портал Azure, отображающей состояние процесса публикации.":::

## <a name="connect-your-function-to-iot-hub"></a>Подключение функции к центру Интернета вещей

Настройте назначение события для данных концентратора.
В [портал Azure](https://portal.azure.com/)перейдите к экземпляру центра Интернета вещей, созданному в разделе [*Предварительные требования*](#prerequisites) . В разделе **события** создайте подписку для функции.

:::image type="content" source="media/how-to-ingest-iot-hub-data/add-event-subscription.png" alt-text="Снимок экрана портал Azure, который показывает добавление подписки на события.":::

На странице **Создание подписки на события** заполните поля следующим образом:
  1. В поле **имя** укажите имя подписки.
  2. В разделе **схема события** выберите _Схема сетки событий_.
  3. В разделе **типы событий** установите флажок _телеметрии устройства_ и снимите флажки для других типов событий.
  4. В разделе **тип конечной точки** выберите _функция Azure_.
  5. В разделе **Конечная точка** выберите ссылку _выбрать конечную точку_ , чтобы создать конечную точку.
    
:::image type="content" source="media/how-to-ingest-iot-hub-data/create-event-subscription.png" alt-text="Снимок экрана портал Azure для создания сведений о подписке на события":::

На открывшейся странице _Выбор функции Azure_ проверьте следующие сведения.
 1. **Подписка**. Ваша подписка Azure
 2. **Группа ресурсов**. Ваша группа ресурсов
 3. **Приложение функции**: имя приложения функции
 4. **Слот**: _Рабочая_
 5. **Функция**. Выберите функцию из раскрывающегося списка.

Сохраните сведения, нажав кнопку _подтвердить выбор_ .            
      
:::image type="content" source="media/how-to-ingest-iot-hub-data/select-azure-function.png" alt-text="Снимок экрана портал Azure для выбора функции.":::

Нажмите кнопку _создать_ , чтобы создать подписку на события.

## <a name="send-simulated-iot-data"></a>Отправка смоделированных данных IoT

Чтобы протестировать новую функцию, используйте симулятор устройств из руководства по [*подключению комплексного решения*](./tutorial-end-to-end.md). Этот учебник основан на примере проекта, написанного на языке C#. Пример кода расположен здесь: [Azure Digital двойников Samples](/samples/azure-samples/digital-twins-samples/digital-twins-samples). Вы будете использовать проект **девицесимулатор** в этом репозитории.

В сквозном руководстве выполните следующие действия.
1. [*Регистрация имитированного устройства в Центре Интернета вещей*](./tutorial-end-to-end.md#register-the-simulated-device-with-iot-hub)
2. [*Настройка и запуск имитации*](./tutorial-end-to-end.md#configure-and-run-the-simulation)

## <a name="validate-your-results"></a>Проверка результатов

При выполнении приведенного выше симулятора устройства значение температуры цифрового двойника будет изменено. В Azure CLI выполните следующую команду, чтобы увидеть значение температуры.

```azurecli-interactive
az dt twin query -q "select * from digitaltwins" -n {digital_twins_instance_name}
```

Выходные данные должны содержать значение температуры следующим образом:

```json
{
  "result": [
    {
      "$dtId": "thermostat67",
      "$etag": "W/\"0000000-1e83-4f7f-b448-524371f64691\"",
      "$metadata": {
        "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
        "Temperature": {
          "ackCode": 200,
          "ackDescription": "Auto-Sync",
          "ackVersion": 1,
          "desiredValue": 69.75806974934324,
          "desiredVersion": 1
        }
      },
      "Temperature": 69.75806974934324
    }
  ]
}
```

Чтобы увидеть изменение значения, повторно выполните команду запроса выше.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте о поступлении и исходящих данных в Azure Digital двойников:
* [*Основные понятия: интеграция с другими службами*](concepts-integration.md)
