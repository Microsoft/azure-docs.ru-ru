---
title: Руководство. Работа с очередями Хранилища очередей Azure в .NET
description: В этом руководстве описано, как использовать Хранилище очередей Azure для создания очередей, а также вставки, получения и удаления сообщений с помощью кода .NET.
author: twooley
ms.author: twooley
ms.reviewer: dineshm
ms.date: 06/09/2020
ms.topic: tutorial
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-csharp
ms.openlocfilehash: dda6fb816fcba4ab9350edd956d895a3b9b204d4
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106276133"
---
# <a name="tutorial-work-with-azure-queue-storage-queues-in-net"></a>Руководство по Работа с очередями Хранилища очередей Azure в .NET

Хранилище очередей Azure реализует облачный механизм очередей для поддержки взаимодействия между компонентами распределенного приложения. Каждая очередь отслеживает список сообщений, которые добавляются компонентом отправителя и обрабатываются компонентом получателя. Очередь позволяет мгновенно масштабировать приложение при любом изменении нагрузки. В этой статье описаны основные процессы работы с очередью Хранилища очередей Azure.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - Создание учетной записи хранения Azure
> - Создайте приложение
> - Добавление клиентских библиотек Azure
> - Добавление поддержки для асинхронных вызовов
> - Создание очереди
> - Вставка сообщения в очередь
> - Вывод сообщений из очереди
> - Удаление пустой очереди
> - Проверка аргументов командной строки
> - Создание и запуск приложения

## <a name="prerequisites"></a>Предварительные требования

- Получите бесплатную копию межплатформенного редактора [Visual Studio Code](https://code.visualstudio.com/download).
- Скачайте и установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download) версии 3.1 или более поздней.
- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="create-an-azure-storage-account"></a>Создание учетной записи хранения Azure

Прежде всего создайте учетную запись хранения Azure. Пошаговые инструкции по созданию учетной записи хранения см. в статье [Создание учетной записи хранения](../common/storage-account-create.md?toc=%2Fazure%2Fstorage%2Fqueues%2Ftoc.json). Это отдельный шаг, который вы выполняете после создания бесплатной учетной записи Azure для выполнения предварительных требований.

## <a name="create-the-app"></a>Создайте приложение

Создайте приложение .NET Core с именем `QueueApp`. Для простоты одно приложение будет отправлять сообщения в очередь и принимать их.

1. В окне консоли (cmd, PowerShell или Azure CLI) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `QueueApp`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: `Program.cs`.

   ```console
   dotnet new console -n QueueApp
   ```

2. Перейдите в созданную папку `QueueApp` и создайте приложение, чтобы убедиться, что все в порядке.

   ```console
   cd QueueApp
   ```

   ```console
   dotnet build
   ```

   Должен отобразиться похожий результат:

   ```output
   C:\Tutorials>dotnet new console -n QueueApp
   The template "Console Application" was created successfully.

   Processing post-creation actions...
   Running 'dotnet restore' on QueueApp\QueueApp.csproj...
     Restore completed in 155.63 ms for C:\Tutorials\QueueApp\QueueApp.csproj.

   Restore succeeded.

   C:\Tutorials>cd QueueApp

   C:\Tutorials\QueueApp>dotnet build
   Microsoft (R) Build Engine version 16.0.450+ga8dc7f1d34 for .NET Core
   Copyright (C) Microsoft Corporation. All rights reserved.

     Restore completed in 40.87 ms for C:\Tutorials\QueueApp\QueueApp.csproj.
     QueueApp -> C:\Tutorials\QueueApp\bin\Debug\netcoreapp3.1\QueueApp.dll

   Build succeeded.
       0 Warning(s)
       0 Error(s)

   Time Elapsed 00:00:02.40

   C:\Tutorials\QueueApp>_
   ```

<!-- markdownlint-disable MD023 -->

## <a name="add-the-azure-client-libraries"></a>Добавление клиентских библиотек Azure

1. Добавьте клиентские библиотеки службы хранилища Azure в проект с помощью команды `dotnet add package`.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   Выполните следующую команду из папки проекта в окне консоли.

   ```console
   dotnet add package Azure.Storage.Queues
   ```

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   Выполните следующие команды из папки проекта в окне консоли.

   ```console
   dotnet add package Microsoft.Azure.Storage.Common
   ```

   ```console
   dotnet add package Microsoft.Azure.Storage.Queue
   ```
   ---

### <a name="add-using-statements"></a>Добавление инструкций using

1. Перейдите в каталог проекта и введите в командной строке `code .`, чтобы открыть Visual Studio Code в этом каталоге. Оставьте окно командной строки открытым. Позднее нам нужно будет выполнить здесь другие команды. Если появится предложение добавить необходимые для сборки и отладки ресурсы C#, нажмите кнопку **Да**.

1. Откройте файл исходного кода `Program.cs` и добавьте следующие пространства имен сразу за инструкцией `using System;`. Это приложение использует типы из добавленных пространств имен, чтобы подключаться к службе хранилища Azure и работать с очередями.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_UsingStatements":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_UsingStatements":::

1. Сохраните файл `Program.cs`.

## <a name="add-support-for-asynchronous-code"></a>Добавление поддержки для асинхронных вызовов

Так как приложение использует облачные ресурсы, код выполняется асинхронно.

1. Обновите метод `Main`, чтобы он выполнялся асинхронно. Замените `void` возвращаемым значением `async Task`.

   ```csharp
   static async Task Main(string[] args)
   ```

1. Сохраните файл `Program.cs`.

## <a name="create-a-queue"></a>Создание очереди

Перед выполнением вызовов к Azure API необходимо получить учетные данные с портала Azure.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

### <a name="add-the-connection-string-to-the-app"></a>Добавление строки подключения в приложение

Добавьте в приложение строку подключения, чтобы оно могло обращаться к учетной записи хранения.

1. Вернитесь к окну Visual Studio Code.

1. В методе `Main` замените код `Console.WriteLine("Hello, World");` следующей строкой, которая получает строку подключения из переменной среды.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_DeclareConnectionString":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_DeclareConnectionString":::

1. Добавьте в `Main` следующий код, который создает объект очереди для последующей передачи в методы отправки и получения данных.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_CreateQueueClient":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_CreateQueueClient":::

1. Сохраните файл.

## <a name="insert-messages-into-the-queue"></a>Вставка сообщения в очередь

Создайте новый метод для отправки сообщения в очередь.

1. Добавьте в класс `Program` следующий метод `InsertMessageAsync`.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   Этому методу передается ссылка на очередь. Если очередь не существует, она создается при вызове [`CreateIfNotExistsAsync`](/dotnet/api/azure.storage.queues.queueclient.createifnotexistsasync). Затем в очередь добавляется `newMessage` с помощью вызова [`SendMessageAsync`](/dotnet/api/azure.storage.queues.queueclient.sendmessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_InsertMessage":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   Этому методу передается ссылка на очередь. Если очередь не существует, она создается при вызове [`CreateIfNotExistsAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.createifnotexistsasync). Затем в очередь добавляется `newMessage` с помощью вызова [`AddMessageAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.addmessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_InsertMessage":::

1. **Необязательно**. По умолчанию максимальное время хранения сообщения составляет 7 дней. Для срока жизни сообщения можно указать любое положительное число. В следующем фрагменте кода добавляется сообщение, для которого срок действия **никогда** не истекает.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

    Чтобы добавить сообщение с неограниченным сроком действия, укажите `Timespan.FromSeconds(-1)` в вызове функции `SendMessageAsync`.

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Initial.cs" id="snippet_SendNonExpiringMessage":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

    Чтобы добавить сообщение с неограниченным сроком действия, укажите `Timespan.FromSeconds(-1)` в вызове функции `AddMessageAsync`.

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Initial.cs" id="snippet_SendNonExpiringMessage":::

1. Сохраните файл.

Сообщение очереди должно иметь формат, совместимый с XML-запросом в кодировке UTF-8. Размер сообщения может достигать 64 КБ. Если сообщение содержит двоичные данные, примените к нему [кодировку Base64](/dotnet/api/system.convert.tobase64string).

## <a name="dequeue-messages"></a>Вывод сообщений из очереди

Создайте новый метод для получения сообщения из очереди. Очень важно удалять из очереди успешно полученные сообщения, чтобы они не обрабатывались более одного раза.

1. Добавьте новый метод с именем `RetrieveNextMessageAsync` в класс `Program`.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   Этот метод получает сообщение из очереди, вызывая [`ReceiveMessagesAsync`](/dotnet/api/azure.storage.queues.queueclient.receivemessagesasync) со значением `1` в первом параметре, чтобы получить только следующее сообщение из очереди. Получив сообщение, удалите его из очереди с помощью метода [`DeleteMessageAsync`](/dotnet/api/azure.storage.queues.queueclient.deletemessageasync).

   Когда сообщение отправляется в очередь с помощью пакета SDK до версии 12, оно автоматически кодируется в Base64. Начиная с версии 12 кодировка не применяется. При извлечении сообщения с помощью пакета SDK версии 12 оно не декодируется автоматически. Необходимо явным образом выполнить для содержимого [декодирование из Base64](/dotnet/api/system.convert.frombase64string).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Initial.cs" id="snippet_InitialRetrieveMessage":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   Этот метод получает сообщение из очереди, вызывая [`GetMessageAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessageasync). Получив сообщение, удалите его из очереди с помощью метода [`DeleteMessageAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.deletemessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Initial.cs" id="snippet_InitialRetrieveMessage":::

1. Сохраните файл.

## <a name="delete-an-empty-queue"></a>Удаление пустой очереди

По завершении работы с проектом мы рекомендуем всегда оценивать, нужны ли вам созданные ресурсы. Работающие ресурсы могут означать лишние затраты. Если очередь существует, но пуста, предложите пользователю удалить ее.

1. Дополните метод `RetrieveNextMessageAsync` запросом на удаление пустой очереди.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_RetrieveMessage":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_RetrieveMessage":::

1. Сохраните файл.

## <a name="check-for-command-line-arguments"></a>Проверка аргументов командной строки

Если в приложение переданы аргументы командной строки, мы будем считать их сообщением для добавления в очередь. Объедините эти аргументы в одну строку. Добавьте эту строку в очередь сообщений, вызвав добавленный ранее метод `InsertMessageAsync`.

Если в командной строке нет аргументов, выполните попытку извлечения. Вызовите метод `RetrieveNextMessageAsync`, чтобы получить следующее сообщение из очереди.

Перед завершением работы дождитесь ввода пользователя, используя метод `Console.ReadLine`.

1. Дополните метод `Main` проверкой аргументов командной строки и ожиданием действий пользователя.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_Main":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_Main":::

1. Сохраните файл.

## <a name="complete-code"></a>Полный код

Ниже приведен полный код для этого проекта.

   # <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_AllCode":::

   # <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_AllCode":::
   ---

## <a name="build-and-run-the-app"></a>Создание и запуск приложения

1. В командной строке в каталоге проекта выполните следующую команду dotnet, чтобы скомпилировать проект.

   ```console
   dotnet build
   ```

1. Когда сборка проекта успешно завершится, выполните следующую команду для добавления первого сообщения в очередь.

   ```console
   dotnet run First queue message
   ```

   Вы должны увидеть такой результат:

   ```output
   C:\Tutorials\QueueApp>dotnet run First queue message
   The queue was created.
   Sent: First queue message
   Press Enter..._
   ```

1. Запустите приложение без аргументов командной строки, чтобы получить и удалить первое сообщение из очереди.

   ```console
   dotnet run
   ```

1. Еще несколько раз выполните приложение, пока не закончатся сообщения в очереди. При следующем запуске вы получите сообщение о том, что очередь пуста, и запрос на ее удаление.

   ```output
   C:\Tutorials\QueueApp>dotnet run First queue message
   The queue was created.
   Sent: First queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run Second queue message
   Sent: Second queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run Third queue message
   Sent: Third queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: First queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: Second queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: Third queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   The queue is empty. Attempt to delete it? (Y/N) Y
   Received: The queue was deleted.
   Press Enter...

   C:\Tutorials\QueueApp>_
   ```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

1. Создание очереди
1. Добавление и удаление сообщений из очереди
1. Удаление очереди Хранилища очередей Azure

Дополнительные сведения см. в кратких руководствах по Хранилищу очередей Azure.

> [!div class="nextstepaction"]
> [Краткое руководство. Создание очереди и добавление сообщения в нее с помощью портала Azure](storage-quickstart-queues-portal.md)

- [Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 для .NET](storage-quickstart-queues-dotnet.md)
- [Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 для Java](storage-quickstart-queues-java.md)
- [Краткое руководство. Клиентская библиотека Хранилища очередей Azure версии 12 для Python](storage-quickstart-queues-python.md)
- [Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 для JavaScript](storage-quickstart-queues-nodejs.md)
