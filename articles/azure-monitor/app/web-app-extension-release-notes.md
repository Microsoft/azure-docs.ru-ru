---
title: Заметки о выпуске для расширения веб-приложения Azure — Application Insights
description: Выпуски заметки о расширении веб-приложений Azure для инструментирования среды выполнения с Application Insights.
ms.topic: conceptual
author: MS-jgol
ms.author: jgol
ms.date: 06/26/2020
ms.openlocfilehash: 07ba61f630b849a377f1c7ba881f95518eb73606
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102042612"
---
# <a name="release-notes-for-azure-web-app-extension-for-application-insights"></a>Заметки о выпуске расширения веб-приложения Azure для Application Insights

Эта статья содержит заметки о выпусках расширения веб-приложений Azure для инструментирования среды выполнения с Application Insights. Это применимо только к предварительно установленным расширениям.

Дополнительные [сведения](azure-web-apps.md) о расширении веб-приложения Azure для Application Insights.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

- Как узнать, какая версия расширения сейчас используется?
    - Перейдите к `https://<yoursitename>.scm.azurewebsites.net/ApplicationInsights`. Дополнительные сведения см. в разделе Пошаговое [руководство по устранению неполадок для мониторинга на основе расширений и агентов](./azure-web-apps.md?tabs=net#troubleshooting) .

- Что делать, если я использую частные расширения?
    - Удалите расширения закрытого сайта, так как оно больше не поддерживается.

## <a name="release-notes"></a>Заметки о выпуске

### <a name="2838"></a>2.8.38

- Расширение JAVA: Обновлено на [Java Agent 3.0.2 (GA)](https://github.com/microsoft/ApplicationInsights-Java/releases/tag/3.0.2) от 2.5.1.
- Расширение Node.js: обновлен пакет SDK для AI в [1.8.8](https://github.com/microsoft/ApplicationInsights-node.js/releases/tag/1.8.8) от 1.8.7.
- .NET Core: удалены неподдерживаемые версии (2,0, 2,2, 3,0). Поддерживаемые версии: 2,1 и 3,1.

### <a name="2837"></a>2.8.37

- Расширение AppSvc для Windows: .Net Core работать с любой версией System.Diagnostics.DiagnosticSource.dll.

### <a name="2836"></a>2.8.36

- Расширение AppSvc Windows: включено взаимодействие между операциями и пакетом SDK для AI в .NET Core.

### <a name="2835"></a>2.8.35

- Расширение AppSvc для Windows: добавлена поддержка .NET Core 3,1.

### <a name="2833"></a>2.8.33

- Агенты .NET, .NET Core, Java и Node.js и расширение Windows: поддержка облаков независимых. Строки соединений можно использовать для отправки данных в облака независимых.

### <a name="2831"></a>2.8.31

- Агент ASP.NET Core: Исправлена проблема, связанная с одним из обновленных ссылок на пакет SDK Application Insights (см. известные проблемы для 2.8.26). Если в `System.Diagnostics.DiagnosticSource.dll` среде выполнения уже загружена неправильная версия, расширение без кода теперь не будет завершать работу приложения и отключается. Для клиентов, затронутых этой проблемой, рекомендуется удалить `System.Diagnostics.DiagnosticSource.dll` из папки Bin или использовать старую версию расширения, установив "ApplicationInsightsAgent_EXTENSIONVERSION = 2.8.24"; в противном случае наблюдение за приложениями не будет включено.

### <a name="2826"></a>2.8.26

- Агент ASP.NET Core: Исправлена проблема, связанная с обновленным пакетом SDK для Application Insights. Агент не будет пытаться загрузиться `AiHostingStartup` , если ApplicationInsights.dll уже существует в папке bin. Это разрешает проблемы, связанные с отражением через сборку \<AiHostingStartup\> . Типы типов ().
- Известные проблемы: `System.IO.FileLoadException: Could not load file or assembly 'System.Diagnostics.DiagnosticSource, Version=4.0.4.0, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51'` при загрузке другой версии библиотеки DLL может возникнуть исключение `DiagnosticSource` . Это может произойти, например, при `System.Diagnostics.DiagnosticSource.dll` наличии в папке публикации. Для устранения рисков используйте предыдущую версию расширения, задав параметры приложения в службах приложений: ApplicationInsightsAgent_EXTENSIONVERSION = 2.8.24.

### <a name="2824"></a>2.8.24

- Переупакованная версия 2.8.21.

### <a name="2823"></a>2.8.23

- Добавлена поддержка мониторинга с неASP.NET Coreным кодом 3,0.
- Обновленный пакет SDK ASP.NET Core [2.8.0](https://github.com/microsoft/ApplicationInsights-aspnetcore/releases/tag/2.8.0) для сред выполнения версии 2,1, 2,2 и 3,0. Приложения, предназначенные для .NET Core 2,0, продолжают использовать пакет SDK.

### <a name="2814"></a>2.8.14

- Обновленная версия пакета SDK ASP.NET Core от 2.3.0 до последней версии (2.6.1) для приложений, предназначенных для .NET Core 2,1, 2,2. Приложения, предназначенные для .NET Core 2,0, продолжают использовать пакет SDK.

### <a name="2812"></a>2.8.12

- Поддержка приложений ASP.NET Core 2,2.
- Исправлена ошибка в расширении ASP.NET Core, которая вызывает пакет SDK, даже если приложение уже оснащено пакетом SDK. Для приложений 2,1 и 2,2 присутствие ApplicationInsights.dll в папке приложения теперь вызывает откат расширения. Для приложений 2,0 расширение создается, только если ApplicationInsights включен с `UseApplicationInsights()` вызовом.

- Постоянное исправление неполного ответа в формате HTML для ASP.NET Core приложений. Теперь это исправление расширено для работы с приложениями .NET Core 2,2.

- Добавлена поддержка для отключения внедрения JavaScript для ASP.NET Coreных приложений ( `APPINSIGHTS_JAVASCRIPT_ENABLED=false appsetting` ). Для ASP.NET Core внедрение JavaScript по умолчанию находится в режиме согласия, если не было явно отключено. (Настройка по умолчанию выполняется для удержания текущего поведения.)

- Исправлена ошибка расширения ASP.NET Core, которая привела к вставке, даже если отсутствует iKey.
- Исправлена ошибка в логике префикса версии пакета SDK, которая привела к неправильной версии пакета SDK в телеметрии.

- Добавлен префикс версии пакета SDK для ASP.NET Core приложений, чтобы указать, как собираются данные телеметрии.
- Исправлена страница SCM-ApplicationInsights для правильного отображения версии предварительно установленного расширения.

### <a name="2810"></a>2.8.10

- Исправление неполного ответа HTML для ASP.NET Core приложений.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о настройке мониторинга для служб приложений Azure см. в [документации по службе приложений Azure](azure-web-apps.md) . 
