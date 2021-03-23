---
title: Вычислительные среды, поддерживаемые фабрикой данных Azure версии 1
description: Сведения о вычислительных средах, которые можно использовать в конвейерах фабрики данных Azure (например, Azure HDInsight) для преобразования или обработки данных.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 5f6fbcb73b4139c0a80ea8352071d8683c401d6e
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104782951"
---
# <a name="compute-environments-supported-by-azure-data-factory-version-1"></a>Вычислительные среды, поддерживаемые фабрикой данных Azure версии 1
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию службы "Фабрика данных", см. руководство по [использованию связанных служб вычисления в службе "Фабрика данных"](../compute-linked-services.md).

В этой статье описываются вычислительные среды, которые можно использовать для обработки и преобразования данных. Здесь содержатся также сведения о различных конфигурациях (конфигурациях по запросу и ваших собственных), которые поддерживаются фабрикой данных при настройке связанных служб, связывающих эти вычислительные среды с фабрикой данных Azure.

Следующая таблица содержит список вычислительных сред, поддерживаемых фабрикой данных, и доступных в них действий. 

| Вычислительная среда                      | Действия                               |
| ---------------------------------------- | ---------------------------------------- |
| [Кластер Azure HDInsight по запросу](#azure-hdinsight-on-demand-linked-service) или [собственный кластер HDInsight](#azure-hdinsight-linked-service) | [DotNet](data-factory-use-custom-activities.md), [Hive](data-factory-hive-activity.md), [Pig](data-factory-pig-activity.md), [MapReduce](data-factory-map-reduce.md), [потоковая передача Hadoop](data-factory-hadoop-streaming-activity.md) |
| [Пакетная служба Azure](#azure-batch-linked-service) | [DotNet](data-factory-use-custom-activities.md) |
| [Машинное обучение Azure Studio (классическая модель)](#azure-machine-learning-studio-classic-linked-service) | [Действия Studio (классическая модель): выполнение пакета и обновление ресурса](data-factory-azure-ml-batch-execution-activity.md) |
| [Azure Data Lake Analytics](#azure-data-lake-analytics-linked-service) | [Аналитика озера данных U-SQL](data-factory-usql-activity.md) |
| [Azure SQL](#azure-sql-linked-service), [azure синапсе Analytics](#azure-synapse-analytics-linked-service) [SQL Server](#sql-server-linked-service) | [Действие хранимой процедуры](data-factory-stored-proc-activity.md) |

## <a name="hdinsight-versions-supported-in-data-factory"></a><a name="supported-hdinsight-versions-in-azure-data-factory"></a>Версии HDInsight, поддерживаемые в фабрике данных
Azure HDInsight поддерживает несколько версий кластера Hadoop, которые могут быть развернуты в любое время. Каждая поддерживаемая версия создает конкретную версию платформы HortonWorks Data Platform (HDP) и набор компонентов этой версии. 

Корпорация Майкрософт обновляет список поддерживаемых версий HDInsight, предоставляя новейшие компоненты и исправления для экосистемы Hadoop. Подробные сведения см. в статье [Поддерживаемые версии HDInsight](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions).

> [!IMPORTANT]
> HDInsight версии 3.3 под управлением Linux не используется с 31 июля 2017 г. До 15 декабря 2017 г. клиентам связанных служб HDInsight по запросу фабрики данных версии 1 нужно было протестировать и обновить службу HDInsight до более новой версии. 31 июля 2018 г. будет прекращено использование HDInsight под управлением Windows.
>
> 

### <a name="after-the-retirement-date"></a>После даты вывода 

После 15 декабря 2017 г.:

- Вы больше не можете создавать кластеры HDInsight версии 3.3 под управлением Linux (или более ранних версий) с помощью связанной службы HDInsight по запросу в фабрике данных версии 1. 
- Если свойства [ **OsType** и **Version**](#azure-hdinsight-on-demand-linked-service) не указаны явно в определении JSON для связанной службы HDInsight по запросу фабрики данных версии 1, значение по умолчанию изменяется с **версии = 3.1, osType = Windows** на **Version = \<latest HDI default version\> ( https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning) , osType = Linux**).

После 31 июля 2018 г.:

- Вы больше не сможете создать любую версию кластеров HDInsight под управлением Windows с помощью связанной службы HDInsight по запросу в фабрике данных версии 1. 

### <a name="recommended-actions"></a>Рекомендованные действия

- Чтобы убедиться, что можно использовать последние компоненты и исправления экосистемы Hadoop, обновите [свойства **osType** и **Version**](#azure-hdinsight-on-demand-linked-service) затронутых определений связанной службы HDInsight по запросу фабрики данных версии 1 до более новых версий HDInsight под управлением Linux (HDInsight 3.6). 
- До 15 декабря 2017 г. можно было протестировать действия потоковой передачи Hive, Pig, MapReduce и Hadoop фабрики данных версии 1, ссылающиеся на затронутую связанную службу, чтобы убедиться, что они совместимы с новыми значениями по умолчанию **osType** и **Version** (**Version=3.6**, **osType=Linux**) или явной версией HDInsight и типом операционной системы, до которой выполняется обновление. 
  Дополнительные сведения о совместимости см. в статье [Миграция из кластера HDInsight под управлением Windows на кластер под управлением Linux](../../hdinsight/index.yml) и в разделе [Заметки о выпуске Hortonworks, связанные с версиями HDInsight](../../hdinsight/hdinsight-component-versioning.md). 
- Чтобы продолжать использовать связанные службы HDInsight по запросу фабрики данных версии 1 для создания кластеров HDInsight под управлением Windows, нужно было задать для свойства **osType** значение **Windows** до 15 декабря 2017 г. Мы рекомендуем начать использовать кластеры HDInsight под управлением Linux до 31 июля 2018 г. 
- Если вы используете связанную службу HDInsight по запросу для выполнения настраиваемого действия DotNet фабрики данных версии 1, обновите определение JSON настраиваемого действия DotNet, чтобы использовать связанную пакетную службу Azure. Дополнительные сведения см. в статье [Использование настраиваемых действий в конвейере фабрики данных Azure](./data-factory-use-custom-activities.md). 

> [!Note]
> Если вы используете связанное устройство HDInsight с собственным кластером в службе "Фабрика данных Azure" версии 1 или собственную связанную службу HDInsight, выполняемую по запросу, в службе "Фабрика данных Azure", дополнительные действия не требуются. В этих сценариях уже применена последняя версия, поддерживающая политику кластеров HDInsight. 
>
> 


## <a name="on-demand-compute-environment"></a>Среда вычислений по запросу
В конфигурации по запросу фабрика данных полностью управляет вычислительной средой. Вычислительная среда автоматически создается фабрикой данных перед отправкой задания для обработки данных и удаляется после его выполнения. 

Вы можете создать связанную службу для вычислительной среды по запросу. С помощью связанной службы можно настроить вычислительную среду и управлять детализированными параметрами выполнения задания, управления кластером и параметрами действий начальной загрузки.

> [!NOTE]
> Сейчас конфигурации по запросу поддерживаются только для кластеров HDInsight.
> 

## <a name="azure-hdinsight-on-demand-linked-service"></a>Связанная служба Azure HDInsight по запросу
Для обработки данных фабрика данных автоматически создает кластер HDInsight по запросу под управлением Windows или Linux. Кластер создается в том же регионе, что и учетная запись хранения, связанная с кластером. С помощью свойства JSON **linkedServiceName** можно создать кластер.

Обратите внимание на следующие *важные* моменты, касающиеся связанной службы HDInsight по запросу.

* Кластер HDInsight по запросу не отображается в вашей подписке Azure. Служба фабрики данных управляет кластером HDInsight по запросу от вашего имени.
* Журналы заданий, которые выполняются в кластере HDInsight по запросу, копируются в учетную запись хранения, связанную с кластером HDInsight. Чтобы получить доступ к этим журналам, на портале Azure перейдите к панели **Подробности о выполнении операции**. Дополнительные сведения см. в статье [о мониторинге конвейеров и управлении ими](data-factory-monitor-manage-pipelines.md).
* Вы оплачиваете только время, когда кластер HDInsight работает и выполняет задания.

> [!IMPORTANT]
> Подготовка к работе кластера HDInsight по запросу обычно занимает от *20 минут*.
>
> 

### <a name="example"></a>Пример
Представленный ниже код JSON определяет связанную службу HDInsight по запросу под управлением Linux. При обработке среза данных фабрика данных автоматически создает кластер HDInsight *под управлением Linux*. 

```json
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.6",
            "osType": "Linux",
            "clusterSize": 1,
            "timeToLive": "00:05:00",            
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

> [!IMPORTANT]
> Кластер HDInsight создает *контейнер по умолчанию* в хранилище BLOB-объектов Azure, указанном в свойстве JSON **linkedServiceName**. По умолчанию при удалении кластера HDInsight этот контейнер не удаляется. В связанной службе HDInsight по запросу кластер HDInsight создается каждый раз, когда нужно обработать срез данных (если не используется динамический кластер **timeToLive**). После завершения обработки кластер удаляется. 
>
> По мере обработки новых срезов количество контейнеров в хранилище BLOB-объектов будет увеличиваться. Если эти контейнеры не используются для устранения неполадок с заданиями, удалите их, чтобы сократить расходы на хранение. Имена этих контейнеров указаны по шаблону `adf<your Data Factory name>-<linked service name>-<date and time>`. Для удаления контейнеров в хранилище BLOB-объектов можно использовать такие средства, как [Обозреватель службы хранилища Microsoft Azure](https://storageexplorer.com/) .
>
> 

### <a name="properties"></a>Свойства
| Свойство                     | Описание                              | Обязательно |
| ---------------------------- | ---------------------------------------- | -------- |
| type                         | Присвойте свойству type значение **HDInsightOnDemand**. | Да      |
| clusterSize                  | Общее количество рабочих узлов и узлов данных в кластере. Кластер HDInsight создается с 2 головными узлами и количеством рабочих узлов, заданным в этом свойстве. Узлы имеют размер Standard_D3 с 4 ядрами. Кластер с 4 рабочими узлами использует 24 ядра (4\*4= 16 для рабочих узлов плюс 2\*4 = 8 для головных узлов). Дополнительные сведения об уровне Standard_D3 см. в статье [Установка кластеров в HDInsight с использованием Hadoop, Spark, Kafka и других технологий](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md). | Да      |
| timeToLive                   | Допустимое время простоя кластера HDInsight по запросу. Указывает, как долго кластер HDInsight по запросу остается активным после выполнения действия, если в кластере нет других активных заданий.<br /><br />Например, если выполнение действия занимает 6 минут, а для **TimeToLive** установлено значение 5 минут, кластер остается активным в течение 5 минут после 6 минут обработки выполнения действия. Если в течение этих 6 минут выполняется другое действие, оно обрабатывается в том же кластере.<br /><br />Создание кластера HDInsight по запросу является ресурсоемкой операцией и может занять некоторое время. При необходимости используйте этот параметр для повышения производительности фабрики данных за счет повторного использования кластера HDInsight по запросу.<br /><br />Если значение **timeToLive** равно **0**, кластер удаляется сразу после выполнения действия. Однако если задать большое значение, кластер может простаивать без необходимости, что приведет к большим затратам. Очень важно установить соответствующее значение с учетом своих потребностей.<br /><br />Если значение свойства **timeToLive** задано правильно, один и тот же экземпляр кластера HDInsight по запросу могут совместно использовать несколько конвейеров. | Да      |
| version                      | Версия кластера HDInsight. Список допустимых версий HDInsight см. в разделе [Поддерживаемые версии HDInsight](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions). Если это значение не указано, используется [новейшая версия HDI по умолчанию](../../hdinsight/hdinsight-component-versioning.md). | Нет       |
| linkedServiceName            | Связанная служба хранилища Azure, которую кластер по запросу должен использовать для хранения и обработки данных. Кластер HDInsight создается в том же регионе, что и учетная запись хранения.<p>Сейчас недоступно создание кластера HDInsight по запросу, который использует в качестве хранилища Azure Data Lake Store. Чтобы сохранить данные результатов обработки HDInsight в Data Lake Store, воспользуйтесь действием копирования и скопируйте данные из хранилища BLOB-объектов в Data Lake Store. </p> | Да      |
| additionalLinkedServiceNames | Указывает дополнительные учетные записи хранения для связанной службы HDInsight. Фабрика данных регистрирует учетные записи хранения от вашего имени. Эти учетные записи хранения должны находиться в том же регионе, что и кластер HDInsight. Кластер HDInsight создается в том же регионе, что и учетная запись хранения, указанная свойством **linkedServiceName**. | Нет       |
| osType                       | Тип операционной системы. Допустимые значения: **Linux** и **Windows**. Если это значение не указано, используется **Linux**.  <br /><br />Мы настоятельно рекомендуем использовать кластеры HDInsight под управлением Linux. Дата вывода HDInsight под управлением Windows — 31 июля 2018 г. | Нет       |
| hcatalogLinkedServiceName    | Имя связанной службы SQL Azure, указывающее на базу данных HCatalog. При создании кластера HDInsight по запросу используется база данных SQL в качестве хранилища метаданных. | Нет       |

#### <a name="example-linkedservicenames-json"></a>Пример: LinkedServiceNames JSON

```json
"additionalLinkedServiceNames": [
    "otherLinkedServiceName1",
    "otherLinkedServiceName2"
  ]
```

### <a name="advanced-properties"></a>Дополнительные свойства
Для детализированной настройки кластера HDInsight по запросу можно указать следующие свойства.

| Свойство               | Описание                              | Обязательно |
| :--------------------- | :--------------------------------------- | :------- |
| coreConfiguration      | Задает параметры конфигурации ядра (core-site.xml) для создаваемого кластера HDInsight. | Нет       |
| hBaseConfiguration     | Задает основные параметры конфигурации HBase (hbase-site.xml) для кластера HDInsight. | Нет       |
| hdfsConfiguration      | Задает основные параметры конфигурации HDFS (hdfs-site.xml) для кластера HDInsight. | Нет       |
| hiveConfiguration      | Задает основные параметры конфигурации Hive (hive-site.xml) для кластера HDInsight. | Нет       |
| mapReduceConfiguration | Задает параметры конфигурации MapReduce (mapred-site.xml) для кластера HDInsight. | Нет       |
| oozieConfiguration     | Задает параметры конфигурации Oozie (oozie-site.xml) для кластера HDInsight. | Нет       |
| stormConfiguration     | Задает параметры конфигурации Storm (storm-site.xml) для кластера HDInsight. | Нет       |
| yarnConfiguration      | Задает параметры конфигурации YARN (yarn-site.xml) для кластера HDInsight. | Нет       |

#### <a name="example-on-demand-hdinsight-cluster-configuration-with-advanced-properties"></a>Пример: конфигурация кластера HDInsight по запросу с дополнительными свойствами

```json
{
  "name": " HDInsightOnDemandLinkedService",
  "properties": {
    "type": "HDInsightOnDemand",
    "typeProperties": {
      "version": "3.6",
      "osType": "Linux",
      "clusterSize": 16,
      "timeToLive": "01:30:00",
      "linkedServiceName": "adfods1",
      "coreConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "hiveConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "mapReduceConfiguration": {
        "mapreduce.reduce.java.opts": "-Xmx4000m",
        "mapreduce.map.java.opts": "-Xmx4000m",
        "mapreduce.map.memory.mb": "5000",
        "mapreduce.reduce.memory.mb": "5000",
        "mapreduce.job.reduce.slowstart.completedmaps": "0.8"
      },
      "yarnConfiguration": {
        "yarn.app.mapreduce.am.resource.mb": "5000",
        "mapreduce.map.memory.mb": "5000"
      },
      "additionalLinkedServiceNames": [
        "datafeeds",
        "adobedatafeed"
      ]
    }
  }
}
```

### <a name="node-sizes"></a>Размеры узлов
С помощью перечисленных ниже свойств можно указать размеры головных узлов, узлов данных и узлов ZooKeeper. 

| Свойство          | Описание                              | Обязательно |
| :---------------- | :--------------------------------------- | :------- |
| headNodeSize      | Задает размер головного узла. Значение по умолчанию — **Standard_D3**. Дополнительные сведения см. в разделе [Указание размеров узла](#specify-node-sizes). | Нет       |
| dataNodeSize      | Задает размер узла данных. Значение по умолчанию — **Standard_D3**. | Нет       |
| zookeeperNodeSize | Задает размер узла ZooKeeper. Значение по умолчанию — **Standard_D3**. | Нет       |

#### <a name="specify-node-sizes"></a>Указание размеров узла
Сведения о строковых значениях, которые необходимо указать для свойств, описанных в предыдущем разделе, см. в статье [Размеры виртуальных машин Linux в Azure](../../virtual-machines/sizes.md). Значения должны соответствовать командлетам и API-интерфейсам, указанным в статье [Размеры виртуальных машин Linux в Azure](../../virtual-machines/sizes.md). Узел данных большого размера (по умолчанию) имеет 7 ГБ памяти. Этого может быть недостаточно для вашего сценария. 

Если вы хотите создать головные узлы и рабочие узлы размером D4, укажите **Standard_D4** в качестве значения для свойств **headNodeSize** и **dataNodeSize**. 

```json
"headNodeSize": "Standard_D4",    
"dataNodeSize": "Standard_D4",
```

Если задать для этих свойств недопустимое значение, может отобразиться следующее сообщение:

  "Не удалось создать кластер. Исключение: не удается завершить операцию создания кластера. Операция завершилась ошибкой с кодом 400. Оставшееся состояние кластера: "Ошибка". Сообщение: "PreClusterCreationValidationFailure". 
  
Если вы видите это сообщение, убедитесь, что используются имена командлета и API из таблицы, указанной в статье [Размеры виртуальных машин Linux в Azure](../../virtual-machines/sizes.md).  

> [!NOTE]
> Сейчас фабрика данных не поддерживает кластеры HDInsight, использующие Data Lake Store в качестве основного хранилища. Используйте службу хранилища Azure в качестве основного хранилища для кластеров HDInsight. 
>
> 


## <a name="bring-your-own-compute-environment"></a>Использование собственной вычислительной среды
Вы можете зарегистрировать имеющуюся вычислительную среду в качестве связанной службы в фабрике данных. Управление вычислительной средой осуществляется пользователем. Служба фабрики данных использует вычислительную среду для выполнения действий.

Такая конфигурация поддерживается в следующих средах вычислений:

* Azure HDInsight
* Пакетная служба Azure
* Машинное обучение Azure Studio (классическая модель)
* Аналитика озера данных Azure
* База данных SQL Azure, Azure синапсе Analytics, SQL Server

## <a name="azure-hdinsight-linked-service"></a>Связанная служба Azure HDInsight
Чтобы зарегистрировать собственный кластер HDInsight в фабрике данных, вы можете создать связанную службу HDInsight.

### <a name="example"></a>Пример

```json
{
  "name": "HDInsightLinkedService",
  "properties": {
    "type": "HDInsight",
    "typeProperties": {
      "clusterUri": " https://<hdinsightclustername>.azurehdinsight.net/",
      "userName": "admin",
      "password": "<password>",
      "linkedServiceName": "MyHDInsightStoragelinkedService"
    }
  }
}
```

### <a name="properties"></a>Свойства
| Свойство          | Описание                              | Обязательно |
| ----------------- | ---------------------------------------- | -------- |
| type              | Присвойте свойству type значение **HDInsight**. | Да      |
| clusterUri        | Универсальный код ресурса (URI) кластера HDInsight.        | Да      |
| username          | Имя учетной записи пользователя, которое будет использоваться для подключения к имеющемуся кластеру HDInsight. | Да      |
| password          | Пароль для учетной записи пользователя.   | Да      |
| linkedServiceName | Имя связанной службы хранилища, которая обращается к хранилищу BLOB-объектов, используемому кластером HDInsight. <p>Сейчас для этого свойства невозможно указать связанную службу Data Lake Store. Вы можете получать доступ к данным в Data Lake Store из скриптов Hive или Pig, если кластер HDInsight имеет доступ к Data Lake Store. </p> | Да      |

## <a name="azure-batch-linked-service"></a>Связанная пакетная служба Azure
Чтобы зарегистрировать пакетный пул виртуальных машин в фабрике данных, можно создать связанную пакетную службу. Пользовательские действия Microsoft .NET можно выполнять с помощью пакетной службы или HDInsight.

Если вы не работали с пакетной службой, сделайте следующее:

* Ознакомьтесь с [основными сведениями о пакетной службе Azure](../../azure-sql/database/sql-database-paas-overview.md).
* Ознакомьтесь со сведениями о командлете [New AzureBatchAccount](/previous-versions/azure/mt125880(v=azure.100)). С его помощью можно создать учетную запись пакетной службы. Это также можно сделать на [портале Azure](../../batch/batch-account-create-portal.md). Дополнительные сведения об использовании этого командлета см. в записи блога [Using Azure PowerShell to Manage Azure Batch Account](/archive/blogs/windowshpc/using-azure-powershell-to-manage-azure-batch-account) (Использование Azure PowerShell для управления учетной записью пакетной службы Azure).
* Ознакомьтесь со сведениями о командлете [New AzureBatchPool](/previous-versions/azure/mt125936(v=azure.100)). С его помощью можно создать пул пакетной службы.

### <a name="example"></a>Пример

```json
{
  "name": "AzureBatchLinkedService",
  "properties": {
    "type": "AzureBatch",
    "typeProperties": {
      "accountName": "<Azure Batch account name>",
      "accessKey": "<Azure Batch account key>",
      "poolName": "<Azure Batch pool name>",
      "linkedServiceName": "<Specify associated storage linked service reference here>"
    }
  }
}
```

Для свойства **AccountName** добавьте **. \<region name\>** в имя учетной записи пакетной службы. Пример:

```json
"accountName": "mybatchaccount.eastus"
```

Кроме того, можно указать конечную точку **batchUri**. Пример:

```json
"accountName": "adfteam",
"batchUri": "https://eastus.batch.azure.com",
```

### <a name="properties"></a>Свойства
| Свойство          | Описание                              | Обязательно |
| ----------------- | ---------------------------------------- | -------- |
| type              | Присвойте свойству type значение **AzureBatch**. | Да      |
| accountName       | Имя учетной записи пакетной службы.         | Да      |
| accessKey         | Ключ доступа к учетной записи пакетной службы.  | Да      |
| poolName          | Имя пула виртуальных машин.    | Да      |
| linkedServiceName | Имя связанной службы хранилища, которая ассоциируется с этой связанной пакетной службой. Эта связанная служба используется для промежуточных файлов, необходимых для выполнения действий и хранения журналов выполнения действий. | Да      |

## <a name="azure-machine-learning-studio-classic-linked-service"></a>Связанная служба Машинное обучение Azure Studio (классическая модель)
Можно создать связанную службу Машинное обучение Azure Studio (классическая модель), чтобы зарегистрировать конечную точку пакетной оценки Studio (классическая) для фабрики данных.

### <a name="example"></a>Пример

```json
{
  "name": "AzureMLLinkedService",
  "properties": {
    "type": "AzureML",
    "typeProperties": {
      "mlEndpoint": "https://[batch scoring endpoint]/jobs",
      "apiKey": "<apikey>"
    }
  }
}
```

### <a name="properties"></a>Свойства
| Свойство   | Описание                              | Обязательно |
| ---------- | ---------------------------------------- | -------- |
| Тип       | Присвойте свойству type значение **AzureML**. | Да      |
| mlEndpoint | URL-адрес пакетной оценки.                   | Да      |
| apiKey     | API модели опубликованной рабочей области.     | Да      |

## <a name="azure-data-lake-analytics-linked-service"></a>Связанная служба Azure Data Lake Analytics
Вы можете создать связанную службу Data Lake Analytics, чтобы связать службу вычислений Data Lake Analytics с фабрикой данных Azure. Действие U-SQL Data Lake Analytics в конвейере ссылается на эту связанную службу. 

В следующей таблице приведено описание общих свойств, используемых в определении JSON.

| Свойство                 | Описание                              | Обязательно                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| type                 | Присвойте свойству type значение **AzureDataLakeAnalytics**. | Да                                      |
| accountName          | Имя учетной записи Data Lake Analytics.  | Да                                      |
| dataLakeAnalyticsUri | Универсальный код ресурса (URI) Data Lake Analytics.           | Нет                                       |
| subscriptionId       | Идентификатор подписки Azure.                    | Нет<br /><br />(Если не указан, используется подписка фабрики данных.) |
| имя_группы_ресурсов    | Имя группы ресурсов Azure.                | Нет<br /><br /> (Если не указано, используется группа ресурсов фабрики данных.) |

### <a name="authentication-options"></a>Параметры проверки подлинности
В связанной службе Data Lake Analytics можно выполнить проверку подлинности на основе субъекта-службы или с помощью учетных данных пользователя.

#### <a name="service-principal-authentication-recommended"></a>Проверка подлинности на основе субъекта-службы (рекомендуется)
При использовании проверки подлинности на основе субъекта-службы необходимо зарегистрировать сущность приложения в Azure Active Directory (Azure AD). Затем предоставьте Azure AD доступ к Data Lake Store. Подробные инструкции см. в статье [Аутентификация между службами в Data Lake Store с помощью Azure Active Directory](../../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md). Запишите следующие значения, которые используются для определения связанной службы:
* Идентификатор приложения
* Ключ приложения 
* Tenant ID

Используйте проверку подлинности на основе субъекта-службы, указав следующие свойства:

| Свойство                | Описание                              | Обязательно |
| :---------------------- | :--------------------------------------- | :------- |
| servicePrincipalId  | Идентификатор клиента приложения.     | Да      |
| servicePrincipalKey | Ключ приложения.           | Да      |
| tenant              | Сведения о клиенте (доменное имя или идентификатор клиента), в котором находится приложение. Чтобы получить эти сведения, наведите указатель мыши на правый верхний угол страницы портала Azure. | Да      |

**Пример. Проверка подлинности субъекта-службы**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="user-credential-authentication"></a>Использование проверки подлинности на основе учетных данных пользователя
Для Data Lake Analytics можно использовать проверку подлинности на основе учетных данных пользователя, указав приведенные ниже свойства.

| Свойство          | Описание                              | Обязательно |
| :---------------- | :--------------------------------------- | :------- |
| авторизация | В редакторе фабрики данных нажмите кнопку **Авторизовать**. Введите учетные данные. URL-адрес авторизации будет создан автоматически и присвоен этому свойству. | Да      |
| sessionID     | Идентификатор сеанса OAuth из сеанса авторизации OAuth. Каждый идентификатор сеанса является уникальным и используется только один раз. Этот параметр создается автоматически при использовании редактора фабрики данных. | Да      |

**Пример. Использование проверки подлинности на основе учетных данных пользователя**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>Срок действия маркера
Срок действия кода авторизации, созданного с помощью кнопки **Авторизовать**, через некоторое время истекает. 

Если срок действия маркера проверки подлинности истечет, может появиться следующее сообщение об ошибке. 

  "Произошла ошибка при операции с учетными данными: invalid_grant — AADSTS70002. Ошибка при проверке учетных данных. AADSTS70008: срок действия предоставленных прав доступа истек или они были отозваны. Идентификатор отслеживания: d18629e8-af88-43c5-88e3-d8419eb1fca1 Идентификатор корреляции: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Временная отметка: 2015-12-15 21:09:31Z".

В следующей таблице показано время прекращения действия для каждого типа учетной записи пользователя: 

| Тип пользователя                                | Срок действия                            |
| :--------------------------------------- | :--------------------------------------- |
| Учетные записи пользователей, которыми *не* управляет Azure AD (Hotmail, Live и т. д.) | 12 часов.                                 |
| Учетные записи пользователей, которыми *управляет* Azure AD | 14 дней после последнего запуска среза. <br /><br />90 дней, если срез, основанный на связанной службе на основе OAuth, выполняется по крайней мере раз в 14 дней. |

Чтобы избежать этой ошибки или исправить ее, вам потребуется повторно авторизоваться с помощью кнопки **Авторизовать**, когда срок действия маркера истечет. Затем повторно разверните связанную службу. Значения свойств **sessionId** и **authorization** также можно задавать программно с помощью следующего кода:

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```

Дополнительные сведения о классах фабрики данных, используемых в этом примере кода, см. в следующих статьях:
* [AzureDataLakeStoreLinkedService — класс](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice)
* [AzureDataLakeAnalyticsLinkedService — класс](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice)
* [AuthorizationSessionGetResponse — класс](/dotnet/api/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse)

Для использования класса **WindowsFormsWebAuthenticationDialog** следует добавить ссылку на Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll. 

## <a name="azure-sql-linked-service"></a>Связанная служба SQL Azure
Связанная служба SQL создается и применяется к [действию хранимой процедуры](data-factory-stored-proc-activity.md) для вызова хранимой процедуры из конвейера фабрики данных. Дополнительные сведения см. в разделе [Свойства связанной службы](data-factory-azure-sql-connector.md#linked-service-properties).

## <a name="azure-synapse-analytics-linked-service"></a>Связанная служба Azure синапсе Analytics
Вы можете создать связанную службу Azure синапсе Analytics и использовать ее с [действием хранимой процедуры](data-factory-stored-proc-activity.md) для вызова хранимой процедуры из конвейера фабрики данных. Дополнительные сведения см. в статье [соединитель Azure синапсе Analytics](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties).

## <a name="sql-server-linked-service"></a>Связанная служба SQL Server
Связанная служба SQL Server создается и применяется к [действию хранимой процедуры](data-factory-stored-proc-activity.md) для вызова хранимой процедуры из конвейера фабрики данных. Дополнительные сведения см. в разделе [Свойства связанной службы](data-factory-sqlserver-connector.md#linked-service-properties).