---
title: Руководство по Создание приложения для перевода текста с помощью WPF и C# — Переводчик
titleSuffix: Azure Cognitive Services
description: В рамках этого руководства вы создадите приложение WPF для перевода текста, определения языка и проверки орфографии с помощью ключа одной подписки.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: tutorial
ms.date: 05/26/2020
ms.author: lajanuar
ms.custom: devx-track-csharp
ms.openlocfilehash: 0665dcbc8de518c5759c52a8fc3aec26859566d6
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101728014"
---
# <a name="tutorial-create-a-translation-app-with-wpf"></a>Руководство по Создание приложения для перевода текста с помощью WPF

В рамках этого руководства вы создадите приложение [Windows Presentation Foundation (WPF)](/visualstudio/designers/getting-started-with-wpf), в котором службы Azure Cognitive Services используются для перевода текста, определения языка и проверки орфографии с помощью ключа одной подписки. В частности, приложение будет вызывать API-интерфейсы из Переводчика и средства [Проверка орфографии Bing](https://azure.microsoft.com/services/cognitive-services/spell-check/).

Что такое WPF? Это инфраструктура пользовательского интерфейса, позволяющая создавать классические клиентские приложения. Платформа разработки WPF поддерживает широкий набор функций для разработки приложений, включая модель приложения, ресурсы, элементы управления, графику, макет, привязку данных, документы и безопасность. Это подмножество платформы .NET. Поэтому, если вы ранее создавали приложения на платформе .NET с помощью ASP.NET или Windows Forms, у вас не возникнет трудностей в процессе программирования. Для реализации декларативной модели программирования приложений, которая рассматривается в следующих разделах, в WPF используется расширяемый язык разметки приложений (XAML).

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * создание проекта WPF в Visual Studio;
> * добавление сборок приложения и пакетов NuGet в проект;
> * создание пользовательского интерфейса для приложения с помощью XAML;
> * использование Переводчика для получения списка языков,перевода текста и определения исходного языка;
> * применение API "Проверка орфографии Bing" для проверки вводимого текста и повышения точности перевода;
> * Запуск приложения WPF

### <a name="cognitive-services-used-in-this-tutorial"></a>Службы Cognitive Services, используемые в этом руководстве

В этом списке представлены службы Cognitive Services, которые используются в этом руководстве. Чтобы получить дополнительные сведения о каждой функции, перейдите в справочник по API.

| Служба | Компонент | Описание |
|---------|---------|-------------|
| API перевода | [Get Languages](./reference/v3-0-languages.md) | Получение полного списка поддерживаемых языков для перевода текста. |
| API перевода | [Translate](./reference/v3-0-translate.md) | Перевод текста на 90 языков и диалектов. |
| API перевода | [Detect](./reference/v3-0-detect.md) | Определение языка исходного текста. Для определения используется показатель достоверности. |
| API Проверки орфографии Bing | [Spell Check](/rest/api/cognitiveservices/bing-spell-check-api-v7-reference) | Исправление орфографических ошибок для повышения точности перевода. |

## <a name="prerequisites"></a>Предварительные требования

Для этого вам потребуются следующие компоненты:

* подписка Azure Cognitive Services; [ключ Cognitive Services](../cognitive-services-apis-create-account.md#create-a-new-azure-cognitive-services-resource);
* компьютер с Windows;
* [Visual Studio 2019](https://www.visualstudio.com/downloads/) — Community или Enterprise

> [!NOTE]
> Для работы с этим руководством рекомендуем создавать подписки в западной части США. Иначе при выполнении упражнения вам потребуется изменить конечные точки и регионы в коде.  

## <a name="create-a-wpf-app-in-visual-studio"></a>Создание приложения WPF в Visual Studio

Прежде всего, настройте проект в Visual Studio.

1. Запустите Visual Studio. Выберите **Создать новый проект**.
1. В окне **Создать новый проект**, найдите и выберите **Приложение WPF (.NET Framework)** . Чтобы ограничить параметры, вы можете выбрать C# в параметре **Язык**.
1. Нажмите кнопку **Далее** и задайте имя проекту `MSTranslatorDemo`.
1. Установите платформу версии **.NET Framework 4.7.2** или более поздней и выберите **Создать**.
   ![Введите имя и версию платформы в Visual Studio](media/name-wpf-project-visual-studio.png)

Вы создали проект. Обратите внимание на две открытые вкладки: `MainWindow.xaml` и `MainWindow.xaml.cs`. В рамках этого руководства мы будем добавлять код в эти два файла. Мы изменим `MainWindow.xaml` для пользовательского интерфейса приложения. Мы изменим `MainWindow.xaml.cs` для наших вызовов Переводчика и Проверки орфографии Bing.
   ![Проверка среды](media/blank-wpf-project.png)

В рамках следующего раздела мы добавим в проект сборки и пакет NuGet, чтобы расширить возможности проекта, например добавить функцию анализа JSON.

## <a name="add-references-and-nuget-packages-to-your-project"></a>Добавление ссылок и пакетов NuGet в проект

Для работы проекта потребуется несколько сборок .NET Framework и файл NewtonSoft.Json, которые необходимо установить с помощью диспетчера пакетов NuGet.

### <a name="add-net-framework-assemblies"></a>Добавление сборок платформы .NET

Добавим в проект сборки для сериализации и десериализации объектов, а также управления HTTP-запросами и ответами.

1. Найдите проект в обозревателе решений Visual Studio. Чтобы открыть **Диспетчер ссылок**, щелкните правой кнопкой мыши проект, а затем выберите **Добавить > Ссылка**.
1. На вкладке **Сборки** перечислены все сборки платформы .NET, для которых можно указать ссылки. Используйте панель поиска в правом верхнем углу для поиска ссылок.
   ![Добавление ссылок для сборки](media/add-assemblies-2019.png)
1. Выберите следующие ссылки для своего проекта:
   * [System.Runtime.Serialization](/dotnet/api/system.runtime.serialization)
   * [System.Web](/dotnet/api/system.web);
   * System.Web.Extensions
   * [System.Windows](/dotnet/api/system.windows)
1. После добавления этих ссылок в проект вы можете нажать кнопку **ОК**, чтобы закрыть **диспетчер ссылок**.

> [!NOTE]
> Дополнительные сведения о ссылках на сборки см. в [руководстве по добавлению или удалению ссылок с помощью диспетчера ссылок](/visualstudio/ide/how-to-add-or-remove-references-by-using-the-reference-manager).

### <a name="install-newtonsoftjson"></a>Установка Newtonsoft.Json

В этом приложении для десериализации объектов JSON будет использоваться файл NewtonSoft.Json. Чтобы установить пакет, следуйте инструкциям ниже.

1. Найдите проект в обозревателе решений Visual Studio и щелкните проект правой кнопкой мыши. Выберите **Manage NuGet Packages...** (Управление пакетами NuGet...).
1. Перейдите на вкладку **Обзор**.
1. Введите [NewtonSoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/) в строке поиска.

    ![Поиск и установка файла NewtonSoft.Json](media/nuget-package-manager.png)

1. Выберите пакет и нажмите **Установить**.
1. По завершении установки закройте вкладку.

## <a name="create-a-wpf-form-using-xaml"></a>Создание формы WPF с помощью XAML

Чтобы работать с приложением, вам потребуется пользовательский интерфейс. С помощью XAML мы создадим форму, которая позволит пользователям выбирать исходные языки и языки перевода, вводить текст и отображать результат перевода.

Давайте посмотрим, что у нас получается.

![Пользовательский интерфейс XAML для WPF](media/translator-text-csharp-xaml.png)

Пользовательский интерфейс состоит из следующих компонентов:

| Имя | Тип | Описание |
|------|------|-------------|
| `FromLanguageComboBox` | ComboBox | Содержит список языков, поддерживаемых Microsoft Translator для перевода текста. Пользователь выбирает язык, с которого нужно переводить. |
| `ToLanguageComboBox` | ComboBox | Содержит тот же список языков, что и `FromComboBox`, но используется для выбора языка перевода. |
| `TextToTranslate` | TextBox | Позволяет пользователю вводить текст для перевода. |
| `TranslateButton` | Кнопка | Воспользуйтесь этой кнопкой, чтобы перевести текст. |
| `TranslatedTextLabel` | Метка | Позволяет отобразить перевод. |
| `DetectedLanguageLabel` | Метка | Позволяет отобразить определенный язык текста, который необходимо перевести (`TextToTranslate`). |

> [!NOTE]
> Мы создаем эту форму с помощью исходного кода XAML. Но вы можете создавать формы с помощью редактора в Visual Studio.

Давайте добавим код в проект.

1. В Visual Studio выберите вкладку для файла `MainWindow.xaml`.
1. Скопируйте этот код в проект, а затем выберите **Файл > Сохранить MainWindow.xaml**, чтобы сохранить изменения.
   ```xaml
   <Window x:Class="MSTranslatorDemo.MainWindow"
           xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
           xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
           xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
           xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
           xmlns:local="clr-namespace:MSTranslatorDemo"
           mc:Ignorable="d"
           Title="Microsoft Translator" Height="400" Width="700" BorderThickness="0">
       <Grid>
           <Label x:Name="label" Content="Microsoft Translator" HorizontalAlignment="Left" Margin="39,6,0,0" VerticalAlignment="Top" Height="49" FontSize="26.667"/>
           <TextBox x:Name="TextToTranslate" HorizontalAlignment="Left" Height="23" Margin="42,160,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="600" FontSize="14" TabIndex="3"/>
           <Label x:Name="EnterTextLabel" Content="Text to translate:" HorizontalAlignment="Left" Margin="40,129,0,0" VerticalAlignment="Top" FontSize="14"/>
           <Label x:Name="toLabel" Content="Translate to:" HorizontalAlignment="Left" Margin="304,58,0,0" VerticalAlignment="Top" FontSize="14"/>

           <Button x:Name="TranslateButton" Content="Translate" HorizontalAlignment="Left" Margin="39,206,0,0" VerticalAlignment="Top" Width="114" Height="31" Click="TranslateButton_Click" FontSize="14" TabIndex="4" IsDefault="True"/>
           <ComboBox x:Name="ToLanguageComboBox"
                   HorizontalAlignment="Left"
                   Margin="306,88,0,0"
                   VerticalAlignment="Top"
                   Width="175" FontSize="14" TabIndex="2">

           </ComboBox>
           <Label x:Name="fromLabel" Content="Translate from:" HorizontalAlignment="Left" Margin="40,58,0,0" VerticalAlignment="Top" FontSize="14"/>
           <ComboBox x:Name="FromLanguageComboBox"
               HorizontalAlignment="Left"
               Margin="42,88,0,0"
               VerticalAlignment="Top"
               Width="175" FontSize="14" TabIndex="1"/>
           <Label x:Name="TranslatedTextLabel" Content="Translation is displayed here." HorizontalAlignment="Left" Margin="39,255,0,0" VerticalAlignment="Top" Width="620" FontSize="14" Height="85" BorderThickness="0"/>
           <Label x:Name="DetectedLanguageLabel" Content="Autodetected language is displayed here." HorizontalAlignment="Left" Margin="39,288,0,0" VerticalAlignment="Top" Width="620" FontSize="14" Height="84" BorderThickness="0"/>
       </Grid>
   </Window>
   ```
После этого в Visual Studio откроется предварительная версия пользовательского интерфейса приложения. Интерфейс должен выглядеть, как на рисунке выше.

Теперь форма готова. Напишем несколько строк кода, чтобы использовать Перевод текстов и Проверку орфографии Bing.

> [!NOTE]
> Вы можете оптимизировать эту форму или создать собственную.

## <a name="create-your-app"></a>Создание приложения

В файле `MainWindow.xaml.cs` содержится код, управляющий приложением. В рамках следующих разделов мы добавим код для заполнения раскрывающихся меню и вызова нескольких API-интерфейсов, предоставляемых Переводчиком и Проверкой орфографии Bing.

* Во время запуска программы и инициализации окна `MainWindow` вызывается метод `Languages` Переводчика, чтобы извлечь и заполнить раскрывающиеся меню выбора языков. Этот метод однократно вызывается в начале каждого сеанса.
* Если нажать кнопку **Перевести**, приложение получает сведения о выбранном пользователем языке и текст. Для этого текста выполняется проверка орфографии и перевод, после чего перевод и сведения об определенном языке отображаются пользователю.
  * Для перевода текста из `TextToTranslate` вызывается метод `Translate` Переводчика. В этом запросе также передаются языки `to` и `from`, выбранные с помощью раскрывающихся меню.
  * Чтобы определить язык текста `TextToTranslate`, вызывается метод `Detect` API Переводчика.
  * Для проверки орфографии `TextToTranslate` и исправления ошибок используется Проверка орфографии Bing.

Все наши проекты инкапсулируются в класс `MainWindow : Window`. Сначала добавим код, чтобы указать ключ подписки, объявить конечные точки для Переводчика и Проверки орфографии Bing, а также инициализировать приложение.

1. В Visual Studio выберите вкладку для файла `MainWindow.xaml.cs`.
1. Замените заполненные инструкции `using` следующим кодом:  
   ```csharp
   using System;
   using System.Windows;
   using System.Net;
   using System.Net.Http;
   using System.IO;
   using System.Collections.Generic;
   using System.Linq;
   using System.Text;
   using Newtonsoft.Json;
   ```
1. Найдите класс `MainWindow : Window` и замените его следующим кодом:
   ```csharp
   {
       // This sample uses the Cognitive Services subscription key for all services. To learn more about
       // authentication options, see: https://docs.microsoft.com/azure/cognitive-services/authentication.
       const string COGNITIVE_SERVICES_KEY = "YOUR_COG_SERVICES_KEY";
       // Endpoints for Translator and Bing Spell Check
       public static readonly string TEXT_TRANSLATION_API_ENDPOINT = "https://api.cognitive.microsofttranslator.com/{0}?api-version=3.0";
       const string BING_SPELL_CHECK_API_ENDPOINT = "https://westus.api.cognitive.microsoft.com/bing/v7.0/spellcheck/";
       // An array of language codes
       private string[] languageCodes;

       // Dictionary to map language codes from friendly name (sorted case-insensitively on language name)
       private SortedDictionary<string, string> languageCodesAndTitles =
           new SortedDictionary<string, string>(Comparer<string>.Create((a, b) => string.Compare(a, b, true)));

       // Global exception handler to display error message and exit
       private static void HandleExceptions(object sender, UnhandledExceptionEventArgs args)
       {
           Exception e = (Exception)args.ExceptionObject;
           MessageBox.Show("Caught " + e.Message, "Error", MessageBoxButton.OK, MessageBoxImage.Error);
           System.Windows.Application.Current.Shutdown();
       }
       // MainWindow constructor
       public MainWindow()
       {
           // Display a message if unexpected error is encountered
           AppDomain.CurrentDomain.UnhandledException += new UnhandledExceptionEventHandler(HandleExceptions);

           if (COGNITIVE_SERVICES_KEY.Length != 32)
           {
               MessageBox.Show("One or more invalid API subscription keys.\n\n" +
                   "Put your keys in the *_API_SUBSCRIPTION_KEY variables in MainWindow.xaml.cs.",
                   "Invalid Subscription Key(s)", MessageBoxButton.OK, MessageBoxImage.Error);
               System.Windows.Application.Current.Shutdown();
           }
           else
           {
               // Start GUI
               InitializeComponent();
               // Get languages for drop-downs
               GetLanguagesForTranslate();
               // Populate drop-downs with values from GetLanguagesForTranslate
               PopulateLanguageMenus();
           }
       }
   // NOTE:
   // In the following sections, we'll add code below this.
   }
   ```
1. Добавьте ключ подписки для службы Cognitive Services и сохраните его.

В этом блоке кода мы объявили две переменные-члена класса, в которых содержатся данные о языках, доступных для перевода:

| Переменная | Тип | Описание |
|----------|------|-------------|
|`languageCodes` | Массив строк |Кэширует коды языков. Служба Translator использует короткие коды, например `en` для английского языка, чтобы идентифицировать язык. |
|`languageCodesAndTitles` | Отсортированный словарь | Отображает понятные имена в пользовательском интерфейсе вместо коротких кодов, используемых в API. Они сортируются в алфавитном порядке без учета регистра. |

В конструкторе `MainWindow` добавим обработчик ошибок с помощью инструкции `HandleExceptions`. Эта обработка ошибок обеспечивает отображение предупреждения, если исключение не обработано. Затем выполняется проверка, позволяющая убедиться, что введенный ключ подписки содержит 32 символа. Если длина ключа больше или меньше 32 символов, возникает ошибка.

Если у нас ключи правильной длины, после вызова `InitializeComponent()` пользовательский интерфейс находит, загружает и создает описание XAML основного окна приложения.

Наконец, добавим код вызова методов для получения списка языков для перевода и заполнения раскрывающихся меню в пользовательском интерфейсе приложения. Не стоит беспокоиться, скоро мы рассмотрим код, требующийся для этих вызовов.

## <a name="get-supported-languages"></a>Получение сведений о поддерживаемых языках

Переводчик сейчас поддерживает более 90 языков и диалектов. Так как со временем будет добавлена поддержка новых языков, вместо того чтобы задавать список в коде приложения, рекомендуем вызвать ресурс "Языки" Переводчика.

В рамках этого раздела мы создадим запрос `GET` к ресурсу языков на получение списка языков, доступных для перевода.

> [!NOTE]
> Ресурс языков позволяет отфильтровать поддерживаемые языки с помощью параметров запроса, таких как транслитерация, словарь и перевод. См. дополнительные сведения в [справочнике по API](./reference/v3-0-languages.md).

Прежде чем продолжить, рассмотрим пример ответа на вызов ресурса языков:

```json
{
  "translation": {
    "af": {
      "name": "Afrikaans",
      "nativeName": "Afrikaans",
      "dir": "ltr"
    },
    "ar": {
      "name": "Arabic",
      "nativeName": "العربية",
      "dir": "rtl"
    }
    // Additional languages are provided in the full JSON output.
}
```

Из этих выходных данных можно получить код языка и `name` (имя) для конкретного языка. В этом приложении для десериализации объектов JSON будет использоваться файл NewtonSoft.Json ([`JsonConvert.DeserializeObject`](https://www.newtonsoft.com/json/help/html/M_Newtonsoft_Json_JsonConvert_DeserializeObject__1.htm)).

Перейдем в коде к строке, на которой мы остановились в предыдущем разделе, и добавим в приложение метод для получения поддерживаемых языков.

1. В Visual Studio выберите вкладку для файла `MainWindow.xaml.cs`.
2. Добавьте следующий код в проект:
   ```csharp
   // ***** GET TRANSLATABLE LANGUAGE CODES
   private void GetLanguagesForTranslate()
   {
       // Send request to get supported language codes
       string uri = String.Format(TEXT_TRANSLATION_API_ENDPOINT, "languages") + "&scope=translation";
       WebRequest WebRequest = WebRequest.Create(uri);
       WebRequest.Headers.Add("Accept-Language", "en");
       WebResponse response = null;
       // Read and parse the JSON response
       response = WebRequest.GetResponse();
       using (var reader = new StreamReader(response.GetResponseStream(), UnicodeEncoding.UTF8))
       {
           var result = JsonConvert.DeserializeObject<Dictionary<string, Dictionary<string, Dictionary<string, string>>>>(reader.ReadToEnd());
           var languages = result["translation"];

           languageCodes = languages.Keys.ToArray();
           foreach (var kv in languages)
           {
               languageCodesAndTitles.Add(kv.Value["name"], kv.Key);
           }
       }
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

Метод `GetLanguagesForTranslate()` позволяет создать HTTP-запрос GET с параметром `scope=translation` в строке запроса, чтобы ограничить область запроса языками, которые поддерживаются для перевода. Заголовок `Accept-Language` со значением `en` добавляется, чтобы поддерживаемые языки возвращались на английском языке.

Ответ в формате JSON анализируется и преобразуется в словарь. Затем коды языков добавляются в переменную-член `languageCodes`. Затем мы перебираем пары "ключ — значение", содержащие коды языков и понятные названия языков, и добавляем их к переменной-члену `languageCodesAndTitles`. В раскрывающихся меню в форме отображаются понятные имена, но для запроса перевода необходимо передавать коды.

## <a name="populate-language-drop-down-menus"></a>Заполнение раскрывающихся меню языками

Пользовательский интерфейс разрабатывается на языке XAML, поэтому вам не нужно настраивать много параметров, кроме вызова `InitializeComponent()`. Достаточно добавить понятные названия языков в раскрывающиеся меню **Перевести с** и **Перевести на**. Метод `PopulateLanguageMenus()` добавляет имена.

1. В Visual Studio выберите вкладку для файла `MainWindow.xaml.cs`.
2. Добавьте следующий код в проект после метода `GetLanguagesForTranslate()`:
   ```csharp
   private void PopulateLanguageMenus()
   {
       // Add option to automatically detect the source language
       FromLanguageComboBox.Items.Add("Detect");

       int count = languageCodesAndTitles.Count;
       foreach (string menuItem in languageCodesAndTitles.Keys)
       {
           FromLanguageComboBox.Items.Add(menuItem);
           ToLanguageComboBox.Items.Add(menuItem);
       }

       // Set default languages
       FromLanguageComboBox.SelectedItem = "Detect";
       ToLanguageComboBox.SelectedItem = "English";
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

Этот метод позволяет выполнить перебор словаря `languageCodesAndTitles` и добавить каждый ключ в оба меню. После заполнения меню языкам исходного текста и перевода по умолчанию присваивается значение **Detect** (Определить) и **Английский** соответственно.

> [!TIP]
> Если не задать выбор по умолчанию для меню, пользователь может нажать кнопку **Перевести**, не выбирая исходный и целевой язык. Параметры по умолчанию устраняют необходимость решения этой проблемы.

Теперь, когда инициализировано окно `MainWindow` и создан пользовательский интерфейс, этот код ожидает, пока пользователь не нажмет кнопку **Перевести**.

## <a name="detect-language-of-source-text"></a>Определение языка исходного текста

Создадим метод определения языка исходного текста (текст, введенный в текстовом поле) с помощью Переводчика. Значение, возвращаемое этим запросом, используется в запросе на перевод.

1. В Visual Studio выберите вкладку для файла `MainWindow.xaml.cs`.
2. Добавьте следующий код в проект после метода `PopulateLanguageMenus()`:
   ```csharp
   // ***** DETECT LANGUAGE OF TEXT TO BE TRANSLATED
   private string DetectLanguage(string text)
   {
       string detectUri = string.Format(TEXT_TRANSLATION_API_ENDPOINT ,"detect");

       // Create request to Detect languages with Translator
       HttpWebRequest detectLanguageWebRequest = (HttpWebRequest)WebRequest.Create(detectUri);
       detectLanguageWebRequest.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
       detectLanguageWebRequest.Headers.Add("Ocp-Apim-Subscription-Region", "westus");
       detectLanguageWebRequest.ContentType = "application/json; charset=utf-8";
       detectLanguageWebRequest.Method = "POST";

       // Send request
       var serializer = new System.Web.Script.Serialization.JavaScriptSerializer();
       string jsonText = serializer.Serialize(text);

       string body = "[{ \"Text\": " + jsonText + " }]";
       byte[] data = Encoding.UTF8.GetBytes(body);

       detectLanguageWebRequest.ContentLength = data.Length;

       using (var requestStream = detectLanguageWebRequest.GetRequestStream())
           requestStream.Write(data, 0, data.Length);

       HttpWebResponse response = (HttpWebResponse)detectLanguageWebRequest.GetResponse();

       // Read and parse JSON response
       var responseStream = response.GetResponseStream();
       var jsonString = new StreamReader(responseStream, Encoding.GetEncoding("utf-8")).ReadToEnd();
       dynamic jsonResponse = serializer.DeserializeObject(jsonString);

       // Fish out the detected language code
       var languageInfo = jsonResponse[0];
       if (languageInfo["score"] > (decimal)0.5)
       {
           DetectedLanguageLabel.Content = languageInfo["language"];
           return languageInfo["language"];
       }
       else
           return "Unable to confidently detect input language.";
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

Этот метод позволяет создать HTTP-запрос `POST` к ресурсу определения языка. Он принимает один аргумент `text`, который передается в тексте запроса. Позже, когда мы создадим запрос на перевод, введенный в пользовательском интерфейсе текст будет передан в этот метод для определения языка.

Кроме того, этот метод позволяет оценить достоверность ответа. Если оценка превышает `0.5`, определенный язык отображается в пользовательском интерфейсе.

## <a name="spell-check-the-source-text"></a>Проверка орфографии в исходном тексте

Теперь с помощью API "Проверка орфографии Bing" создадим метод проверки орфографии в исходном тексте. Проверка орфографии гарантирует, что мы получим точный перевод от Переводчика. При нажатии кнопки **Перевести** любые изменения исходного текста передаются в запросе на перевод.

1. В Visual Studio выберите вкладку для файла `MainWindow.xaml.cs`.
2. Добавьте следующий код в проект после метода `DetectLanguage()`:

```csharp
// ***** CORRECT SPELLING OF TEXT TO BE TRANSLATED
private string CorrectSpelling(string text)
{
    string uri = BING_SPELL_CHECK_API_ENDPOINT + "?mode=spell&mkt=en-US";

    // Create a request to Bing Spell Check API
    HttpWebRequest spellCheckWebRequest = (HttpWebRequest)WebRequest.Create(uri);
    spellCheckWebRequest.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
    spellCheckWebRequest.Method = "POST";
    spellCheckWebRequest.ContentType = "application/x-www-form-urlencoded"; // doesn't work without this

    // Create and send the request
    string body = "text=" + System.Web.HttpUtility.UrlEncode(text);
    byte[] data = Encoding.UTF8.GetBytes(body);
    spellCheckWebRequest.ContentLength = data.Length;
    using (var requestStream = spellCheckWebRequest.GetRequestStream())
        requestStream.Write(data, 0, data.Length);
    HttpWebResponse response = (HttpWebResponse)spellCheckWebRequest.GetResponse();

    // Read and parse the JSON response; get spelling corrections
    var serializer = new System.Web.Script.Serialization.JavaScriptSerializer();
    var responseStream = response.GetResponseStream();
    var jsonString = new StreamReader(responseStream, Encoding.GetEncoding("utf-8")).ReadToEnd();
    dynamic jsonResponse = serializer.DeserializeObject(jsonString);
    var flaggedTokens = jsonResponse["flaggedTokens"];

    // Construct sorted dictionary of corrections in reverse order (right to left)
    // This ensures that changes don't impact later indexes
    var corrections = new SortedDictionary<int, string[]>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
    for (int i = 0; i < flaggedTokens.Length; i++)
    {
        var correction = flaggedTokens[i];
        var suggestion = correction["suggestions"][0];  // Consider only first suggestion
        if (suggestion["score"] > (decimal)0.7)         // Take it only if highly confident
            corrections[(int)correction["offset"]] = new string[]   // dict key   = offset
                { correction["token"], suggestion["suggestion"] };  // dict value = {error, correction}
    }

    // Apply spelling corrections, in order, from right to left
    foreach (int i in corrections.Keys)
    {
        var oldtext = corrections[i][0];
        var newtext = corrections[i][1];

        // Apply capitalization from original text to correction - all caps or initial caps
        if (text.Substring(i, oldtext.Length).All(char.IsUpper)) newtext = newtext.ToUpper();
        else if (char.IsUpper(text[i])) newtext = newtext[0].ToString().ToUpper() + newtext.Substring(1);

        text = text.Substring(0, i) + newtext + text.Substring(i + oldtext.Length);
    }
    return text;
}
// NOTE:
// In the following sections, we'll add code below this.
```

## <a name="translate-text-on-click"></a>Перевод текста по щелчку

Наконец, создадим метод, который вызывается при нажатии кнопки **Перевести** в пользовательском интерфейсе.

1. В Visual Studio выберите вкладку для файла `MainWindow.xaml.cs`.
1. Добавьте следующий код в проект после метода `CorrectSpelling()` и сохраните его:  
   ```csharp
   // ***** PERFORM TRANSLATION ON BUTTON CLICK
   private async void TranslateButton_Click(object sender, EventArgs e)
   {
       string textToTranslate = TextToTranslate.Text.Trim();

       string fromLanguage = FromLanguageComboBox.SelectedValue.ToString();
       string fromLanguageCode;

       // auto-detect source language if requested
       if (fromLanguage == "Detect")
       {
           fromLanguageCode = DetectLanguage(textToTranslate);
           if (!languageCodes.Contains(fromLanguageCode))
           {
               MessageBox.Show("The source language could not be detected automatically " +
                   "or is not supported for translation.", "Language detection failed",
                   MessageBoxButton.OK, MessageBoxImage.Error);
               return;
           }
       }
       else
           fromLanguageCode = languageCodesAndTitles[fromLanguage];

       string toLanguageCode = languageCodesAndTitles[ToLanguageComboBox.SelectedValue.ToString()];

       // spell-check the source text if the source language is English
       if (fromLanguageCode == "en")
       {
           if (textToTranslate.StartsWith("-"))    // don't spell check in this case
               textToTranslate = textToTranslate.Substring(1);
           else
           {
               textToTranslate = CorrectSpelling(textToTranslate);
               TextToTranslate.Text = textToTranslate;     // put corrected text into input field
           }
       }
       // handle null operations: no text or same source/target languages
       if (textToTranslate == "" || fromLanguageCode == toLanguageCode)
       {
           TranslatedTextLabel.Content = textToTranslate;
           return;
       }

       // send HTTP request to perform the translation
       string endpoint = string.Format(TEXT_TRANSLATION_API_ENDPOINT, "translate");
       string uri = string.Format(endpoint + "&from={0}&to={1}", fromLanguageCode, toLanguageCode);

       System.Object[] body = new System.Object[] { new { Text = textToTranslate } };
       var requestBody = JsonConvert.SerializeObject(body);

       using (var client = new HttpClient())
       using (var request = new HttpRequestMessage())
       {
           request.Method = HttpMethod.Post;
           request.RequestUri = new Uri(uri);
           request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
           request.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
           request.Headers.Add("Ocp-Apim-Subscription-Region", "westus");
           request.Headers.Add("X-ClientTraceId", Guid.NewGuid().ToString());

           var response = await client.SendAsync(request);
           var responseBody = await response.Content.ReadAsStringAsync();

           var result = JsonConvert.DeserializeObject<List<Dictionary<string, List<Dictionary<string, string>>>>>(responseBody);
           var translation = result[0]["translations"][0]["text"];

           // Update the translation field
           TranslatedTextLabel.Content = translation;
       }
   }
   ```

Прежде всего необходимо получить сведения о языках исходного текста и перевода, а также текст, который пользователь ввел в форму. Если для исходного языка установлено значение **Detect** (Определить), для определения языка текста вызывается метод `DetectLanguage()`. Текст может быть написан на языке, который не поддерживает Переводчик. В этом случае выводится сообщение с информацией для пользователя и исходный текст возвращается без перевода.

Если исходным языком является английский (независимо от того, указан ли он или обнаружен), проверяется правописание с помощью `CorrectSpelling()` и применяются все исправления. Исправленный текст добавляется обратно в текстовое поле, чтобы продемонстрировать пользователю сделанные изменения.

Код для перевода текста должен быть вам знаком: создайте URI, создайте запрос, отправьте его и проанализируйте ответ. Массив JSON может содержать более одного объекта для перевода, но в нашем приложении нужен только один объект.

После успешного выполнения запроса объект `TranslatedTextLabel.Content` заменяется на `translation`. Это позволяет отображать переведенный текст в пользовательском интерфейсе.

## <a name="run-your-wpf-app"></a>Запуск приложения WPF

Теперь у вас есть работающее приложение для перевода текста, созданное с помощью WPF. Чтобы запустить приложение, нажмите кнопку **запуска** в Visual Studio.

## <a name="source-code"></a>Исходный код

Исходный код этого проекта доступен на GitHub.

* [Ознакомьтесь с исходным кодом](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-C-Sharp-Tutorial)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Справочник по Переводчику Майкрософт](./reference/v3-0-reference.md)
