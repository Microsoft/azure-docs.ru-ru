---
title: Запрос данных Avro с помощью Azure Data Lake Analytics | Документы Майкрософт
description: В этой статье описано, как с помощью свойств текста сообщения передать данные телеметрии устройства в хранилище BLOB-объектов и запросить записанные в это хранилище данные в формате Avro.
author: ash2017
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/15/2019
ms.author: asrastog
ms.openlocfilehash: 3cfe75edcf338f5248baf396147a5b77803fbfb3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97655940"
---
# <a name="query-avro-data-by-using-azure-data-lake-analytics"></a>Запрос данных Avro с помощью Azure Data Lake Analytics

В этой статье показано, как запросить данные Avro, чтобы повысить эффективность маршрутизации сообщений из Центра Интернета вещей Azure в службы Azure. [Маршрутизация сообщений](iot-hub-devguide-messages-d2c.md) позволяет фильтровать данные с помощью сложных запросов на основе свойства сообщения, текста сообщения, тегов двойников устройств и свойств двойников устройств. Подробнее о возможностях выполнения запросов в маршрутизации сообщений см. в статье о [синтаксисе запросов маршрутизации сообщения](iot-hub-devguide-routing-query-syntax.md).

Раньше сложность была в том, что при маршрутизации сообщений в хранилище BLOB-объектов Azure Центр Интернета вещей Azure записывал содержимое в формате Avro, который включает и свойство текста сообщения, и свойство сообщения. Формат Avro не используется для других конечных точек. Хотя формат Avro прекрасно подходит для хранения данных и сообщений, использовать его для опроса данных сложно. в отличие от формата JSON или CSV. Теперь Центр Интернета вещей поддерживает запись данных в хранилище BLOB-объектов не только в формате Avro, но и в JSON.

Подробнее см. в статье [Использование службы хранилища Azure в качестве конечной точки маршрутизации](iot-hub-devguide-messages-d2c.md#azure-storage-as-a-routing-endpoint).

Чтобы устранить проблемы, связанные с требованиями и форматами нереляционных больших данных, и преодолеть эти сложности, вы можете применить шаблоны больших данных к данным масштабирования и преобразования. Один из шаблонов ("оплата за запрос") — это служба Azure Data Lake Analytics, которой посвящена эта статья. Вы можете легко выполнить этот запрос в Hadoop или другом решении, но зачастую Azure Data Lake Analytics лучше подходит при использовании подхода с оплатой за запрос.

В U-SQL есть средство извлечения данных в формате Avro. Дополнительные сведения см. в [примере U-SQL Avro](https://github.com/Azure/usql/tree/master/Examples/AvroExamples).

## <a name="query-and-export-avro-data-to-a-csv-file"></a>Запрос данных Avro и их экспорт в файл CSV

В этом разделе описано, как запросить данные Avro и экспортировать их в CSV-файл в хранилище BLOB-объектов Azure. При этом эти данные можно легко поместить в другие хранилища или репозитории.

1. Настройте маршрутизацию данных на конечную точку хранилища BLOB-объектов Azure в Центре Интернета вещей, используя свойство в тексте сообщения для выбора сообщений.

   ![Раздел "Пользовательские конечные точки"](./media/iot-hub-query-avro-data/query-avro-data-1a.png)

   ![Правила маршрутизации](./media/iot-hub-query-avro-data/query-avro-data-1b.png)

   Дополнительные сведения о настройке маршрутов и пользовательских конечных точек см. в разделе о [маршрутизации сообщений для центра Интернета вещей](iot-hub-create-through-portal.md#message-routing-for-an-iot-hub).

2. Убедитесь, что в свойствах или тексте сообщения на устройстве настроены кодировка, тип содержимого и необходимые данные в соответствии с документацией по продукту. При просмотре этих атрибутов в средстве Device Explorer, как показано ниже, можно убедиться, что они установлены правильно.

   ![Область данных концентратора событий](./media/iot-hub-query-avro-data/query-avro-data-2.png)

3. Настройте экземпляр Azure Data Lake Store и экземпляр Data Lake Analytics. Центр Интернета вещей не выполняет маршрутизацию в экземпляр Data Lake Store, но эта маршрутизация необходима для экземпляра Data Lake Analytics.

   ![Экземпляры Data Lake Store и Data Lake Analytics](./media/iot-hub-query-avro-data/query-avro-data-3.png)

4. В Azure Data Lake Analytics настройте хранилище BLOB-объектов Azure как дополнительное хранилище. Это то же хранилище BLOB-объектов, в которое Центр Интернета вещей Azure маршрутизирует данные.

   ![Область "Источники данных"](./media/iot-hub-query-avro-data/query-avro-data-4.png)

5. Как было описано в разделе [Пример U-SQL Avro](https://github.com/Azure/usql/tree/master/Examples/AvroExamples), вам потребуется четыре файла DLL. Отправьте эти файлы в расположение в экземпляре Data Lake Store.

   ![Четыре отправляемых файла DLL](./media/iot-hub-query-avro-data/query-avro-data-5.png)

6. В Visual Studio создайте проект U-SQL.

   ![Создание проекта U-SQL](./media/iot-hub-query-avro-data/query-avro-data-6.png)

7. Вставьте содержимое приведенного ниже сценария и в созданный файл. Измените три выделенных раздела: учетная запись Data Lake Analytics, связанные пути к файлам DLL и путь к учетной записи хранения.

   ![Три раздела, которые необходимо изменить](./media/iot-hub-query-avro-data/query-avro-data-7a.png)

   Фактический сценарий U-SQL, который выводит данные в файл CSV, приведен ниже:

    ```sql
        DROP ASSEMBLY IF EXISTS [Avro];
        CREATE ASSEMBLY [Avro] FROM @"/Assemblies/Avro/Avro.dll&quot;;
        DROP ASSEMBLY IF EXISTS [Microsoft.Analytics.Samples.Formats];
        CREATE ASSEMBLY [Microsoft.Analytics.Samples.Formats] FROM @&quot;/Assemblies/Avro/Microsoft.Analytics.Samples.Formats.dll&quot;;
        DROP ASSEMBLY IF EXISTS [Newtonsoft.Json];
        CREATE ASSEMBLY [Newtonsoft.Json] FROM @&quot;/Assemblies/Avro/Newtonsoft.Json.dll&quot;;
        DROP ASSEMBLY IF EXISTS [log4net];
        CREATE ASSEMBLY [log4net] FROM @&quot;/Assemblies/Avro/log4net.dll&quot;;

        REFERENCE ASSEMBLY [Newtonsoft.Json];
        REFERENCE ASSEMBLY [log4net];
        REFERENCE ASSEMBLY [Avro];
        REFERENCE ASSEMBLY [Microsoft.Analytics.Samples.Formats];

        // Blob container storage account filenames, with any path
        DECLARE @input_file string = @&quot;wasb://hottubrawdata@kevinsayazstorage/kevinsayIoT/{*}/{*}/{*}/{*}/{*}/{*}&quot;;
        DECLARE @output_file string = @&quot;/output/output.csv&quot;;

        @rs =
        EXTRACT
        EnqueuedTimeUtc string,
        Body byte[]
        FROM @input_file

        USING new Microsoft.Analytics.Samples.Formats.ApacheAvro.AvroExtractor(@&quot;
        {
            &quot;&quot;type&quot;&quot;:&quot;&quot;record&quot;&quot;,
            &quot;&quot;name&quot;&quot;:&quot;&quot;Message&quot;&quot;,
            &quot;&quot;namespace&quot;&quot;:&quot;&quot;Microsoft.Azure.Devices&quot;&quot;,
            &quot;&quot;fields&quot;&quot;:
           [{
                &quot;&quot;name&quot;&quot;:&quot;&quot;EnqueuedTimeUtc&quot;&quot;,
                &quot;&quot;type&quot;&quot;:&quot;&quot;string&quot;&quot;
            },
            {
                &quot;&quot;name&quot;&quot;:&quot;&quot;Properties&quot;&quot;,
                &quot;&quot;type&quot;&quot;:
                {
                    &quot;&quot;type&quot;&quot;:&quot;&quot;map&quot;&quot;,
                    &quot;&quot;values&quot;&quot;:&quot;&quot;string&quot;&quot;
                }
            },
            {
                &quot;&quot;name&quot;&quot;:&quot;&quot;SystemProperties&quot;&quot;,
                &quot;&quot;type&quot;&quot;:
                {
                    &quot;&quot;type&quot;&quot;:&quot;&quot;map&quot;&quot;,
                    &quot;&quot;values&quot;&quot;:&quot;&quot;string&quot;&quot;
                }
            },
            {
                &quot;&quot;name&quot;&quot;:&quot;&quot;Body&quot;&quot;,
                &quot;&quot;type&quot;&quot;:[&quot;&quot;null&quot;&quot;,&quot;&quot;bytes&quot;&quot;]
            }]
        }"
        );

        @cnt =
        SELECT EnqueuedTimeUtc AS time, Encoding.UTF8.GetString(Body) AS jsonmessage
        FROM @rs;

        OUTPUT @cnt TO @output_file USING Outputters.Text(); 
    ```

    Для запуска этого сценария в Data Lake Analytics потребовалось пять минут. Сценарий был ограничен 10 аналитическими единицами и обработал 177 файлов. Результат в виде файла CSV показан на следующем рисунке:

    ![Результаты вывода в файл CSV](./media/iot-hub-query-avro-data/query-avro-data-7b.png)

    ![Выходные данные, преобразованные в файл CSV](./media/iot-hub-query-avro-data/query-avro-data-7c.png)

    Для анализа JSON перейдите к шагу 8.

8. Большинство сообщений Центра Интернета вещей находятся в формате файла JSON. Добавив следующие строки, вы сможете проанализировать сообщение в файле JSON. Это позволит добавить предложения WHERE и выводить только необходимые данные.

    ```sql
       @jsonify =
         SELECT Microsoft.Analytics.Samples.Formats.Json.JsonFunctions.JsonTuple(Encoding.UTF8.GetString(Body))
           AS message FROM @rs;
    
        /*
        @cnt =
            SELECT EnqueuedTimeUtc AS time, Encoding.UTF8.GetString(Body) AS jsonmessage
            FROM @rs;
        
        OUTPUT @cnt TO @output_file USING Outputters.Text();
        */
        
        @cnt =
            SELECT message["message"] AS iotmessage,
                message["event"] AS msgevent,
                message["object"] AS msgobject,
                message["status"] AS msgstatus,
                message["host"] AS msghost
            FROM @jsonify;
            
        OUTPUT @cnt TO @output_file USING Outputters.Text();
    ```

    В выводе отображается столбец для каждого элемента в команде `SELECT`.

    ![Вывод со столбцами для каждого элемента](./media/iot-hub-query-avro-data/query-avro-data-8.png)

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как запросить данные Avro, чтобы повысить эффективность маршрутизации сообщений из Центра Интернета вещей Azure в службы Azure.

Примеры комплексных решений, в которых используется центр Интернета вещей, см. в [документации по акселераторам решений для Интернета вещей Azure](../iot-accelerators/index.yml).

Дополнительные сведения о разработке решений с помощью Центра Интернета вещей см. в [руководстве разработчика для Центра Интернета вещей](iot-hub-devguide.md).

Дополнительные сведения о маршрутизации сообщений в центре Интернета вещей см. в статье об [отправке и получении сообщений в центре Интернета вещей](iot-hub-devguide-messaging.md).
