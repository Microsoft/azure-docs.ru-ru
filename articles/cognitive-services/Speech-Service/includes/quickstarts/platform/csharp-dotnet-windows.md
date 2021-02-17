---
title: Краткое руководство. Настройка платформы (Windows) для пакета SDK службы "Речь" для .NET Framework — служба "Речь"
titleSuffix: Azure Cognitive Services
description: В этом руководстве показано, как настроить платформу для приложения .NET Framework на C# для Windows с пакетом SDK службы "Речь".
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.custom: devx-track-dotnet
ms.openlocfilehash: 1a9faf24c5a82b815b40afe15769480b69074dc9
ms.sourcegitcommit: 5a999764e98bd71653ad12918c09def7ecd92cf6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552844"
---
а также как установить [пакет SDK для службы "Речь"](~/articles/cognitive-services/speech-service/speech-sdk.md) для .NET Framework (Windows). Если вам нужно только имя пакета, чтобы приступить к работе самостоятельно, выполните `Install-Package Microsoft.CognitiveServices.Speech` в консоли NuGet.

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством вам понадобится:

* В Windows для вашей платформы необходим [распространяемый компонент Microsoft Visual C++ для Visual Studio 2019](https://support.microsoft.com/en-us/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0). При первой установке может потребоваться перезагрузка.
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)

## <a name="create-a-visual-studio-project-and-install-the-speech-sdk"></a>Создание проекта Visual Studio и установка пакета SDK службы "Речь"

Вам необходимо установить [пакет SDK NuGet для службы "Речь"](https://aka.ms/csspeech/nuget), чтобы вы могли ссылаться на него в коде. Для этого может потребоваться сначала создать проект **helloworld**. Если у вас уже есть проект с рабочей нагрузкой **Разработка классических приложений .NET**, можно использовать этот проект и перейти к разделу [Установка пакета SDK для службы "Речь" с помощью диспетчера пакетов NuGet](#use-nuget-package-manager-to-install-the-speech-sdk).

### <a name="create-helloworld-project"></a>Создание проекта helloworld

1. Запустите Visual Studio 2019.

1. В окне Начало работы выберите **Создать проект**. 

1. В окне **Создание проекта** выберите **Консольное приложение (.NET Framework)** и нажмите кнопку **Далее**.

1. В окне **Настройка нового проекта** введите *helloworld* в **Имя проекта**, выберите или создайте путь каталога в **Расположение**, а затем выберите **Создать**.

1. В строке меню Visual Studio выберите **Инструменты** > **Get Tools and Features** (Получить инструменты и компоненты), открывающие Visual Studio Installer и показывающие диалоговое окно **Идет изменение**.

1. Проверьте, доступна ли рабочая нагрузка **разработки классического приложения .NET**. Если рабочая нагрузка не была установлена, установите флажок возле нее, а затем выберите **Изменить**, чтобы начать установку. Скачивание и установка может занять несколько минут.

   Если флажок рядом с **Разработка классических приложений .NET** уже установлен, выберите **Закрыть**, чтобы выйти из диалогового окна.

   ![Включение рабочей нагрузки "Разработка классического приложения .NET"](~/articles/cognitive-services/speech-service/media/sdk/vs-enable-net-desktop-workload.png)

1. Закройте Visual Studio Installer.

### <a name="use-nuget-package-manager-to-install-the-speech-sdk"></a>Установка пакета SDK для службы "Речь" с помощью диспетчера пакетов NuGet

1. В Обозревателе решений щелкните правой кнопкой мыши на проект **helloworld** и выберите **Управление пакетами NuGet**, чтобы отобразить Диспетчер пакетов NuGet.

   ![Диспетчер пакетов NuGet](~/articles/cognitive-services/speech-service/media/sdk/vs-nuget-package-manager.png)

1. В правом верхнем углу найдите раскрывающийся список **Источник пакета** и убедитесь, что выбран вариант **`nuget.org`** .

1. В левом верхнем углу нажмите кнопку **Просмотреть**.

1. В поле поиска введите *Microsoft.CognitiveServices.Speech* и выберите **Ввести**.

1. В результатах поиска выберите пакет **Microsoft.CognitiveServices.Speech**, а затем выберите **Установить** для установки последней стабильной версии.

   ![Установка пакета Microsoft.CognitiveServices.Speech NuGet](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-03-nuget-install-1.0.0.png)

1. Примите все соглашения и лицензии для запуска установки.

   После установки пакета на **Консоли диспетчера пакетов** появится подтверждение.

### <a name="choose-target-architecture"></a>Выбор целевой архитектуры

Чтобы создать и запустить консольное приложение, создайте конфигурацию платформы, соответствующую архитектуре компьютера.

1. В строке меню выберите **Сборка** > **Configuration Manager** (Диспетчер конфигураций). Откроется диалоговое окно **ConfigurationManager** (Диспетчер конфигураций).

   ![Диалоговое окно Configuration Manager (Диспетчер конфигураций)](~/articles/cognitive-services/speech-service/media/sdk/vs-configuration-manager-dialog-box.png)

1. В раскрывающемся списке **Активная платформа решения** выберите команду **Новый**. Откроется диалоговое окно **Создание платформы решения**.

1. В раскрывающемся списке **Введите или выберите новую платформу**.
   - Если вы используете 64-разрядную версию Windows, выберите **x64**.
   - Если вы используете 32-разрядную версию Windows, выберите **x86**.

1. Нажмите **ОК**, а затем **Закрыть**.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [windows](../quickstart-list.md)]
