---
title: Интеграция с клиентским приложением с использованием пакета SDK службы "Речь"
titleSuffix: Azure Cognitive Services
description: как выполнять запросы к опубликованному приложению настраиваемых команд из пакета Speech SDK, выполняемого в приложении UWP.
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.custom: devx-track-csharp
ms.openlocfilehash: fa3a6d16b79800043bdcd3f183dd86fa278dd1a9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026033"
---
# <a name="integrate-with-a-client-application-using-speech-sdk"></a>Интеграция с клиентским приложением с помощью речевого пакета SDK

Из этой статьи вы узнаете, как выполнять запросы к опубликованному приложению настраиваемых команд из пакета Speech SDK, выполняемого в приложении UWP. Чтобы установить подключение к приложению настраиваемых команд, вам потребуется:

- Публикация приложения настраиваемых команд и получение идентификатора приложения (идентификатор приложения)
- Создание клиентского приложения универсальная платформа Windows (UWP) с помощью речевого пакета SDK, позволяющего взаимодействовать с приложением пользовательских команд.

## <a name="prerequisites"></a>Предварительные условия

Для выполнения этой статьи требуется приложение с пользовательскими командами. Если вы еще не создали приложение с пользовательскими командами, это можно сделать, следуя кратким руководствам:
> [!div class = "checklist"]
> * [Создание приложения с пользовательскими командами](quickstart-custom-commands-application.md)

Вам также потребуется:
> [!div class = "checklist"]
> * [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) или более поздней версии. Это руководством основано на Visual Studio 2019.
> * Ключ подписки Azure для служб "Речь". [Получите его бесплатно](overview.md#try-the-speech-service-for-free) или создайте его на [портал Azure](https://portal.azure.com)
> * [Включение устройства для разработки](/windows/uwp/get-started/enable-your-device-for-development)

## <a name="step-1-publish-custom-commands-application"></a>Шаг 1. Публикация приложения настраиваемых команд

1. Откройте созданное ранее приложение пользовательских команд.
1. Перейдите в раздел **Параметры**, выберите **ресурс Luis** .
1. Если **ресурс прогноза** не назначен, выберите ключ прогнозирования запроса или создайте новый.

    Перед публикацией приложения всегда требуется прогнозный ключ запроса. Дополнительные сведения о ресурсах LUIS см. в статье [Создание ресурса Luis](../luis/luis-how-to-azure-subscription.md) .

1. Вернитесь к командам редактирования, выберите **опубликовать** .

   > [!div class="mx-imgBorder"]
   > ![публикации приложения](media/custom-commands/setup-speech-sdk-publish-application.png)

1. Скопируйте идентификатор приложения из уведомления об опубликовании для последующего использования
1. Копирование ключа ресурса речи для последующего использования

## <a name="step-2-create-a-visual-studio-project"></a>Шаг 2. Создание проекта Visual Studio

[!INCLUDE [](../../../includes/cognitive-services-speech-service-quickstart-uwp-create-proj.md)]

## <a name="step-3-add-sample-code"></a>Шаг 3. Добавление примера кода

На этом шаге мы добавим код XAML, определяющий пользовательский интерфейс приложения, и добавим реализацию кода программной части C#.

### <a name="xaml-code"></a>Код XAML

Создайте пользовательский интерфейс приложения, добавив код XAML.

1. В **Обозреватель решений** откройте `MainPage.xaml`

1. В представлении XAML конструктора замените все содержимое следующим фрагментом кода:

   ```xml
   <Page
       x:Class="helloworld.MainPage"
       xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
       xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
       xmlns:local="using:helloworld"
       xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
       xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
       mc:Ignorable="d"
       Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">

       <Grid>
           <StackPanel Orientation="Vertical" HorizontalAlignment="Center"
                       Margin="20,50,0,0" VerticalAlignment="Center" Width="800">
               <Button x:Name="EnableMicrophoneButton" Content="Enable Microphone"
                       Margin="0,10,10,0" Click="EnableMicrophone_ButtonClicked"
                       Height="35"/>
               <Button x:Name="ListenButton" Content="Talk"
                       Margin="0,10,10,0" Click="ListenButton_ButtonClicked"
                       Height="35"/>
               <StackPanel x:Name="StatusPanel" Orientation="Vertical"
                           RelativePanel.AlignBottomWithPanel="True"
                           RelativePanel.AlignRightWithPanel="True"
                           RelativePanel.AlignLeftWithPanel="True">
                   <TextBlock x:Name="StatusLabel" Margin="0,10,10,0"
                              TextWrapping="Wrap" Text="Status:" FontSize="20"/>
                   <Border x:Name="StatusBorder" Margin="0,0,0,0">
                       <ScrollViewer VerticalScrollMode="Auto"
                                     VerticalScrollBarVisibility="Auto" MaxHeight="200">
                           <!-- Use LiveSetting to enable screen readers to announce
                                the status update. -->
                           <TextBlock
                               x:Name="StatusBlock" FontWeight="Bold"
                               AutomationProperties.LiveSetting="Assertive"
                               MaxWidth="{Binding ElementName=Splitter, Path=ActualWidth}"
                               Margin="10,10,10,20" TextWrapping="Wrap"  />
                       </ScrollViewer>
                   </Border>
               </StackPanel>
           </StackPanel>
           <MediaElement x:Name="mediaElement"/>
       </Grid>
   </Page>
   ```

Конструктор обновляется для отображения пользовательского интерфейса приложения.

### <a name="c-code-behind-source"></a>Источник кода C# программной части

Добавьте источник кода программной части, чтобы приложение правильно работала. Источник кода программной части включает в себя:

- Обязательные `using` инструкции для `Speech` `Speech.Dialog` пространств имен и
- простая реализация доступа к микрофону с привязкой к обработчику кнопки;
- базовые вспомогательные функции пользовательского интерфейса для представления сообщений и ошибок в приложении;
- целевая точка для пути кода инициализации, который будет добавлен позже;
- вспомогательное приложение для воспроизведения текста, преобразованного в речь (без поддержки потоковой передачи данных);
- пустой обработчик кнопки воспроизведения, который будет заполнен позже.

Добавьте источник кода программной части следующим образом:

1. В **Обозреватель решений** откройте исходный файл кода программной части `MainPage.xaml.cs` (сгруппированный в разделе `MainPage.xaml` ).

1. Замените содержимое файла следующим кодом: 

   ```csharp
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Audio;
   using Microsoft.CognitiveServices.Speech.Dialog;
   using System;
   using System.IO;
   using System.Text;
   using Windows.UI.Xaml;
   using Windows.UI.Xaml.Controls;
   using Windows.UI.Xaml.Media;

   namespace helloworld
   {
       public sealed partial class MainPage : Page
       {
           private DialogServiceConnector connector;

           private enum NotifyType
           {
               StatusMessage,
               ErrorMessage
           };

           public MainPage()
           {
               this.InitializeComponent();
           }

           private async void EnableMicrophone_ButtonClicked(
               object sender, RoutedEventArgs e)
           {
               bool isMicAvailable = true;
               try
               {
                   var mediaCapture = new Windows.Media.Capture.MediaCapture();
                   var settings =
                       new Windows.Media.Capture.MediaCaptureInitializationSettings();
                   settings.StreamingCaptureMode =
                       Windows.Media.Capture.StreamingCaptureMode.Audio;
                   await mediaCapture.InitializeAsync(settings);
               }
               catch (Exception)
               {
                   isMicAvailable = false;
               }
               if (!isMicAvailable)
               {
                   await Windows.System.Launcher.LaunchUriAsync(
                       new Uri("ms-settings:privacy-microphone"));
               }
               else
               {
                   NotifyUser("Microphone was enabled", NotifyType.StatusMessage);
               }
           }

           private void NotifyUser(
               string strMessage, NotifyType type = NotifyType.StatusMessage)
           {
               // If called from the UI thread, then update immediately.
               // Otherwise, schedule a task on the UI thread to perform the update.
               if (Dispatcher.HasThreadAccess)
               {
                   UpdateStatus(strMessage, type);
               }
               else
               {
                   var task = Dispatcher.RunAsync(
                       Windows.UI.Core.CoreDispatcherPriority.Normal,
                       () => UpdateStatus(strMessage, type));
               }
           }

           private void UpdateStatus(string strMessage, NotifyType type)
           {
               switch (type)
               {
                   case NotifyType.StatusMessage:
                       StatusBorder.Background = new SolidColorBrush(
                           Windows.UI.Colors.Green);
                       break;
                   case NotifyType.ErrorMessage:
                       StatusBorder.Background = new SolidColorBrush(
                           Windows.UI.Colors.Red);
                       break;
               }
               StatusBlock.Text += string.IsNullOrEmpty(StatusBlock.Text)
                   ? strMessage : "\n" + strMessage;

               if (!string.IsNullOrEmpty(StatusBlock.Text))
               {
                   StatusBorder.Visibility = Visibility.Visible;
                   StatusPanel.Visibility = Visibility.Visible;
               }
               else
               {
                   StatusBorder.Visibility = Visibility.Collapsed;
                   StatusPanel.Visibility = Visibility.Collapsed;
               }
               // Raise an event if necessary to enable a screen reader
               // to announce the status update.
               var peer = Windows.UI.Xaml.Automation.Peers.FrameworkElementAutomationPeer.FromElement(StatusBlock);
               if (peer != null)
               {
                   peer.RaiseAutomationEvent(
                       Windows.UI.Xaml.Automation.Peers.AutomationEvents.LiveRegionChanged);
               }
           }

           // Waits for and accumulates all audio associated with a given
           // PullAudioOutputStream and then plays it to the MediaElement. Long spoken
           // audio will create extra latency and a streaming playback solution
           // (that plays audio while it continues to be received) should be used --
           // see the samples for examples of this.
           private void SynchronouslyPlayActivityAudio(
               PullAudioOutputStream activityAudio)
           {
               var playbackStreamWithHeader = new MemoryStream();
               playbackStreamWithHeader.Write(Encoding.ASCII.GetBytes("RIFF"), 0, 4); // ChunkID
               playbackStreamWithHeader.Write(BitConverter.GetBytes(UInt32.MaxValue), 0, 4); // ChunkSize: max
               playbackStreamWithHeader.Write(Encoding.ASCII.GetBytes("WAVE"), 0, 4); // Format
               playbackStreamWithHeader.Write(Encoding.ASCII.GetBytes("fmt "), 0, 4); // Subchunk1ID
               playbackStreamWithHeader.Write(BitConverter.GetBytes(16), 0, 4); // Subchunk1Size: PCM
               playbackStreamWithHeader.Write(BitConverter.GetBytes(1), 0, 2); // AudioFormat: PCM
               playbackStreamWithHeader.Write(BitConverter.GetBytes(1), 0, 2); // NumChannels: mono
               playbackStreamWithHeader.Write(BitConverter.GetBytes(16000), 0, 4); // SampleRate: 16kHz
               playbackStreamWithHeader.Write(BitConverter.GetBytes(32000), 0, 4); // ByteRate
               playbackStreamWithHeader.Write(BitConverter.GetBytes(2), 0, 2); // BlockAlign
               playbackStreamWithHeader.Write(BitConverter.GetBytes(16), 0, 2); // BitsPerSample: 16-bit
               playbackStreamWithHeader.Write(Encoding.ASCII.GetBytes("data"), 0, 4); // Subchunk2ID
               playbackStreamWithHeader.Write(BitConverter.GetBytes(UInt32.MaxValue), 0, 4); // Subchunk2Size

               byte[] pullBuffer = new byte[2056];

               uint lastRead = 0;
               do
               {
                   lastRead = activityAudio.Read(pullBuffer);
                   playbackStreamWithHeader.Write(pullBuffer, 0, (int)lastRead);
               }
               while (lastRead == pullBuffer.Length);

               var task = Dispatcher.RunAsync(
                   Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
               {
                   mediaElement.SetSource(
                       playbackStreamWithHeader.AsRandomAccessStream(), "audio/wav");
                   mediaElement.Play();
               });
           }

           private void InitializeDialogServiceConnector()
           {
               // New code will go here
           }

           private async void ListenButton_ButtonClicked(
               object sender, RoutedEventArgs e)
           {
               // New code will go here
           }
       }
   }
   ```
    > [!NOTE]
    > Если отображается ошибка: "тип" Object "определен в сборке, на которую нет ссылок"
    > 1. Щелкните правой кнопкой мыши клиентское решение.
    > 1. Выберите **Управление пакетами NuGet для решения**, а затем — **обновления** . 
    > 1. Если в списке обновлений отображается **Microsoft. NETCore. UniversalWindowsPlatform** , обновите **Microsoft. NETCore. UniversalWindowsPlatform** до последней версии.

1. Добавьте следующий код в тело метода `InitializeDialogServiceConnector`

   ```csharp
   // This code creates the `DialogServiceConnector` with your subscription information.
   // create a DialogServiceConfig by providing a Custom Commands application id and Cognitive Services subscription key
   // the RecoLanguage property is optional (default en-US); note that only en-US is supported in Preview
   const string speechCommandsApplicationId = "YourApplicationId"; // Your application id
   const string speechSubscriptionKey = "YourSpeechSubscriptionKey"; // Your subscription key
   const string region = "YourServiceRegion"; // The subscription service region. Note: only 'westus2' is currently supported

   var speechCommandsConfig = CustomCommandsConfig.FromSubscription(speechCommandsApplicationId, speechSubscriptionKey, region);
   speechCommandsConfig.SetProperty(PropertyId.SpeechServiceConnection_RecoLanguage, "en-us");
   connector = new DialogServiceConnector(speechCommandsConfig);
   ```

1. Замените строки `YourApplicationId` , `YourSpeechSubscriptionKey` и `YourServiceRegion` собственными значениями для приложения, подписки на речь и [региона](regions.md) .

1. Добавьте следующий фрагмент кода в конец тела метода `InitializeDialogServiceConnector`

   ```csharp
   //
   // This code sets up handlers for events relied on by `DialogServiceConnector` to communicate its activities,
   // speech recognition results, and other information.
   //
   // ActivityReceived is the main way your client will receive messages, audio, and events
   connector.ActivityReceived += (sender, activityReceivedEventArgs) =>
   {
       NotifyUser(
           $"Activity received, hasAudio={activityReceivedEventArgs.HasAudio} activity={activityReceivedEventArgs.Activity}");

       if (activityReceivedEventArgs.HasAudio)
       {
           SynchronouslyPlayActivityAudio(activityReceivedEventArgs.Audio);
       }
   };

   // Canceled will be signaled when a turn is aborted or experiences an error condition
   connector.Canceled += (sender, canceledEventArgs) =>
   {
       NotifyUser($"Canceled, reason={canceledEventArgs.Reason}");
       if (canceledEventArgs.Reason == CancellationReason.Error)
       {
           NotifyUser(
               $"Error: code={canceledEventArgs.ErrorCode}, details={canceledEventArgs.ErrorDetails}");
       }
   };

   // Recognizing (not 'Recognized') will provide the intermediate recognized text
   // while an audio stream is being processed
   connector.Recognizing += (sender, recognitionEventArgs) =>
   {
       NotifyUser($"Recognizing! in-progress text={recognitionEventArgs.Result.Text}");
   };

   // Recognized (not 'Recognizing') will provide the final recognized text
   // once audio capture is completed
   connector.Recognized += (sender, recognitionEventArgs) =>
   {
       NotifyUser($"Final speech-to-text result: '{recognitionEventArgs.Result.Text}'");
   };

   // SessionStarted will notify when audio begins flowing to the service for a turn
   connector.SessionStarted += (sender, sessionEventArgs) =>
   {
       NotifyUser($"Now Listening! Session started, id={sessionEventArgs.SessionId}");
   };

   // SessionStopped will notify when a turn is complete and
   // it's safe to begin listening again
   connector.SessionStopped += (sender, sessionEventArgs) =>
   {
       NotifyUser($"Listening complete. Session ended, id={sessionEventArgs.SessionId}");
   };
   ```

1. Добавьте следующий фрагмент кода в тело `ListenButton_ButtonClicked` метода в `MainPage` классе.

   ```csharp
   // This code sets up `DialogServiceConnector` to listen, since you already established the configuration and
   // registered the event handlers.
   if (connector == null)
   {
       InitializeDialogServiceConnector();
       // Optional step to speed up first interaction: if not called,
       // connection happens automatically on first use
       var connectTask = connector.ConnectAsync();
   }

   try
   {
       // Start sending audio
       await connector.ListenOnceAsync();
   }
   catch (Exception ex)
   {
       NotifyUser($"Exception: {ex.ToString()}", NotifyType.ErrorMessage);
   }
   ```

1. В строке меню выберите **файл**  >  **сохранить все** , чтобы сохранить изменения.

## <a name="try-it-out"></a>Попробуйте сейчас

1. В строке меню выберите **Сборка** > **Построить решение**, чтобы создать приложение. Теперь код должен компилироваться без ошибок.

1. Выберите **Отладка** > **Начать отладку**(или нажмите клавишу **F5**), чтобы запустить приложение. Откроется окно **helloworld**.

   ![Пример приложения виртуального помощника UWP в C# — краткое руководство](media/sdk/qs-voice-assistant-uwp-helloworld-window.png)

1. Выберите **Включить микрофон**. Если появится запрос на разрешение доступа, выберите **Да**.

   ![Запрос на разрешение доступа к микрофону](media/sdk/qs-csharp-uwp-10-access-prompt.png)

1. Выберите **разговор** и говорите в микрофоне устройства фразу на английском языке или предложение. Ваша речь передастся в канал "Речь Direct Line" и преобразуется в текст, который появится в том же окне.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Пошаговое руководство. Отправка действия в клиентское приложение (Предварительная версия)](./how-to-custom-commands-send-activity-to-client.md)