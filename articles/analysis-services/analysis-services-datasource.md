---
title: Источники данных, поддерживаемые в службах Azure Analysis Services | Документы Майкрософт
description: Описание источников данных и соединителей, поддерживаемых для табличных моделей данных уровня 1200 и выше в службах Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 02/08/2021
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 03f4d151fc948e4c060989a7d101ad91aecdecb6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99981499"
---
# <a name="data-sources-supported-in-azure-analysis-services"></a>Источники данных, поддерживаемые в службах Azure Analysis Services

Источники данных и соединители, представленные в функции получения данных или мастере импорта таблиц в Visual Studio, отображаются как для служб Azure Analysis Services, так и для служб SQL Server Analysis Services. Однако в Azure Analysis Services поддерживаются не все показанные источники данных и соединители. Типы источников данных, к которым можно подключиться, зависят от многих факторов, таких как уровень совместимости модели, доступные соединители данных, тип аутентификации и поддержка локального шлюза данных. В следующих таблицах описаны поддерживаемые источники данных для Azure Analysis Services.

## <a name="azure-data-sources"></a>Источники данных Azure

|Источник данных  |В памяти  |DirectQuery  |Примечания |
|---------|---------|---------|---------|
|База данных SQL Azure      |   Да      |    Да      |<sup>[2](#azprovider)</sup>, <sup>[3](#azsqlmanaged)</sup>|
|Azure Synapse Analytics (хранилище данных SQL)      |   Да      |   Да       |<sup>[2](#azprovider)</sup>|
|хранилище BLOB-объектов Azure      |   Да       |    Нет      | <sup>[1](#tab1400a)</sup> |
|Хранилище таблиц Azure     |   Да       |    Нет      | <sup>[1](#tab1400a)</sup>|
|Azure Cosmos DB     |  Да        |  Нет        |<sup>[1](#tab1400a)</sup> |
|Azure Data Lake Store 1-го поколения      |   Да       |    Нет      |<sup>[1](#tab1400a)</sup> |
|Azure Data Lake Store 2-го поколения       |   Да       |    Нет      |<sup>[1](#tab1400a)</sup>, <sup>[5](#gen2)</sup>|
|Azure HDInsight — HDFS    |     Да     |   Нет       |<sup>[1](#tab1400a)</sup> |
|Azure HDInsight — Spark     |   Да       |   Нет       |<sup>[1](#tab1400a)</sup>, <sup>[4](#databricks)</sup>|
||||

**Примечания.**

<a name="tab1400a">1</a> Только табличная модель 1400 и более поздние.  
<a name="azprovider">2</a> — при указании в качестве источника данных *поставщика* в табличных 1200 и более высоких моделях модели в памяти и в моделях DirectQuery требуется драйвер Microsoft OLE DB для SQL Server мсоледбскл (рекомендуется) или платформа .NET Framework поставщик данных для SQL Server.  
<a name="azsqlmanaged">3</a> — поддерживается управляемый экземпляр Azure SQL. Поскольку Управляемый экземпляр SQL выполняется в виртуальной сети Azure с частным IP-адресом, на экземпляре должна быть включена общедоступная конечная точка. Если она не включена, то требуется [Локальный шлюз данных](analysis-services-gateway.md).  
<a name="databricks">4</a> — экземпляр Azure Databricks, использующий соединитель Spark, в настоящее время не поддерживается.  
<a name="gen2">5</a> — соединитель с ADLS 2-го поколения в настоящее время не поддерживается, однако соединитель хранилища BLOB-объектов Azure можно использовать с источниками данных ADLS 2-го поколения.

## <a name="other-data-sources"></a>Другие источники данных

|Источник данных | В памяти | DirectQuery |Примечания   |
|  --- | --- | --- | --- |
|База данных Access     |  Да | Нет |  |
|Active Directory     |  Да | Нет | <sup>[6](#tab1400b)</sup>  |
|Службы Analysis Services     |  Да | Нет |  |
|Система платформы аналитики     |  Да | Нет |  |
|CSV-файл  |Да | Нет |  |
|Dynamics 365     |  Да | Нет | <sup>[6](#tab1400b)</sup> |
|Книга Excel     |  Да | Нет |  |
|Exchange      |  Да | Нет | <sup>[6](#tab1400b)</sup> |
|Папка      |Да | Нет | <sup>[6](#tab1400b)</sup> |
|IBM Informix  |Да | Нет |  |
|Документ JSON      |  Да | Нет | <sup>[6](#tab1400b)</sup> |
|Строки из двоичного файла      | Да | Нет | <sup>[6](#tab1400b)</sup> |
|База данных MySQL     | Да | Нет |  |
|Веб-канал OData      |  Да | Нет | <sup>[6](#tab1400b)</sup> |
|Запрос ODBC     | Да | Нет |  |
|OLE DB     |   Да | Нет |  |
|Oracle;  | Да  |Да  | <sup>[9](#oracle)</sup> |
|База данных PostgreSQL   | Да | Нет | <sup>[6](#tab1400b)</sup> |
|Объекты Salesforce|  Да | Нет | <sup>[6](#tab1400b)</sup> |
|Отчеты SalesForce |Да | Нет | <sup>[6](#tab1400b)</sup> |
|SAP HANA     |  Да | Нет |  |
|SAP Business Warehouse    |  Да | Нет | <sup>[6](#tab1400b)</sup> |
|SharePoint      |   Да | Нет | <sup>[6](#tab1400b)</sup>, <sup>[11](#filesSP)</sup> |
|SQL Server |Да   | Да  | <sup>[7](#sqlim)</sup>, <sup>[8](#instgw)</sup> |
|хранилище данных SQL Server. |Да   | Да  | <sup>[7](#sqlim)</sup>, <sup>[8](#instgw)</sup> |
|База данных Sybase     |  Да | Нет |  |
|Teradata | Да  | Да  | <sup>[10](#teradata)</sup> |
|TXT-файл  |Да | Нет |  |
|Таблица XML    |  Да | Нет | <sup>[6](#tab1400b)</sup> |
| | | |

**Примечания.**  
<a name="tab1400b">6</a> — только табличная модель 1400 и более поздние.  
<a name="sqlim">7</a> — при указании в качестве *поставщика* источника данных в табличных моделях данных 1200 и выше укажите драйвер MSOLEDBSQL (рекомендуется), Microsoft OLE DB Driver for SQL Server, SQL Server Native Client 11.0 или поставщик данных .NET Framework для SQL Server.  
<a name="instgw">8</a> — при указании MSOLEDBSQL в качестве поставщика данных может потребоваться скачать и установить [Microsoft OLE DB Driver for SQL Server](/sql/connect/oledb/oledb-driver-for-sql-server) на том же компьютере, что и локальный шлюз данных.  
<a name="oracle">9</a> — укажите поставщик данных Oracle для .NET для табличных моделей 1200 или как *поставщик* источника данных в табличных моделях 1400 и выше. При указании в качестве структурированного источника данных обязательно [Включите управляемый поставщик Oracle](#enable-oracle-managed-provider).   
<a name="teradata">10</a> — укажите поставщик данных Teradata для .NET для табличных моделей уровня 1200 или как *поставщик* источника данных в табличных моделях 1400 и выше.  
<a name="filesSP">11</a> — файлы в локальной среде SharePoint не поддерживаются.

Для подключения к локальным источникам данных с сервера Azure Analysis Services требуется [локальный шлюз](analysis-services-gateway.md). При использовании шлюза требуются 64-разрядные поставщики.

## <a name="understanding-providers"></a>Основные сведения о поставщиках

При создании табличных проектов уровня 1400 и выше в Visual Studio по умолчанию поставщик данных не указывается при подключении к источнику данных с помощью функции **получения данных**. Табличные модели 1400 и выше используют соединители [Power Query](/power-query/power-query-what-is-power-query) для управления подключениями, запросами к данным и комбинированием между источником данных и Analysis Services. Их иногда называют *структурированными* соединениями с источниками данных, поскольку параметры свойств соединения тут настроены заранее. Однако можно включить и устаревшие источники данных для проекта модели в Visual Studio. Если они включены, можно использовать **мастер импорта таблиц** для подключения к определенным источникам данных, которые традиционно поддерживаются в табличных моделях 1200 и ниже, как к *устаревшим* источникам данных или источникам данных *поставщика*. При указании в качестве источника данных поставщика можно указать конкретного поставщика данных и другие дополнительные свойства соединения. Например, можно подключиться к SQL Server экземпляру хранилища данных или даже к базе данных SQL Azure в качестве устаревшего источника данных. Затем можно выбрать драйвер OLE DB для поставщика данных MSOLEDBSQL SQL Server. В этом случае выбор поставщика данных OLE DB может повысить производительность по сравнению с соединителем Power Query. 

При использовании мастера импорта таблиц в Visual Studio подключение к любому источнику данных требует поставщика данных. Поставщик данных по умолчанию выбирается за вас. При необходимости поставщика данных можно изменить. Выбранный тип поставщика может зависеть от производительности, независимо от того, использует ли модель хранение в памяти или DirectQuery, а также того, на какой платформе Analysis Services развертывается модель.

### <a name="specify-provider-data-sources-in-tabular-1400-and-higher-model-projects"></a>Указание источников данных поставщика в табличных проектах моделей 1400 и выше

Чтобы включить источники данных поставщика, в Visual Studio щелкните **Средства** > **Параметры** > **Табличный режим служб Analysis Services** > **Импорт данных** и выберите **Включить устаревшие источники данных**.

![Включение устаревших источников данных](media/analysis-services-datasource/aas-enable-legacy-datasources.png)

После включения устаревших источников данных в **Обозревателе табличных моделей** щелкните правой кнопкой мыши **Источники данных** > **Импортировать из источника данных (устаревшего)** .

![Устаревшие источники данных в обозревателе табличных моделей](media/analysis-services-datasource/aas-import-legacy-datasources.png)

Как и в случае с проектами табличных моделей 1200, используйте **Мастер импорта таблиц** для подключения к источнику данных. На странице "Подключение" щелкните **Дополнительно**. Укажите поставщика данных и другие параметры подключения в разделе **Установка дополнительных свойств**.

![Дополнительные свойства устаревших источников данных](media/analysis-services-datasource/aas-import-legacy-advanced.png)

## <a name="impersonation"></a>Олицетворение
В некоторых случаях может быть необходимо указать другую учетную запись олицетворения. Учетную запись олицетворения можно указать в Visual Studio или SQL Server Management Studio (SSMS).

Для локальных источников данных:

* При использовании проверки подлинности SQL олицетворением должна быть учетная запись службы.
* При использовании проверки подлинности Windows задайте пользователя и пароль Windows. Для SQL Server проверка подлинности Windows с использованием конкретной учетной записи олицетворения поддерживается только для моделей данных в памяти.

Для облачных источников данных

* При использовании проверки подлинности SQL олицетворением должна быть учетная запись службы.


## <a name="oauth-credentials"></a>Учетные данные OAuth

Для табличных моделей на уровне совместимости 1400 и выше с использованием режима *в памяти* , базы данных SQL Azure, Azure синапсе, Dynamics 365 и списка SharePoint поддерживаются учетные данные OAuth. Чтобы создать допустимые токены, задайте учетные данные с помощью Power Query. Azure Analysis Services управляет обновлением токена для источников данных OAuth во избежание истечения времени ожидания для длительных операций обновления. 
> [!NOTE]
> Обновление управляемого токена не поддерживается для источников данных, доступ к которым осуществляется через шлюз. Например, к одному или нескольким источникам данных гибридных запросов можно получить доступ через шлюз, а для свойства [аспаас\алвайсусегатевай](analysis-services-vnet-gateway.md) — **значение true**. 

Режим прямого запроса не поддерживается с учетными данными OAuth.

## <a name="enable-oracle-managed-provider"></a>Включить управляемый поставщик Oracle

В некоторых случаях запросы DAX к источнику данных Oracle могут возвращать непредвиденные результаты. Это может быть вызвано тем, что поставщик используется для соединения с источником данных.

Как описано в разделе [Общие сведения о поставщиках](#understanding-providers) , табличные модели подключаются к источникам данных как к *структурированному* источнику данных или к источнику данных *поставщика* . Для моделей с источником данных Oracle, указанным в качестве источника данных поставщика, убедитесь, что указанный поставщик является поставщиком данных Oracle для .NET (Oracle. Data Access. Client). 

Если источник данных Oracle указан в качестве структурированного источника данных, включите свойство сервера **мдатаенгине\усеманажедораклепровидер** . Задание этого свойства гарантирует, что модель подключится к источнику данных Oracle с помощью рекомендованного поставщика данных Oracle для управляемого поставщика .NET.
 
Включение управляемого поставщика Oracle:

1. В SQL Server Management Studio подключитесь к серверу.
2. Создайте запрос XMLA со следующим скриптом. Замените **ServerName** на полное имя сервера, а затем выполните запрос.

    ```xml
    <Alter AllowCreate="true" ObjectExpansion="ObjectProperties" xmlns="http://schemas.microsoft.com/analysisservices/2003/engine">
        <Object />
        <ObjectDefinition>
            <Server xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:ddl2="http://schemas.microsoft.com/analysisservices/2003/engine/2" xmlns:ddl2_2="http://schemas.microsoft.com/analysisservices/2003/engine/2/2" 
    xmlns:ddl100_100="http://schemas.microsoft.com/analysisservices/2008/engine/100/100" xmlns:ddl200="http://schemas.microsoft.com/analysisservices/2010/engine/200" xmlns:ddl200_200="http://schemas.microsoft.com/analysisservices/2010/engine/200/200" 
    xmlns:ddl300="http://schemas.microsoft.com/analysisservices/2011/engine/300" xmlns:ddl300_300="http://schemas.microsoft.com/analysisservices/2011/engine/300/300" xmlns:ddl400="http://schemas.microsoft.com/analysisservices/2012/engine/400" 
    xmlns:ddl400_400="http://schemas.microsoft.com/analysisservices/2012/engine/400/400" xmlns:ddl500="http://schemas.microsoft.com/analysisservices/2013/engine/500" xmlns:ddl500_500="http://schemas.microsoft.com/analysisservices/2013/engine/500/500">
                <ID>ServerName</ID>
                <Name>ServerName</Name>
                <ServerProperties>
                    <ServerProperty>
                        <Name>MDataEngine\UseManagedOracleProvider</Name>
                        <Value>1</Value>
                    </ServerProperty>
                </ServerProperties>
            </Server>
        </ObjectDefinition>
    </Alter>
    ```

3. Перезапустите сервер.


## <a name="next-steps"></a>Дальнейшие действия

* [Локальный шлюз](analysis-services-gateway.md)
* [Управление службами Analysis Services](analysis-services-manage.md)
