---
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 11/09/2018
ms.author: clemensv
ms.openlocfilehash: 2d6836b2bf667e4170e67a95dc1daad72a769eb9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185521"
---
### <a name="create-a-console-application"></a>Создание консольного приложение

В Visual Studio создайте проект **Консольное приложение (.NET Framework)**.

### <a name="add-the-relay-nuget-package"></a>Добавление пакета ретранслятора NuGet

1. Щелкните созданный проект правой кнопкой мыши и выберите **Управление пакетами NuGet**.
2. Выберите **Обзор** и выполните поиск по ключевой фразе **Microsoft.Azure.Relay**. В результатах поиска выберите **Ретранслятор Microsoft Azure**. 
3. Выберите **Установить** для завершения установки. Закройте диалоговое окно.

### <a name="write-code-to-send-messages"></a>Написание кода для отправки сообщений

1. В начале файла Program.cs замените существующие операторы `using` следующими операторами `using`:
   
    ```csharp
    using System;
    using System.IO;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.Relay;
    ```
2. Добавьте константы в класс `Program`, чтобы получить сведения о гибридном подключении. Замените заполнители в скобках значениями, которые получены при создании гибридного подключения. Необходимо использовать полное имя пространства имен.
   
    ```csharp
    private const string RelayNamespace = "{RelayNamespace}.servicebus.windows.net";
    private const string ConnectionName = "{HybridConnectionName}";
    private const string KeyName = "{SASKeyName}";
    private const string Key = "{SASKey}";
    ```
3. Добавьте в класс `Program` следующий метод:
   
    ```csharp
    private static async Task RunAsync()
    {
        Console.WriteLine("Enter lines of text to send to the server with ENTER");
   
        // Create a new hybrid connection client.
        var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(KeyName, Key);
        var client = new HybridConnectionClient(new Uri(String.Format("sb://{0}/{1}", RelayNamespace, ConnectionName)), tokenProvider);
   
        // Initiate the connection.
        var relayConnection = await client.CreateConnectionAsync();
   
        // Run two concurrent loops on the connection. One 
        // reads input from the console and writes it to the connection 
        // with a stream writer. The other reads lines of input from the 
        // connection with a stream reader and writes them to the console. 
        // Entering a blank line shuts down the write task after 
        // sending it to the server. The server then cleanly shuts down
        // the connection, which terminates the read task.
   
        var reads = Task.Run(async () => {
            // Initialize the stream reader over the connection.
            var reader = new StreamReader(relayConnection);
            var writer = Console.Out;
            do
            {
                // Read a full line of UTF-8 text up to newline.
                string line = await reader.ReadLineAsync();
                // If the string is empty or null, you are done.
                if (String.IsNullOrEmpty(line))
                    break;
                // Write to the console.
                await writer.WriteLineAsync(line);
            }
            while (true);
        });
   
        // Read from the console and write to the hybrid connection.
        var writes = Task.Run(async () => {
            var reader = Console.In;
            var writer = new StreamWriter(relayConnection) { AutoFlush = true };
            do
            {
                // Read a line from the console.
                string line = await reader.ReadLineAsync();
                // Write the line out, also when it's empty.
                await writer.WriteLineAsync(line);
                // Quit when the line is empty,
                if (String.IsNullOrEmpty(line))
                    break;
            }
            while (true);
        });
   
        // Wait for both tasks to finish.
        await Task.WhenAll(reads, writes);
        await relayConnection.CloseAsync(CancellationToken.None);
    }
    ```
4. В метод `Main` в классе `Program` добавьте следующую строку кода.
   
    ```csharp
    RunAsync().GetAwaiter().GetResult();
    ```
   
    Файл Program.cs должен выглядеть следующим образом:
   
    ```csharp
    using System;
    using System.IO;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.Relay;
   
    namespace Client
    {
        class Program
        {
            private const string RelayNamespace = "{RelayNamespace}.servicebus.windows.net";
            private const string ConnectionName = "{HybridConnectionName}";
            private const string KeyName = "{SASKeyName}";
            private const string Key = "{SASKey}";
   
            static void Main(string[] args)
            {
                RunAsync().GetAwaiter().GetResult();
            }
   
            private static async Task RunAsync()
            {
                Console.WriteLine("Enter lines of text to send to the server with ENTER");
   
                // Create a new hybrid connection client.
                var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(KeyName, Key);
                var client = new HybridConnectionClient(new Uri(String.Format("sb://{0}/{1}", RelayNamespace, ConnectionName)), tokenProvider);
   
                // Initiate the connection.
                var relayConnection = await client.CreateConnectionAsync();
   
                // Run two concurrent loops on the connection. One 
                // reads input from the console and then writes it to the connection 
                // with a stream writer. The other reads lines of input from the 
                // connection with a stream reader and then writes them to the console. 
                // Entering a blank line shuts down the write task after 
                // sending it to the server. The server then cleanly shuts down
                // the connection, which terminates the read task.
   
                var reads = Task.Run(async () => {
                    // Initialize the stream reader over the connection.
                    var reader = new StreamReader(relayConnection);
                    var writer = Console.Out;
                    do
                    {
                        // Read a full line of UTF-8 text up to newline.
                        string line = await reader.ReadLineAsync();
                        // If the string is empty or null, you are done.
                        if (String.IsNullOrEmpty(line))
                            break;
                        // Write to the console.
                        await writer.WriteLineAsync(line);
                    }
                    while (true);
                });
   
                // Read from the console and write to the hybrid connection.
                var writes = Task.Run(async () => {
                    var reader = Console.In;
                    var writer = new StreamWriter(relayConnection) { AutoFlush = true };
                    do
                    {
                        // Read a line from the console.
                        string line = await reader.ReadLineAsync();
                        // Write the line out, also when it's empty.
                        await writer.WriteLineAsync(line);
                        // Quit when the line is empty.
                        if (String.IsNullOrEmpty(line))
                            break;
                    }
                    while (true);
                });
   
                // Wait for both tasks to finish.
                await Task.WhenAll(reads, writes);
                await relayConnection.CloseAsync(CancellationToken.None);
            }
        }
    }
    ```

