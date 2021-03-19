---
title: Разработка приложений Java с помощью Visual Studio Code
description: В этой статье показано, как создавать, развертывать и отлаживать приложения Java Service Fabric с помощью редактора Visual Studio Code.
author: peterpogorski
ms.topic: article
ms.date: 06/29/2018
ms.custom: devx-track-java
ms.author: pepogors
ms.openlocfilehash: cc65deb924a9f3367c2ea1d7c71544743ccf2697
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87327367"
---
# <a name="develop-java-service-fabric-applications-with-visual-studio-code"></a>Разработка приложений Java Service Fabric с помощью Visual Studio Code

[Расширение Service Fabric Reliable Services для VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services) позволяет легко создавать приложения Java Service Fabric в операционных системах Windows, Linux и macOS.

В этой статье показано, как создавать, развертывать и отлаживать приложение Java Service Fabric с помощью Visual Studio Code.

> [!IMPORTANT]
> Приложения Java Service Fabric могут разрабатываться на компьютерах Windows, но развернуть их можно только в кластерах Linux в Azure. Отладка приложений Java в ОС Windows не поддерживается.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что уже установлено VS Code, расширение Service Fabric Reliable Services для VS Code и все зависимости, необходимые для среды разработки. Дополнительные сведения см в разделе [Руководство](./service-fabric-get-started-vs-code.md#prerequisites).

## <a name="download-the-sample"></a>Скачивание примера приложения
В этой статье используется приложение для голосования в [примере быстрого начала работы с приложением Service Fabric Java в репозитории GitHub](https://github.com/Azure-Samples/service-fabric-java-quickstart). 

Чтобы клонировать репозиторий на компьютер разработки, в окне терминала (командное окно в Windows) необходимо выполнить следующую команду.

```sh
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
```

## <a name="open-the-application-in-vs-code"></a>Открытие приложения в VS Code

Откройте VS Code.  Щелкните значок Обозревателя на **панели действий** и нажмите кнопку **Открыть папку**, или щелкните **файл -> Открыть папку**. Перейдите к каталогу в папке *./service-fabric-java-quickstart/Voting*, где вы клонировали репозиторий, затем нажмите кнопку **ОК**. Рабочая область должна содержать те же файлы, что показаны на следующем снимке экрана.

![Приложение для голосования Java в рабочей области](./media/service-fabric-develop-java-applications-with-vs-code/java-voting-application.png)

## <a name="build-the-application"></a>Построение приложения

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть **палитру команд** в VS Code.
2. Найдите и выберите команду **Service Fabric: Build Application (Создание приложения Service Fabric)**. Выходные данные сборки передаются во встроенный терминал.

   ![Команда сборки приложения в VS Code](./media/service-fabric-develop-java-applications-with-vs-code/sf-build-application.png)

## <a name="deploy-the-application-to-the-local-cluster"></a>Развертывание приложения в локальном кластере
Созданное приложение можно развернуть в локальном кластере. 

> [!IMPORTANT]
> Развертывание приложений Java не поддерживается на компьютерах под управлением Windows.

1. В **Палитре команд** выберите **Service Fabric: развертывания приложения (Localhost)**. Выходные данные процесса установки отправляются в интегрированный терминал.

   ![Команда развертывания приложения в VS Code](./media/service-fabric-develop-java-applications-with-vs-code/sf-deploy-application.png)

4. Когда развертывание будет завершено, запустите браузер и откройте Service Fabric Explorer: `http://localhost:19080/Explorer`. Будет видно, что приложение запущено. Это может занять некоторое время. 

   ![Приложение для голосования в Service Fabric Explorer](./media/service-fabric-develop-java-applications-with-vs-code/sfx-localhost-java.png)

4. Когда вы проверите, что приложение запущено, запустите браузер и откройте эту страницу: `http://localhost:8080`. Это веб-интерфейс приложения. Можно добавить элементы и щелкнуть их, чтобы проголосовать.

   ![Приложение для голосования в браузере](./media/service-fabric-develop-java-applications-with-vs-code/voting-sample-in-browser.png)

5. Чтобы удалить приложение из кластера, выберите команду **Service Fabric: Remove Application** (Service Fabric: удалить приложение) на **палитре команд**. Выходные данные процесса удаления отправляются в интегрированный терминал. Чтобы убедиться, что приложение было удалено из локального кластера можно использовать Service Fabric Explorer.

## <a name="debug-the-application"></a>Отладка приложения
При отладке приложения в VS Code оно должно выполняться в локальном кластере. Затем в код можно добавить контрольные точки.

> [!IMPORTANT]
> Отладка приложений Java не поддерживается на компьютерах под управлением Windows.

Чтобы подготовить VotingDataService и приложение для голосования для отладки, выполните следующие действия.

1. Обновите файл *Voting/VotingApplication/VotingDataServicePkg/Code/entryPoint.sh*.
Закомментируйте команду в строке 6 (используя "#") и добавьте в конец файла следующую команду.

   ```
   java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar VotingDataService.jar
   ```

2. Обновите файл *Voting/VotingApplication/ApplicationManifest.xml*. Задайте атрибутам **MinReplicaSetSize** и **TargetReplicaSetSize** значение "1" в элементе **StatefulService**.
   
   ```xml
         <StatefulService MinReplicaSetSize="1" ServiceTypeName="VotingDataServiceType" TargetReplicaSetSize="1">
   ```

3. Чтобы открыть представление отладчика в VS Code, щелкните значок отладки в **Панели действий**. Щелкните значок шестеренки в верхней части представления отладчика и выберите **Java** в меню раскрывающееся среды. Откроется файл launch.json. 

   ![Значок отладки в рабочей области VS Code](./media/service-fabric-develop-java-applications-with-vs-code/debug-icon-workspace.png)

3. В launch.json-файле задайте значение порту в конфигурации с именем **Отладка (Вложение)** для **8001**. Сохраните файл.

   ![Конфигурация отладки для launch.json-файла](./media/service-fabric-develop-java-applications-with-vs-code/launch-json-java.png)

4. Развертывание приложения в локальном кластере с помощью команды **Service Fabric: развертывание приложения (Localhost)**. Убедитесь, что приложение выполняется в Service Fabric Explorer. Теперь приложение готово к отладке.

Чтобы задать точку останова, выполните следующее.

1. В проводнике откройте файла */Voting/VotingDataService/src/statefulservice/VotingDataService.java*. Задайте точку останова на первой строке кода в `try` блоке `addItem` метода (строка 80).
   
   ![Задайте точку останова в службе данных для голосования](./media/service-fabric-develop-java-applications-with-vs-code/breakpoint-set.png)

   > [!IMPORTANT]
   > Убедитесь, что вы установили точки останова на исполняемые строки кода. Например точки останова, заданные в объявлениях метода, `try` инструкциях или `catch` инструкциях отброшенных отладчиком.
2. Чтобы начать отладку, щелкните значок отладки на **панели действий**,выберите настройку **Отладка (Вложение)** из меню "Отладка" и нажмите кнопку выполнения (зеленая стрелка).

   ![Конфигурация "Отладка (Вложение)"](./media/service-fabric-develop-java-applications-with-vs-code/debug-attach-java.png)

3. В веб-браузере перейдите по адресу `http://localhost:8080`. Введите новый элемент в текстовом поле и нажмите кнопку **+ добавить**. Ваша точка останова должна быть выполненной. Вы можете использовать панель инструментов "Отладка" в верхней части VS Code для продолжения выполнения, перехода по строкам, перехода к методам или выхода из текущего метода. 
   
   ![Выполнение точки останова](./media/service-fabric-develop-java-applications-with-vs-code/breakpoint-hit.png)
       
4. Чтобы завершить сеанс отладки, щелкните значок подключения на панели инструментов отладки в верхней части VS Code.
   
   ![Отключение отладчика](./media/service-fabric-develop-java-applications-with-vs-code/debug-bar-disconnect.png)
       
5. Закончив отладку, вы можете использовать команду **Service Fabric: Remove Application** (​​Service Fabric: удалить приложение) для удаления приложения для голосования из локального кластера. 

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в разделе [Разработка и отладка приложений Service Fabric на C# с помощью VS Code](./service-fabric-develop-csharp-applications-with-vs-code.md).
