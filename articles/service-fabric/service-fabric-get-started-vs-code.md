---
title: Service Fabric Azure с VS Code начало работы
description: В этой статье содержатся общие сведения о создании приложений Service Fabric с помощью Visual Studio Code.
author: peterpogorski
ms.topic: article
ms.date: 06/29/2018
ms.author: pepogors
ms.custom: devx-track-js
ms.openlocfilehash: a655199d854462285ca98a2d8f454c483c061f99
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92316674"
---
# <a name="service-fabric-for-visual-studio-code"></a>Service Fabric для Visual Studio Code

[Расширение Reliable Services на платформе Service Fabric для VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services) предоставляет инструменты, необходимые для создания, сборки и отладки приложений Service Fabric в операционных системах Windows, Linux и macOS.

В статье представлены общие сведения о требованиях и надстройках расширения, а также использовании различных команд, которые оно предоставляет. 

> [!IMPORTANT]
> Приложения Java Service Fabric могут разрабатываться на компьютерах Windows, но развернуть их можно только в кластерах Linux в Azure. Отладка приложений Java в ОС Windows не поддерживается.

## <a name="prerequisites"></a>Предварительные условия

Необходимо обязательно установить следующие компоненты во всех средах:

* [Visual Studio Code](https://code.visualstudio.com/)
* [Node.js](https://nodejs.org/)
* [Git](https://git-scm.com/);
* [Пакет SDK для Service Fabric](./service-fabric-get-started.md)
* Генераторы Yeoman (установите соответствующие генераторы для своего приложения)

   ```sh
   npm install -g yo
   npm install -g generator-azuresfjava
   npm install -g generator-azuresfcsharp
   npm install -g generator-azuresfcontainer
   npm install -g generator-azuresfguest
   ```

Для разработки на Java необходимо обязательно установить следующие компоненты:

* [Пакет SDK для Java](/azure/developer/java/fundamentals/java-jdk-long-term-support) (версия 1.8)
* [Gradle](https://gradle.org/install/)
* [Отладчик для расширения Java в VS Code](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug). Он необходим для отладки служб Java. Отладка служб Java поддерживается только в ОС Linux. Для установки воспользуйтесь VS Code Marketplace или щелкните значок "Расширения" на **панели действий** в VS Code и выполните поиск расширения.

Для разработки .NET Core/C# необходимо установить следующие компоненты:

* [.NET Core](https://www.microsoft.com/net/learn/get-started) (версия 2.0.0 или более новая)
* [Расширение VS Code C# для Visual Studio Code (на платформе OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp). Необходимо для отладки служб C#. Для установки воспользуйтесь VS Code Marketplace или щелкните значок "Расширения" на **панели действий** в VS Code и выполните поиск расширения.

## <a name="setup"></a>Настройка

1. Откройте VS Code.
2. Щелкните значок "Расширения" на **панели действий** в левой части VS Code. Выполните поиск словосочетания "Service Fabric". Нажмите кнопку **Установить** для расширения Reliable Services на платформе Service Fabric.

## <a name="commands"></a>Команды
Расширение Reliable Services на платформе Service Fabric для VS Code предоставляет множество команд, которые должны помочь разработчикам создавать и развертывать проекты Service Fabric. Команды можно вызывать из **палитры команд**. Для этого нажмите `(Ctrl + Shift + p)`, введите имя команды в строке ввода и выберите нужную команду из списка подсказок. 

* Service Fabric: Create Application (Service Fabric: создать приложение) 
* Service Fabric: Publish Application (Service Fabric: опубликовать приложение) 
* Service Fabric: Deploy Application (Service Fabric: развернуть приложение) 
* Service Fabric: Remove Application (Service Fabric: удалить приложение)  
* Service Fabric: Build Application (Service Fabric: компилировать приложение) 
* Service Fabric: Clean Application (Service Fabric: очистить приложение) 

### <a name="service-fabric-create-application"></a>Service Fabric: Create Application (Service Fabric: создать приложение)

С помощью команды **Service Fabric: Create Application** (Service Fabric: создать приложение) в текущей рабочей области создается приложение Service Fabric. В зависимости от того, какие генераторы Yeoman установлены на компьютере разработки, возможно создание нескольких типов приложения Service Fabric, в том числе проектов Java, C#, а также контейнерных и гостевых проектов. 

1.  Выберите команду **Service Fabric: создать приложение.**
2.  Выберите тип нового приложения Service Fabric. 
3.  Введите имя приложения, которое необходимо создать.
3.  Выберите тип службы, которую необходимо добавить в приложение Service Fabric. 
4.  Следуйте инструкциям, чтобы задать имя службы. 
5.  Новое приложение Service Fabric появится в рабочей области.
6.  Откройте папку нового приложения, чтобы она стала корневой в рабочей области. Вы можете продолжить выполнение команд отсюда.

### <a name="service-fabric-add-service"></a>Service Fabric: Add Service (Service Fabric: добавить службу)
Команда **Service Fabric: Add Service** (Service Fabric: добавить службу) добавляет новую службу в уже существующее приложение Service Fabric. Папка приложения, в которое добавляется служба, должна быть корневым каталогом рабочей области. 

1.  Выберите команду **Service Fabric: добавить службу** .
2.  Выберите тип текущего приложения Service Fabric. 
3.  Выберите тип службы, которую необходимо добавить в приложение Service Fabric. 
4.  Следуйте инструкциям, чтобы задать имя службы. 
5.  Новая служба появится в каталоге проекта. 

### <a name="service-fabric-publish-application"></a>Service Fabric: Publish Application (Service Fabric: опубликовать приложение)
Команда **Service Fabric: Publish Application** (Service Fabric: опубликовать приложение) развертывает приложение Service Fabric на удаленном кластере. Целевой кластер может быть безопасным или незащищенным. Если параметры не заданы в Cloud.json, приложение развертывается в локальном кластере.

1.  При первой сборке приложения в каталоге проекта создается файл Cloud.json.
2.  Введите в файл Cloud.json значения для кластера, к которому планируется подключение.
3.  Выберите команду **Service Fabric: Publish Application** (Service Fabric: опубликовать приложение).
4.  Проверьте, установлено ли приложение. Для этого просмотрите целевой кластер с помощью Service Fabric Explorer. 

### <a name="service-fabric-deploy-application-localhost"></a>Service Fabric: Deploy Application (Service Fabric: развернуть приложение) (localhost)
Команда **Service Fabric: Deploy Application** (Service Fabric: развернуть приложение) развертывает приложение Service Fabric в локальном кластере. Перед использованием команды убедитесь, что локальный кластер запущен. 

1. Выберите команду **Service Fabric: Deploy Application** (Service Fabric: развернуть приложение).
2. Просмотрите локальный кластер с Service Fabric Explorer (http: \/ /ЛОКАЛХОСТ: 19080/Explorer), чтобы убедиться, что приложение установлено. Это может занять некоторое время.
3. Для развертывания в локальном кластере можно также использовать команду **Service Fabric: Publish Application** (Service Fabric: опубликовать приложение) без параметров, заданных в файле Cloud.json.

> [!NOTE]
> Развертывание приложений Java не поддерживается на компьютерах под управлением Windows.

### <a name="service-fabric-remove-application"></a>Service Fabric: Remove Application (Service Fabric: удалить приложение)
Команда **Service Fabric: Remove Application** (Service Fabric: удалить приложение) удаляет приложение Service Fabric из кластера, в котором оно было ранее развернуто с помощью расширения VS Code. 

1.  Выберите команду **Service Fabric: Remove Application** (Service Fabric: удалить приложение).
2.  Проверьте, удалено ли приложение. Для этого просмотрите кластер с помощью Service Fabric Explorer. Это может занять некоторое время.

### <a name="service-fabric-build-application"></a>Service Fabric: Build Application (Service Fabric: компилировать приложение)
Команда **Service Fabric: Build Application** может создавать приложения Service Fabric Java или C#. 

1.  Перед выполнением команды убедитесь, что открыта корневая папка приложения. Команда определяет тип приложения (C# или Java) и компилирует его соответствующим образом.
2.  Выберите команду **Service Fabric: Build Application** (Service Fabric: компилировать приложение).
3.  Выходные данные процесса сборки пишутся во встроенный терминал.

### <a name="service-fabric-clean-application"></a>Service Fabric: Clean Application (Service Fabric: очистить приложение)
Команда **Service Fabric: Clean Application** (Service Fabric: очистить приложение) удаляет все JAR-файлы и собственные библиотеки, которые были созданы в процессе сборки. Действует только для приложений Java. 

1.  Перед выполнением команды убедитесь, что открыта корневая папка приложения. 
2.  Выберите команду **Service Fabric: Clean Application** (Service Fabric: очистить приложение).
3.  Выходные данные процесса очистки пишутся во встроенный терминал.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в разделе [Разработка и отладка приложений Service Fabric на C# с помощью VS Code](./service-fabric-develop-csharp-applications-with-vs-code.md).
* Дополнительные сведения см. в статье [Develop Java Service Fabric applications with Visual Studio Code](./service-fabric-develop-java-applications-with-vs-code.md) (Разработка и отладка приложений Java Service Fabric в VS Code).