---
title: Щелкните подключаемый модуль автосбора аналитики для Application Insights пакет SDK для JavaScript.
description: Как установить и использовать подключаемый модуль автоколлекции для Application Insights JavaScript SDK.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 01/14/2021
ms.author: lagayhar
ms.openlocfilehash: e48d669321ad8c58681e8a92e68f2089962bdc17
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102429856"
---
# <a name="click-analytics-auto-collection-plugin-for-application-insights-javascript-sdk"></a>Щелкните подключаемый модуль автосбора аналитики для Application Insights пакет SDK для JavaScript.

Этот подключаемый модуль автоматически отслеживает события щелчка на веб-страницах и использует атрибуты Data-* в элементах HTML для заполнения данных телеметрии событий.

## <a name="getting-started"></a>Начало работы

Пользователи могут настроить подключаемый модуль автосбора данных Click Analytics с помощью NPM.

### <a name="npm-setup"></a>Установка NPM

Установить пакет NPM:

```bash
npm install --save @microsoft/applicationinsights-clickanalytics-js @microsoft/applicationinsights-web
```

```js

import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ClickAnalyticsPlugin } from '@microsoft/applicationinsights-clickanalytics-js';

const clickPluginInstance = new ClickAnalyticsPlugin();
// Click Analytics configuration
const clickPluginConfig = {
  autoCapture: true
};
// Application Insights Configuration
const configObj = {
  instrumentationKey: "YOUR INSTRUMENTATION KEY",
  extensions: [clickPluginInstance],
  extensionConfig: {
    [clickPluginInstance.identifier]: clickPluginConfig
  },
};

const appInsights = new ApplicationInsights({ config: configObj });
appInsights.loadAppInsights();
```

## <a name="how-to-effectively-use-the-plugin"></a>Эффективное использование подключаемого модуля

1. Данные телеметрии, созданные на основе событий щелчка, сохраняются `customEvents` в разделе Application Insights портал Azure.
2. Значение параметра `name` customEvent заполняется на основе следующих правил.
    1.  , `id` Указанный в, `data-*-id` будет использоваться в качестве имени customEvent. Например, если элемент HTML, который щелкнули, имеет атрибут "Data-Sample-ID" = "Button1", то "Button1" будет customEvent именем.
    2. Если такого атрибута не существует и если в `useDefaultContentNameOrId` конфигурации задано значение, то в `true` `id` качестве имени customEvent будет использоваться HTML-атрибут или имя содержимого элемента, который щелкнул элемент.
    3. Если `useDefaultContentNameOrId` параметр имеет значение false, то имя customEvent будет иметь значение "not_specified".

    > [!TIP]
    > Мы рекомендуем установить значение `useDefaultContentNameOrId` true для создания осмысленных данных.  
3. `parentDataTag` выполняет две задачи:
    1. Если этот тег имеется, подключаемый модуль выберет `data-*` атрибуты и значения из всех родительских HTML-элементов элемента, который щелкнули.
    2. Для повышения эффективности подключаемый модуль использует этот тег в качестве флага, когда он обнаруживает, что он будет останавливаться на дальнейшей обработке модели DOM (модель DOM) назад.
    
    > [!CAUTION]
    > После `parentDataTag` использования пакет SDK начнет искать родительские теги во всем приложении, а не только HTML-элемент, где он использовался.
4. `customDataPrefix` Этот пользователь всегда должен начинаться с `data-` , например `data-sample-` . В HTML `data-*` Глобальные атрибуты образуют класс атрибутов, называемых пользовательскими атрибутами данных, которые позволяют осуществлять обмен собственными данными между HTML и его представлением DOM с помощью скриптов. Более старые браузеры (Internet Explorer, Safari) удаляют непонятные атрибуты, если они не начинаются с `data-` .

    `*`В `data-*` может быть заменен любым именем, которое следует за [рабочим правилом имен XML](https://www.w3.org/TR/REC-xml/#NT-Name) со следующими ограничениями.
    - Имя не должно начинаться с "XML", для этих букв используется любой регистр.
    - Имя не должно содержать точки с запятой (U + 003A).
    - Имя не должно содержать прописные буквы.

## <a name="configuration"></a>Конфигурация

| Имя                  | Тип                               | По умолчанию | Описание                                                                                                                              |
| --------------------- | -----------------------------------| --------| ---------------------------------------------------------------------------------------------------------------------------------------- |
| Автозапись           | Логическое                            | true    | Автоматическая настройка записи.                                                                                                         |
| обратный вызов              | [ивалуекаллбакк](#ivaluecallback)  | null    | Настройка обратных вызовов.                                                                                                                 |
| пажетагс              | строка                             | null    | Теги страницы.                                                                                                                               |
| Теги              | [икустомдататагс](#icustomdatatags)| null    | Пользовательские теги данных, предоставленные для переопределения тегов по умолчанию, используемых для сбора данных по щелчку.                                                           |
| урлколлексаш        | Логическое                            | false   | Включает ведение журнала значений после символа "#" URL-адреса.                                                                          |
| урлколлекткуери       | Логическое                            | false   | Включает ведение журнала строки запроса URL-адреса.                                                                                      |
| бехавиорвалидатор     | Компонент                           | null  | Функция обратного вызова, используемая для `data-*-bhvr` проверки значения. Дополнительные сведения см. в [разделе бехавиорвалидатор](#behaviorvalidator).|
| дефаултригхткликкбхвр | строка (или) номер                 | ''      | Значение поведения по умолчанию при возникновении события щелчка правой кнопкой мыши. Это значение будет переопределено, если элемент имеет `data-*-bhvr` атрибут. |
| дропинвалидевентс     | Логическое                            | false   | Флаг для удаления событий, которые не имеют полезных данных по щелчку.                                                                                   |

### <a name="ivaluecallback"></a>ивалуекаллбакк

| Имя               | Тип     | По умолчанию | Описание                                                                             |
| ------------------ | -------- | ------- | --------------------------------------------------------------------------------------- |
| pageName           | Компонент | null    | Функция для переопределения поведения записи pageName по умолчанию.                           |
| пажеактионпажетагс | Компонент | null    | Функция обратного вызова для дополнения Пажетагс по умолчанию, собираемого во время события Пажеактион.  |
| contentName        | Компонент | null    | Функция обратного вызова для заполнения настраиваемого contentName.                                 |

### <a name="icustomdatatags"></a>икустомдататагс

| Имя                      | Тип    | По умолчанию   | Тег по умолчанию для использования в HTML |   Описание                                                                                |
|---------------------------|---------|-----------|-------------|----------------------------------------------------------------------------------------------|
| уседефаултконтентнамеорид | Логическое | false     | Н/Д         |Собирает стандартный атрибут HTML для contentName, если определенный элемент не помечен как Кустомдатапрефикс по умолчанию или если Кустомдатапрефикс не предоставлен пользователем. |
| кустомдатапрефикс          | строка  | `data-`   | `data-*`| Автоматически записывать имя и значение элементов, помеченных указанным префиксом. Например, `data-*-id` `data-<yourcustomattribute>` можно использовать в HTML-тегах.   |
| аиблобаттрибутетаг        | строка  | `ai-blob` |  `data-ai-blob`| Подключаемый модуль поддерживает атрибут BLOB-объекта JSON, а не отдельные `data-*` атрибуты. |
| метадатапрефикс            | строка  | null      | Н/Д  | Автоматически записать имя и содержимое элемента meta заголовка HTML с предоставленным префиксом при захвате. Например, `custom-` можно использовать в ТЕГЕ HTML meta. |
| каптуреаллметадатаконтент | Логическое | false     | Н/Д   | Автоматически захватить все имена и содержимое элементов meta заголовка HTML. Значение по умолчанию — false. Если этот флажок включен, переопределяет предоставленные Метадатапрефикс. |
| парентдататаг             | строка  | null      |  Н/Д  | Прекращает обход модели DOM для записи имени содержимого и значения элементов при обнаружении этого тега. Например, `data-<yourparentDataTag>` можно использовать в HTML-тегах.|
| днтдататаг                | строка  | `ai-dnt`  |  `data-ai-dnt`| Элементы HTML с этим атрибутом будут игнорироваться подключаемым модулем для сбора данных телеметрии.|

### <a name="behaviorvalidator"></a>бехавиорвалидатор

Функции Бехавиорвалидатор автоматически проверяют, соответствуют ли помеченные поведения в коде предварительно определенному списку. Это гарантирует, что пометки поведений будут соответствовать установленной в вашей организации таксономии. Это не обязательно или ожидается, что большинство Azure Monitorных клиентов будут использовать его, но оно доступно для расширенных сценариев. Существует три различные функции обратного вызова Бехавиорвалидатор, предоставляемые в рамках этого расширения. Тем не менее пользователи могут использовать собственные функции обратного вызова, если предоставляемые функции не решают ваше требование. Цель заключается в том, чтобы использовать собственную структуру данных о поведении, подключаемый модуль использует эту функцию проверяющего элемента управления при извлечении поведений из тегов данных.

| Имя                   | Описание                                                                        |
| ---------------------- | -----------------------------------------------------------------------------------|
| бехавиорвалуевалидатор | Используйте эту функцию обратного вызова, если структура данных поведений является массивом строк.|
| бехавиормапвалидатор   | Используйте эту функцию обратного вызова, если структура данных поведений является словарем.       |
| бехавиоренумвалидатор  | Используйте эту функцию обратного вызова, если структура данных поведений является перечислением.            |

#### <a name="sample-usage-with-behaviorvalidator"></a>Пример использования с Бехавиорвалидатор

```js
var clickPlugin = Microsoft.ApplicationInsights.ClickAnalyticsPlugin;
var clickPluginInstance = new clickPlugin();

// Behavior enum values
var behaviorMap = {
  UNDEFINED: 0, // default, Undefined

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Page Experience [1-19]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  NAVIGATIONBACK: 1, // Advancing to the previous index position within a webpage
  NAVIGATION: 2, // Advancing to a specific index position within a webpage
  NAVIGATIONFORWARD: 3, // Advancing to the next index position within a webpage
  APPLY: 4, // Applying filter(s) or making selections
  REMOVE: 5, // Applying filter(s) or removing selections
  SORT: 6, // Sorting content
  EXPAND: 7, // Expanding content or content container
  REDUCE: 8, // Sorting content
  CONTEXTMENU: 9, // Context Menu
  TAB: 10, // Tab control
  COPY: 11, // Copy the contents of a page
  EXPERIMENTATION: 12, // Used to identify a third party experimentation event
  PRINT: 13, // User printed page
  SHOW: 14, //  Displaying an overlay
  HIDE: 15, //  Hiding an overlay
  MAXIMIZE: 16, //  Maximizing an overlay
  MINIMIZE: 17, // Minimizing an overlay
  BACKBUTTON: 18, //  Clicking the back button

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Scenario Process [20-39]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  STARTPROCESS: 20, // Initiate a web process unique to adopter
  PROCESSCHECKPOINT: 21, // Represents a checkpoint in a web process unique to adopter
  COMPLETEPROCESS: 22, // Page Actions that complete a web process unique to adopter
  SCENARIOCANCEL: 23, // Actions resulting from cancelling a process/scenario

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Download [40-59]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  DOWNLOADCOMMIT: 40, // Initiating an unmeasurable off-network download
  DOWNLOAD: 41, // Initiating a download

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Search [60-79]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  SEARCHAUTOCOMPLETE: 60, // Auto-completing a search query during user input
  SEARCH: 61, // Submitting a search query
  SEARCHINITIATE: 62, // Initiating a search query
  TEXTBOXINPUT: 63, // Typing or entering text in the text box

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Commerce [80-99]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  VIEWCART: 82, // Viewing the cart
  ADDWISHLIST: 83, // Adding a physical or digital good or services to a wishlist
  FINDSTORE: 84, // Finding a physical store
  CHECKOUT: 85, // Before you fill in credit card info
  REMOVEFROMCART: 86, // Remove an item from the cart
  PURCHASECOMPLETE: 87, // Used to track the pageView event that happens when the CongratsPage or Thank You page loads after a successful purchase
  VIEWCHECKOUTPAGE: 88, // View the checkout page
  VIEWCARTPAGE: 89, // View the cart page
  VIEWPDP: 90, // View a PDP
  UPDATEITEMQUANTITY: 91, // Update an item's quantity
  INTENTTOBUY: 92, // User has the intent to buy an item
  PUSHTOINSTALL: 93, // User has selected the push to install option

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Authentication [100-119]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  SIGNIN: 100, // User sign-in
  SIGNOUT: 101, // User sign-out

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Social [120-139]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  SOCIALSHARE: 120, // "Sharing" content for a specific social channel
  SOCIALLIKE: 121, // "Liking" content for a specific social channel
  SOCIALREPLY: 122, // "Replying" content for a specific social channel
  CALL: 123, // Click on a "call" link
  EMAIL: 124, // Click on an "email" link
  COMMUNITY: 125, // Click on a "community" link

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Feedback [140-159]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  VOTE: 140, // Rating content or voting for content
  SURVEYCHECKPOINT: 145, // reaching the survey page/form

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Registration, Contact [160-179]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  REGISTRATIONINITIATE: 161, // Initiating a registration process
  REGISTRATIONCOMPLETE: 162, // Completing a registration process
  CANCELSUBSCRIPTION: 163, // Canceling a subscription
  RENEWSUBSCRIPTION: 164, // Renewing a subscription
  CHANGESUBSCRIPTION: 165, // Changing a subscription
  REGISTRATIONCHECKPOINT: 166, // Reaching the registration page/form

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Chat [180-199]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  CHATINITIATE: 180, // Initiating a chat experience
  CHATEND: 181, // Ending a chat experience

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Trial [200-209]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  TRIALSIGNUP: 200, // Signing-up for a trial
  TRIALINITIATE: 201, // Initiating a trial

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Signup [210-219]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  SIGNUP: 210, // Signing-up for a notification or service
  FREESIGNUP: 211, // Signing-up for a free service

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Referals [220-229]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  PARTNERREFERRAL: 220, // Navigating to a partner's web property

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Intents [230-239]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  LEARNLOWFUNNEL: 230, // Engaging in learning behavior on a commerce page (ex. "Learn more click")
  LEARNHIGHFUNNEL: 231, // Engaging in learning behavior on a non-commerce page (ex. "Learn more click")
  SHOPPINGINTENT: 232, // Shopping behavior prior to landing on a commerce page

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  // Video [240-259]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  VIDEOSTART: 240, // Initiating a video
  VIDEOPAUSE: 241, // Pausing a video
  VIDEOCONTINUE: 242, // Pausing or resuming a video.
  VIDEOCHECKPOINT: 243, // Capturing predetermined video percentage complete.
  VIDEOJUMP: 244, // Jumping to a new video location.
  VIDEOCOMPLETE: 245, // Completing a video (or % proxy)
  VIDEOBUFFERING: 246, // Capturing a video buffer event
  VIDEOERROR: 247, // Capturing a video error
  VIDEOMUTE: 248, // Muting a video
  VIDEOUNMUTE: 249, // Unmuting a video
  VIDEOFULLSCREEN: 250, // Making a video full screen
  VIDEOUNFULLSCREEN: 251, // Making a video return from full screen to original size
  VIDEOREPLAY: 252, // Making a video replay
  VIDEOPLAYERLOAD: 253, // Loading the video player
  VIDEOPLAYERCLICK: 254, //  Click on a button within the interactive player
  VIDEOVOLUMECONTROL: 255, //  Click on video volume control
  VIDEOAUDIOTRACKCONTROL: 256, // Click on audio control within a video
  VIDEOCLOSEDCAPTIONCONTROL: 257, //  Click on the closed caption control
  VIDEOCLOSEDCAPTIONSTYLE: 258, //  Click to change closed caption style
  VIDEORESOLUTIONCONTROL: 259, //  Click to change resolution

  ///////////////////////////////////////////////////////////////////////////////////////////////////
  //    Advertisement Engagement [280-299]
  ///////////////////////////////////////////////////////////////////////////////////////////////////
  ADBUFFERING: 283, // Ad is buffering
  ADERROR: 284, // Ad error
  ADSTART: 285, // Ad start
  ADCOMPLETE: 286, // Ad complete
  ADSKIP: 287, // Ad skipped
  ADTIMEOUT: 288, // Ad timed-out
  OTHER: 300 // Other
};

// Application Insights Configuration
var configObj = {
  instrumentationKey: "YOUR INSTRUMENTATION KEY",
  extensions: [clickPluginInstance],
  extensionConfig: {
    [clickPluginInstance.identifier]: {
      behaviorValidator: Microsoft.ApplicationInsights.BehaviorMapValidator(behaviorMap),
      defaultRightClickBhvr: 9
    },
  },
};
var appInsights = new Microsoft.ApplicationInsights.ApplicationInsights({
  config: configObj
});
appInsights.loadAppInsights();
```

## <a name="sample-app"></a>Пример приложения

[Простое веб-приложение с включенным подключаемым модулем автоматической коллекции Click Analytics](https://go.microsoft.com/fwlink/?linkid=2152871).

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [репозиторием GitHub](https://github.com/microsoft/ApplicationInsights-JS/tree/master/extensions/applicationinsights-clickanalytics-js) и [пакетом NPM](https://www.npmjs.com/package/@microsoft/applicationinsights-clickanalytics-js) для подключаемого модуля автосбора данных Click Analytics.
- Используйте [Анализ событий в процессе использования](usage-segmentation.md) , чтобы анализировать верхние щелчки и срезы по доступным измерениям.
- Найдите элемент данные в поле содержимое в атрибуте customDimensions в таблице CustomEvents в [log Analytics](../logs/log-analytics-tutorial.md#write-a-query). Дополнительные рекомендации см. в разделе [пример приложения](https://go.microsoft.com/fwlink/?linkid=2152871) .
- Создайте [книгу](../visualize/workbooks-overview.md) или [экспортируйте ее в Power BI](../logs/log-powerbi.md#integrating-queries) для создания пользовательских визуализаций по щелчку данных.
