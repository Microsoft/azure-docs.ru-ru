---
title: Настройка конечных веб-точек (предварительная версия)
titleSuffix: Azure Cognitive Services
description: Настройка конечных веб-точек для Пользовательских голосовых команд
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: 6f2dfdbb5833b34441b4abba7359ad70c4717d1d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98602165"
---
# <a name="set-up-web-endpoints"></a>Настройка конечных веб-точек

Из этой статьи вы узнаете, как настроить конечные веб-точки в приложении Пользовательских голосовых команд. Эти конечные веб-точки позволят вам принимать HTTP-запросы от клиентского приложения. Вы выполните следующие задачи:

- настройка конечных веб-точек в приложении Пользовательских голосовых команд;
- вызов конечных веб-точек в приложении Пользовательских голосовых команд;
- получение ответа конечных веб-точек;
- интеграция ответа конечных веб-точек в пользовательские полезные данные JSON, их отправка и визуализация из клиентского приложения UWP пакета SDK службы "Речь" на C#.

## <a name="prerequisites"></a>Предварительные требования
> [!div class = "checklist"]
> * [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)
> * Ключ подписки Azure для служб "Речь". [Получите бесплатно](overview.md#try-the-speech-service-for-free) или создайте его на [портале Azure](https://portal.azure.com).
> * Предварительно [созданное приложение Пользовательских голосовых команд](quickstart-custom-commands-application.md).
> * Клиентское приложение с поддержкой пакета SDK службы "Речь". Узнайте, [как отправлять действия в клиентское приложение](./how-to-custom-commands-setup-speech-sdk.md).

## <a name="setup-web-endpoints"></a>Настройка конечных веб-точек

1. Откройте созданное ранее приложение Пользовательских голосовых команд.
1. Выберите "Web endpoints" (Конечные веб-точки) и "New web endpoint" (Создать конечную веб-точку).

   > [!div class="mx-imgBorder"]
   > ![Создание конечной веб-точки](media/custom-commands/setup-web-endpoint-new-endpoint.png)

   | Параметр | Рекомендуемое значение | Описание |
   | ------- | --------------- | ----------- |
   | Имя | UpdateDeviceState | Имя конечной веб-точки. |
   | URL-адрес | https://webendpointexample.azurewebsites.net/api/DeviceState | URL-адрес конечной точки, к которой будет обращаться ваше приложение Пользовательских голосовых команд. |
   | Метод | POST | Разрешенные взаимодействия (например, GET, POST) с вашей конечной точкой.|
   | Заголовки | Ключ: app; значение: первые 8 цифр вашего идентификатора приложения | Параметры заголовков для включения в заголовок запроса.|

    > [!NOTE]
    > - Пример конечной веб-точки, созданной с использованием [функции Azure](../../azure-functions/index.yml), которая привязывается к базе данных, где хранится состояние устройства для телевизора и вентилятора.
    > - Предлагаемый заголовок нужен только для примера конечной точки.
    > - Чтобы значение заголовка было уникальным в нашем примере конечной точки, используйте первые 8 цифр вашего идентификатора приложения.
    > - В реальных средах конечная веб-точка может быть конечной точкой для [центра Интернета вещей](../../iot-hub/about-iot-hub.md), который управляет вашими устройствами.

1. Выберите команду **Сохранить**.

## <a name="call-web-endpoints"></a>Вызов конечных веб-точек

1. Перейдите к команде **TurnOnOff**, выберите **ConfirmationResponse** для правила завершения и щелкните **Добавить действие**.
1. В разделе **New Action-Type** (Создание типа действия) выберите **Call web endpoint** (Вызов конечной веб-точки).
1. В разделе **Изменение действия — конечные точки** выберите **UpdateDeviceState**, то есть созданную нами конечную веб-точку.  
1. В разделе **Конфигурация** введите следующие значения:
   > [!div class="mx-imgBorder"]
   > ![Параметры действия вызова конечных веб-точек](media/custom-commands/setup-web-endpoint-edit-action-parameters.png)

   | Параметр | Рекомендуемое значение | Описание |
   | ------- | --------------- | ----------- |
   | Конечные точки | UpdateDeviceState | Конечная веб-точка, которую вы хотите вызвать в этом действии. |
   | Параметры запроса | item={SubjectDevice}&&value={OnOff} | Параметры запроса, добавляемые к URL-адресу конечной веб-точки.  |
   | Содержимое текста | Н/Д | Содержимое текста запроса. |

    > [!NOTE]
    > - Предлагаемые параметры запроса необходимы только для примера конечной точки.

1. В разделе **On Success - Action to execute** (При успешном выполнении — действие для выполнения) выберите **Send speech response** (Отправить ответ службы "Речь").

    В разделе **Simple editor** (Простой редактор) введите `{SubjectDevice} is {OnOff}`.

   > [!div class="mx-imgBorder"]
   > ![Снимок экрана, на котором отображается экран "при успешном выполнении действия".](media/custom-commands/setup-web-endpoint-edit-action-on-success-send-response.png)

   | Параметр | Рекомендуемое значение | Описание |
   | ------- | --------------- | ----------- |
   | Действие для выполнения | Отправить ответ службы "Речь" | Действие, выполняемое при успешном запросе к конечной веб-точке |

   > [!NOTE]
   > - Вы также можете напрямую обращаться к полям в HTTP-ответе с помощью `{YourWebEndpointName.FieldName}`. Пример: `{UpdateDeviceState.TV}`

1. В разделе **On Failure - Action to execute** (При неуспешном выполнении — действие для выполнения) выберите **Send speech response** (Отправить ответ службы "Речь").

    В разделе **Simple editor** (Простой редактор) введите `Sorry, {WebEndpointErrorMessage}`.

   > [!div class="mx-imgBorder"]
   > ![Действие при неуспешном вызове конечных веб-точек](media/custom-commands/setup-web-endpoint-edit-action-on-fail.png)

   | Параметр | Рекомендуемое значение | Описание |
   | ------- | --------------- | ----------- |
   | Действие для выполнения | Отправить ответ службы "Речь" | Действие, выполняемое при неуспешном запросе к конечной веб-точке |

   > [!NOTE]
   > - Аргумент `{WebEndpointErrorMessage}` является необязательным. Вы можете удалить его, если не хотите, чтобы отображались сообщения об ошибках.
   > - В нашем примере конечной точки отправляется HTTP-ответ с подробным сообщением об ошибке для распространенных ошибок, например связанных с отсутствующими параметрами заголовка.

### <a name="try-it-out-in-test-portal"></a>Опробуйте на портале тестирования
- Ответ при успешном выполнении.
Сохраните, запустите обучение и протестируйте.
   > [!div class="mx-imgBorder"]
   > ![Снимок экрана, на котором показан ответ при успешном выполнении.](media/custom-commands/setup-web-endpoint-on-success-response.png)
- Ответ при неуспешном выполнении.
Удалите один из параметров запроса, сохраните, запустите обучение и протестируйте.
   > [!div class="mx-imgBorder"]
   > ![Действие при успешном вызове конечных веб-точек](media/custom-commands/setup-web-endpoint-on-fail-response.png)

## <a name="integrate-with-client-application"></a>Интеграция с клиентским приложением

При работе с [практическим руководством по отправке действия в клиентское приложение (предварительная версия)](./how-to-custom-commands-send-activity-to-client.md) вы добавили действие **Отправить действие в клиент**. Это действие отправляется в клиентское приложение независимо от успешности действия **Вызов конечной веб-точки**.
Но в большинстве случаев вам требуется отправить действие в клиентское приложение только при успешном выполнении вызова к конечной веб-точке. В этом примере именно при таких условиях выполняется успешное обновление состояния устройства.

1. Удалите добавленное ранее действие **Отправить действие в клиент**.
1. Измените вызов конечной веб-точки:
    1. В разделе **Конфигурация** обязательно введите в **параметры запроса** следующее значение: `item={SubjectDevice}&&value={OnOff}`.
    1. В разделе **При успешном выполнении** введите в поле **Действие для выполнения** значение **Отправить действие в клиент**.
    1. Скопируйте приведенный ниже код JSON в поле **Содержимое действия**.
   ```json
   {
      "type": "event",
      "name": "UpdateDeviceState",
      "value": {
        "state": "{OnOff}",
        "device": "{SubjectDevice}"
      }
    }
   ```
Теперь вы отправляете действие в клиент только при успешном выполнении запроса к конечной веб-точке.

### <a name="create-visuals-for-syncing-device-state"></a>Создание визуальных элементов для синхронизации состояния устройства
Добавьте следующий код XML в файл `MainPage.xaml` над блоком `"EnableMicrophoneButton"`.

```xml
<Button x:Name="SyncDeviceStateButton" Content="Sync Device State"
        Margin="0,10,10,0" Click="SyncDeviceState_ButtonClicked"
        Height="35"/>
<Button x:Name="EnableMicrophoneButton" ......
        .........../>
```

### <a name="sync-device-state"></a>Синхронизация состояния устройства

В файл `MainPage.xaml.cs` добавьте ссылку `using Windows.Web.Http;`. Добавьте в класс `MainPage` приведенный далее код. Этот метод отправит запрос GET в пример конечной точки и извлечет текущее состояние устройства для вашего приложения. Обязательно измените `<your_app_name>` на значение, которое вы использовали в **заголовке** в конечной веб-точке Пользовательских голосовых команд.

```C#
private async void SyncDeviceState_ButtonClicked(object sender, RoutedEventArgs e)
{
    //Create an HTTP client object
    var httpClient = new HttpClient();

    //Add a user-agent header to the GET request.
    var your_app_name = "<your-app-name>";

    Uri endpoint = new Uri("https://webendpointexample.azurewebsites.net/api/DeviceState");
    var requestMessage = new HttpRequestMessage(HttpMethod.Get, endpoint);
    requestMessage.Headers.Add("app", $"{your_app_name}");

    try
    {
        //Send the GET request
        var httpResponse = await httpClient.SendRequestAsync(requestMessage);
        httpResponse.EnsureSuccessStatusCode();
        var httpResponseBody = await httpResponse.Content.ReadAsStringAsync();
        dynamic deviceState = JsonConvert.DeserializeObject(httpResponseBody);
        var TVState = deviceState.TV.ToString();
        var FanState = deviceState.Fan.ToString();
        await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(
            CoreDispatcherPriority.Normal,
            () =>
            {
                State_TV.Text = TVState;
                State_Fan.Text = FanState;
            });
    }
    catch (Exception ex)
    {
        NotifyUser(
            $"Unable to sync device status: {ex.Message}");
    }
}
```

## <a name="try-it-out"></a>Попробуйте!

1. Запуск приложения
1. Щелкните "Sync Device State" (Синхронизировать состояние устройства).
Если вы проверили приложение с помощью команды `turn on tv` в предыдущем разделе, вы увидите, что для состояния телевизора отображается значение "on" (включен).
    > [!div class="mx-imgBorder"]
    > ![Синхронизация состояния устройства](media/custom-commands/setup-web-endpoint-sync-device-state.png)
1. Выберите "Enable microphone" (Включить микрофон).
1. Нажмите кнопку речи.
1. Скажите `turn on the fan`.
1. Визуальное состояние вентилятора должно измениться на "on" (включен).
    > [!div class="mx-imgBorder"]
    > ![Включение вентилятора](media/custom-commands/setup-web-endpoint-turn-on-fan.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Экспорт приложения настраиваемых команд в качестве удаленного навыка](./how-to-custom-commands-integrate-remote-skills.md)

