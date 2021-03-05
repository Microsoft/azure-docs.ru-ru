---
title: Расширение IoT Central Azure с помощью настраиваемых правил и уведомлений | Документация Майкрософт
description: В качестве разработчика решения настройте IoT Centralное приложение для отправки уведомлений по электронной почте при прекращении отправки телеметрических данных устройством. Это решение использует Azure Stream Analytics, функции Azure и SendGrid.
author: TheJasonAndrew
ms.author: v-anjaso
ms.date: 02/09/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: mvc, devx-track-csharp
manager: philmea
ms.openlocfilehash: 6146676121bac0089d5f520d60a97d74567a32bc
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102179346"
---
# <a name="extend-azure-iot-central-with-custom-rules-using-stream-analytics-azure-functions-and-sendgrid"></a>Расширение возможностей Azure IoT Central с помощью настраиваемых правил с использованием Stream Analytics, Функций Azure и SendGrid

В этом пошаговом руководство показано, как разработчик решения, как расширить приложение IoT Central с помощью настраиваемых правил и уведомлений. В примере показано, как отправить уведомление оператору, когда устройство прекращает отправку данных телеметрии. Решение использует [Azure Stream Analytics](../../stream-analytics/index.yml) запрос, чтобы определить, когда устройство остановило отправку данных телеметрии. Задание Stream Analytics использует [функции Azure](../../azure-functions/index.yml) для отправки уведомлений по электронной почте с помощью [SendGrid](https://sendgrid.com/docs/for-developers/partners/microsoft-azure/).

В этом пошаговом руководство показано, как расширить IoT Central за пределами того, что он уже может сделать с помощью встроенных правил и действий.

В этом пошаговом руководство вы узнаете, как выполнять следующие задачи:

* Потоковая передача телеметрии из IoT Central приложения с помощью *непрерывного экспорта данных*.
* Создание Stream Analytics запроса, который определяет, когда устройство остановило отправку данных.
* Отправка уведомления по электронной почте с помощью функций Azure и SendGrid Services.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этом руководстве, вам потребуется активная подписка Azure.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

### <a name="iot-central-application"></a>Приложение IoT Central

Создайте приложение IoT Central на веб-сайте [диспетчера приложений Azure IOT Central](https://aka.ms/iotcentral) со следующими параметрами:

| Параметр | Значение |
| ------- | ----- |
| Ценовой план | Standard |
| Шаблон приложения | Аналитика в магазине — мониторинг условий |
| имя приложения; | Примите значение по умолчанию или выберите собственное имя. |
| URL-адрес | Примите значение по умолчанию или выберите собственный уникальный префикс URL-адреса. |
| Каталог | Клиент Azure Active Directory |
| Подписка Azure. | Ваша подписка Azure. |
| Регион | Ближайший регион |

В примерах и снимках экрана в этой статье используется **США** регион. Выберите расположение, близкое к вам, и убедитесь, что вы создали все ресурсы в том же регионе.

Этот шаблон приложения включает два смоделированных устройства термостата, которые отправляют данные телеметрии.

### <a name="resource-group"></a>Группа ресурсов

Используйте [портал Azure, чтобы создать группу ресурсов](https://portal.azure.com/#create/Microsoft.ResourceGroup) с именем **детектстоппеддевицес** , которая будет содержать другие создаваемые ресурсы. Создайте ресурсы Azure в том же расположении, что и приложение IoT Central.

### <a name="event-hubs-namespace"></a>пространство имен Центров событий;

Используйте [портал Azure, чтобы создать пространство имен концентраторов событий](https://portal.azure.com/#create/Microsoft.EventHub) со следующими параметрами:

| Параметр | Значение |
| ------- | ----- |
| Имя    | Выберите имя пространства имен |
| Ценовая категория | Basic |
| Подписка | Ваша подписка |
| Группа ресурсов | детектстоппеддевицес |
| Расположение | Восточная часть США |
| Единицы пропускной способности | 1 |

### <a name="stream-analytics-job"></a>Задание Stream Analytics

Используйте [портал Azure, чтобы создать задание Stream Analytics](https://portal.azure.com/#create/Microsoft.StreamAnalyticsJob)  со следующими параметрами:

| Параметр | Значение |
| ------- | ----- |
| Имя    | Выберите имя задания |
| Подписка | Ваша подписка |
| Группа ресурсов | детектстоппеддевицес |
| Расположение | Восточная часть США |
| Среда размещения | Cloud |
| Единицы потоковой передачи | 3 |

### <a name="function-app"></a>Приложение-функция

Используйте [портал Azure, чтобы создать приложение-функцию](https://portal.azure.com/#create/Microsoft.FunctionApp) со следующими параметрами:

| Параметр | Значение |
| ------- | ----- |
| Имя приложения    | Выберите имя приложения функции |
| Подписка | Ваша подписка |
| Группа ресурсов | детектстоппеддевицес |
| ОС | Windows |
| План размещения | План потребления |
| Расположение | Восточная часть США |
| Стек времени выполнения | .NET |
| Память | Создание |

### <a name="sendgrid-account-and-api-keys"></a>Учетная запись SendGrid и ключи API

Если у вас нет учетной записи Sendgrid, создайте [бесплатную учетную запись](https://app.sendgrid.com/) , прежде чем начинать работу.

1. На панели мониторинга Sendgrid в меню слева выберите **ключи API**.
1. Нажмите кнопку **создать ключ API.**
1. Назовите новый ключ API **азурефунктионакцесс.**
1. Щелкните **создать & представление**.

    :::image type="content" source="media/howto-create-custom-rules/sendgrid-api-keys.png" alt-text="Снимок экрана: ключ API создания SendGrid.":::

После этого вам будет предоставлен ключ API. Сохраните эту строку для последующего использования.

## <a name="create-an-event-hub"></a>Создание концентратора событий

Вы можете настроить IoT Central приложение для непрерывного экспорта данных телеметрии в концентратор событий. В этом разделе вы создадите концентратор событий для получения данных телеметрии из приложения IoT Central. Концентратор событий доставляет данные телеметрии в задание Stream Analytics для обработки.

1. В портал Azure перейдите к пространству имен концентраторов событий и выберите **+ концентратор событий**.
1. Назовите концентратор событий **централекспорт** и выберите **создать**.

Пространство имен концентраторов событий выглядит как на следующем снимке экрана: 

:::image type="content" source="media/howto-create-custom-rules/event-hubs-namespace.png" alt-text="Снимок экрана пространства имен концентраторов событий." border="false":::


## <a name="define-the-function"></a>Определение функции

Это решение использует приложение "функции Azure" для отправки уведомления по электронной почте, когда задание Stream Analytics обнаруживает остановленное устройство. Чтобы создать приложение функции, сделайте следующее:

1. В портал Azure перейдите к экземпляру **службы приложений** в группе ресурсов **детектстоппеддевицес** .
1. Выберите **+** , чтобы создать новую функцию.
1. Выберите **триггер HTTP**.
1. Нажмите **Добавить**.

    :::image type="content" source="media/howto-create-custom-rules/add-function.png" alt-text="Изображение функции триггера HTTP по умолчанию"::: 

## <a name="edit-code-for-http-trigger"></a>Изменение кода для триггера HTTP

На портале создается функция по умолчанию с именем **HttpTrigger1**:

:::image type="content" source="media/howto-create-custom-rules/default-function.png" alt-text="Снимок экрана функции изменения триггера HTTP.":::


1. Замените код C# следующим кодом:

    ```csharp
    #r "Newtonsoft.Json"
    #r "SendGrid"
    using System;
    using SendGrid.Helpers.Mail;
    using Microsoft.Azure.WebJobs.Host;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    public static SendGridMessage Run(HttpRequest req, ILogger log)
    {
        string requestBody = new StreamReader(req.Body).ReadToEnd();
        log.LogInformation(requestBody);
        var notifications = JsonConvert.DeserializeObject<IList<Notification>>(requestBody);

        SendGridMessage message = new SendGridMessage();
        message.Subject = "Contoso device notification";

        var content = "The following device(s) have stopped sending telemetry:<br/><br/><table><tr><th>Device ID</th><th>Time</th></tr>";
        foreach(var notification in notifications) {
            log.LogInformation($"No message received - Device: {notification.deviceid}, Time: {notification.time}");
            content += $"<tr><td>{notification.deviceid}</td><td>{notification.time}</td></tr>";
        }
        content += "</table>";
        message.AddContent("text/html", content);  

        return message;
    }

    public class Notification
    {
        public string deviceid { get; set; }
        public string time { get; set; }
    }
    ```

    Вы можете увидеть сообщение об ошибке, пока не будет сохранен новый код.
1. Нажмите кнопку **сохранить** , чтобы сохранить функцию.

## <a name="add-sendgrid-key"></a>Добавить ключ SendGrid

Чтобы добавить ключ API SendGrid, необходимо добавить его в **ключи функции** следующим образом:

1. Выберите **функции ключи**.
1. Выберите **+ создать функциональную клавишу**.
1. Введите *имя* и *значение* созданного ранее ключа API.
1. Нажмите кнопку **ОК.**

    :::image type="content" source="media/howto-create-custom-rules/add-key.png" alt-text="Снимок экрана: Добавление ключа Сангрид.":::


## <a name="configure-httptrigger-function-to-use-sendgrid"></a>Настройка функции HttpTrigger для использования SendGrid

Чтобы отправить сообщения электронной почты с помощью SendGrid, необходимо настроить привязки для функции следующим образом:

1. Выберите **Интеграция**.
1. Выберите **Добавить выходные данные** в разделе **http ($Return)**.
1. Выберите **Удалить.**
1. Выберите **+ новый выход**.
1. В качестве типа привязки выберите **SendGrid**.
1. Для параметра тип настройки ключа API SendGrid нажмите кнопку Создать.
1. Введите *имя* и *значение* ключа API SendGrid.
1. Добавьте следующие данные:

| Параметр | Значение |
| ------- | ----- |
| Имя параметра сообщения | Выберите свое имя |
| Адрес | Выберите имя для адреса |
| Адрес отправителя | Выберите имя из адреса |
| Тема сообщения | Введите заголовок темы |
| Текст сообщения | Введите сообщение из интеграции |

1. Щелкните **ОК**.

    :::image type="content" source="media/howto-create-custom-rules/add-output.png" alt-text="Снимок экрана: Добавление выходных данных Сандгрид.":::


### <a name="test-the-function-works"></a>Проверка работы функции

Чтобы проверить функцию на портале, сначала выберите **журналы** в нижней части редактора кода. Затем выберите **тест** справа от редактора кода. Используйте следующий код JSON в качестве **текста запроса**:

```json
[{"deviceid":"test-device-1","time":"2019-05-02T14:23:39.527Z"},{"deviceid":"test-device-2","time":"2019-05-02T14:23:50.717Z"},{"deviceid":"test-device-3","time":"2019-05-02T14:24:28.919Z"}]
```

Сообщения журнала функций отображаются на панели **журналы** :

:::image type="content" source="media/howto-create-custom-rules/function-app-logs.png" alt-text="Выходные данные журнала функций":::

Через несколько минут адрес электронной почты будет получать сообщение **электронной почты со** следующим содержимым:

```txt
The following device(s) have stopped sending telemetry:

Device ID    Time
test-device-1    2019-05-02T14:23:39.527Z
test-device-2    2019-05-02T14:23:50.717Z
test-device-3    2019-05-02T14:24:28.919Z
```

## <a name="add-stream-analytics-query"></a>Добавление Stream Analytics запроса

Это решение использует Stream Analytics запрос для обнаружения того, когда устройство прекращает отправку данных телеметрии более 120 секунд. В качестве входных данных в запросе используются данные телеметрии из концентратора событий. Задание отправляет результаты запроса в приложение функции. В этом разделе вы настроите задание Stream Analytics:

1. В портал Azure перейдите к заданию Stream Analytics в разделе **топология заданий** выберите **входные данные**, щелкните **+ добавить потоковый вход** и выберите **концентратор событий**.
1. Используйте сведения в следующей таблице для настройки входных данных с помощью созданного ранее концентратора событий, а затем выберите **сохранить**:

    | Параметр | Значение |
    | ------- | ----- |
    | Псевдоним входных данных | централтелеметри |
    | Подписка | Ваша подписка |
    | пространство имен концентратора событий; | Пространство имен концентратора событий |
    | имя концентратора событий; | Использовать existing- **централекспорт** |

1. В разделе **топология заданий** выберите **выходные данные**, нажмите кнопку **+ Добавить**, а затем выберите **функция Azure**.
1. Используйте сведения в следующей таблице, чтобы настроить выходные данные, а затем выберите **сохранить**:

    | Параметр | Значение |
    | ------- | ----- |
    | Псевдоним выходных данных | емаилнотификатион |
    | Подписка | Ваша подписка |
    | Приложение-функция | Приложение функции |
    | Компонент  | HttpTrigger1 |

1. В разделе **топология заданий** выберите **запрос** и замените существующий запрос на следующий SQL:

    ```sql
    with
    LeftSide as
    (
        SELECT
        -- Get the device ID from the message metadata and create a column
        GetMetadataPropertyValue([centraltelemetry], '[EventHub].[IoTConnectionDeviceId]') as deviceid1, 
        EventEnqueuedUtcTime AS time1
        FROM
        -- Use the event enqueued time for time-based operations
        [centraltelemetry] TIMESTAMP BY EventEnqueuedUtcTime
    ),
    RightSide as
    (
        SELECT
        -- Get the device ID from the message metadata and create a column
        GetMetadataPropertyValue([centraltelemetry], '[EventHub].[IoTConnectionDeviceId]') as deviceid2, 
        EventEnqueuedUtcTime AS time2
        FROM
        -- Use the event enqueued time for time-based operations
        [centraltelemetry] TIMESTAMP BY EventEnqueuedUtcTime
    )

    SELECT
        LeftSide.deviceid1 as deviceid,
        LeftSide.time1 as time
    INTO
        [emailnotification]
    FROM
        LeftSide
        LEFT OUTER JOIN
        RightSide 
        ON
        LeftSide.deviceid1=RightSide.deviceid2 AND DATEDIFF(second,LeftSide,RightSide) BETWEEN 1 AND 120
        where
        -- Find records where a device didn't send a message 120 seconds
        RightSide.deviceid2 is NULL
    ```

1. Щелкните **Сохранить**.
1. Чтобы запустить задание Stream Analytics, выберите **Обзор**, затем **запустите**, **а затем** **запустите**:

    :::image type="content" source="media/howto-create-custom-rules/stream-analytics.png" alt-text="Снимок экрана Stream Analytics.":::

## <a name="configure-export-in-iot-central"></a>Настройка экспорта в IoT Central 

На веб-сайте [Azure IOT Central Application Manager](https://aka.ms/iotcentral) перейдите к созданному IOT Central приложении.

В этом разделе вы настроите приложение для потоковой передачи данных телеметрии из виртуальных устройств в концентратор событий. Чтобы настроить экспорт, выполните следующие действия.

1. Перейдите на страницу **Экспорт данных** , выберите **+ создать**, а затем **концентраторы событий Azure**.
1. Используйте следующие параметры для настройки экспорта, а затем выберите **сохранить**: 

    | Параметр | Значение |
    | ------- | ----- |
    | Отображаемое имя | Экспорт в концентраторы событий |
    | Активировано | С |
    | Тип данных для экспорта | Телеметрия |
    | Усовершенствования | Введите требуемый ключ и значение, как должны упорядочиваться экспортированные данные | 
    | Назначение | Создайте новый и введите данные для экспорта данных. |

    :::image type="content" source="media/howto-create-custom-rules/cde-configuration.png" alt-text="Снимок экрана экспорта данных.":::

Прежде чем продолжить, подождите, пока состояние экспорта будет **выполняться** .

## <a name="test"></a>Тест

Чтобы протестировать решение, можно отключить экспорт непрерывных данных из IoT Central в имитацию остановленных устройств:

1. В приложении IoT Central перейдите на страницу **Export Data (экспорт данных** ) и выберите **Экспорт в конфигурацию экспорта концентраторов событий** .
1. Установите для параметра **включено** значение **выкл** и нажмите кнопку **сохранить**.
1. По истечении двух минут адрес **электронной почты будет** принимать одно или несколько сообщений, которые выглядят как в следующем примере:

    ```txt
    The following device(s) have stopped sending telemetry:

    Device ID         Time
    Thermostat-Zone1  2019-11-01T12:45:14.686Z
    ```

## <a name="tidy-up"></a>Удаление ненужных элементов

Чтобы устранить эту работу и избежать ненужных затрат, удалите группу ресурсов **детектстоппеддевицес** в портал Azure.

Вы можете удалить IoT Centralное приложение со страницы **Управление** в приложении.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали следующее:

* Потоковая передача телеметрии из IoT Central приложения с помощью *непрерывного экспорта данных*.
* Создание Stream Analytics запроса, который определяет, когда устройство остановило отправку данных.
* Отправка уведомления по электронной почте с помощью функций Azure и SendGrid Services.

Теперь, когда вы узнали, как создавать настраиваемые правила и уведомления, предлагаем следующий шаг: Узнайте, как [расширить Azure IOT Central с помощью пользовательской аналитики](howto-create-custom-analytics.md).
