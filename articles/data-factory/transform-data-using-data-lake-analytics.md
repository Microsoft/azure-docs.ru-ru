---
title: Преобразование данных с помощью сценария U-SQL
description: Узнайте, как обрабатывать и преобразовывать данные с помощью сценариев U-SQL в службе вычислений Azure Data Lake Analytics.
ms.author: abnarain
author: nabhishek
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/01/2018
ms.openlocfilehash: 94b584f0bea01754a4e955d1418a7a9a2e9aeace
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100373918"
---
# <a name="process-data-by-running-u-sql-scripts-on-azure-data-lake-analytics"></a>Обработка данных путем выполнения скриптов U-SQL на Azure Data Lake Analytics 
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-usql-activity.md)
> * [Текущая версия](transform-data-using-data-lake-analytics.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Конвейер в фабрике данных Azure обрабатывает данные в связанной службе хранилища с помощью связанных вычислительных служб. В нем содержится последовательность действий, каждое из которых выполняет определенную операцию обработки. В этой статье описывается **действие U-SQL в Data Lake Analytics**, которое запускает сценарий **U-SQL** в связанной службе вычислений **Azure Data Lake Analytics**. 

Перед созданием конвейера с действием U-SQL в Data Lake Analytics следует создать учетную запись Data Lake Analytics. Дополнительные сведения об Azure Data Lake Analytics см. в статье [Начало работы с аналитикой озера данных Azure](../data-lake-analytics/data-lake-analytics-get-started-portal.md).


## <a name="azure-data-lake-analytics-linked-service"></a>Связанная служба Azure Data Lake Analytics
Можно создать связанную службу **Azure Data Lake Analytics** , чтобы связать службу вычислений Azure Data Lake Analytics с фабрикой данных Azure. Действие U-SQL Data Lake Analytics в конвейере ссылается на эту связанную службу. 

В следующей таблице приведены описания универсальных свойств из определения JSON. 

| Свойство                 | Описание                              | Обязательно                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| **type**                 | Свойству type необходимо присвоить значение **AzureDataLakeAnalytics**. | Да                                      |
| **accountName**          | Имя учетной записи аналитики озера данных Azure.  | Да                                      |
| **dataLakeAnalyticsUri** | Универсальный код ресурса (URI) аналитики озера данных Azure.           | Нет                                       |
| **subscriptionId**       | Идентификатор подписки Azure.                    | Нет                                       |
| **resourceGroupName**    | Имя группы ресурсов Azure                | Нет                                       |

### <a name="service-principal-authentication"></a>Проверка подлинности субъекта-службы
Для связанной службы Azure Data Lake Analytics необходимо выполнить проверку подлинности субъекта-службы, чтобы подключиться к службе Azure Data Lake Analytics. При использовании проверки подлинности на основе субъекта-службы необходимо зарегистрировать сущность приложения в Azure Active Directory (Azure AD) и предоставить ей доступ к Data Lake Analytics и Data Lake Store, которые она использует. Подробные инструкции см. в статье [Аутентификация между службами в Data Lake Store с помощью Azure Active Directory](../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md). Запишите следующие значения, которые используются для определения связанной службы:

* Идентификатор приложения
* Ключ приложения 
* Tenant ID

Предоставьте разрешение субъекта-службы службе Azure Data Lake Anatlyics с помощью [мастера добавления пользователей](../data-lake-analytics/data-lake-analytics-manage-use-portal.md#add-a-new-user).

Используйте проверку подлинности на основе субъекта-службы, указав следующие свойства:

| Свойство                | Описание                              | Обязательно |
| :---------------------- | :--------------------------------------- | :------- |
| **servicePrincipalId**  | Укажите идентификатора клиента приложения.     | Да      |
| **servicePrincipalKey** | Укажите ключ приложения.           | Да      |
| **tenant**              | Укажите сведения о клиенте (доменное имя или идентификатор клиента), в котором находится приложение. Эти сведения можно получить, наведя указатель мыши на правый верхний угол страницы портала Azure. | Да      |

**Пример. Проверка подлинности субъекта-службы**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "<account name>",
            "dataLakeAnalyticsUri": "<azure data lake analytics URI>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "value": "<service principal key>",
                "type": "SecureString"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }       
    }
}
```

Дополнительные сведения о связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md).

## <a name="data-lake-analytics-u-sql-activity"></a>Действие U-SQL в Data Lake Analytics
В следующем фрагменте кода JSON определяется конвейер с действием U-SQL в аналитике озера данных. Определение действия содержит ссылку на созданную ранее связанную службу аналитики озера данных Azure. Чтобы выполнить скрипт U-SQL Data Lake Analytics, фабрика данных отправляет указанный скрипт в Data Lake Analytics. Необходимые входные и выходные данные определяются в скрипте Data Lake Analytics для извлечения и вывода. 

```json
{
    "name": "ADLA U-SQL Activity",
    "description": "description",
    "type": "DataLakeAnalyticsU-SQL",
    "linkedServiceName": {
        "referenceName": "<linked service name of Azure Data Lake Analytics>",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "scriptLinkedService": {
            "referenceName": "<linked service name of Azure Data Lake Store or Azure Storage which contains the U-SQL script>",
            "type": "LinkedServiceReference"
        },
        "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
        "degreeOfParallelism": 3,
        "priority": 100,
        "parameters": {
            "in": "/datalake/input/SearchLog.tsv",
            "out": "/datalake/output/Result.tsv"
        }
    }   
}
```

В следующей таблице описаны имена и описания свойств, относящихся к этому действию. 

| Свойство            | Описание                              | Обязательно |
| :------------------ | :--------------------------------------- | :------- |
| name                | Имя действия в конвейере.     | Да      |
| description         | Описание действия.  | Нет       |
| type                | Для действия U-SQL Data Lake Analytics в качестве типа действия используется **DataLakeAnalyticsU-SQL**. | Да      |
| linkedServiceName   | Связанная служба Azure Data Lake Analytics. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md).  |Да       |
| scriptPath          | Путь к папке, содержащей скрипт U-SQL В имени файла учитывается регистр. | Да      |
| scriptLinkedService | Связанная служба, которая связывает с фабрикой данных службу **Azure Data Lake Store** или **службу хранилища Azure**, содержащую скрипт. | Да      |
| degreeOfParallelism | Максимальное количество узлов, используемых одновременно для выполнения задания. | Нет       |
| priority            | Определяет, какие задания из всех в очереди должны запускаться в первую очередь. Чем меньше число, тем выше приоритет. | Нет       |
| параметры          | Параметры для передачи в скрипт U-SQL.    | Нет       |
| runtimeVersion      | Версия среды выполнения обработчика U-SQL, которую нужно использовать. | Нет       |
| compilationMode     | <p>Режим компиляции U-SQL. Может иметь одно из следующих значений. **Semantic**: выполнение только семантических проверок и необходимых проверок работоспособности. **Full**: выполнение полной компиляции, включая проверку синтаксиса, оптимизацию, создание кода и т. д. **SingleBox**: выполнение полной компиляции с параметром TargetType, заданным для SingleBox. Если не указать значение для этого свойства, сервер определит оптимальный режим компиляции. | Нет |

Определение сценария см. в разделе [Пример скрипта U-SQL](#sample-u-sql-script). 

## <a name="sample-u-sql-script"></a>Пример скрипта U-SQL

```
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int,
            Urls            string,
            ClickedUrls     string
    FROM @in
    USING Extractors.Tsv(nullEscape:"#NULL#");

@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";

@rs1 =
    SELECT Start, Region, Duration
    FROM @rs1
    WHERE Start <= DateTime.Parse("2012/02/19");

OUTPUT @rs1   
    TO @out
      USING Outputters.Tsv(quoting:false, dateTimeFormat:null);
```

В приведенном выше примере входные и выходные данные скрипта определяются в параметрах **\@in** и **\@out**. Значения параметров **\@in** и **\@out** в скрипте U-SQL передаются динамически с помощью Фабрики данных, с использованием раздела parameters. 

В определении конвейера для заданий, которые выполняются в службе Azure Data Lake Analytics, можно определить другие свойства, например degreeOfParallelism и priority.

## <a name="dynamic-parameters"></a>Динамические параметры
В примере определения конвейера параметрам in и out присвоено жестко заданные значения. 

```json
"parameters": {
    "in": "/datalake/input/SearchLog.tsv",
    "out": "/datalake/output/Result.tsv"
}
```

Вместо этого можно использовать динамические параметры. Пример: 

```json
"parameters": {
    "in": "/datalake/input/@{formatDateTime(pipeline().parameters.WindowStart,'yyyy/MM/dd')}/data.tsv",
    "out": "/datalake/output/@{formatDateTime(pipeline().parameters.WindowStart,'yyyy/MM/dd')}/result.tsv"
}
```

В этом случае входные файлы по-прежнему берутся из папки /datalake/input, а выходные файлы создаются в папке /datalake/output. Имена файлов являются динамическими и зависят от времени начала окна, которое передается, когда активируется конвейер.  

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами: 

* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие Pig](transform-data-using-hadoop-pig.md)
* [Действие MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Действие потоковой передачи Hadoop](transform-data-using-hadoop-streaming.md)
* [Действие Spark](transform-data-using-spark.md)
* [Настраиваемое действие .NET](transform-data-using-dotnet-custom-activity.md)
* [Действие выполнения пакета Машинное обучение Azure Studio (классическая модель)](transform-data-using-machine-learning.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)