---
title: Использование команд устройств в решении Azure IoT Central
description: Как использовать команды устройства в решении IoT Central Azure. В этом руководстве показано, как разработчику устройства использовать команды устройства в клиентском приложении для приложения IoT Central Azure.
author: dominicbetts
ms.author: dobett
ms.date: 01/07/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: e53bf377a7ef8f2293debd288ba25ef8f04ff4fc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98611004"
---
# <a name="how-to-use-commands-in-an-azure-iot-central-solution"></a>Использование команд в решении IoT Central Azure

В этом пошаговом руководство показано, как разработчику устройства следует использовать команды, определенные в шаблоне устройства.

Оператор может использовать пользовательский интерфейс IoT Central для вызова команды на устройстве. Команды управляют поведением устройства. Например, оператор может вызвать команду для перезагрузки устройства или для получения данных диагностики.

Устройство может:

* Немедленное реагирование на команду.
* Ответьте на IoT Central при получении команды и последующем уведомлении IoT Central о завершении *длительной команды* .

По умолчанию команды предполагают подключение устройства и завершаются ошибкой, если устройство недоступно. Если в пользовательском интерфейсе шаблона устройства выбран параметр **очередь при автономной работе** , команда может быть поставлена в очередь, пока устройство не будет подключено к сети. Эти *автономные команды* описаны в отдельном разделе далее в этой статье.

## <a name="define-your-commands"></a>Определение команд

Стандартные команды отправляются на устройство, чтобы сообщить устройству о необходимости выполнить какие-либо действия. Команда может включать параметры с дополнительными сведениями. Например, команда для открытия клапана на устройстве может иметь параметр, указывающий, какой объем открыть клапан. Команды также могут получать возвращаемое значение, когда устройство завершает команду. Например, команда, запрашивающая устройство для запуска некоторой диагностики, может получить отчет о диагностике в качестве возвращаемого значения.

Команды определяются как часть шаблона устройства. На следующем снимке экрана показано определение команды **отчета Get Max-Min** в шаблоне устройства **термостата** . У этой команды есть параметры запроса и ответа: 

:::image type="content" source="media/howto-use-commands/command-definition.png" alt-text="Снимок экрана: команда Get max min Report в шаблоне устройства термостата":::

В следующей таблице показаны параметры конфигурации для возможности "Команда".

| Поле             |Описание|
|-------------------|-----------|
|Отображаемое имя       |Значение команды, используемое на панелях мониторинга и формах.|
| Имя            | Имя команды. IoT Central создает значение для этого поля на основе отображаемого имени, но при необходимости можно выбрать собственное значение. Это поле должно быть буквенно-цифровым. Код устройства использует это значение **имени** .|
| Тип возможности | Команда|
| Поставить в очередь в автономном режиме | Следует ли сделать эту команду *автономной* командой. |
| Описание     | Описание возможности "Команда".|
| Комментировать     | Любые комментарии о возможности "Команда".|
| Запрос     | Полезная нагрузка для команды устройства.|
| Ответ     | Полезная нагрузка командного ответа устройства.|

В следующем фрагменте кода показано представление JSON команды в модели устройства. В этом примере значение ответа является сложным типом **объекта** с несколькими полями:

```json
{
  "@type": "Command",
  "name": "getMaxMinReport",
  "displayName": "Get Max-Min report.",
  "description": "This command returns the max, min and average temperature from the specified time to the current time.",
  "request": {
    "name": "since",
    "displayName": "Since",
    "description": "Period to return the max-min report.",
    "schema": "dateTime"
  },
  "response": {
    "name" : "tempReport",
    "displayName": "Temperature Report",
    "schema": {
      "@type": "Object",
      "fields": [
        {
          "name": "maxTemp",
          "displayName": "Max temperature",
          "schema": "double"
        },
        {
          "name": "minTemp",
          "displayName": "Min temperature",
          "schema": "double"
        },
        {
          "name" : "avgTemp",
          "displayName": "Average Temperature",
          "schema": "double"
        },
        {
          "name" : "startTime",
          "displayName": "Start Time",
          "schema": "dateTime"
        },
        {
          "name" : "endTime",
          "displayName": "End Time",
          "schema": "dateTime"
        }
      ]
    }
  }
}
```

> [!TIP]
> Модель устройства можно экспортировать на странице шаблона устройства.

Вы можете связать это определение команды с снимком экрана пользовательского интерфейса, используя следующие поля:

* `@type` чтобы указать тип возможности, сделайте следующее: `Command`
* `name` для значения команды.

Необязательные поля, такие как отображаемое имя и описание, позволяют добавлять дополнительные сведения в интерфейс и возможности.

## <a name="standard-commands"></a>Стандартные команды

В этом разделе показано, как устройство отправляет значение ответа сразу после получения команды.

В следующем фрагменте кода показано, как устройство может ответить на команду, немедленно отправив код успешного выполнения:

> [!NOTE]
> В этой статье для простоты используется Node.js. Примеры для других языков см. в руководстве по [созданию и подключению клиентского приложения к IOT Centralному приложению Azure](tutorial-connect-device.md) .

```javascript
client.onDeviceMethod('getMaxMinReport', commandHandler);

// ...

const commandHandler = async (request, response) => {
  switch (request.methodName) {
  case 'getMaxMinReport': {
    console.log('MaxMinReport ' + request.payload);
    await sendCommandResponse(request, response, 200, deviceTemperatureSensor.getMaxMinReportObject());
    break;
  }
  default:
    await sendCommandResponse(request, response, 404, 'unknown method');
    break;
  }
};

const sendCommandResponse = async (request, response, status, payload) => {
  try {
    await response.send(status, payload);
    console.log('Response to method \'' + request.methodName +
              '\' sent successfully.' );
  } catch (err) {
    console.error('An error ocurred when sending a method response:\n' +
              err.toString());
  }
};
```

Вызов для `onDeviceMethod` установки `commandHandler` метода. Этот обработчик команд:

1. Проверяет имя команды.
1. Для `getMaxMinReport` команды вызывает метод, `getMaxMinReportObject` чтобы получить значения для включения в возвращаемый объект.
1. Вызывает метод `sendCommandResponse` для отправки ответа обратно в IOT Central. Этот ответ включает `200` код ответа, указывающий на успешное выполнение.

На следующем снимке экрана показано, как в пользовательском интерфейсе IoT Central отображается успешный ответ команды:

:::image type="content" source="media/howto-use-commands/simple-command-ui.png" alt-text="Снимок экрана, показывающий, как просмотреть полезные данные команды для стандартной команды":::

## <a name="long-running-commands"></a>Долго выполняющиеся команды

В этом разделе показано, как устройство может отложить отправку подтверждения о завершении команды.

В следующем фрагменте кода показано, как устройство может реализовать долго выполняющуюся команду:

> [!NOTE]
> В этой статье для простоты используется Node.js. Примеры для других языков см. в руководстве по [созданию и подключению клиентского приложения к IOT Centralному приложению Azure](tutorial-connect-device.md) .

```javascript
client.onDeviceMethod('rundiagnostics', commandHandler);

// ...

const commandHandler = async (request, response) => {
  switch (request.methodName) {
  case 'rundiagnostics': {
    console.log('Starting long-running diagnostics run ' + request.payload);
    await sendCommandResponse(request, response, 202, 'Diagnostics run started');

    // Long-running operation here
    // ...

    const patch = {
      rundiagnostics: {
        value: 'Diagnostics run complete at ' + new Date().toLocaleString()
      }
    };

    deviceTwin.properties.reported.update(patch, function (err) {
      if (err) throw err;
      console.log('Properties have been reported for component');
    });
    break;
  }
  default:
    await sendCommandResponse(request, response, 404, 'unknown method');
    break;
  }
};
```

Вызов для `onDeviceMethod` установки `commandHandler` метода. Этот обработчик команд:

1. Проверяет имя команды.
1. Вызывает метод `sendCommandResponse` для отправки ответа обратно в IOT Central. Этот ответ включает `202` код ответа для указания ожидающих результатов.
1. Завершает выполнение длительной операции.
1. Использует сообщаемое свойство с тем же именем, что и команда, чтобы сообщить IoT Central о завершении команды.

На следующем снимке экрана показано, как ответ команды отображается в пользовательском интерфейсе IoT Central при получении кода ответа 202:

:::image type="content" source="media/howto-use-commands/long-running-start.png" alt-text="Снимок экрана, показывающий немедленный ответ от устройства":::

На следующем снимке экрана показан пользовательский интерфейс IoT Central при получении обновления свойства, указывающего на завершение команды:

:::image type="content" source="media/howto-use-commands/long-running-finish.png" alt-text="Снимок экрана, на котором отображается длинная команда завершена":::

## <a name="offline-commands"></a>Автономные команды

В этом разделе показано, как устройство обрабатывает команду в режиме «вне сети». Если устройство находится в оперативном режиме, оно может работать с автономной командой, как только оно будет получено. Если устройство находится в автономном режиме, при следующем подключении к IoT Central обрабатывается команда offline. Устройства не могут отправить возвращаемое значение в ответ на команду offline.

> [!NOTE]
> В этой статье для простоты используется Node.js.

На следующем снимке экрана показана автономная команда с именем **женератедиагностикс**. Параметр запроса — это объект со свойством DateTime с именем **StartTime** и целочисленным свойством перечисления под названием **Bank**:

:::image type="content" source="media/howto-use-commands/offline-command.png" alt-text="Снимок экрана, показывающий пользовательский интерфейс для автономной команды":::

В следующем фрагменте кода показано, как клиент может прослушивать автономные команды и отображать содержимое сообщения:

```javascript
client.on('message', function (msg) {
  console.log('Body: ' + msg.data);
  console.log('Properties: ' + JSON.stringify(msg.properties));
  client.complete(msg, function (err) {
    if (err) {
      console.error('complete error: ' + err.toString());
    } else {
      console.log('complete sent');
    }
  });
});
```

Выходные данные из предыдущего фрагмента кода показывают полезные данные со значениями **StartTime** и **Bank** . Список свойств содержит имя команды в элементе списка **имя метода** :

```output
Body: {"StartTime":"2021-01-06T06:00:00.000Z","Bank":2}
Properties: {"propertyList":[{"key":"iothub-ack","value":"none"},{"key":"method-name","value":"GenerateDiagnostics"}]}
```

> [!NOTE]
> По умолчанию срок жизни для команд offline составляет 24 часа, после чего срок действия сообщения истекает.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как использовать команды в приложении IoT Central Azure, ознакомьтесь с [полезными данными](concepts-telemetry-properties-commands.md) , чтобы узнать больше о параметрах команды, а также о [создании и подключении клиентского приложения к приложению IOT Central Azure](tutorial-connect-device.md) для просмотра полных примеров кода на разных языках.
