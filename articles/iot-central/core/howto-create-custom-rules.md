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
ms.openlocfilehash: 824308b66803d2dfa05383ff06ce97c48626619d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102179346"
---
# <a name="extend-azure-iot-central-with-custom-rules-using-stream-analytics-azure-functions-and-sendgrid"></a>Расширение возможностей Azure IoT Central с помощью настраиваемых правил с использованием Stream Analytics, Функций Azure и SendGrid

В этом пошаговом руководство показано, как разработчик решения, как расширить приложение IoT Central с помощью настраиваемых правил и уведомлений. В примере показано, как отправить уведомление оператору, когда устройство прекращает отправку данных телеметрии. Решение использует [Azure Stream Analytics](../../stream-analytics/index.yml) запрос, чтобы определить, когда устройство остановило отправку данных телеметрии. Задание Stream Analytics использует [функции Azure](../../azure-functions/index.yml) для отправки уведомлений по электронной почте с помощью [SendGrid](https://sendgrid.com/docs/for-developers/partners/microsoft-azure/).

В этом пошаговом руководство показано, как расширить IoT Central за пределами того, что он уже может сделать с помощью встроенных правил и действий.

В этом пошаговом руководство вы узнаете, как выполнять следующие задачи:

* Потоковая передача телеметрии из IoT Central приложения с помощью *непрерывного экспорта данных*.
* Создание Stream Analytics запроса, который определяет, когда устройство остановило отправку данных.
* Отправка уведомления по электронной почте с помощью функций Azure и SendGrid Services.

## <a name="prerequisites"></a>Предварительные условия

Чтобы выполнить действия, описанные в этом руководстве, вам потребуется активная подписка Azure.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

### <a name="iot-central-application"></a>Приложение IoT Central

Создайте приложение IoT Central на веб-сайте [диспетчера приложений Azure IOT Central](https://aka.ms/iotcentral) со следующими параметрами:

| Параметр | Значение |
| ------- | ----- |
| Ценовой план | Стандартный |
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

```:::image type="content" source="media/howto-create-custom-rules/event-hubs-namespace.png" alt-text="Screenshot of Event Hubs namespace." border="false":::

## Define the function

This solution uses an Azure Functions app to send an email notification when the Stream Analytics job detects a stopped device. To create your function app:

1. In the Azure portal, navigate to the **App Service** instance in the **DetectStoppedDevices** resource group.
1. Select **+** to create a new function.
1. Select **HTTP Trigger**.
1. Select **Add**.

    :::image type="content" source="media/howto-create-custom-rules/add-function.png" alt-text="Image of the Default HTTP trigger function"::: 

## Edit code for HTTP Trigger

The portal creates a default function called **HttpTrigger1**:

```:::image type="content" source="media/howto-create-custom-rules/default-function.png" alt-text="Screenshot of Edit HTTP trigger function.":::

1. Replace the C# code with the following code:

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

    You may see an error message until you save the new code.
1. Select **Save** to save the function.

## Add SendGrid Key

To add your SendGrid API Key, you need to add it to your **Function Keys** as follows:

1. Select **Function Keys**.
1. Choose **+ New Function Key**.
1. Enter the *Name* and *Value* of the API Key you created before.
1. Click **OK.**

    :::image type="content" source="media/howto-create-custom-rules/add-key.png" alt-text="Screenshot of Add Sangrid Key.":::


## Configure HttpTrigger function to use SendGrid

To send emails with SendGrid, you need to configure the bindings for your function as follows:

1. Select **Integrate**.
1. Choose **Add Output** under **HTTP ($return)**.
1. Select **Delete.**
1. Choose **+ New Output**.
1. For Binding Type, then choose **SendGrid**.
1. For SendGrid API Key Setting Type, click New.
1. Enter the *Name* and *Value* of your SendGrid API key.
1. Add the following information:

| Setting | Value |
| ------- | ----- |
| Message parameter name | Choose your name |
| To address | Choose the name of your To Address |
| From address | Choose the name of your From Address |
| Message subject | Enter your subject header |
| Message text | Enter the message from your integration |

1. Select **OK**.

    :::image type="content" source="media/howto-create-custom-rules/add-output.png" alt-text="Screenshot of Add SandGrid Output.":::


### Test the function works

To test the function in the portal, first choose **Logs** at the bottom of the code editor. Then choose **Test** to the right of the code editor. Use the following JSON as the **Request body**:

```json
[{"deviceid":"test-device-1","time":"2019-05-02T14:23:39.527Z"},{"deviceid":"test-device-2","time":"2019-05-02T14:23:50.717Z"},{"deviceid":"test-device-3","time":"2019-05-02T14:24:28.919Z"}]
```

Сообщения журнала функций отображаются на панели **журналы** :

```:::image type="content" source="media/howto-create-custom-rules/function-app-logs.png" alt-text="Function log output":::

After a few minutes, the **To** email address receives an email with the following content:

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
    | Активировано | Включено |
    | Тип данных для экспорта | Телеметрия |
    | Усовершенствования | Введите требуемый ключ и значение, как должны упорядочиваться экспортированные данные | 
    | Назначение | Создайте новый и введите данные для экспорта данных. |

    :::image type="content" source="media/howto-create-custom-rules/cde-configuration.png" alt-text="Снимок экрана экспорта данных.":::

Прежде чем продолжить, подождите, пока состояние экспорта будет **выполняться** .

## <a name="test"></a>Проверить

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
