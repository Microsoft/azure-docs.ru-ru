---
title: Отправка действия настраиваемых команд клиентскому приложению
titleSuffix: Azure Cognitive Services
description: Из этой статьи вы узнаете, как отправить действие из приложения настраиваемых команд в клиентское приложение, запускающее пакет SDK для распознавания речи.
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: 52a4dbc4ff01515af8cd7d2503877184a09f7e64
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94566101"
---
# <a name="send-custom-commands-activity-to-client-application"></a>Отправка действия настраиваемых команд клиентскому приложению

Из этой статьи вы узнаете, как отправить действие из приложения настраиваемых команд в клиентское приложение, запускающее пакет SDK для распознавания речи.

Вы выполняете следующие задачи:

- Определение и Отправка пользовательских полезных данных JSON из приложения пользовательских команд
- Получение и визуализация содержимого настраиваемой полезной нагрузки JSON из клиентского приложения SDK для языка C# UWP

## <a name="prerequisites"></a>Предварительные условия
> [!div class = "checklist"]
> * [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) или более поздней версии. В этом руководством используется Visual Studio 2019
> * Ключ подписки Azure для служб "Речь". [Получите бесплатно](overview.md#try-the-speech-service-for-free) или создайте его на [портале Azure](https://portal.azure.com).
> * Предварительно [созданное приложение Пользовательских голосовых команд](quickstart-custom-commands-application.md).
> * Клиентское приложение с поддержкой Speech SDK: [практические руководства. Интеграция с клиентским приложением с помощью пакета SDK для речи](./how-to-custom-commands-setup-speech-sdk.md)

## <a name="setup-send-activity-to-client"></a>Настройка отправки действия клиенту 
1. Откройте созданное ранее приложение пользовательских команд.
1. Выберите команду **турнонофф** , выберите **конфирматионреспонсе** в разделе правило завершения, а затем выберите **Добавить действие** .
1. В разделе **новый тип действия** выберите **Отправить действие клиенту** .
1. Скопируйте приведенный ниже код JSON в **содержимое действия** .
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
1. Нажмите кнопку **сохранить** , чтобы создать новое правило с действием Отправить действие, **обучить** и **опубликовать** изменение

   > [!div class="mx-imgBorder"]
   > ![Правило завершения отправки действия](media/custom-commands/send-activity-to-client-completion-rules.png)

## <a name="integrate-with-client-application"></a>Интеграция с клиентским приложением

В [пошаговом окне Настройка клиентского приложения с помощью речевого пакета SDK (Предварительная версия)](./how-to-custom-commands-setup-speech-sdk.md)вы создали клиентское приложение UWP с пакетом SDK для распознавания речи, которое обрабатывает такие команды `turn on the tv` , как, `turn off the fan` . После добавления некоторых визуальных элементов можно увидеть результат этих команд.

Чтобы добавить помеченные поля с текстом, **указывающим** или **выключая**, добавьте следующий XML-блок StackPanel в `MainPage.xaml` .

```xml
<StackPanel Orientation="Vertical" H......>
......
</StackPanel>
<StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Margin="20">
    <Grid x:Name="Grid_TV" Margin="50, 0" Width="100" Height="100" Background="LightBlue">
        <StackPanel>
            <TextBlock Text="TV" Margin="0, 10" TextAlignment="Center"/>
            <TextBlock x:Name="State_TV" Text="off" TextAlignment="Center"/>
        </StackPanel>
    </Grid>
    <Grid x:Name="Grid_Fan" Margin="50, 0" Width="100" Height="100" Background="LightBlue">
        <StackPanel>
            <TextBlock Text="Fan" Margin="0, 10" TextAlignment="Center"/>
            <TextBlock x:Name="State_Fan" Text="off" TextAlignment="Center"/>
        </StackPanel>
    </Grid>
</StackPanel>
<MediaElement ....../>
```

### <a name="add-reference-libraries"></a>Добавление библиотек ссылок

Так как вы создали полезные данные JSON, необходимо добавить ссылку на библиотеку [JSON.NET](https://www.newtonsoft.com/json) для управления десериализацией.

1. Щелкните правой кнопкой мыши клиентское решение.
1. Выберите **Управление пакетами NuGet для решения**, нажмите кнопку **Обзор** . 
1. Если вы уже установили **Newtonsoft.jsв**, убедитесь, что его версия по крайней мере 12.0.3. Если это не так, перейдите к разделу **Управление пакетами NuGet для обновления решения**, выполните поиск по **Newtonsoft.json** , чтобы обновить его. В этом руководством используется версия 12.0.3.

    > [!div class="mx-imgBorder"]
    > ![Отправка полезных данных действия](media/custom-commands/send-activity-to-client-json-nuget.png)

1. Кроме того, убедитесь, что пакет NuGet **Microsoft. NETCore. UniversalWindowsPlatform** имеет по крайней мере 6.2.10. В этом руководством используется версия 6.2.10.

В "MainPage. XAML. cs" добавьте

```C#
using Newtonsoft.Json; 
using Windows.ApplicationModel.Core;
using Windows.UI.Core;
```

### <a name="handle-the-received-payload"></a>Обрабатывает полученные полезные данные

В `InitializeDialogServiceConnector` замените `ActivityReceived` обработчик событий следующим кодом. Измененный `ActivityReceived` обработчик событий извлечет полезные данные из действия и изменит визуальное состояние телевизора или вентилятора соответственно.

```C#
connector.ActivityReceived += async (sender, activityReceivedEventArgs) =>
{
    NotifyUser($"Activity received, hasAudio={activityReceivedEventArgs.HasAudio} activity={activityReceivedEventArgs.Activity}");

    dynamic activity = JsonConvert.DeserializeObject(activityReceivedEventArgs.Activity);
    var name = activity?.name != null ? activity.name.ToString() : string.Empty;

    if (name.Equals("UpdateDeviceState"))
    {
        Debug.WriteLine("Here");
        var state = activity?.value?.state != null ? activity.value.state.ToString() : string.Empty;
        var device = activity?.value?.device != null ? activity.value.device.ToString() : string.Empty;

        if (state.Equals("on") || state.Equals("off"))
        {
            switch (device)
            {
                case "tv":
                    await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(
                        CoreDispatcherPriority.Normal, () => { State_TV.Text = state; });
                    break;
                case "fan":
                    await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(
                        CoreDispatcherPriority.Normal, () => { State_Fan.Text = state; });
                    break;
                default:
                    NotifyUser($"Received request to set unsupported device {device} to {state}");
                    break;
            }
        }
        else { 
            NotifyUser($"Received request to set unsupported state {state}");
        }
    }

    if (activityReceivedEventArgs.HasAudio)
    {
        SynchronouslyPlayActivityAudio(activityReceivedEventArgs.Audio);
    }
};
```

## <a name="try-it-out"></a>Попробуйте!

1. Запуск приложения
1. Выберите "Enable microphone" (Включить микрофон).
1. Нажмите кнопку речи.
1. Скажите `turn on the tv`.
1. Визуальное состояние телевизора должно измениться на "вкл."
   > [!div class="mx-imgBorder"]
   > ![Снимок экрана, на котором показано, что теперь визуальное состояние T V включено.](media/custom-commands/send-activity-to-client-turn-on-tv.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Как настроить конечные веб-точки (Предварительная версия)](./how-to-custom-commands-setup-web-endpoints.md)
