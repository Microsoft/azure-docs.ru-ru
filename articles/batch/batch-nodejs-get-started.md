---
title: Использование клиентской библиотеки пакетной службы Azure для Node.js
description: Изучите основные принципы работы пакетной службы Azure и создайте простое решение с использованием Node.js.
ms.topic: how-to
ms.date: 10/08/2020
ms.openlocfilehash: a2d86582c94804a20a3a9e952bc64d60414e25a5
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104656028"
---
# <a name="get-started-with-batch-sdk-for-nodejs"></a>Приступая к работе с пакетом SDK для пакетной службы для Node.js

Изучите основы создания клиента пакетной службы в Node.js с помощью [пакета SDK для пакетной службы Azure для Node.js](/javascript/api/overview/azure/batch). Мы определим ключевые аспекты приложения пакетной службы, а затем настроим его с помощью клиента Node.js.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что вы уже работали с Node.js и знаете, как работать в Linux. Также предполагается, что у вас есть настроенная учетная запись Azure с правами доступа для создания пакетной службы и службы хранилища.

Перед изучением шагов, описанных в этой статье, советуем ознакомиться со статьей [Выполнение реальных параллельных рабочих нагрузок с использованием пакетной службы](batch-technical-overview.md).

## <a name="understand-the-scenario"></a>Ознакомление со сценарием

У нас есть простой скрипт, написанный на языке Python, который загружает все CSV-файлы из контейнера хранилища BLOB-объектов Azure и преобразует их в формат JSON. Для параллельной обработки нескольких контейнеров учетных записей хранения можно развернуть скрипт как задание пакетной службы Azure.

## <a name="azure-batch-architecture"></a>Архитектура пакетной службы Azure

На следующей схеме показано, как можно масштабировать скрипт Python с помощью пакетной службы Azure и клиента Node.js.

![Схема, демонстрирующая архитектуру сценария.](./media/batch-nodejs-get-started/BatchScenario.png)

Клиент Node.js развертывает задание пакетной службы с задачей подготовки (описанной далее) и набор задач в зависимости от числа контейнеров в учетной записи хранения. Скрипты можно скачать из репозитория GitHub.

- [Клиент Node.js](https://github.com/Azure-Samples/azure-batch-samples/blob/master/JavaScript/Node.js/sample.js)
- [Скрипты оболочки для задач подготовки](https://github.com/Azure-Samples/azure-batch-samples/blob/master/JavaScript/Node.js/startup_prereq.sh)
- [Преобразователь Python CSV в JSON](https://github.com/Azure-Samples/azure-batch-samples/blob/master/JavaScript/Node.js/processcsv.py)

> [!TIP]
> Клиент Node.js по указанной ссылке не содержит код, который необходимо развертывать в качестве приложения-функции Azure. Перейдите по следующим ссылкам, чтобы ознакомиться с инструкциями по его созданию.
> - [Создание приложения-функции](../azure-functions/functions-get-started.md)
> - [Триггер таймера](../azure-functions/functions-bindings-timer.md)

## <a name="build-the-application"></a>создание приложения;

Рассмотрим пошаговый процесс создания клиента Node.js.

### <a name="step-1-install-azure-batch-sdk"></a>Шаг 1. Установка пакета SDK для пакетной службы Azure

Пакет SDK для пакетной службы Azure для Node.js можно установить с помощью команды npm install.

`npm install azure-batch`

Эта команда устанавливает последнюю версию пакета SDK узла azure-batch.

>[!Tip]
> Для запуска команд npm install в приложении-функции Azure перейдите к консоли Kudu на вкладке параметров функций Azure. В этом случае для установки пакета SDK для пакетной службы Azure для Node.js.

### <a name="step-2-create-an-azure-batch-account"></a>Шаг 2. Создание учетной записи пакетной службы Azure

Ее можно создать на портале [Azure](batch-account-create-portal.md) или с помощью командной строки ([PowerShell](batch-powershell-cmdlets-get-started.md) /[Azure CLI](/cli/azure)).

Ниже перечислены команды для создания учетной записи пакетной службы Azure с помощью Azure CLI.

Создайте группу ресурсов. (Пропустите этот шаг, если у вас уже есть группа, в которой вы хотите создать учетную запись пакетной службы.)

`az group create -n "<resource-group-name>" -l "<location>"`

Создайте учетную запись пакетной службы Azure.

`az batch account create -l "<location>"  -g "<resource-group-name>" -n "<batch-account-name>"`

Каждая учетная запись пакетной службы имеет соответствующие ключи доступа. Эти ключи необходимы для создания дополнительных ресурсов в учетной записи пакетной службы Azure. В рабочей среде рекомендуется использовать Azure Key Vault для хранения этих ключей. Теперь необходимо создать субъект-службу для приложения. С помощью этого субъекта-службы приложение может создать токен OAuth для доступа к ключам из хранилища ключей.

`az batch account keys list -g "<resource-group-name>" -n "<batch-account-name>"`

Скопируйте и сохраните ключ, который будет использоваться на последующих шагах.

### <a name="step-3-create-an-azure-batch-service-client"></a>Шаг 3. Создание клиента пакетной службы Azure

Следующий фрагмент кода сначала импортирует модуль Node.js azure-batch, а затем создает клиент пакетной службы. Сначала необходимо создать объект SharedKeyCredentials, используя ключ учетной записи пакетной службы, скопированный на предыдущем шаге.

```nodejs
// Initializing Azure Batch variables

var batch = require('azure-batch');

var accountName = '<azure-batch-account-name>';

var accountKey = '<account-key-downloaded>';

var accountUrl = '<account-url>'

// Create Batch credentials object using account name and account key

var credentials = new batch.SharedKeyCredentials(accountName,accountKey);

// Create Batch service client

var batch_client = new batch.ServiceClient(credentials,accountUrl);

```

URI пакетной службы Azure можно найти на вкладке "Обзор" на портале Azure. Он имеет следующий формат:

`https://accountname.location.batch.azure.com`

См. снимок экрана:

![URI пакетной службы Azure](./media/batch-nodejs-get-started/azurebatchuri.png)

### <a name="step-4-create-an-azure-batch-pool"></a>Шаг 4. Создание пула пакетной службы Azure

Пул пакетной службы Azure состоит из нескольких виртуальных машин (также известных как узлы пакетной службы). На этих узлах пакетная служба Azure развертывает задачи и управляет ими. Задайте следующие параметры конфигурации для пула:

- тип образа виртуальной машины;
- размер узлов виртуальной машины;
- число узлов виртуальной машины.

> [!TIP]
> Размер и количество узлов виртуальной машины во многом зависит от числа задач, которые требуется выполнить параллельно, а также от самой задачи. Мы советуем провести тестирование для определения идеального количества и размера.

Следующий фрагмент кода создает объекты параметров конфигурации.

```nodejs
// Creating Image reference configuration for Ubuntu Linux VM
var imgRef = {publisher:"Canonical",offer:"UbuntuServer",sku:"14.04.2-LTS",version:"latest"}

// Creating the VM configuration object with the SKUID
var vmconfig = {imageReference:imgRef,nodeAgentSKUId:"batch.node.ubuntu 14.04"}

// Setting the VM size to Standard F4
var vmSize = "STANDARD_F4"

//Setting number of VMs in the pool to 4
var numVMs = 4
```

> [!TIP]
> Список образов виртуальных машин Linux для пакетной службы Azure и соответствующие номера SKU см. в разделе [Список образов виртуальных машин](batch-linux-nodes.md#list-of-virtual-machine-images).

После определения конфигурации можно создать пул пакетной службы Azure. Команда пула пакетной службы создает узлы виртуальных машин Azure и подготавливает их к получению задач для выполнения. Каждый пул должен иметь уникальный идентификатор, чтобы на него можно было ссылаться на последующих шагах.

Следующий фрагмент кода создает пул пакетной службы Azure.

```nodejs
// Create a unique Azure Batch pool ID
var poolid = "pool" + customerDetails.customerid;
var poolConfig = {id:poolid, displayName:poolid,vmSize:vmSize,virtualMachineConfiguration:vmconfig,targetDedicatedComputeNodes:numVms,enableAutoScale:false };
// Creating the Pool for the specific customer
var pool = batch_client.pool.add(poolConfig,function(error,result){
    if(error!=null){console.log(error.response)};
});
```

Убедитесь, что пул находится в активном состоянии, прежде чем отправить в него задание.

```nodejs
var cloudPool = batch_client.pool.get(poolid,function(error,result,request,response){
        if(error == null)
        {

            if(result.state == "active")
            {
                console.log("Pool is active");
            }
        }
        else
        {
            if(error.statusCode==404)
            {
                console.log("Pool not found yet returned 404...");

            }
            else
            {
                console.log("Error occurred while retrieving pool data");
            }
        }
        });
```

Ниже приведен пример результирующего объекта, возвращаемого функцией pool.get.

```
{ id: 'processcsv_201721152',
  displayName: 'processcsv_201721152',
  url: 'https://<batch-account-name>.centralus.batch.azure.com/pools/processcsv_201721152',
  eTag: '<eTag>',
  lastModified: 2017-03-27T10:28:02.398Z,
  creationTime: 2017-03-27T10:28:02.398Z,
  state: 'active',
  stateTransitionTime: 2017-03-27T10:28:02.398Z,
  allocationState: 'resizing',
  allocationStateTransitionTime: 2017-03-27T10:28:02.398Z,
  vmSize: 'standard_a1',
  virtualMachineConfiguration:
   { imageReference:
      { publisher: 'Canonical',
        offer: 'UbuntuServer',
        sku: '14.04.2-LTS',
        version: 'latest' },
     nodeAgentSKUId: 'batch.node.ubuntu 14.04' },
  resizeTimeout:
   { [Number: 900000]
     _milliseconds: 900000,
     _days: 0,
     _months: 0,
     _data:
      { milliseconds: 0,
        seconds: 0,
        minutes: 15,
        hours: 0,
        days: 0,
        months: 0,
        years: 0 },
     _locale:
      Locale {
        _calendar: [Object],
        _longDateFormat: [Object],
        _invalidDate: 'Invalid date',
        ordinal: [Function: ordinal],
        _ordinalParse: /\d{1,2}(th|st|nd|rd)/,
        _relativeTime: [Object],
        _months: [Object],
        _monthsShort: [Object],
        _week: [Object],
        _weekdays: [Object],
        _weekdaysMin: [Object],
        _weekdaysShort: [Object],
        _meridiemParse: /[ap]\.?m?\.?/i,
        _abbr: 'en',
        _config: [Object],
        _ordinalParseLenient: /\d{1,2}(th|st|nd|rd)|\d{1,2}/ } },
  currentDedicated: 0,
  targetDedicated: 4,
  enableAutoScale: false,
  enableInterNodeCommunication: false,
  taskSlotsPerNode: 1,
  taskSchedulingPolicy: { nodeFillType: 'Spread' } }
```

### <a name="step-4-submit-an-azure-batch-job"></a>Шаг 4. Отправка задания пакетной службы Azure

Задание пакетной службы Azure представляет собой логическую группу схожих задач. В нашем случае это преобразование CSV в JSON. В нем каждая задача может обрабатывать CSV-файлы, которые присутствуют в каждом контейнере службы хранилища Azure.

Эти задачи могут выполняться параллельно и развертываться на нескольких узлах, управляемых пакетной службой Azure.

> [!TIP]
> Чтобы указать максимальное число задач, которые могут выполняться параллельно на одном узле, используйте свойство [taskSlotsPerNode](https://azure.github.io/azure-sdk-for-node/azure-batch/latest/Pool.html#add).

#### <a name="preparation-task"></a>Задача подготовки

Созданные узлы виртуальных машин являются пустыми узлами Ubuntu. Часто необходимо установить набор программ в качестве необходимых компонентов.
Обычно для узлов Linux можно использовать скрипт оболочки, который устанавливает необходимые компоненты перед запуском фактических задач. Однако это может быть любой программируемый исполняемый файл.

[Скрипт оболочки](https://github.com/shwetams/azure-batchclient-sample-nodejs/blob/master/startup_prereq.sh) в этом примере устанавливает Python-pip и пакет SDK для службы хранилища Azure для Python.

Можно отправить скрипт в учетную запись хранения Azure и создать URI SAS для доступа к нему. Этот процесс можно автоматизировать с помощью пакета SDK для службы хранилища Azure для Node.js.

> [!TIP]
> Задача подготовки задания выполняется только на узлах виртуальных машин, где необходимо выполнить определенную задачу. Если требуемые компоненты нужно установить на всех узлах независимо от выполняемых задач, при добавлении пула можно использовать свойство [startTask](https://azure.github.io/azure-sdk-for-node/azure-batch/latest/Pool.html#add). Для справки можно использовать следующее определение задачи подготовки.

Задача подготовки указывается во время отправки задания пакетной службы Azure. Ниже перечислены параметры конфигурации задачи подготовки.

- **Идентификатор**. уникальный идентификатор приложения.
- **commandLine:** командная строка для выполнения исполняемого файла задачи.
- **resourceFiles:** массив объектов, предоставляющих сведения о файлах, которые нужно скачать для выполнения этой задачи.  Ниже приведены его параметры.
  - blobSource: URI SAS файла.
  - filePath: локальный путь для скачивания и сохранения файла.
  - fileMode: применимо только для узлов Linux. fileMode имеет восьмеричный формат и значение по умолчанию 0770.
- **waitForSuccess:** если задано значение true, задача не выполняется при сбое задачи подготовки.
- **runElevated:** задайте значение true, если для выполнения задачи требуются повышенные привилегии.

В следующем фрагменте кода показан пример конфигурации скрипта задачи подготовки:

```nodejs
var job_prep_task_config = {id:"installprereq",commandLine:"sudo sh startup_prereq.sh > startup.log",resourceFiles:[{'blobSource':'Blob SAS URI','filePath':'startup_prereq.sh'}],waitForSuccess:true,runElevated:true}
```

Если обязательных компонентов, которые необходимо установить для выполнения задач, нет, задачи подготовки можно пропустить. Следующий код создает задание с отображаемым именем process csv files.

 ```nodejs
 // Setting up Batch pool configuration
 var pool_config = {poolId:poolid}
 // Setting up Job configuration along with preparation task
 var jobId = "processcsvjob"
 var job_config = {id:jobId,displayName:"process csv files",jobPreparationTask:job_prep_task_config,poolInfo:pool_config}
 // Adding Azure batch job to the pool
 var job = batch_client.job.add(job_config,function(error,result){
     if(error != null)
     {
         console.log("Error submitting job : " + error.response);
     }});
```

### <a name="step-5-submit-azure-batch-tasks-for-a-job"></a>Шаг 5. Отправка задач пакетной службы Azure для задания

Теперь, когда задание преобразования CSV-файлов создано, можно создать задачи для него. Предположим, у нас есть четыре контейнера, для каждого из которых необходимо создать задачу.

[Скрипт Python](https://github.com/shwetams/azure-batchclient-sample-nodejs/blob/master/processcsv.py) принимает два параметра:

- container_name: контейнер хранилища, откуда будут скачиваться файлы.
- pattern: необязательный параметр шаблона имени файла.

Предположим, у нас есть четыре контейнера con1, con2, con3, con4. Следующий код отправляет задачи в задание пакетной службы Azure process csv, созданное ранее.

```nodejs
// storing container names in an array
var container_list = ["con1","con2","con3","con4"]
    container_list.forEach(function(val,index){

           var container_name = val;
           var taskID = container_name + "_process";
           var task_config = {id:taskID,displayName:'process csv in ' + container_name,commandLine:'python processcsv.py --container ' + container_name,resourceFiles:[{'blobSource':'<blob SAS URI>','filePath':'processcsv.py'}]}
           var task = batch_client.task.add(poolid,task_config,function(error,result){
                if(error != null)
                {
                    console.log(error.response);
                }
                else
                {
                    console.log("Task for container : " + container_name + "submitted successfully");
                }



           });

    });
```

Код добавляет несколько задач в пул. Все задачи выполняются на узле созданного пула виртуальных машин. Если количество задач превышает число виртуальных машин в пуле или значение свойства taskSlotsPerNode, задачи ожидают, пока узел станет доступным. Это задание оркестрации пакетная служба Azure обрабатывает автоматически.

Портал содержит подробные представления задач и состояния заданий. Можно также использовать список, чтобы получить функции в пакете SDK узла Azure. Подробные сведения приведены в документации по [ссылке](https://azure.github.io/azure-sdk-for-node/azure-batch/latest/Job.html).

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте подробнее о [рабочем процессе и основных ресурсах пакетной службы](batch-service-workflow-features.md), таких как пулы, узлы, задания и задачи.
- Сведения об API пакетной службы см. в статье [Microsoft Azure SDK for Node.js - Batch Service](/javascript/api/overview/azure/batch) (Пакет Microsoft Azure SDK для Node.js. Пакетная служба).