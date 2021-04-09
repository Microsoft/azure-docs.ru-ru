---
title: Руководство по Реализация шаблона сохранения озера данных для обновления таблицы Azure Databricks Delta | Документация Майкрософт
description: Из этого руководства вы узнаете, как вставлять строки данных в таблицу, которая хранится в Azure Data Lake Storage 2-го поколения, с помощью подписки на Сетку событий, функции Azure и задания Azure Databricks.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: tutorial
ms.date: 08/20/2019
ms.author: normesta
ms.reviewer: sumameh
ms.custom: devx-track-csharp
ms.openlocfilehash: f5fa4ad357e937fed7df5be24a1fc78409a0259b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100516402"
---
# <a name="tutorial-implement-the-data-lake-capture-pattern-to-update-a-databricks-delta-table"></a>Руководство по Реализация шаблона сохранения озера данных для обновления таблицы Databricks Delta

В этом руководстве описана обработка событий в учетной записи хранения с иерархическим пространством имен.

Вы создадите небольшое решение, которое позволяет пользователю заполнить таблицу Databricks Delta, загружая файл разделенных запятыми значений (CSV) с описанием заказа на продажу. Чтобы создать это решение, вы объедините подписку на Сетку событий, функцию Azure и [задание](/azure/databricks/jobs) в Azure Databricks.

Выполняя данное руководство, вы сделаете следующее:

> [!div class="checklist"]
> * создавать событие в подписке на Сетку событий, которое вызывает функцию Azure;
> * создавать функцию Azure, которая получает от события уведомление и запускает задание в Azure Databricks;
> * создавать задание Databricks, в рамках которого заказ клиента вставляется в таблицу Databricks Delta в учетной записи хранения.

Мы будем создавать решение в обратном порядке, то есть начнем с рабочей области Azure Databricks.

## <a name="prerequisites"></a>Предварительные требования

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

* Создайте учетную запись хранения, которая использует иерархическое пространство имен (Azure Data Lake Storage 2-го поколения). В рамках этого руководства используется учетная запись хранения с именем `contosoorders`. Убедитесь, что учетная запись пользователя содержит назначенную ей [роль участника для данных хранилища BLOB-объектов](../common/storage-auth-aad-rbac-portal.md).

   См. статью [Создание учетной записи хранения для использования с Azure Data Lake Storage 2-го поколения](create-data-lake-storage-account.md).

* Создание субъекта-службы. В разделе [Руководство. Создание приложения Azure Active Directory и субъект-службы с доступом к ресурсам с помощью портала](../../active-directory/develop/howto-create-service-principal-portal.md).

  Существует несколько конкретных действий, которые необходимо выполнить при изучении этой статьи.

  :heavy_check_mark: При выполнении действий, описанных в разделе [Назначение приложению роли](../../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application) этой статьи, не забудьте назначить субъекту-службе роль **участника данных BLOB-объектов хранилища**.

  > [!IMPORTANT]
  > Убедитесь в том, что роль назначается в учетной записи хранения Data Lake Storage 2-го поколения. Можно назначить роль родительской группе ресурсов или подписке, но вы будете получать ошибки, связанные с разрешениями, пока роль не будет назначена учетной записи хранения.

  :heavy_check_mark: При выполнении действий, описанных в разделе [Получение значений для входа](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) этой статьи, вставьте идентификатор клиента, идентификатор приложения и значения паролей в текстовый файл. Эти значения вам скоро понадобятся.

## <a name="create-a-sales-order"></a>Создание заказа на продажу

Сначала создайте CSV-файл с описанием заказа на продажу и отправьте его в учетную запись хранения. Позже вы примените данные из этого файла для заполнения первой строки в таблице Databricks Delta.

1. Откройте обозреватель службы хранилища Azure. Затем перейдите к учетной записи хранения и создайте в разделе **Контейнеры больших двоичных объектов** новый контейнер с именем **data**.

   ![папка данных](./media/data-lake-storage-events/data-container.png "папка данных")

   Подробнее это описано в статье [Управление данными в учетной записи Azure Data Lake Storage 2-го поколения с помощью Обозревателя службы хранилища Azure](data-lake-storage-explorer.md).

2. В контейнере **data** создайте папку **input**.

3. Вставьте приведенный ниже текст в текстовый редактор.

   ```
   InvoiceNo,StockCode,Description,Quantity,InvoiceDate,UnitPrice,CustomerID,Country
   536365,85123A,WHITE HANGING HEART T-LIGHT HOLDER,6,12/1/2010 8:26,2.55,17850,United Kingdom
   ```

4. Сохраните этот файл на локальном компьютере и присвойте ему имя **data.csv.**

5. С помощью Обозревателя службы хранилища отправьте этот файл в папку **input**.  

## <a name="create-a-job-in-azure-databricks"></a>Создание задания в Azure Databricks

В этом разделе описано выполнение таких задач:

* создание рабочей области Azure Databricks;
* Создайте записную книжку.
* создание и заполнение таблицы Databricks Delta;
* добавление кода для вставки строк в таблицу Databricks Delta;
* создание задания.

### <a name="create-an-azure-databricks-workspace"></a>Создание рабочей области Azure Databricks

В этом разделе вы создадите рабочую область Azure Databricks с помощью портала Azure.

1. На портале Azure выберите **Создать ресурс** > **Analytics** > **Azure Databricks**.

    ![Databricks на портале Azure](./media/data-lake-storage-quickstart-create-databricks-account/azure-databricks-on-portal.png "Databricks на портале Azure")

2. В разделе **службы Azure Databricks** укажите значения для создания рабочей области Databricks.

    ![Создайте рабочую область Azure Databricks](./media/data-lake-storage-events/new-databricks-service.png "Создание рабочей области Azure Databricks").

    Создание рабочей области займет несколько минут. Чтобы отслеживать состояние операции, просмотрите индикатор выполнения вверху.

### <a name="create-a-spark-cluster-in-databricks"></a>Создание кластера Spark в Databricks

1. На [портале Azure](https://portal.azure.com) перейдите к созданной рабочей области Azure Databricks, а затем выберите **Launch Workspace** (Запуск рабочей области).

2. Вы будете перенаправлены на портал Azure Databricks. На портале выберите **Новый** > **Кластер**.

    ![Databricks в Azure](./media/data-lake-storage-events/databricks-on-azure.png "Databricks в Azure")

3. На странице **создания кластера** укажите значения для создания кластера.

    ![Создание кластера Databricks Spark в Azure](./media/data-lake-storage-events/create-databricks-spark-cluster.png "Создание кластера Databricks Spark в Azure")

    Для всех остальных параметров, кроме следующих, примите значения по умолчанию:

    * Введите имя кластера.
    * Убедитесь, что установлен флажок **Terminate after 120 minutes of activity** (Завершить работу после 120 минут отсутствия активности). Укажите длительность (в минутах) для завершения работы кластера, если тот не используется.

4. Выберите **Create cluster** (Создать кластер). После запуска кластера можно вложить записные книжки в кластер и запустить задания Spark.

Дополнительные сведения о создании кластеров см. в статье [о создании кластера Spark в Azure Databricks](/azure/databricks/clusters/create).

### <a name="create-a-notebook"></a>Создание записной книжки

1. В левой области выберите **Рабочая область**. В раскрывающемся списке **Рабочая область** выберите **Создать** > **Notebook** (Записная книжка).

    ![Создание записной книжки в Databricks](./media/data-lake-storage-quickstart-create-databricks-account/databricks-create-notebook.png "Создание записной книжки в Databricks")

2. В диалоговом окне **создания записной книжки** введите имя записной книжки. Выберите **Python** в качестве языка, а затем выберите созданный ранее кластер Spark.

    ![Снимок экрана для диалогового окна создания записной книжки, где можно выбрать Python в качестве языка.](./media/data-lake-storage-events/new-databricks-notebook.png "Создание записной книжки в Databricks")

    Нажмите кнопку **создания**.

### <a name="create-and-populate-a-databricks-delta-table"></a>Создание и заполнение таблицы Databricks Delta

1. Скопируйте приведенный ниже блок кода и вставьте его в первую ячейку в новой записной книжке, но пока не выполняйте этот код.  

   В этом блоке кода замените значения заполнителей `appId`, `password` и `tenant` значениями, которые вы собрали при подготовке предварительных условий для этого руководства.

    ```Python
    dbutils.widgets.text('source_file', "", "Source File")

    spark.conf.set("fs.azure.account.auth.type", "OAuth")
    spark.conf.set("fs.azure.account.oauth.provider.type", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
    spark.conf.set("fs.azure.account.oauth2.client.id", "<appId>")
    spark.conf.set("fs.azure.account.oauth2.client.secret", "<password>")
    spark.conf.set("fs.azure.account.oauth2.client.endpoint", "https://login.microsoftonline.com/<tenant>/oauth2/token")

    adlsPath = 'abfss://data@contosoorders.dfs.core.windows.net/'
    inputPath = adlsPath + dbutils.widgets.get('source_file')
    customerTablePath = adlsPath + 'delta-tables/customers'
    ```

    Этот код позволяет создать мини-приложение с именем **source_file**. Позже вы создадите функцию Azure, которая вызывает этот код и передает мини-приложению путь к файлу.  Этот код также позволяет выполнить аутентификацию субъекта-службы в учетной записи хранения, а затем создать переменные для использования в других ячейках.

    > [!NOTE]
    > При настройке рабочей среды рассмотрите возможность сохранения ключа проверки подлинности в Azure Databricks. Затем в блоке кода замените ключ проверки подлинности ключом поиска. <br><br>Например, вместо использования строки кода `spark.conf.set("fs.azure.account.oauth2.client.secret", "<password>")` следует использовать строку `spark.conf.set("fs.azure.account.oauth2.client.secret", dbutils.secrets.get(scope = "<scope-name>", key = "<key-name-for-service-credential>"))`. <br><br>Выполнив инструкции из этого руководства, ознакомьтесь с примерами такого подхода в статье о [Data Lake Storage 2-го поколения](/azure/databricks/data/data-sources/azure/azure-datalake-gen2) на веб-сайте Azure Databricks.

2. Нажмите клавиши **SHIFT + ВВОД**, чтобы запустить код в этом блоке.

3. Скопируйте приведенный ниже блок кода и вставьте его в другую ячейку записной книжки. Затем нажмите сочетание клавиш **SHIFT+ВВОД**, чтобы выполнить этот блок кода.

   ```Python
   from pyspark.sql.types import StructType, StructField, DoubleType, IntegerType, StringType


   inputSchema = StructType([
   StructField("InvoiceNo", IntegerType(), True),
   StructField("StockCode", StringType(), True),
   StructField("Description", StringType(), True),
   StructField("Quantity", IntegerType(), True),
   StructField("InvoiceDate", StringType(), True),
   StructField("UnitPrice", DoubleType(), True),
   StructField("CustomerID", IntegerType(), True),
   StructField("Country", StringType(), True)
   ])

   rawDataDF = (spark.read
    .option("header", "true")
    .schema(inputSchema)
    .csv(adlsPath + 'input')
   )

   (rawDataDF.write
     .mode("overwrite")
     .format("delta")
     .saveAsTable("customer_data", path=customerTablePath))
   ```

   Этот код позволяет создать в учетной записи хранения таблицу Databricks Delta, а затем загрузить в нее начальные данные из ранее отправленного CSV-файла.

4. После успешного выполнения этого блока кода удалите его из записной книжки.

### <a name="add-code-that-inserts-rows-into-the-databricks-delta-table"></a>Добавление кода для вставки строк в таблицу Databricks Delta

1. Скопируйте приведенный ниже блок кода и вставьте его в другую ячейку, но пока не запускайте ее.

   ```Python
   upsertDataDF = (spark
     .read
     .option("header", "true")
     .csv(inputPath)
   )
   upsertDataDF.createOrReplaceTempView("customer_data_to_upsert")
   ```

   Этот код позволяет вставлять во временное табличное представление данные из CSV-файла. Путь к этому CSV-файлу получен из входных данных мини-приложения, которое вы создали на предыдущем шаге.

2. Добавьте приведенный ниже код, который объединяет содержимое временного табличного представления с таблицей Databricks Delta.

   ```
   %sql
   MERGE INTO customer_data cd
   USING customer_data_to_upsert cu
   ON cd.CustomerID = cu.CustomerID
   WHEN MATCHED THEN
     UPDATE SET
       cd.StockCode = cu.StockCode,
       cd.Description = cu.Description,
       cd.InvoiceNo = cu.InvoiceNo,
       cd.Quantity = cu.Quantity,
       cd.InvoiceDate = cu.InvoiceDate,
       cd.UnitPrice = cu.UnitPrice,
       cd.Country = cu.Country
   WHEN NOT MATCHED
     THEN INSERT (InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country)
     VALUES (
       cu.InvoiceNo,
       cu.StockCode,
       cu.Description,
       cu.Quantity,
       cu.InvoiceDate,
       cu.UnitPrice,
       cu.CustomerID,
       cu.Country)
   ```

### <a name="create-a-job"></a>Создание задания

Создайте задание для запуска созданной ранее записной книжки. Позже вы создадите функцию Azure, которая запускает это задание при возникновении события.

1. Щелкните **Задания**.

2. На странице **Задания** щелкните **Создать задание**.

3. Присвойте заданию имя и выберите книгу `upsert-order-data`.

   ![Создание задания](./media/data-lake-storage-events/create-spark-job.png "Создание задания")

## <a name="create-an-azure-function"></a>Создание функции Azure

Создайте функцию Azure, которая запускает это задание.

1. В верхнем углу рабочей области Databricks Delta щелкните значок "Люди", а затем щелкните **Параметры пользователя**.

   ![Управление учетной записью](./media/data-lake-storage-events/generate-token.png "Параметры пользователей")

2. Нажмите кнопку **Создать новый токен**, а затем кнопку **Создать**.

   Не забудьте скопировать значение токена в надежное расположение. Этот токен потребуется функции Azure для аутентификации в Databricks, чтобы запустить задание.
  
3. Нажмите кнопку **Создать ресурс** в верхнем левом углу портала Azure, а затем выберите **Вычисления > Приложение-функция**.

   ![Создание функции Azure](./media/data-lake-storage-events/function-app-create-flow.png "Создание функции Azure")

4. На странице **Создание** для приложения-функция выберите стек среды выполнения **.NET Core**, а также настройте экземпляр Application Insights.

   ![Настройка приложения-функции](./media/data-lake-storage-events/new-function-app.png "Настройка приложения-функции")

5. На странице **Обзор** для приложения-функции щелкните **Конфигурация**.

   ![Снимок экрана, на котором выделен параметр "Конфигурация" в разделе "Настроенные функции".](./media/data-lake-storage-events/configure-function-app.png "Настройка приложения-функции")

6. На странице **Параметры приложения** нажмите кнопку **Новый параметр приложения** поочередно для каждого параметра.

   ![Добавление параметра конфигурации](./media/data-lake-storage-events/add-application-setting.png "Добавление параметра конфигурации")

   Добавьте следующие параметры:

   |Имя параметра | Значение |
   |----|----|
   |**DBX_INSTANCE**| Регион для рабочей области Databricks. Например: `westus2.azuredatabricks.net`|
   |**DBX_PAT**| Личный маркер доступа, который вы создали ранее. |
   |**DBX_JOB_ID**|Уникальный идентификатор выполняемого задания. В нашем примере он имеет значение `1`.|
7. На странице обзорных сведений для приложения-функции нажмите кнопку **Новая функция**.

   ![Новая функция](./media/data-lake-storage-events/new-function.png "Новая функция")

8. Щелкните **Azure Event Grid Trigger** (Триггер Сетки событий Azure).

   Установите расширение **Microsoft.Azure.WebJobs.Extensions.EventGrid**, если появится такое предложение. Если потребуется установить расширение, повторно щелкните **Azure Event Grid Trigger** (Триггер Сетки событий Azure), чтобы создать функцию.

   Появится область **Новая функция**.

9. В области **Новая функция** присвойте этой функции имя **UpsertOrder** и нажмите кнопку **Создать**.

10. Замените содержимое файла приведенным ниже кодом и нажмите кнопку **Сохранить**:

    ```cs
    using "Microsoft.Azure.EventGrid"
    using "Newtonsoft.Json"
    using Microsoft.Azure.EventGrid.Models;
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;

    private static HttpClient httpClient = new HttpClient();

    public static async Task Run(EventGridEvent eventGridEvent, ILogger log)
    {
        log.LogInformation("Event Subject: " + eventGridEvent.Subject);
        log.LogInformation("Event Topic: " + eventGridEvent.Topic);
        log.LogInformation("Event Type: " + eventGridEvent.EventType);
        log.LogInformation(eventGridEvent.Data.ToString());

        if (eventGridEvent.EventType == "Microsoft.Storage.BlobCreated" | | eventGridEvent.EventType == "Microsoft.Storage.FileRenamed") {
            var fileData = ((JObject)(eventGridEvent.Data)).ToObject<StorageBlobCreatedEventData>();
            if (fileData.Api == "FlushWithClose") {
                log.LogInformation("Triggering Databricks Job for file: " + fileData.Url);
                var fileUrl = new Uri(fileData.Url);
                var httpRequestMessage = new HttpRequestMessage {
                    Method = HttpMethod.Post,
                    RequestUri = new Uri(String.Format("https://{0}/api/2.0/jobs/run-now", System.Environment.GetEnvironmentVariable("DBX_INSTANCE", EnvironmentVariableTarget.Process))),
                    Headers = {
                        { System.Net.HttpRequestHeader.Authorization.ToString(), "Bearer " +  System.Environment.GetEnvironmentVariable ("DBX_PAT", EnvironmentVariableTarget.Process)},
                        { System.Net.HttpRequestHeader.ContentType.ToString (), "application/json" }
                    },
                    Content = new StringContent(JsonConvert.SerializeObject(new {
                        job_id = System.Environment.GetEnvironmentVariable ("DBX_JOB_ID", EnvironmentVariableTarget.Process) ,
                        notebook_params = new {
                            source_file = String.Join("", fileUrl.Segments.Skip(2))
                        }
                    }))
                 };
                var response = await httpClient.SendAsync(httpRequestMessage);
                response.EnsureSuccessStatusCode();
            }
        }
    }
    ```

   Этот код позволяет проанализировать сведения о возникшем событии хранилища и создать сообщение запроса с URL-адресом файла, вызвавшего событие. В составе этого сообщения функция передает значение в мини-приложение **source_file**, которое вы создали ранее. С помощью кода функции сообщение отправляется в задание Databricks Delta и для аутентификации применяется токен, который вы получили ранее.

## <a name="create-an-event-grid-subscription"></a>Создание подписки Сетки событий

В этом разделе вы создадите подписку на службу "Сетка событий", которая вызывает функцию Azure при отправке файлов в учетную запись хранения.

1. На странице кода функции нажмите кнопку **Добавить подписку сетки событий**.

   ![Снимок экрана, на котором выделена кнопка "Добавить подписку сетки событий".](./media/data-lake-storage-events/new-event-subscription.png "Создание подписки на события")

2. На странице **Создать подписку на события** присвойте подписке имя и выберите учетную запись хранения с помощью полей на этой странице.

   ![Создание подписки на события](./media/data-lake-storage-events/new-event-subscription-2.png "Создание подписки на события")

3. В раскрывающемся списке **Filter to Event Types** (Фильтровать по типам событий) выберите события **Blob Created** (Создание BLOB-объекта) и **Blob Deleted** (Удаление BLOB-объекта), а затем нажмите кнопку **Создать**.

## <a name="test-the-event-grid-subscription"></a>Тестирование подписки на Сетку событий

1. Создайте файл с именем `customer-order.csv`, вставьте в него приведенный ниже код JSON и сохраните файл на локальном компьютере.

   ```
   InvoiceNo,StockCode,Description,Quantity,InvoiceDate,UnitPrice,CustomerID,Country
   536371,99999,EverGlow Single,228,1/1/2018 9:01,33.85,20993,Sierra Leone
   ```

2. С помощью Обозревателя службы хранилища отправьте этот файл в папку **input** в учетной записи хранения.

   При отправке файла вызывается событие **Microsoft.Storage.BlobCreated**. Сетка событий уведомляет всех, кто подписался на это событие. В нашем примере единственным подписчиком является наша функция Azure. Эта функция Azure анализирует параметры события и определяет, какое событие произошло. Затем она передает URL-адрес файла в задание Databricks. В рамках задания Databricks файл считывается и в таблицу Databricks Delta, размещенную в учетной записи хранения, добавляется соответствующая строка.

3. Чтобы убедиться, что задание выполнено успешно, откройте рабочую область Databricks, нажмите кнопку **Задания** и откройте нужное задание.

4. Выберите задание, чтобы открыть его страницу.

   ![Задание Spark](./media/data-lake-storage-events/spark-job.png "Задание Spark")

   Когда задание будет выполнено, отобразится уведомление о завершении.

   ![Задание успешно выполнено](./media/data-lake-storage-events/spark-job-completed.png "Задание успешно выполнено")

5. В новой ячейке книги выполните приведенный ниже запрос, чтобы просмотреть обновленную таблицу Databricks Delta.

   ```
   %sql select * from customer_data
   ```

   Возвращается таблица, которая содержит последнюю запись.

   ![Последняя запись отображается в таблице](./media/data-lake-storage-events/final_query.png "Последняя запись отображается в таблице")

6. Чтобы обновить эту запись, создайте файл с именем `customer-order-update.csv`, вставьте в него приведенный ниже код и сохраните файл на локальном компьютере.

   ```
   InvoiceNo,StockCode,Description,Quantity,InvoiceDate,UnitPrice,CustomerID,Country
   536371,99999,EverGlow Single,22,1/1/2018 9:01,33.85,20993,Sierra Leone
   ```

   Этот CSV-файл почти идентичен предыдущему, за исключением того, что количество в заказе изменилось с `228` на `22`.

7. С помощью Обозревателя службы хранилища отправьте этот файл в папку **input** в учетной записи хранения.

8. Снова выполните запрос `select`, чтобы просмотреть обновленную разностную таблицу.

   ```
   %sql select * from customer_data
   ```

   Будет возвращена таблица, которая содержит обновленную запись.

   ![Обновленная запись отображается в таблице](./media/data-lake-storage-events/final_query-2.png "Обновленная запись отображается в таблице")

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите группу ресурсов и все связанные с ней ресурсы, когда надобность в них отпадет. Для этого выберите группу ресурсов для учетной записи хранения и выберите **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Reacting to Blob storage events (preview)](storage-blob-event-overview.md) (Реагирование на события хранилища BLOB-объектов)