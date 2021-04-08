---
title: Учебник по запуску решения "Функции Azure" из заданий Azure Stream Analytics
description: Из этого руководства вы узнаете, как настроить службу "Функции Azure" в качестве приемника выходных данных для заданий Stream Analytics.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 01/27/2020
ms.openlocfilehash: 74e09e61a6132858d716686bdb6687bb670f0d33
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98879516"
---
# <a name="tutorial-run-azure-functions-from-azure-stream-analytics-jobs"></a>Руководство по Запуск решения "Функции Azure" из заданий Azure Stream Analytics 

Службу "Функции Azure" можно запустить из Azure Stream Analytics, настроив службу "Функции" в качестве приемника выходных данных для задания Stream Analytics. Служба "Функции" — это ориентированная на события среда вычислений по требованию, которая позволяет реализовывать код, активируемый по событиям, возникающим в Azure или в сторонних службах. Эта возможность службы "Функции" реагировать на триггеры упрощает вывод данных в задания Stream Analytics.

Stream Analytics вызывает службу "Функции" с помощью триггеров HTTP. Выходной адаптер службы "Функции" позволяет пользователям подключать службу "Функции" к Stream Analytics таким образом, что события могут запускаться на основе запросов Stream Analytics. 

> [!NOTE]
> Подключение к Функциям Azure в виртуальной сети из задания Stream Analytics, которое выполняется в многотенантном кластере, не поддерживается.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание и выполнение заданий Stream Analytics
> * Создание экземпляра кэша Azure для Redis
> * Создание функции Azure
> * Проверка кэша Azure для Redis на наличие результатов

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="configure-a-stream-analytics-job-to-run-a-function"></a>Настройка задания Stream Analytics для запуска функции 

В этом разделе показано, как настроить задание Stream Analytics для запуска функции, которая записывает данные в кэш Azure для Redis. Задание Stream Analytics считывает события из Центров событий Azure, и выполняется запрос, который вызывает функцию. Эта функция считывает данные из задания Stream Analytics и записывает его в кэш Azure для Redis.

![Схема, на которой показаны связи между службами Azure](./media/stream-analytics-with-azure-functions/image1.png)

## <a name="create-a-stream-analytics-job-with-event-hubs-as-input"></a>Создание задания Stream Analytics с Центрами событий в качестве входных данных

Следуйте руководству [по выявлению мошенничества в режиме реального времени](stream-analytics-real-time-fraud-detection.md), чтобы создать концентратор событий, запустить приложение создания событий и создать задание Stream Analytics. Пропустите шаги для создания запроса и выходных данных. Вместо этого ознакомьтесь со следующими разделами, чтобы настроить выходные данные Функций Azure.

## <a name="create-an-azure-cache-for-redis-instance"></a>Создание экземпляра кэша Azure для Redis

1. Создайте кэш в кэше Azure для Redis, выполнив действия, описанные в [этом разделе](../azure-cache-for-redis/cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).  

2. После создания кэша в разделе **Параметры** выберите **Ключи доступа**. Запишите **основную строку подключения**.

   ![Снимок экрана строки подключения кэша Azure для Redis](./media/stream-analytics-with-azure-functions/image2.png)

## <a name="create-a-function-in-azure-functions-that-can-write-data-to-azure-cache-for-redis"></a>Создание функции в службе "Функции Azure", которая может записывать данные в кэш Azure для Redis

1. Сведения см. в разделе [Создание приложения-функции](../azure-functions/functions-get-started.md) документации по службе "Функции". В этом разделе описано, как создать приложение-функцию и [функцию, активируемую с помощью HTTP, в службе "Функции Azure"](../azure-functions/functions-get-started.md) с помощью языка CSharp.  

2. Перейдите к функции **run.csx**. Обновите ее, используя следующий код. Замените **"\<your Azure Cache for Redis connection string goes here\>"** основной строкой подключения кэша Azure для Redis, полученной ранее. 

    ```csharp
    using System;
    using System.Net;
    using System.Threading.Tasks;
    using StackExchange.Redis;
    using Newtonsoft.Json;
    using System.Configuration;

    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");
    
        // Get the request body
        dynamic dataArray = await req.Content.ReadAsAsync<object>();

        // Throw an HTTP Request Entity Too Large exception when the incoming batch(dataArray) is greater than 256 KB. Make sure that the size value is consistent with the value entered in the Stream Analytics portal.

        if (dataArray.ToString().Length > 262144)
        {
            return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
        }
        var connection = ConnectionMultiplexer.Connect("<your Azure Cache for Redis connection string goes here>");
        log.Info($"Connection string.. {connection}");
    
        // Connection refers to a property that returns a ConnectionMultiplexer
        IDatabase db = connection.GetDatabase();
        log.Info($"Created database {db}");
    
        log.Info($"Message Count {dataArray.Count}");

        // Perform cache operations using the cache object. For example, the following code block adds few integral data types to the cache
        for (var i = 0; i < dataArray.Count; i++)
        {
            string time = dataArray[i].time;
            string callingnum1 = dataArray[i].callingnum1;
            string key = time + " - " + callingnum1;
            db.StringSet(key, dataArray[i].ToString());
            log.Info($"Object put in database. Key is {key} and value is {dataArray[i].ToString()}");
       
            // Simple get of data types from the cache
            string value = db.StringGet(key);
            log.Info($"Database got: {value}");
        }

        return req.CreateResponse(HttpStatusCode.OK, "Got");
    }

   ```

   Когда Stream Analytics получает исключение "Сущность запроса HTTP слишком большая" из функции, размер пакетов, отправляемых в службу "Функции", уменьшается. Следующий код гарантирует, что Stream Analytics не отправляет пакеты слишком большого размера. Убедитесь, что максимальное количество пакетов и размеры значений, используемые в функции, соответствуют значениям, введенным на портале Stream Analytics.

    ```csharp
    if (dataArray.ToString().Length > 262144)
        {
            return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
        }
   ```

3. В выбранном текстовом редакторе создайте JSON-файл с именем **project.json**. Вставьте следующий код и сохраните его на своем локальном компьютере. Этот файл содержит зависимости пакетов NuGet, необходимых для функции C#.  
   
    ```json
    {
        "frameworks": {
            "net46": {
                "dependencies": {
                    "StackExchange.Redis":"1.1.603",
                    "Newtonsoft.Json": "9.0.1"
                }
            }
        }
    }

   ```
 
4. Вернитесь на портал Azure. С вкладки **Функции платформы** перейдите к своей функции. В разделе **Средства разработки** выберите **Редактор службы приложений**. 
 
   ![Снимок экрана: вкладка с компонентами платформы с выбранным Редактором службы приложений.](./media/stream-analytics-with-azure-functions/image3.png)

5. В редакторе службы приложений щелкните корневой каталог правой кнопкой мыши и отправьте файл **project.json**. После успешной отправки обновите страницу. Теперь вы должны увидеть автоматически сформированный файл с именем **project.lock.json**. Он содержит ссылки на DLL-файлы, указанные в файле project.json.  

   ![Снимок экрана: выбранный в меню пункт отправки файлов.](./media/stream-analytics-with-azure-functions/image4.png)

## <a name="update-the-stream-analytics-job-with-the-function-as-output"></a>Обновление задания Stream Analytics с использованием функции в качестве выходных данных

1. Откройте задание Stream Analytics на портале Azure.  

2. Перейдите к функции и выберите **Обзор** > **Выходные данные** > **Добавить**. Чтобы добавить новые выходные данные, выберите **Функция Azure** в качестве приемника. Новый выходной адаптер службы "Функции" поддерживает следующие свойства:  

   |**Имя свойства**|**Описание**|
   |---|---|
   |Псевдоним выходных данных| Понятное имя, используемое в запросах задания для ссылки на эти выходные данные. |
   |Вариант импорта| Вы можете использовать эту функцию из текущей подписки или указать параметры вручную, если функция находится в другой подписке. |
   |Приложение-функция| Имя приложения службы "Функции". |
   |Компонент| Имя функции в приложении службы "Функции" (имя функции run.csx).|
   |Максимальный размер пакета|Задает максимальный размер каждого пакета выходных данных в байтах, который передается в функцию. По умолчанию установлено значение 262 144 байт (256 КБ).|
   |Максимальное количество пакетов|Указывает максимальное число событий в каждом пакете, который отправляется в функцию. По умолчанию используется значение 100. Это необязательное свойство.|
   |Клавиши|Позволяет использовать функцию из другой подписки. Укажите значение ключа для доступа к функции. Это необязательное свойство.|

3. Введите имя для выходного псевдонима. В этом учебнике он называется **saop1**, но можно использовать любое имя по своему усмотрению. Заполните другие сведения.

4. Откройте задание Stream Analytics и обновите запрос следующим образом. Если вы не присвоили приемнику выходных данных имя **saop1**, не забудьте изменить его в запросе.  

   ```sql
    SELECT
            System.Timestamp as Time, CS1.CallingIMSI, CS1.CallingNum as CallingNum1,
            CS2.CallingNum as CallingNum2, CS1.SwitchNum as Switch1, CS2.SwitchNum as Switch2
        INTO saop1
        FROM CallStream CS1 TIMESTAMP BY CallRecTime
           JOIN CallStream CS2 TIMESTAMP BY CallRecTime
            ON CS1.CallingIMSI = CS2.CallingIMSI AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
        WHERE CS1.SwitchNum != CS2.SwitchNum
   ```

5. Запустите приложение telcodatagen.exe, выполнив следующую команду в командной строке. Команда использует следующий формат: `telcodatagen.exe [#NumCDRsPerHour] [SIM Card Fraud Probability] [#DurationHours]`.  
   
   ```cmd
   telcodatagen.exe 1000 0.2 2
   ```
    
6.  Запустите задание Stream Analytics.

## <a name="check-azure-cache-for-redis-for-results"></a>Проверка кэша Azure для Redis на наличие результатов

1. Перейдите на портал Azure и найдите кэш Azure для Redis. Выберите **Консоль**.  

2. Используйте [команды кэша Azure для Redis](https://redis.io/commands), чтобы проверить наличие данных в кэше Azure для Redis. (Команда имеет формат Get {key}.) Пример:

   **Get "12/19/2017 21:32:24 - 123414732"**

   Эта команда должна вывести значение для указанного ключа:

   ![Снимок экрана с выходными данными кэша Azure для Redis](./media/stream-analytics-with-azure-functions/image5.png)

## <a name="error-handling-and-retries"></a>Обработка ошибок и повторные попытки

Если при отправке событий в Функции Azure происходит сбой, Stream Analytics повторяет большинство операций. Все исключения HTTP повторяются до успешного завершения за исключением ошибки HTTP 413 (Сущность слишком большая). Ошибка "Сущность слишком большая" рассматривается как ошибка данных, к которой применяется [политика повтора или удаления](stream-analytics-output-error-policy.md).

> [!NOTE]
> Время ожидания для запросов HTTP от Stream Analytics к Функциям Azure устанавливается на 100 секунд. Если обработка пакета приложением "Функции Azure" занимает более 100 секунд, Stream Analytics выдает сообщение об ошибке и повторяет попытку обработки пакета.

Повторная попытка после истечения времени ожидания может привести к записи дубликатов событий в приемник выходных данных. Когда Stream Analytics пытается повторно обработать пакет с ошибкой, он повторяет попытки для всех событий в этом пакете. Рассмотрим, например, пакет из 20 событий, которые отправляются в Функции Azure из Stream Analytics. Предположим, что приложению "Функции Azure" требуется 100 секунд для обработки первых 10 событий в этом пакете. По истечении 100 секунд модуль Stream Analytics приостанавливает обработку запроса, так не получает положительного ответа от Функций Azure, и для этого пакета отправляется другой запрос. Первые 10 событий в пакете обрабатываются приложением "Функции Azure" повторно, что приводит к дублированию. 

## <a name="known-issues"></a>Известные проблемы

На портале Azure при попытке сброса значения максимального размера пакета или максимального числа пакетов до нуля (значения по умолчанию) после сохранения значение возвращается к ранее введенному. В этом случае введите значения по умолчанию для этих полей вручную.

Служба Stream Analytics в настоящее время не поддерживает использование [маршрутизации HTTP](/sandbox/functions-recipes/routes?tabs=csharp) в Функциях Azure.

Поддержка подключения к Функциям Azure, размещенным в виртуальной сети, не включена.

## <a name="clean-up-resources"></a>Очистка ресурсов

Ставшие ненужными группу ресурсов, задание потоковой передачи и все связанные ресурсы можно удалить. При удалении задания будет прекращена тарификация за единицы потоковой передачи, потребляемые заданием. Если вы планируете использовать это задание в будущем, вы можете остановить и перезапустить его позже. Если вы не собираетесь использовать это задание дальше, удалите все ресурсы, созданные в ходе работы с этим руководством, выполнив следующие шаги:

1. В меню слева на портале Azure щелкните **Группы ресурсов**, а затем выберите имя созданного ресурса.  
2. На странице группы ресурсов щелкните **Удалить**, в текстовом поле введите имя ресурса для удаления и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве было создано простое задание Stream Analytics, которое запускает Функцию Azure. Чтобы узнать больше о заданиях Stream Analytics, перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Запуск определяемых пользователем функций JavaScript в рамках заданий Stream Analytics](stream-analytics-javascript-user-defined-functions.md)
