---
title: Разработка приложений .NET Core с помощью Visual Studio Code
description: В этой статье показано, как создавать, развертывать и отлаживать приложения .NET Core Service Fabric с помощью Visual Studio Code.
author: peterpogorski
ms.topic: article
ms.date: 06/29/2018
ms.author: pepogors
ms.openlocfilehash: 5fbd523a38b3c4860316e45b8b7c03a17de19499
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92678342"
---
# <a name="develop-c-service-fabric-applications-with-visual-studio-code"></a>Разработка приложений Service Fabric на C# с помощью Visual Studio Code

[Расширение Service Fabric Reliable Services для VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services) позволяет легко создавать приложения .NET Core Service Fabric в операционных системах Windows, Linux и macOS.

В этой статье показано, как создавать, развертывать и отлаживать приложение .NET Core Service Fabric с помощью Visual Studio Code.

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что уже установлено VS Code, расширение Service Fabric Reliable Services для VS Code и все зависимости, необходимые для среды разработки. Дополнительные сведения см в разделе [Руководство](./service-fabric-get-started-vs-code.md#prerequisites).

## <a name="download-the-sample"></a>Скачивание примера приложения
В этой статье используется приложение CounterService для примеров из репозитория GitHub [Getting started with Service Fabric with .NET](https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started) (Краткое руководство по Service Fabric с использованием .NET). 

Чтобы клонировать репозиторий на компьютер разработки, в окне терминала (командное окно в Windows) необходимо выполнить следующую команду.

```
git clone https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started.git
```

## <a name="open-the-application-in-vs-code"></a>Открытие приложения в VS Code

### <a name="windows"></a>Windows
Щелкните правой кнопкой мыши значок VS Code в меню "Пуск" и выберите **Запуск от имени администратора**. Чтобы подключить отладчик к службам, необходимо запустить VS Code от имени администратора.

### <a name="linux"></a>Linux
Используя терминал, перейдите по указному пути /service-fabric-dotnet-core-getting-started/Services/CounterService из каталога, в который локально было клонировано приложение.

Для открытия VS Code в качестве пользователя root, что позволит подключать отладчик к службам пользователя, выполните следующую команду.
```
sudo code . --user-data-dir='.'
```

Теперь приложение должно появиться в рабочей области VS Code.

![Приложение Counter Service в рабочей области](./media/service-fabric-develop-csharp-applications-with-vs-code/counter-service-application-in-workspace.png)

## <a name="build-the-application"></a>Построение приложения
1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть **палитру команд** в VS Code.
2. Найдите и выберите команду **Service Fabric: Build Application (Создание приложения Service Fabric)**. Выходные данные сборки передаются во встроенный терминал.

   ![Команда сборки приложения в VS Code](./media/service-fabric-develop-csharp-applications-with-vs-code/sf-build-application.png)

## <a name="deploy-the-application-to-the-local-cluster"></a>Развертывание приложения в локальном кластере
Созданное приложение можно развернуть в локальном кластере. 

1. В **Палитре команд** выберите **Service Fabric: развертывания приложения (Localhost)**. Выходные данные процесса установки отправляются в интегрированный терминал.

   ![Команда развертывания приложения в VS Code](./media/service-fabric-develop-csharp-applications-with-vs-code/sf-deploy-application.png)

4. После завершения развертывания запустите браузер и откройте Service Fabric Explorer: http: \/ /ЛОКАЛХОСТ: 19080/Explorer. Будет видно, что приложение запущено. Это может занять некоторое время. 

   ![Приложение службы счетчиков в Service Fabric Explorer](./media/service-fabric-develop-csharp-applications-with-vs-code/sfx-verify-deploy.png)

4. Убедившись, что приложение запущено, запустите браузер и откройте следующую страницу: http: \/ /ЛОКАЛХОСТ: 31002. Это веб-интерфейс приложения. Чтобы увидеть текущее значение счетчика при его изменении, обновите страницу.

   ![Приложение Counter Service в браузере](./media/service-fabric-develop-csharp-applications-with-vs-code/counter-service-running.png)

## <a name="publish-the-application-to-an-azure-service-fabric-cluster"></a>Публикация приложения в кластере Azure Service Fabric
Вместе с развертыванием приложения в локальном кластере можно также опубликовать приложение в удаленном кластере Azure Service Fabric. 

1. Убедитесь, что приложение создано с помощью приведенных выше инструкций. Обновите созданный файл конфигурации, `Cloud.json` указав сведения об удаленном кластере, в котором необходимо выполнить публикацию.

2. В **палитре команд** выберите **команду Service Fabric: Публикация приложения**. Выходные данные процесса установки отправляются в интегрированный терминал.

   ![Команда публикации приложения в VS Code](./media/service-fabric-develop-csharp-applications-with-vs-code/sf-publish-application.png)

3. Когда развертывание будет завершено, запустите браузер и откройте Service Fabric Explorer: `https:<clusterurl>:19080/Explorer`. Будет видно, что приложение запущено. Это может занять некоторое время. 

## <a name="debug-the-application"></a>Отладка приложения
При отладке приложения в VS Code оно должно выполняться в локальном кластере. Затем в код можно добавить контрольные точки.

Чтобы настроить контрольную точку, выполните следующие действия.
1. Откройте файл, который находится по адресу */src/CounterServiceApplication/CounterService/CounterService.cs* и настройте контрольную точку, которая находится в 62 строке внутри метода `RunAsync` в Explorer.
3. Чтобы открыть представление отладчика в VS Code, щелкните значок отладки в **Панели действий**. Щелкните значок шестеренки в верхней части представления отладчика и выберите **.NET Core** в раскрывающемся меню среды. Откроется файл launch.json. Теперь этот файл можно закрыть. Теперь в меню конфигурации отладки должно появиться меню выбора конфигурации, расположенное рядом с кнопкой запуска (зеленая стрелка).

   ![Значок отладки в рабочей области VS Code](./media/service-fabric-develop-csharp-applications-with-vs-code/debug-icon-workspace.png)

2. Выберите **.NET Core Attach** (Присоединить .NET Core) из меню конфигурации отладки.

   ![Снимок экрана, на котором показано присоединение .NET Core, выбранное в меню конфигурации отладки.](./media/service-fabric-develop-csharp-applications-with-vs-code/debug-start.png)

3. Откройте Service Fabric Explorer в браузере: http: \/ /ЛОКАЛХОСТ: 19080/Explorer. Чтобы определить первичный узел, на котором работает CounterService, щелкните **Приложения** и разверните его. Значение первичного узла CounterService, который находится на рисунке ниже, соответствует узлу 0.

   ![Первичный узел для CounterService](./media/service-fabric-develop-csharp-applications-with-vs-code/counter-service-primary-node.png)

4. В VS Code щелкните значок запуска (зеленая стрелка) рядом с конфигурацией отладки **.NET Core Attach** (Присоединить .NET Core). В диалоговом окне выбора процесса выберите процесс CounterService, который запущен на первичном узле и был определен на 4 шаге.

   ![Первичный процесс](./media/service-fabric-develop-csharp-applications-with-vs-code/select-process.png)

5. Переход к контрольной точке файла *CounterService.cs* будет выполнен очень быстро. Теперь можно просматривать значения локальных переменных. Используйте панель инструментов "Отладка" в верхней части VS Code для продолжения выполнения перехода по строкам, перехода к методам или выхода из текущего метода. 

   ![Значения отладки](./media/service-fabric-develop-csharp-applications-with-vs-code/breakpoint-hit.png)

6. Чтобы завершить сеанс отладки, щелкните значок подключения на панели инструментов отладки верхней части VS Code.
   
   ![Отключение отладчика](./media/service-fabric-develop-csharp-applications-with-vs-code/debug-bar-disconnect.png)
       
7. После завершения отладки можно использовать команду **​​Service Fabric: Remove Application** (Service Fabric: удалить приложение) для удаления приложения CounterService из локального кластера. 

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в статье [Develop Java Service Fabric applications with Visual Studio Code](./service-fabric-develop-java-applications-with-vs-code.md) (Разработка и отладка приложений Java Service Fabric в VS Code).



