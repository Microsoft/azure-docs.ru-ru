---
title: Руководство по приложениям Магазина Windows с потоковой передачей Smooth Streaming | Документация Майкрософт
description: Сведения об использовании служб мультимедиа Azure для создания приложения для магазина Windows на C# с управляющим элементом XML MediaElement для воспроизведения контента Smooth Stream
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 0fa5d8c5-3d5f-4886-ae55-fb6de4f5256d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 20c7068fa0cb56699fdbc6d75b279abadd583832
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103017126"
---
# <a name="how-to-build-a-smooth-streaming-windows-store-application"></a>Создание приложения для магазина Windows с бесперебойной потоковой передачей

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

Клиентский пакет SDK для бесперебойной потоковой передачи для Windows 8 позволяет разработчикам создавать приложения для магазина Windows, способные воспроизводить контент по требованию и в режиме реального времени с бесперебойной потоковой передачей. Помимо основных функций воспроизведения контента с бесперебойной потоковой передачей этот пакет SDK также предоставляет богатые возможности, такие как защита Microsoft PlayReady, ограничение уровня качества, Live DVR, переключение потока аудио, прослушивание событий ошибок и обновления состояния (например, изменения уровня качества) и т. д. Дополнительные сведения о поддерживаемых возможностях см. в [заметках к выпуску](https://www.iis.net/learn/media/smooth-streaming/smooth-streaming-client-sdk-for-windows-8-release-notes). Дополнительные сведения см. на странице, посвященной [Player Framework для Windows 8](https://playerframework.codeplex.com/). 

Этот учебный курс состоит из четырех занятий:

1. Создание базового приложения магазина с бесперебойной потоковой передачей
2. Добавление ползунка для управления ходом воспроизведения мультимедиа
3. Выбор потоков для бесперебойной потоковой передачи
4. Выбор дорожек для бесперебойной потоковой передачи

## <a name="prerequisites"></a>Предварительные требования
> [!NOTE]
> Проекты для Магазина Windows версии 8.1 и более ранней версии не поддерживаются в Visual Studio 2017.  Дополнительные сведения см. в статье [Целевая платформа и совместимость для Visual Studio 2017](https://www.visualstudio.com/en-us/productinfo/vs2017-compatibility-vs).

* Windows 8 (32-разрядная или 64-разрядная).
* Visual Studio версий 2012–2015.
* [Клиентский пакет SDK для бесперебойной потоковой передачи Microsoft для Windows 8](https://visualstudiogallery.msdn.microsoft.com/04423d13-3b3e-4741-a01c-1ae29e84fea6?SRC=Home)

Готовое решение для каждого урока можно загрузить из раздела образцов кода для разработчиков MSDN (коллекция кодов): 

* [Урок 1.](https://code.msdn.microsoft.com/Smooth-Streaming-Client-0bb1471f) Простой мультимедиапроигрыватель для Windows 8 с потоковой передачей Smooth Streaming. 
* [Урок 2.](https://code.msdn.microsoft.com/A-simple-Windows-8-Smooth-ee98f63a) Простой мультимедиапроигрыватель для Windows 8 с потоковой передачей Smooth Streaming и элементом управления «Ползунок». 
* [Урок 3.](https://code.msdn.microsoft.com/A-Windows-8-Smooth-883c3b44) Простой мультимедиапроигрыватель для Windows 8 с потоковой передачей Smooth Streaming и выбором потока.  
* [Урок 4.](https://code.msdn.microsoft.com/A-Windows-8-Smooth-aa9e4907) Мультимедиапроигрыватель для Windows 8 с потоковой передачей Smooth Streaming и выбором дорожки.

## <a name="lesson-1-create-a-basic-smooth-streaming-store-application"></a>Урок 1. Создание базового приложения для магазина с бесперебойной потоковой передачей

На этом занятии предстоит создать приложение для магазина Windows с элементом управления MediaElement для воспроизведения контента с бесперебойной потоковой передачей.  Работающее приложение имеет следующий вид:

![Пример приложения для магазина Windows с бесперебойной потоковой передачей][PlayerApplication]

Дополнительные сведения о разработке приложений Магазина Windows см. в разделе, посвященном [разработке потрясающих приложений для Windows 8](https://developer.microsoft.com/en-us/windows/). Это занятие содержит следующие процедуры:

1. Создание проекта для магазина Windows
2. Проектирование пользовательского интерфейса (XAML)
3. Изменение кода в файле
4. Компиляция и тестирование приложения

### <a name="to-create-a-windows-store-project"></a>Создание проекта приложения Магазина Windows

1. Запустите Visual Studio. Поддерживаются Visual Studio версии 2012–2015.
1. В меню **Файл** выберите команду **Создать**, а затем — **Проект**.
1. В диалоговом окне "Новый проект" введите или выберите следующие значения:

    | Имя | Значение |
    | --- | --- |
    | Группа шаблонов |Installed/Templates/Visual C#/Windows Store |
    | Шаблон |Пустое приложение (XAML) |
    | Имя |SSPlayer |
    | Расположение |C:\SSTutorials |
    | Имя решения |SSPlayer |
    | Создать каталог для решения |(выбрано) |

1. Нажмите кнопку **ОК**.

### <a name="to-add-a-reference-to-the-smooth-streaming-client-sdk"></a>Добавление ссылки на пакет SDK клиента Smooth Streaming

1. В обозревателе решений щелкните правой кнопкой мыши **SSPlayer**, а затем выберите команду **Добавить ссылку**.
1. Введите или выберите следующие значения:

    | Имя | Значение |
    | --- | --- |
    | Ссылочная группа |Windows/Расширения |
    | Справка |Выберите клиентский пакет SDK бесперебойной потоковой передачи Microsoft для Windows 8 и пакет среды выполнения Microsoft Visual C++ |

1. Нажмите кнопку **ОК**. 

После добавления ссылок необходимо выбрать целевую платформу (x64 x86), добавление ссылок будет действовать не для всех конфигураций платформы ЦП.  Для таких добавленных ссылок в обозревателе решений будет выведен желтый значок предупреждения.

### <a name="to-design-the-player-user-interface"></a>Создание пользовательского интерфейса проигрывателя

1. В обозревателе решений дважды щелкните **MainPage.xaml** , чтобы открыть его в режиме конструктора.
2. Найдите теги **&lt;Grid&gt;** и **&lt;/Grid&gt;** в XAML-файле и вставьте между ними код ниже:

   ```xml
         <Grid.RowDefinitions>

            <RowDefinition Height="20"/>    <!-- spacer -->
            <RowDefinition Height="50"/>    <!-- media controls -->
            <RowDefinition Height="100*"/>  <!-- media element -->
            <RowDefinition Height="80*"/>   <!-- media stream and track selection -->
            <RowDefinition Height="50"/>    <!-- status bar -->
         </Grid.RowDefinitions>

         <StackPanel Name="spMediaControl" Grid.Row="1" Orientation="Horizontal">
            <TextBlock x:Name="tbSource" Text="Source :  " FontSize="16" FontWeight="Bold" VerticalAlignment="Center" />
            <TextBox x:Name="txtMediaSource" Text="https://ecn.channel9.msdn.com/o9/content/smf/smoothcontent/elephantsdream/Elephants_Dream_1024-h264-st-aac.ism/manifest" FontSize="10" Width="700" Margin="0,4,0,10" />
            <Button x:Name="btnSetSource" Content="Set Source" Width="111" Height="43" Click="btnSetSource_Click"/>
            <Button x:Name="btnPlay" Content="Play" Width="111" Height="43" Click="btnPlay_Click"/>
            <Button x:Name="btnPause" Content="Pause"  Width="111" Height="43" Click="btnPause_Click"/>
            <Button x:Name="btnStop" Content="Stop"  Width="111" Height="43" Click="btnStop_Click"/>
            <CheckBox x:Name="chkAutoPlay" Content="Auto Play" Height="55" Width="Auto" IsChecked="{Binding AutoPlay, ElementName=mediaElement, Mode=TwoWay}"/>
            <CheckBox x:Name="chkMute" Content="Mute" Height="55" Width="67" IsChecked="{Binding IsMuted, ElementName=mediaElement, Mode=TwoWay}"/>
         </StackPanel>

         <StackPanel Name="spMediaElement" Grid.Row="2" Height="435" Width="1072"
                    HorizontalAlignment="Center" VerticalAlignment="Center">
            <MediaElement x:Name="mediaElement" Height="356" Width="924" MinHeight="225"
                          HorizontalAlignment="Center" VerticalAlignment="Center" 
                          AudioCategory="BackgroundCapableMedia" />
            <StackPanel Orientation="Horizontal">
                <Slider x:Name="sliderProgress" Width="924" Height="44"
                        HorizontalAlignment="Center" VerticalAlignment="Center"
                        PointerPressed="sliderProgress_PointerPressed"/>
                <Slider x:Name="sliderVolume" 
                        HorizontalAlignment="Right" VerticalAlignment="Center" Orientation="Vertical" 
                        Height="79" Width="148" Minimum="0" Maximum="1" StepFrequency="0.1" 
                        Value="{Binding Volume, ElementName=mediaElement, Mode=TwoWay}" 
                        ToolTipService.ToolTip="{Binding Value, RelativeSource={RelativeSource Mode=Self}}"/>
            </StackPanel>
         </StackPanel>

         <StackPanel Name="spStatus" Grid.Row="4" Orientation="Horizontal">
            <TextBlock x:Name="tbStatus" Text="Status :  " 
               FontSize="16" FontWeight="Bold" VerticalAlignment="Center" HorizontalAlignment="Center" />
            <TextBox x:Name="txtStatus" FontSize="10" Width="700" VerticalAlignment="Center"/>
         </StackPanel>
   ```
   Элемент управления MediaElement используется для воспроизведения мультимедиа. Для управления мультимедиа в следующем уроке будет использоваться ползунок с именем sliderProgress.
3. Нажмите клавиши **CTRL+S** , чтобы сохранить файл.

Элемент управления MediaElement поддерживает бесперебойную потоковую передачу контента без дополнительной настройки. Чтобы включить поддержку бесперебойной потоковой передачи, необходимо зарегистрировать обработчик потока байтов для бесперебойной потоковой передачи по расширению файла и MIME-типу.  Для регистрации используется метод MediaExtensionManager.RegisterByteStreamHandler в пространстве имен Windows.Media.

В этом XAML-файле некоторые обработчики событий связаны с элементами управления.  Такие обработчики событий необходимо определить.

### <a name="to-modify-the-code-behind-file"></a>Изменение файла кода программной части

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Просмотреть код**.
2. Добавьте следующую инструкцию using в начало файла:

    ```csharp
        using Windows.Media;
    ```

3. В начале класса **MainPage** добавьте следующие члены данных:

    ```csharp
        private MediaExtensionManager extensions = new MediaExtensionManager();
    ```

4. В конце конструктора **MainPage** добавьте следующие две строки:

    ```csharp
        extensions.RegisterByteStreamHandler("Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", ".ism", "text/xml");
        extensions.RegisterByteStreamHandler("Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", ".ism", "application/vnd.ms-sstr+xml");
    ```

5. Добавьте в конце класса **MainPage** следующий код:
   ```csharp
         # region UI Button Click Events
         private void btnPlay_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Play();
         txtStatus.Text = "MediaElement is playing ...";
         }
         private void btnPause_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Pause();
         txtStatus.Text = "MediaElement is paused";
         }
         private void btnSetSource_Click(object sender, RoutedEventArgs e)
         {

         sliderProgress.Value = 0;
         mediaElement.Source = new Uri(txtMediaSource.Text);

         if (chkAutoPlay.IsChecked == true)
         {
             txtStatus.Text = "MediaElement is playing ...";
         }
         else
         {
             txtStatus.Text = "Click the Play button to play the media source.";
         }
         }
         private void btnStop_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Stop();
         txtStatus.Text = "MediaElement is stopped";
         }
         private void sliderProgress_PointerPressed(object sender, PointerRoutedEventArgs e)
         {

         txtStatus.Text = "Seek to position " + sliderProgress.Value;
         mediaElement.Position = new TimeSpan(0, 0, (int)(sliderProgress.Value));
         }
         # endregion
   ```
   Здесь определяется обработчик события sliderProgress_PointerPressed.  Для его запуска требуется выполнить ряд дополнительных действий, которые будут рассматриваться на следующем уроке этого учебного курса.
6. Нажмите клавиши **CTRL+S** , чтобы сохранить файл.

Готовый код файла должен выглядеть следующим образом:

![Просмотр кода в Visual Studio для приложения для магазина Windows с бесперебойной потоковой передачей][CodeViewPic]

### <a name="to-compile-and-test-the-application"></a>Компиляция и тестирование приложения

1. В меню **СБОРКА** выберите **Диспетчер конфигураций**.
2. Измените параметр **Платформа активного решения** в соответствии с вашей платформой разработки.
3. Нажмите клавишу **F6** для компиляции проекта. 
4. Нажмите клавишу **F5** для запуска приложения.
5. В верхней части приложения можно выбрать URL-адрес бесперебойной потоковой передачи по умолчанию или ввести другой. 
6. Щелкните **Задать источник**. Так как по умолчанию включен параметр **Автовоспроизведение** , мультимедиа должно воспроизводиться автоматически.  Управление мультимедиа ведется с помощью кнопок **Воспроизвести**, **Пауза** и **Остановить**.  Громкость звука выбирается вертикальным ползунком.  Однако горизонтальный ползунок для управления воспроизведением мультимедиа пока не реализован полностью. 

Вы завершили урок 1.  В этом уроке с помощью элемента управления MediaElement воспроизводится контент с бесперебойной потоковой передачей.  На следующем занятии будет добавлен ползунок, который позволит управлять воспроизведением такого контента.

## <a name="lesson-2-add-a-slider-bar-to-control-the-media-progress"></a>Урок 2. Добавление ползунка для управления ходом воспроизведения мультимедиа

На занятии 1 было создано приложение для магазина Windows с элементом управления XAML MediaElement для воспроизведения контента мультимедиа с бесперебойной потоковой передачей.  Этот элемент изначально имеет такие функции, как начало, останов и приостановка воспроизведения.  На этом занятии в приложение будет добавлен еще один элемент управления — ползунок.

В этом учебном курсе для обновления положения ползунка на основе текущего положения элемента управления MediaElement будет использоваться таймер.  При воспроизведении контента в режиме реального времени также требуется обновлять время начала и окончания ползунка.  Для обработки лучше использовать событие обновления адаптивного источника.

Источники мультимедиа — это объекты, которые создают данные мультимедиа.  Средство выбора источника принимает URL-адрес или поток байтов и создает соответствующий источник мультимедиа для этого контента.  Средство выбора источника является стандартным способом для создания источников мультимедиа в приложениях. 

Это занятие содержит следующие процедуры:

1. Регистрация обработчика бесперебойной потоковой передачи 
2. Добавление обработчиков событий уровня диспетчера адаптивных источников.
3. Добавление обработчиков событий уровня адаптивного источника.
4. Добавление обработчиков событий MediaElement
5. Добавление кода, связанного с ползунком
6. Компиляция и тестирование приложения

### <a name="to-register-the-smooth-streaming-byte-stream-handler-and-pass-the-propertyset"></a>Регистрация обработчика байтового потока Smooth Streaming и передача набора свойств propertyset

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Просмотреть код**.
2. Добавьте следующую инструкцию using в начало файла:

   ```csharp
        using Microsoft.Media.AdaptiveStreaming;
   ```
3. В начале класса MainPage добавьте следующие члены данных:

   ```csharp
         private Windows.Foundation.Collections.PropertySet propertySet = new Windows.Foundation.Collections.PropertySet();             
         private IAdaptiveSourceManager adaptiveSourceManager;
   ```
4. В конструкторе **MainPage** добавьте следующий код после строки **this.Initialize Components();** и строк кода регистрации, добавленных на предыдущем занятии:

   ```csharp
        // Gets the default instance of AdaptiveSourceManager which manages Smooth 
        //Streaming media sources.
        adaptiveSourceManager = AdaptiveSourceManager.GetDefault();
        // Sets property key value to AdaptiveSourceManager default instance.
        // {A5CE1DE8-1D00-427B-ACEF-FB9A3C93DE2D}" must be hardcoded.
        propertySet["{A5CE1DE8-1D00-427B-ACEF-FB9A3C93DE2D}"] = adaptiveSourceManager;
   ```
5. В конструкторе **MainPage** измените два метода RegisterByteStreamHandler для добавления следующих параметров:

   ```csharp
         // Registers Smooth Streaming byte-stream handler for ".ism" extension and, 
         // "text/xml" and "application/vnd.ms-ss" mime-types and pass the propertyset. 
         // http://*.ism/manifest URI resources will be resolved by Byte-stream handler.
         extensions.RegisterByteStreamHandler(

            "Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", 
            ".ism", 
            "text/xml", 
            propertySet );
         extensions.RegisterByteStreamHandler(

            "Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", 
            ".ism", 
            "application/vnd.ms-sstr+xml", 
         propertySet);
   ```
6. Нажмите клавиши **CTRL+S** , чтобы сохранить файл.

### <a name="to-add-the-adaptive-source-manager-level-event-handler"></a>Добавление обработчиков событий уровня диспетчера адаптивных источников

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Просмотреть код**.
2. В классе **MainPage** добавьте следующий член данных:

   ```csharp
     private AdaptiveSource adaptiveSource = null;
   ```
3. В конце класса **MainPage** добавьте следующий обработчик событий:

   ```csharp
         # region Adaptive Source Manager Level Events
         private void mediaElement_AdaptiveSourceOpened(AdaptiveSource sender, AdaptiveSourceOpenedEventArgs args)
         {

            adaptiveSource = args.AdaptiveSource;
         }

         # endregion Adaptive Source Manager Level Events
   ```
4. В конце конструктора **MainPage** добавьте следующую строку для подписки на событие открытия адаптивного источника:

   ```csharp
         adaptiveSourceManager.AdaptiveSourceOpenedEvent += 
           new AdaptiveSourceOpenedEventHandler(mediaElement_AdaptiveSourceOpened);
   ```
5. Нажмите клавиши **CTRL+S** , чтобы сохранить файл.

### <a name="to-add-adaptive-source-level-event-handlers"></a>Добавление обработчиков событий уровня адаптивного источника

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Просмотреть код**.
2. В классе **MainPage** добавьте следующий член данных:

   ```csharp
     private AdaptiveSourceStatusUpdatedEventArgs adaptiveSourceStatusUpdate; 
     private Manifest manifestObject;
   ```
3. В конце класса **MainPage** добавьте следующие обработчики событий:

   ```csharp
         # region Adaptive Source Level Events
         private void mediaElement_ManifestReady(AdaptiveSource sender, ManifestReadyEventArgs args)
         {

            adaptiveSource = args.AdaptiveSource;
            manifestObject = args.AdaptiveSource.Manifest;
         }

         private void mediaElement_AdaptiveSourceStatusUpdated(AdaptiveSource sender, AdaptiveSourceStatusUpdatedEventArgs args)
         {

            adaptiveSourceStatusUpdate = args;
         }

         private void mediaElement_AdaptiveSourceFailed(AdaptiveSource sender, AdaptiveSourceFailedEventArgs args)
         {

            txtStatus.Text = "Error: " + args.HttpResponse;
         }

         # endregion Adaptive Source Level Events
   ```
4. В конце метода **mediaElement AdaptiveSourceOpened** добавьте следующий код, чтобы подписаться на события:

   ```csharp
         adaptiveSource.ManifestReadyEvent +=

                    mediaElement_ManifestReady;
         adaptiveSource.AdaptiveSourceStatusUpdatedEvent += 

            mediaElement_AdaptiveSourceStatusUpdated;
         adaptiveSource.AdaptiveSourceFailedEvent += 

            mediaElement_AdaptiveSourceFailed;
   ```
5. Нажмите клавиши **CTRL+S** , чтобы сохранить файл.

Эти же события доступны на уровне диспетчера адаптивных источников, который может использоваться для обработки функций, общих для всех элементов мультимедиа в приложении. Каждый адаптивный источник имеет свои собственные события, и все события AdaptiveSource будут передаваться каскадом в диспетчер AdaptiveSourceManager.

### <a name="to-add-media-element-event-handlers"></a>Добавление обработчиков событий MediaElement

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Просмотреть код**.
2. В конце класса **MainPage** добавьте следующие обработчики событий:

   ```csharp
         # region Media Element Event Handlers
         private void MediaOpened(object sender, RoutedEventArgs e)
         {

            txtStatus.Text = "MediaElement opened";
         }

         private void MediaFailed(object sender, ExceptionRoutedEventArgs e)
         {

            txtStatus.Text= "MediaElement failed: " + e.ErrorMessage;
         }

         private void MediaEnded(object sender, RoutedEventArgs e)
         {

            txtStatus.Text ="MediaElement ended.";
         }

         # endregion Media Element Event Handlers
   ```
3. В конце конструктора **MainPage** добавьте следующий код для подписки на события:

   ```csharp
         mediaElement.MediaOpened += MediaOpened;
         mediaElement.MediaEnded += MediaEnded;
         mediaElement.MediaFailed += MediaFailed;
   ```
4. Нажмите клавиши **CTRL+S** , чтобы сохранить файл.

### <a name="to-add-slider-bar-related-code"></a>Добавление кода, связанного с ползунком

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Просмотреть код**.
2. Добавьте следующую инструкцию using в начало файла:

   ```csharp
        using Windows.UI.Core;
   ```
3. В классе **MainPage** добавьте следующий член данных:

   ```csharp
         public static CoreDispatcher _dispatcher;
         private DispatcherTimer sliderPositionUpdateDispatcher;
   ```
4. В конце конструктора **MainPage** добавьте следующий код:

   ```csharp
         _dispatcher = Window.Current.Dispatcher;
         PointerEventHandler pointerpressedhandler = new PointerEventHandler(sliderProgress_PointerPressed);
         sliderProgress.AddHandler(Control.PointerPressedEvent, pointerpressedhandler, true);    
   ```
5. Добавьте в конце класса **MainPage** следующий код:

   ```csharp
         # region sliderMediaPlayer
         private double SliderFrequency(TimeSpan timevalue)
         {

            long absvalue = 0;
            double stepfrequency = -1;

            if (manifestObject != null)
            {
                absvalue = manifestObject.Duration - (long)manifestObject.StartTime;
            }
            else
            {
                absvalue = mediaElement.NaturalDuration.TimeSpan.Ticks;
            }

            TimeSpan totalDVRDuration = new TimeSpan(absvalue);

            if (totalDVRDuration.TotalMinutes >= 10 && totalDVRDuration.TotalMinutes < 30)
            {
               stepfrequency = 10;
            }
            else if (totalDVRDuration.TotalMinutes >= 30 
                     && totalDVRDuration.TotalMinutes < 60)
            {
                stepfrequency = 30;
            }
            else if (totalDVRDuration.TotalHours >= 1)
            {
                stepfrequency = 60;
            }

            return stepfrequency;
         }

         void updateSliderPositionoNTicks(object sender, object e)
         {

            sliderProgress.Value = mediaElement.Position.TotalSeconds;
         }

         public void setupTimer()
         {

            sliderPositionUpdateDispatcher = new DispatcherTimer();
            sliderPositionUpdateDispatcher.Interval = new TimeSpan(0, 0, 0, 0, 300);
            startTimer();
         }

         public void startTimer()
         {

            sliderPositionUpdateDispatcher.Tick += updateSliderPositionoNTicks;
            sliderPositionUpdateDispatcher.Start();
         }

         // Slider start and end time must be updated in case of live content
         public async void setSliderStartTime(long startTime)
         {

            await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                TimeSpan timespan = new TimeSpan(adaptiveSourceStatusUpdate.StartTime);
                double absvalue = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero);
                sliderProgress.Minimum = absvalue;
            });
         }

         // Slider start and end time must be updated in case of live content
         public async void setSliderEndTime(long startTime)
         {

            await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                TimeSpan timespan = new TimeSpan(adaptiveSourceStatusUpdate.EndTime);
                double absvalue = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero);
                sliderProgress.Maximum = absvalue;
            });
         }

         # endregion sliderMediaPlayer
   ```

   > [!NOTE]
   > CoreDispatcher используется для внесения изменений в поток пользовательского интерфейса из других потоков. В случае возникновения узких мест в потоке-отправителе разработчик может использовать отправитель, предоставленный элементом пользовательского интерфейса, который планируется обновить.  Пример:

   ```csharp
         await sliderProgress.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () => { TimeSpan 

         timespan = new TimeSpan(adaptiveSourceStatusUpdate.EndTime); 
         double absvalue  = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero); 

         sliderProgress.Maximum = absvalue; }); 
   ```
6. В конце метода **mediaElement_AdaptiveSourceStatusUpdated** добавьте следующий код:

   ```csharp
         setSliderStartTime(args.StartTime);
         setSliderEndTime(args.EndTime);
   ```
7. В конце метода **MediaOpened** добавьте следующий код:

   ```csharp
         sliderProgress.StepFrequency = SliderFrequency(mediaElement.NaturalDuration.TimeSpan);
         sliderProgress.Width = mediaElement.Width;
         setupTimer();
   ```
8. Нажмите клавиши **CTRL+S** , чтобы сохранить файл.

### <a name="to-compile-and-test-the-application"></a>Компиляция и тестирование приложения

1. Нажмите клавишу **F6** для компиляции проекта. 
2. Нажмите клавишу **F5** для запуска приложения.
3. В верхней части приложения можно выбрать URL-адрес бесперебойной потоковой передачи по умолчанию или ввести другой. 
4. Щелкните **Задать источник**. 
5. Протестируйте работу ползунка.

Вы завершили урок 2.  В этом уроке в приложение был добавлен ползунок. 

## <a name="lesson-3-select-smooth-streaming-streams"></a>Урок 3. Выбор потоков для бесперебойной потоковой передачи
Бесперебойная потоковая передача способна передавать контент с аудиодорожками на нескольких языках, которые могут быть выбраны просматривающими.  На этом занятии предстоит включить средства просмотра для выбора потоков. Это занятие содержит следующие процедуры:

1. Изменение XAML-файла
2. Изменение кода в файле
3. Компиляция и тестирование приложения

### <a name="to-modify-the-xaml-file"></a>Изменение XAML-файла

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Конструктор представлений**.
2. Найдите &lt;Grid.RowDefinitions&gt; и измените RowDefinitions следующим образом:

   ```xml
         <Grid.RowDefinitions>            
            <RowDefinition Height="20"/>
            <RowDefinition Height="50"/>
            <RowDefinition Height="100"/>
            <RowDefinition Height="80"/>
            <RowDefinition Height="50"/>
         </Grid.RowDefinitions>
   ```
3. Внутри тегов &lt;Grid&gt;&lt;/Grid&gt; добавьте следующий код, чтобы определить элемент управления типа "поле со списком", который позволит пользователям просматривать список доступных потоков и выбирать нужные:

   ```xml
         <Grid Name="gridStreamAndBitrateSelection" Grid.Row="3">
            <Grid.RowDefinitions>
                <RowDefinition Height="300"/>
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="250*"/>
                <ColumnDefinition Width="250*"/>
            </Grid.ColumnDefinitions>
            <StackPanel Name="spStreamSelection" Grid.Row="1" Grid.Column="0">
                <StackPanel Orientation="Horizontal">
                    <TextBlock Name="tbAvailableStreams" Text="Available Streams:" FontSize="16" VerticalAlignment="Center"></TextBlock>
                    <Button Name="btnChangeStreams" Content="Submit" Click="btnChangeStream_Click"/>
                </StackPanel>
                <ListBox x:Name="lbAvailableStreams" Height="200" Width="200" Background="Transparent" HorizontalAlignment="Left" 
                    ScrollViewer.VerticalScrollMode="Enabled" ScrollViewer.VerticalScrollBarVisibility="Visible">
                    <ListBox.ItemTemplate>
                        <DataTemplate>
                            <CheckBox Content="{Binding Name}" IsChecked="{Binding isChecked, Mode=TwoWay}" />
                        </DataTemplate>
                    </ListBox.ItemTemplate>
                </ListBox>
            </StackPanel>
         </Grid>
   ```
4. Нажмите клавишу **CTRL+S** , чтобы сохранить изменения.

### <a name="to-modify-the-code-behind-file"></a>Изменение файла кода программной части

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Просмотреть код**.
2. В пространстве имен SSPlayer добавьте следующий новый класс:

   ```csharp
        #region class Stream
   
        public class Stream
        {
            private IManifestStream stream;
            public bool isCheckedValue;
            public string name;
   
            public string Name
            {
                get { return name; }
                set { name = value; }
            }
   
            public IManifestStream ManifestStream
            {
                get { return stream; }
                set { stream = value; }
            }
   
            public bool isChecked
            {
                get { return isCheckedValue; }
                set
                {
                    // mMke the video stream always checked.
                    if (stream.Type == MediaStreamType.Video)
                    {
                        isCheckedValue = true;
                    }
                    else
                    {
                        isCheckedValue = value;
                    }
                }
            }
   
            public Stream(IManifestStream streamIn)
            {
                stream = streamIn;
                name = stream.Name;
            }
        }
        #endregion class Stream
   ```
3. В начале класса MainPage добавьте следующие определения переменных:

   ```csharp
         private List<Stream> availableStreams;
         private List<Stream> availableAudioStreams;
         private List<Stream> availableTextStreams;
         private List<Stream> availableVideoStreams;
   ```
4. Добавьте в класс MainPage следующую область:
   ```csharp
        #region stream selection
        ///<summary>
        ///Functionality to select streams from IManifestStream available streams
        /// </summary>
   
        // This function is called from the mediaElement_ManifestReady event handler 
        // to retrieve the streams and populate them to the local data members.
        public void getStreams(Manifest manifestObject)
        {
            availableStreams = new List<Stream>();
            availableVideoStreams = new List<Stream>();
            availableAudioStreams = new List<Stream>();
            availableTextStreams = new List<Stream>();
   
            try
            {
                for (int i = 0; i<manifestObject.AvailableStreams.Count; i++)
                {
                    Stream newStream = new Stream(manifestObject.AvailableStreams[i]);
                    newStream.isChecked = false;
   
                    //populate the stream lists based on the types
                    availableStreams.Add(newStream);
   
                    switch (newStream.ManifestStream.Type)
                    {
                        case MediaStreamType.Video:
                            availableVideoStreams.Add(newStream);
                            break;
                        case MediaStreamType.Audio:
                            availableAudioStreams.Add(newStream);
                            break;
                        case MediaStreamType.Text:
                            availableTextStreams.Add(newStream);
                            break;
                    }
   
                    // Select the default selected streams from the manifest.
                    for (int j = 0; j<manifestObject.SelectedStreams.Count; j++)
                    {
                        string selectedStreamName = manifestObject.SelectedStreams[j].Name;
                        if (selectedStreamName.Equals(newStream.Name))
                        {
                            newStream.isChecked = true;
                            break;
                        }
                    }
                }
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
   
        // This function set the list box ItemSource
        private async void refreshAvailableStreamsListBoxItemSource()
        {
            try
            {
                //update the stream check box list on the UI
                await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, ()
                    => { lbAvailableStreams.ItemsSource = availableStreams; });
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
   
        // This function creates a selected streams list
        private void createSelectedStreamsList(List<IManifestStream> selectedStreams)
        {
            bool isOneVideoSelected = false;
            bool isOneAudioSelected = false;
   
            // Only one video stream can be selected
            for (int j = 0; j<availableVideoStreams.Count; j++)
            {
                if (availableVideoStreams[j].isChecked && (!isOneVideoSelected))
                {
                    selectedStreams.Add(availableVideoStreams[j].ManifestStream);
                    isOneVideoSelected = true;
                }
            }
   
            // Select the first video stream from the list if no video stream is selected
            if (!isOneVideoSelected)
            {
                availableVideoStreams[0].isChecked = true;
                selectedStreams.Add(availableVideoStreams[0].ManifestStream);
            }
   
            // Only one audio stream can be selected
            for (int j = 0; j<availableAudioStreams.Count; j++)
            {
                if (availableAudioStreams[j].isChecked && (!isOneAudioSelected))
                {
                    selectedStreams.Add(availableAudioStreams[j].ManifestStream);
                    isOneAudioSelected = true;
                    txtStatus.Text = "The audio stream is changed to " + availableAudioStreams[j].ManifestStream.Name;
                }
            }
   
            // Select the first audio stream from the list if no audio steam is selected.
            if (!isOneAudioSelected)
            {
                availableAudioStreams[0].isChecked = true;
                selectedStreams.Add(availableAudioStreams[0].ManifestStream);
            }
   
            // Multiple text streams are supported.
            for (int j = 0; j < availableTextStreams.Count; j++)
            {
                if (availableTextStreams[j].isChecked)
                {
                    selectedStreams.Add(availableTextStreams[j].ManifestStream);
                }
            }
        }
   
        // Change streams on a smooth streaming presentation with multiple video streams.
        private async void changeStreams(List<IManifestStream> selectStreams)
        {
            try
            {
                IReadOnlyList<IStreamChangedResult> returnArgs =
                    await manifestObject.SelectStreamsAsync(selectStreams);
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
        #endregion stream selection
   ```
5. Найдите метод mediaElement_ManifestReady, измените следующий код в конце функции:
   ```csharp
        getStreams(manifestObject);
        refreshAvailableStreamsListBoxItemSource();
   ```
    Таким образом, если манифест MediaElement готов, код получает список доступных потоков и заполняет поле со списком пользовательского интерфейса этими значениями.
6. В классе MainPage найдите область событий щелчка кнопок пользовательского интерфейса и добавьте следующее определение функции:
   ```csharp
        private void btnChangeStream_Click(object sender, RoutedEventArgs e)
        {
            List<IManifestStream> selectedStreams = new List<IManifestStream>();
   
            // Create a list of the selected streams
            createSelectedStreamsList(selectedStreams);
   
            // Change streams on the presentation
            changeStreams(selectedStreams);
        }
   ```

### <a name="to-compile-and-test-the-application"></a>Компиляция и тестирование приложения

1. Нажмите клавишу **F6** для компиляции проекта. 
2. Нажмите клавишу **F5** для запуска приложения.
3. В верхней части приложения можно выбрать URL-адрес бесперебойной потоковой передачи по умолчанию или ввести другой. 
4. Щелкните **Задать источник**. 
5. Язык по умолчанию — audio_eng. Попробуйте переключиться между audio_eng и audio_es. Каждый раз при выборе нового потока необходимо нажать кнопку "Отправить".

Вы завершили урок 3.  В этом уроке были добавлены функциональные возможности для выбора потоков.

## <a name="lesson-4-select-smooth-streaming-tracks"></a>Урок 4. Выбор дорожек для бесперебойной потоковой передачи

Представление бесперебойной потоковой передачи может содержать несколько видеофайлов, зашифрованных с разными уровнями качества (скоростью) и разрешениями. На этом занятии предстоит включить возможность выбора потоков для пользователей. Это занятие содержит следующие процедуры:

1. Изменение XAML-файла
2. Изменение кода в файле
3. Компиляция и тестирование приложения

### <a name="to-modify-the-xaml-file"></a>Изменение XAML-файла

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Конструктор представлений**.
2. Найдите тег &lt;Grid&gt; с именем **gridStreamAndBitrateSelection** и добавьте следующий код в конце этого тега:
   ```xml
         <StackPanel Name="spBitRateSelection" Grid.Row="1" Grid.Column="1">
         <StackPanel Orientation="Horizontal">
             <TextBlock Name="tbBitRate" Text="Available Bitrates:" FontSize="16" VerticalAlignment="Center"/>
             <Button Name="btnChangeTracks" Content="Submit" Click="btnChangeTrack_Click" />
         </StackPanel>
         <ListBox x:Name="lbAvailableVideoTracks" Height="200" Width="200" Background="Transparent" HorizontalAlignment="Left" 
                  ScrollViewer.VerticalScrollMode="Enabled" ScrollViewer.VerticalScrollBarVisibility="Visible">
             <ListBox.ItemTemplate>
                 <DataTemplate>
                     <CheckBox Content="{Binding Bitrate}" IsChecked="{Binding isChecked, Mode=TwoWay}"/>
                 </DataTemplate>
             </ListBox.ItemTemplate>
         </ListBox>
         </StackPanel>
   ```
3. Нажмите **CTRL+S**, чтобы сохранить изменения.

### <a name="to-modify-the-code-behind-file"></a>Изменение файла кода программной части

1. В обозревателе решений щелкните правой кнопкой мыши файл **MainPage.xaml** и выберите команду **Просмотреть код**.
2. В пространстве имен SSPlayer добавьте следующий новый класс:
   ```csharp
        #region class Track
        public class Track
        {
            private IManifestTrack trackInfo;
            public string _bitrate;
            public bool isCheckedValue;
   
            public IManifestTrack TrackInfo
            {
                get { return trackInfo; }
                set { trackInfo = value; }
            }
   
            public string Bitrate
            {
                get { return _bitrate; }
                set { _bitrate = value; }
            }
   
            public bool isChecked
            {
                get { return isCheckedValue; }
                set
                {
                    isCheckedValue = value;
                }
            }
   
            public Track(IManifestTrack trackInfoIn)
            {
                trackInfo = trackInfoIn;
                _bitrate = trackInfoIn.Bitrate.ToString();
            }
            //public Track() { }
        }
        #endregion class Track
   ```
3. В начале класса MainPage добавьте следующие определения переменных:
   ```csharp
        private List<Track> availableTracks;
   ```
4. Добавьте в класс MainPage следующую область:
   ```csharp
        #region track selection
        /// <summary>
        /// Functionality to select video streams
        /// </summary>
   
        /// This Function gets the tracks for the selected video stream
        public void getTracks(Manifest manifestObject)
        {
            availableTracks = new List<Track>();
   
            IManifestStream videoStream = getVideoStream();
            IReadOnlyList<IManifestTrack> availableTracksLocal = videoStream.AvailableTracks;
            IReadOnlyList<IManifestTrack> selectedTracksLocal = videoStream.SelectedTracks;
   
            try
            {
                for (int i = 0; i < availableTracksLocal.Count; i++)
                {
                    Track thisTrack = new Track(availableTracksLocal[i]);
                    thisTrack.isChecked = true;
   
                    for (int j = 0; j < selectedTracksLocal.Count; j++)
                    {
                        string selectedTrackName = selectedTracksLocal[j].Bitrate.ToString();
                        if (selectedTrackName.Equals(thisTrack.Bitrate))
                        {
                            thisTrack.isChecked = true;
                            break;
                        }
                    }
                    availableTracks.Add(thisTrack);
                }
            }
            catch (Exception e)
            {
                txtStatus.Text = e.Message;
            }
        }
   
        // This function gets the video stream that is playing
        private IManifestStream getVideoStream()
        {
            IManifestStream videoStream = null;
            for (int i = 0; i < manifestObject.SelectedStreams.Count; i++)
            {
                if (manifestObject.SelectedStreams[i].Type == MediaStreamType.Video)
                {
                    videoStream = manifestObject.SelectedStreams[i];
                    break;
                }
            }
            return videoStream;
        }
   
        // This function set the UI list box control ItemSource
        private async void refreshAvailableTracksListBoxItemSource()
        {
            try
            {
                // Update the track check box list on the UI 
                await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, ()
                    => { lbAvailableVideoTracks.ItemsSource = availableTracks; });
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }        
        }
   
        // This function creates a list of the selected tracks.
        private void createSelectedTracksList(List<IManifestTrack> selectedTracks)
        {
            // Create a list of selected tracks
            for (int j = 0; j < availableTracks.Count; j++)
            {
                if (availableTracks[j].isCheckedValue == true)
                {
                    selectedTracks.Add(availableTracks[j].TrackInfo);
                }
            }
        }
   
        // This function selects the tracks based on user selection 
        private void changeTracks(List<IManifestTrack> selectedTracks)
        {
            IManifestStream videoStream = getVideoStream();
            try
            {
                videoStream.SelectTracks(selectedTracks);
            }
            catch (Exception ex)
            {
                txtStatus.Text = ex.Message;
            }
        }
        #endregion track selection
   ```
5. Найдите метод mediaElement_ManifestReady, измените следующий код в конце функции:
   ```csharp
         getTracks(manifestObject);
         refreshAvailableTracksListBoxItemSource();
   ```
6. В классе MainPage найдите область событий щелчка кнопок пользовательского интерфейса и добавьте следующее определение функции:
   ```csharp
         private void btnChangeStream_Click(object sender, RoutedEventArgs e)
         {
            List<IManifestStream> selectedStreams = new List<IManifestStream>();

            // Create a list of the selected streams
            createSelectedStreamsList(selectedStreams);

            // Change streams on the presentation
            changeStreams(selectedStreams);
         }
   ```
   
### <a name="to-compile-and-test-the-application"></a>Компиляция и тестирование приложения

1. Нажмите клавишу **F6** для компиляции проекта. 
2. Нажмите клавишу **F5** для запуска приложения.
3. В верхней части приложения можно выбрать URL-адрес бесперебойной потоковой передачи по умолчанию или ввести другой. 
4. Щелкните **Задать источник**. 
5. По умолчанию выбраны все дорожки потока видеоданных. Чтобы поэкспериментировать со скоростью, выберите минимально возможную скорость, а затем — максимально возможную. После каждого изменения необходимо нажать кнопку "Отправить".  Вы увидите, как изменилось качество видеоизображения.

Вы завершили урок 4.  В этом уроке были добавлены функциональные возможности для выбора дорожек.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="other-resources"></a>Другие ресурсы:
* [Создание приложения для Windows 8 с потоковой передачей Smooth Streaming на JavaScript с расширенными возможностями](https://blogs.iis.net/cenkd/archive/2012/08/10/how-to-build-a-smooth-streaming-windows-8-javascript-application-with-advanced-features.aspx)
* [Технический обзор Smooth Streaming](https://www.iis.net/learn/media/on-demand-smooth-streaming/smooth-streaming-technical-overview)

[PlayerApplication]: ./media/media-services-build-smooth-streaming-apps/SSClientWin8-1.png
[CodeViewPic]: ./media/media-services-build-smooth-streaming-apps/SSClientWin8-2.png
