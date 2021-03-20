---
title: Фабрика данных Azure — часто задаваемые вопросы
description: Часто задаваемые вопросы о фабрике данных Azure.
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 4ddadec327f1a4f6aa22d51ac073b9aebb0dee82
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100383200"
---
# <a name="azure-data-factory---frequently-asked-questions"></a>Фабрика данных Azure — часто задаваемые вопросы
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию службы "Фабрика данных", см. [вопросы и ответы по службе "Фабрика данных"](../frequently-asked-questions.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="general-questions"></a>Общие вопросы
### <a name="what-is-azure-data-factory"></a>Что такое Фабрика данных Azure?
Фабрика данных представляет собой облачную службу интеграции информации, которая **автоматизирует перемещение и преобразование данных**. Как на фабрике сырье превращается в готовую продукцию с помощью оборудования, так и в фабриках данных необработанные данные собираются и преобразовываются в готовые к использованию сведения с помощью специальных служб.

Фабрика данных позволяет создавать управляемые данными рабочие процессы для перемещения данных между локальными и облачными хранилищами данных, а также обрабатывать или преобразовывать данные с помощью служб вычислений, таких как Azure HDInsight и Azure Data Lake Analytics. Для созданного конвейера, который выполняет необходимое действие, можно запланировать периодический запуск (ежечасно, ежедневно, еженедельно и т. д.).   

Дополнительные сведения см. в статье [Общие сведения о службе фабрики данных Azure, службе интеграции данных в облаке](data-factory-introduction.md).

### <a name="where-can-i-find-pricing-details-for-azure-data-factory"></a>Где можно найти подробную информацию о ценах на фабрику данных Azure?
Сведения о ценах на фабрику данных Azure см. на [странице сведений о ценах на фабрику данных][adf-pricing-details] .  

### <a name="how-do-i-get-started-with-azure-data-factory"></a>Вопрос. Как приступить к работе с фабрикой данных Azure?
* Общие сведения о фабрике данных Azure см. в разделе [Общие сведения о службе фабрики данных Azure, службе интеграции данных в облаке](data-factory-introduction.md).
* Инструкции по **копированию и перемещению данных** с помощью действия копирования см. в статье [Копирование данных из хранилища BLOB-объектов Azure в Базу данных SQL с помощью фабрики данных](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
* Учебник по **преобразованию данных** с помощью действия HDInsight Hive доступен в разделе [Начало работы с фабрикой данных Azure](data-factory-build-your-first-pipeline.md).

### <a name="what-is-the-data-factorys-region-availability"></a>Какова региональная доступность службы "Фабрика данных"?
Фабрика данных доступна в **западной части США** и **Северной Европе**. Службы вычислений и хранения, используемые фабриками данных, могут быть и в других регионах. Ознакомьтесь с разделом [Поддерживаемые регионы](data-factory-introduction.md#supported-regions).

### <a name="what-are-the-limits-on-number-of-data-factoriespipelinesactivitiesdatasets"></a>Каковы ограничения на число фабрик данных/конвейеров/действий/наборов данных?
Ознакомьтесь с разделом **Ограничения фабрики данных Azure** в статье [Подписка Azure, границы, квоты и ограничения службы](../../azure-resource-manager/management/azure-subscription-service-limits.md#data-factory-limits).

### <a name="what-is-the-authoringdeveloper-experience-with-azure-data-factory-service"></a>В чем заключается взаимодействие разработчика со службой фабрики данных Azure?
Вы можете проектировать и создавать фабрики данных с помощью одного из следующих инструментов или пакетов SDK.

* **Visual Studio.** Фабрику данных Azure можно создать с помощью Visual Studio. Дополнительные сведения см. в разделе [Создание первой фабрики данных Azure с помощью Microsoft Visual Studio](data-factory-build-your-first-pipeline-using-vs.md).
* **Azure PowerShell.** В разделе [Создание первой фабрики данных Azure с помощью Azure PowerShell](data-factory-build-your-first-pipeline-using-powershell.md) приведено пошаговое руководство по созданию фабрики данных с помощью PowerShell. Полная документация по командлетам фабрики данных содержится в [справочнике по командлетам фабрики данных][adf-powershell-reference], который можно найти в библиотеке MSDN.
* **Библиотека классов .NET.** Фабрики данных можно создавать программными средствами с помощью пакета SDK .NET для фабрик данных. Пошаговое руководство по созданию фабрики данных с помощью пакета SDK для .NET см. в разделе [Создание, мониторинг фабрик данных и управление ими с помощью пакета SDK для .NET](data-factory-create-data-factories-programmatically.md). Полную документация по пакету SDK .NET для фабрик данных см. в [справочнике по библиотеке классов фабрики данных][msdn-class-library-reference].
* Для создания и развертывания фабрик данных можно также использовать **REST API**, предоставляемый службой фабрики данных Azure. Полную документацию по REST API для фабрик данных см. в [справочнике по REST API фабрики данных][msdn-rest-api-reference].
* **Шаблон Azure Resource Manager.** Дополнительные сведения см. в статье [Руководство. Создание фабрики данных Azure с помощью шаблона Azure Resource Manager](data-factory-build-your-first-pipeline-using-arm.md).

### <a name="can-i-rename-a-data-factory"></a>Можно ли переименовать фабрику данных?
Нет. Как и для других ресурсов Azure, имя фабрики данных Azure изменить нельзя.

### <a name="can-i-move-a-data-factory-from-one-azure-subscription-to-another"></a>Можно ли переместить фабрику данных из одной подписки Azure в другую?
Да. Используйте кнопку **переместить** в колонке фабрики данных, как показано на следующей схеме:

![Перемещение фабрики данных](media/data-factory-faq/move-data-factory.png)

### <a name="what-are-the-compute-environments-supported-by-data-factory"></a>Какие вычислительные среды поддерживает фабрика данных?
Следующая таблица содержит список вычислительных сред, поддерживаемых фабрикой данных, и доступных в них действий.

| Вычислительная среда | Действия |
| --- | --- |
| [Кластер HDInsight по запросу](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) или [собственный кластер HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) |[DotNet](data-factory-use-custom-activities.md), [Hive](data-factory-hive-activity.md), [Pig](data-factory-pig-activity.md), [MapReduce](data-factory-map-reduce.md), [потоковая передача Hadoop](data-factory-hadoop-streaming-activity.md) |
| [Пакетная служба Azure](data-factory-compute-linked-services.md#azure-batch-linked-service) |[DotNet](data-factory-use-custom-activities.md) |
| [Машинное обучение Azure Studio (классическая модель)](data-factory-compute-linked-services.md#azure-machine-learning-studio-classic-linked-service) |[Действия Studio (классическая модель): выполнение пакета и обновление ресурса](data-factory-azure-ml-batch-execution-activity.md) |
| [Azure Data Lake Analytics](data-factory-compute-linked-services.md#azure-data-lake-analytics-linked-service) |[Аналитика озера данных U-SQL](data-factory-usql-activity.md) |
| [Azure SQL](data-factory-compute-linked-services.md#azure-sql-linked-service), [azure синапсе Analytics](data-factory-compute-linked-services.md#azure-synapse-analytics-linked-service) [SQL Server](data-factory-compute-linked-services.md#sql-server-linked-service) |[Хранимая процедура](data-factory-stored-proc-activity.md) |

### <a name="how-does-azure-data-factory-compare-with-sql-server-integration-services-ssis"></a>Чем отличаются возможности фабрики данных Azure и SQL Server Integration Services (SSIS)? 
Ознакомьтесь с представлением [фабрики данных Azure и служб SSIS](https://www.sqlbits.com/Sessions/Event15/Azure_Data_Factory_vs_SSIS) от одного из наших MVP (наиболее ценных специалистов): Рэд Reza RAD. Некоторые из последних изменений в фабрике данных могут отсутствовать в наборе слайдов. Мы постоянно добавляем в фабрику данных Azure дополнительные возможности. Мы постоянно добавляем в фабрику данных Azure дополнительные возможности. Мы внесем эти обновления в сравнение технологий интеграции данных от корпорации Майкрософт позже в этом году.   

## <a name="activities---faq"></a>Действия — вопросы и ответы
### <a name="what-are-the-different-types-of-activities-you-can-use-in-a-data-factory-pipeline"></a>Какие типы действий использовать в конвейере фабрики данных Azure?
* [Действия перемещения данных](data-factory-data-movement-activities.md) для перемещения данных.
* [Действия преобразования данных](data-factory-data-transformation-activities.md) для обработки или преобразования данных.

### <a name="when-does-an-activity-run"></a>Когда запускается действие?
Параметр конфигурации **availability** в таблице выходных данных определяет, когда выполняется действие. Если указаны входные наборы данных, то прежде чем действие начнет выполняться, оно проверяет, все ли зависимости входных данных выполнены (т. е. находятся в состоянии **Готов**).

## <a name="copy-activity---faq"></a>Действие копирования — вопросы и ответы
### <a name="is-it-better-to-have-a-pipeline-with-multiple-activities-or-a-separate-pipeline-for-each-activity"></a>Что лучше: конвейер с несколькими действиями или отдельный конвейер для каждого действия?
Предполагается, что конвейеры объединяют связанные действия. Действия можно хранить в одном конвейере, если связывающие их наборы данных не используются каким-либо действием за пределами конвейера. Таким образом, вам не придется объединять активные периоды конвейера в цепочку, чтобы они были согласованы друг с другом. Кроме того, целостность данных в таблицах конвейера лучше сохраняется при обновлении конвейера. В сущности, обновление конвейера останавливает все действия в конвейере, удаляет их и создает их снова. С точки зрения разработки также будет легче просматривать поток данных в связанных действиях в одном JSON-файле для конвейера.

### <a name="what-are-the-supported-data-stores"></a>Какие хранилища данных поддерживаются?
Действие копирования в фабрике данных копирует данные из хранилища-источника в хранилище-приемник. Фабрика данных поддерживает приведенные ниже хранилища данных. Данные из любого источника можно записывать в любой приемник. Щелкните название хранилища, чтобы узнать, как скопировать данные из него или в него.

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> Хранилища данных, отмеченные звездочкой (*), могут находиться в локальном расположении или в IaaS Azure и требовать установки [шлюза управления данными](data-factory-data-management-gateway.md) на локальном компьютере или компьютере IaaS Azure.

### <a name="what-are-the-supported-file-formats"></a>Какие форматы файлов поддерживаются?
[!INCLUDE [data-factory-file-format](../../../includes/data-factory-file-format.md)]

### <a name="where-is-the-copy-operation-performed"></a>Где выполняется операция копирования?
Подробные сведения см. в разделе [Глобально доступное перемещение данных](data-factory-data-movement-activities.md#global). Если задействовано локальное хранилище данных, то операцию копирования выполняет шлюз управления данными в локальной среде. Когда данные перемещаются между двумя облачными хранилищами, операция копирования выполняется в ближайшем к принимающему хранилищу регионе в той же географической области.

## <a name="hdinsight-activity---faq"></a>Действие HDInsight — вопросы и ответы
### <a name="what-regions-are-supported-by-hdinsight"></a>В каких регионах поддерживается HDInsight?
Ознакомьтесь с разделом "Географическая доступность" в следующей статье или см. [сведения о ценах на HDInsight][hdinsight-supported-regions].

### <a name="what-region-is-used-by-an-on-demand-hdinsight-cluster"></a>Какой регион используется кластером HDInsight по запросу?
Кластер HDInsight по запросу создается в том же регионе, где существует хранилище, которое вы указали для использования с кластером.    

### <a name="how-to-associate-additional-storage-accounts-to-your-hdinsight-cluster"></a>Как связать дополнительные учетные записи хранения с кластером HDInsight?
Если вы используете собственный кластер HDInsight (BYOC), ознакомьтесь со следующими разделами:

* [Использование кластера HDInsight с дополнительными учетными записями хранения и метахранилищами][hdinsight-alternate-storage]
* [Использование дополнительных учетных записей хранения с Hive HDInsight][hdinsight-alternate-storage-2]

При использовании кластера по запросу, созданного службой фабрики данных, следует указать дополнительные учетные записи хранения для связанной службы HDInsight, чтобы служба фабрики данных могла зарегистрировать их от вашего имени. В определении JSON для связанной службы по запросу используйте свойство **additionalLinkedServiceNames** , чтобы указать дополнительные хранилища учетных записей, как показано в следующем фрагменте кода JSON:

```JSON
{
    "name": "MyHDInsightOnDemandLinkedService",
    "properties":
    {
        "type": "HDInsightOnDemandLinkedService",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "LinkedService-SampleData",
            "additionalLinkedServiceNames": [ "otherLinkedServiceName1", "otherLinkedServiceName2" ]
        }
    }
}
```
В приведенном выше примере otherLinkedServiceName1 и otherLinkedServiceName2 представляют связанные службы, определения которых содержат учетные данные, необходимые кластеру HDInsight для доступа к дополнительным учетным записям хранения.

## <a name="slices---faq"></a>Срезы — вопросы и ответы
### <a name="why-are-my-input-slices-not-in-ready-state"></a>Почему мои срезы входных данных не находятся в состоянии готовности?
Обычно это происходит, если свойству **external** не присвоено значение **True** во входном наборе данных, когда входные данные являются внешними для фабрики (не производятся фабрикой данных).

В следующем примере свойству **external** следует присвоить значение True только в наборе данных **dataset1**.  

Конвейер 1 **фабрики данных 1**: набор данных 1 -> действие 1 -> набор данных 2 -> действие 2 -> набор данных 3. Конвейер 2: набор данных 3-> действие 3 -> набор данных 4.

Если имеется другая фабрика данных с конвейером, который принимает набор данных 4 (созданный конвейером 2 в фабрике данных 1), то следует пометить набор данных 4 как внешний (external), так как он производится фабрикой данных (фабрикой данных 1, не фабрикой данных 2).  

**DataFactory2**    
Конвейер 1: набор данных 4 -> действие 4 -> набор данных 5.

Если свойство external задано неверно, проверьте, существуют ли входные данные в расположении, заданном в определении входного набора данных.

### <a name="how-to-run-a-slice-at-another-time-than-midnight-when-the-slice-is-being-produced-daily"></a>Как выполнить срез в другое время (не в полночь), если срез формируется ежедневно?
Время формирования необходимого среза можно указать с помощью свойства **offset**. Сведения об этом свойстве см. в разделе [Доступность набора данных](data-factory-create-datasets.md#dataset-availability). Приведем краткий пример:

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
Ежедневно срезы создаются в **06:00** , а не в полночь (значение по умолчанию).     

### <a name="how-can-i-rerun-a-slice"></a>Как повторно выполнять срез?
Вы можете повторно выполнить срез одним из следующих способов:

* Запустите окно действия или срез повторно с помощью приложения для мониторинга и управления. Инструкции см. в разделе [Повторное выполнение выбранных окон действий](data-factory-monitor-manage-app.md#perform-batch-actions).   
* Щелкните **Выполнить** в командной строке в колонке **Срез данных** для среза на портале Azure.
* Выполните командлет **Set-аздатафакторислицестатус** с параметром status, имеющим значение **Ожидание** среза.   

    ```powershell
    Set-AzDataFactorySliceStatus -Status Waiting -ResourceGroupName $ResourceGroup -DataFactoryName $df -TableName $table -StartDateTime "02/26/2015 19:00:00" -EndDateTime "02/26/2015 20:00:00"
    ```
  Дополнительные сведения о командлете см. в разделе [Set-аздатафакторислицестатус][set-azure-datafactory-slice-status] .

### <a name="how-long-did-it-take-to-process-a-slice"></a>Сколько времени занимает обработка среза?
Узнать, сколько времени занимает обработка среза данных, можно в обозревателе окон действий в приложении мониторинга и управления. Дополнительные сведения см. в разделе [Обозреватель окон действий](data-factory-monitor-manage-app.md#activity-window-explorer).

На портале Azure можно также выполнить следующие действия:  

1. Щелкните элемент **Наборы данных** в колонке **Фабрика данных** для своей фабрики данных.
2. Щелкните конкретный набор данных в колонке **Наборы данных** .
3. Выберите интересующий вас срез из списка **последних срезов** в колонке **Таблица**.
4. Щелкните выполняемое действие в списке **выполняемых действий** в колонке **Срез данных**.
5. Щелкните плитку **Свойства** в колонке **Подробности о выполнении операции**.
6. Вы увидите поле **ДЛИТЕЛЬНОСТЬ** со значением. Это время, затраченное на обработку среза.   

### <a name="how-to-stop-a-running-slice"></a>Как остановить выполнение среза?
Если необходимо остановить выполнение конвейера, можно использовать командлет [Suspend-аздатафакторипипелине](/powershell/module/az.datafactory/suspend-azdatafactorypipeline) . В настоящее время приостановка конвейера не останавливает выполняющиеся срезы. После завершения текущих выполняемых задач никакие дополнительные срезы выбираются.

Если вы действительно хотите немедленно прекратить все выполняемые задачи, единственным способом является удаление конвейера и создание его заново. Если вы решили удалить конвейер, НЕ обязательно удалять таблицы и связанные службы, используемые конвейером.

[create-factory-using-dotnet-sdk]: data-factory-create-data-factories-programmatically.md
[msdn-class-library-reference]: /dotnet/api/microsoft.azure.management.datafactories.models
[msdn-rest-api-reference]: /rest/api/datafactory/

[adf-powershell-reference]: /powershell/module/az.datafactory/
[azure-portal]: https://portal.azure.com
[set-azure-datafactory-slice-status]: /powershell/module/az.datafactory/set-Azdatafactoryslicestatus

[adf-pricing-details]: https://go.microsoft.com/fwlink/?LinkId=517777
[hdinsight-supported-regions]: https://azure.microsoft.com/pricing/details/hdinsight/
[hdinsight-alternate-storage]: https://social.technet.microsoft.com/wiki/contents/articles/23256.using-an-hdinsight-cluster-with-alternate-storage-accounts-and-metastores.aspx
[hdinsight-alternate-storage-2]: /archive/blogs/cindygross/use-additional-storage-accounts-with-hdinsight-hive