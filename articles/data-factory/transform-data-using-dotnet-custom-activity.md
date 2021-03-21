---
title: Использование настраиваемых действий в конвейере
description: Узнайте, как создавать настраиваемые действия с помощью .NET, а затем использовать действия в конвейере фабрики данных Azure.
ms.service: data-factory
author: nabhishek
ms.author: abnarain
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/26/2018
ms.openlocfilehash: 64588d5968df635c3bb017bd1ff1d10951968f32
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101724954"
---
# <a name="use-custom-activities-in-an-azure-data-factory-pipeline"></a>Использование настраиваемых действий в конвейере фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-use-custom-activities.md)
> * [Текущая версия](transform-data-using-dotnet-custom-activity.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Существует два типа действий, которые можно использовать в конвейере фабрики данных Azure.

- [Действия перемещения данных](copy-activity-overview.md) для перемещения данных между [поддерживаемыми хранилищами данных источника и приемника](copy-activity-overview.md#supported-data-stores-and-formats).
- [Действия преобразования данных](transform-data.md) для преобразования данных с помощью служб вычислений, таких как Azure HDInsight, пакетная служба azure и машинное обучение Azure.

Чтобы переместить данные в хранилище данных, которое не поддерживает фабрика данных, и обратно, или чтобы преобразовать или обработать данные способом, который не поддерживается фабрикой данных Azure, создайте **пользовательское действие** с собственной логикой перемещения или преобразования данных и используйте это действие в конвейере. Настраиваемые действия запускают настраиваемую логику кода в пуле **пакетной службы Azure** виртуальных машин.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Если вы еще не знакомы с пакетной службой Azure, см. следующие статьи.

* [Основные сведения о пакетной службе Azure](../batch/batch-technical-overview.md) — общие сведения о пакетной службе Azure.
* Статья о командлете [New-AzBatchAccount](/powershell/module/az.batch/New-azBatchAccount) со сведениями о создании учетной записи пакетной службы Azure или статья о [портале Azure](../batch/batch-account-create-portal.md) со сведениями о создании учетной записи пакетной службы Azure с помощью портала Azure. Подробные инструкции по использованию этого командлета см. в статье [Using PowerShell to manage Azure Batch Account](/archive/blogs/windowshpc/using-azure-powershell-to-manage-azure-batch-account) (Использование PowerShell для управления учетной записью пакетной службы Azure).
* [New-AzBatchPool](/powershell/module/az.batch/New-AzBatchPool) со сведениями о создании пула пакетной службы Azure.

> [!IMPORTANT]
> При создании нового пула пакетной службы Azure необходимо использовать "VirtualMachineConfiguration", а не "CloudServiceConfiguration". Дополнительные сведения см. в [руководстве по миграции пула пакетной службы Azure](../batch/batch-pool-cloud-service-to-virtual-machine-configuration.md). 

## <a name="azure-batch-linked-service"></a>Связанная пакетная служба Azure

Ниже приведен фрагмент кода JSON, который определяет пример связанной пакетной службы Azure. Дополнительные сведения см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md).

```json
{
    "name": "AzureBatchLinkedService",
    "properties": {
        "type": "AzureBatch",
        "typeProperties": {
            "accountName": "batchaccount",
            "accessKey": {
                "type": "SecureString",
                "value": "access key"
            },
            "batchUri": "https://batchaccount.region.batch.azure.com",
            "poolName": "poolname",
            "linkedServiceName": {
                "referenceName": "StorageLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
}
```

 Дополнительные сведения о связанной пакетной службе Azure см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md).

## <a name="custom-activity"></a>Настраиваемое действие

В следующем фрагменте кода JSON определяется конвейер с простым настраиваемым действием. Определение действия содержит ссылку на связанную пакетную службу Azure.

```json
{
  "name": "MyCustomActivityPipeline",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "helloworld.exe",
        "folderPath": "customactv2/helloworld",
        "resourceLinkedService": {
          "referenceName": "StorageLinkedService",
          "type": "LinkedServiceReference"
        }
      }
    }]
  }
}
```

В этом примере helloworld.exe является пользовательским приложением, которое хранится в папке customactv2/helloworld учетной записи хранения Azure, используемой в resourceLinkedService. Настраиваемое действие отправляет пользовательское приложение для выполнения в пакетной службе Azure. Вы можете заменить команду на любое предпочитаемое приложение, которое можно выполнить в целевой операционной системе узлов пула пакетной службы Azure.

В следующей таблице описаны имена и описания свойств, относящихся к этому действию.

| Свойство              | Описание                              | Обязательно |
| :-------------------- | :--------------------------------------- | :------- |
| name                  | Имя действия в конвейере.     | Да      |
| description           | Описание действия.  | Нет       |
| type                  | Для пользовательского действия используется тип действия **Custom**. | Да      |
| linkedServiceName     | Связанная служба пакетной службы Azure. Дополнительные сведения об этой связанной службе см. в статье [Вычислительные среды, поддерживаемые фабрикой данных Azure](compute-linked-services.md).  | Да      |
| .               | Команда для выполнения пользовательского приложения. Если приложение уже находится в узле пула пакетной службы Azure, resourceLinkedService и folderPath могут быть пропущены. Например, можно указать команду `cmd /c dir`, которая изначально поддерживается узлом пула пакетной службы Windows. | Да      |
| resourceLinkedService | Связанная служба хранилища Azure учетной записи хранения, в которой хранится пользовательское приложение. | Нет&#42;       |
| folderPath            | Путь к папке пользовательского приложения и всех его зависимостей.<br/><br/>При наличие зависимостей, хранящихся в подпапках, то есть в иерархической структуре папок в *folderPath*. Структура папок в настоящее время преобразована в плоскую структуру, когда файлы скопированы в пакетную службу Azure. То есть все файлы копируются в одну папку без подпапок. Чтобы обойти эту реакцию на событие, рассмотрите возможность сжатия файлов, копирования сжатого файла и затем разархивирования его с помощью специального кода в нужном месте. | Нет&#42;       |
| referenceObjects      | Массив имеющихся связанных служб и наборов данных. Указанные связанные службы и наборы данных передаются в пользовательское приложение в формате JSON. Пользовательский код может использовать ресурсы фабрики данных. | Нет       |
| extendedProperties    | Определенные пользователем свойства, которые могут быть переданы в пользовательское приложение в формате JSON. Пользовательский код может использовать дополнительные свойства. | Нет       |
| ретентионтимеиндайс | Время хранения файлов, отправленных для настраиваемого действия. Значение по умолчанию — 30 дней. | Нет |

&#42; Свойства `resourceLinkedService` и `folderPath` (оба) должны быть указаны либо пропущены.

> [!NOTE]
> Если связанные службы передаются как referenceObjects в пользовательском действии, рекомендуется передать связанную службу с поддержкой Azure Key Vault (так как она не содержит защищенных строк) и получить учетные данные с помощью имени секрета непосредственно из Key Vault кода. [Здесь](https://github.com/nabhishek/customactivity_sample/tree/linkedservice) можно найти пример, который ссылается на связанную службу AKV с поддержкой, извлекает учетные данные из Key Vault, а затем обращается к хранилищу в коде.

## <a name="custom-activity-permissions"></a>Разрешения настраиваемых действий

Настраиваемые действия предоставляют автоматической учетной записи пользователя пакетной службы Azure возможность *доступа от имени стандартного пользователя с областью задачи* (спецификация автоматических пользователей по умолчанию). Уровень разрешений автоматической учетной записи пользователя изменить невозможно. Дополнительные сведения см. в разделе [Автоматические учетные записи пользователей](../batch/batch-user-accounts.md#auto-user-accounts) статьи "Выполнение задач с учетными записями пользователей в пакетной службе".

## <a name="executing-commands"></a>Выполнение команд

Вы можете непосредственно выполнить команду, используя настраиваемое действие. В приведенном ниже примере мы выполняем команду "echo hello world" на целевых узлах пула пакетной службы Azure, которая выводит выходные данные в stdout.

```json
{
  "name": "MyCustomActivity",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "cmd /c echo hello world"
      }
    }]
  }
}
```

## <a name="passing-objects-and-properties"></a>Передача объектов и свойств

В этом примере показано, как использовать свойства referenceObjects и extendedProperties для передачи объектов фабрики данных и определенных пользователем свойств в пользовательское приложение.

```json
{
  "name": "MyCustomActivityPipeline",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "SampleApp.exe",
        "folderPath": "customactv2/SampleApp",
        "resourceLinkedService": {
          "referenceName": "StorageLinkedService",
          "type": "LinkedServiceReference"
        },
        "referenceObjects": {
          "linkedServices": [{
            "referenceName": "AzureBatchLinkedService",
            "type": "LinkedServiceReference"
          }]
        },
        "extendedProperties": {          
          "connectionString": {
            "type": "SecureString",
            "value": "aSampleSecureString"
          },
          "PropertyBagPropertyName1": "PropertyBagValue1",
          "propertyBagPropertyName2": "PropertyBagValue2",
          "dateTime1": "2015-04-12T12:13:14Z"
        }
      }
    }]
  }
}
```

При выполнении действия referenceObjects и extendedProperties сохраняются в указанных ниже файлах. Эти файлы будут развернуты в той же папке выполнения SampleApp.exe.

- `activity.json`

  Хранит свойство extendedProperties и свойства настраиваемых действий.

- `linkedServices.json`

  Хранит массив связанных служб, определенных в свойстве referenceObjects.

- `datasets.json`

  Хранит массив наборов данных, определенных в свойстве referenceObjects.

В следующем примере кода показано, как SampleApp.exe может получить доступ к необходимой информации в JSON-файлах.

```csharp
using Newtonsoft.Json;
using System;
using System.IO;

namespace SampleApp
{
    class Program
    {
        static void Main(string[] args)
        {
            //From Extend Properties
            dynamic activity = JsonConvert.DeserializeObject(File.ReadAllText("activity.json"));
            Console.WriteLine(activity.typeProperties.extendedProperties.connectionString.value);

            // From LinkedServices
            dynamic linkedServices = JsonConvert.DeserializeObject(File.ReadAllText("linkedServices.json"));
            Console.WriteLine(linkedServices[0].properties.typeProperties.accountName);
        }
    }
}
```

## <a name="retrieve-execution-outputs"></a>Извлечение выходных данных выполнения

Чтобы запустить выполнение конвейера, выполните следующую команду PowerShell:

```powershell
$runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName $pipelineName
```

Во время выполнения конвейера его текущие выходные данные можно проверить с помощью следующих команд:

```powershell
while ($True) {
    $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)

    if(!$result) {
        Write-Host "Waiting for pipeline to start..." -foregroundcolor "Yellow"
    }
    elseif (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
        Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
    }
    else {
        Write-Host "Pipeline '"$pipelineName"' run finished. Result:" -foregroundcolor "Yellow"
        $result
        break
    }
    ($result | Format-List | Out-String)
    Start-Sleep -Seconds 15
}

Write-Host "Activity `Output` section:" -foregroundcolor "Yellow"
$result.Output -join "`r`n"

Write-Host "Activity `Error` section:" -foregroundcolor "Yellow"
$result.Error -join "`r`n"
```

Свойства **stdout** и **stderr** пользовательского приложения сохраняются в контейнер **adfjobs** в связанной службе хранилища Azure, определенной при создании связанной пакетной службы Azure с помощью GUID задания. Вы можете получить подробный путь из выходных данных выполнения действия, как показано в следующем фрагменте кода:

```
Pipeline ' MyCustomActivity' run finished. Result:

ResourceGroupName : resourcegroupname
DataFactoryName   : datafactoryname
ActivityName      : MyCustomActivity
PipelineRunId     : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
PipelineName      : MyCustomActivity
Input             : {command}
Output            : {exitcode, outputs, effectiveIntegrationRuntime}
LinkedServiceName :
ActivityRunStart  : 10/5/2017 3:33:06 PM
ActivityRunEnd    : 10/5/2017 3:33:28 PM
DurationInMs      : 21203
Status            : Succeeded
Error             : {errorCode, message, failureType, target}

Activity Output section:
"exitcode": 0
"outputs": [
  "https://<container>.blob.core.windows.net/adfjobs/<GUID>/output/stdout.txt",
  "https://<container>.blob.core.windows.net/adfjobs/<GUID>/output/stderr.txt"
]
"effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
Activity Error section:
"errorCode": ""
"message": ""
"failureType": ""
"target": "MyCustomActivity"
```

Если вы хотите использовать содержимое stdout.txt в последующих действиях, путь к файлу stdout.txt можно получить в значении выражения "\@activity('MyCustomActivity').output.outputs[0]".

> [!IMPORTANT]
> - Свойства activity.json, linkedServices.json и datasets.json хранятся в папке среды выполнения пакетной задачи. В этом примере activity.json, linkedServices.jsи datasets.jsв сохраняются по `https://adfv2storage.blob.core.windows.net/adfjobs/<GUID>/runtime/` пути. При необходимости их следует очищать отдельно.
> - Если связанные службы используют локальную среду выполнения интеграции, конфиденциальная информация, например ключи и пароли, шифруется локальной средой выполнения интеграции. Это гарантирует, что учетные данные останутся в пределах частных сетевых сред клиентов. Некоторые поля с конфиденциальными данными, на которые таким образом ссылается пользовательский код приложения, могут отсутствовать. При необходимости в extendedProperties используйте SecureString, а не ссылку на связанную службу.

## <a name="pass-outputs-to-another-activity"></a>Передача выходных данных в другое действие

Пользовательские значения из кода настраиваемого действия можно отправлять обратно в Фабрику данных Azure. Для этого ваше приложение должно записывать их в файл `outputs.json`. Фабрика данных копирует содержимое файла `outputs.json` и добавляет его в выходные данные действия в качестве значения свойства `customOutput` (Максимальный размер — 2 МБ.) Если вы хотите использовать содержимое `outputs.json` в нисходящих действиях, можно получить значение с помощью выражения `@activity('<MyCustomActivity>').output.customOutput` .

## <a name="retrieve-securestring-outputs"></a>Получение выходных данные SecureString

Значения свойств, обозначенные как тип *SecureString*, как показано в некоторых примерах в этой статье, замаскированы на вкладке "Мониторинг" в пользовательском интерфейсе "Data Factory".  Однако при фактическом выполнении конвейера свойство *SecureString* сериализуется как JSON в файле `activity.json` в виде обычного текста. Пример:

```json
"extendedProperties": {
  "connectionString": {
    "type": "SecureString",
    "value": "aSampleSecureString"
  }
}
```

Сериализация на самом деле небезопасна и не предназначена для обеспечения безопасности. Намерение состоит в том, чтобы указывать Фабрике данных скрыть значение на вкладке "Мониторинг".

Чтобы получить доступ из пользовательских действий к свойствам типа *SecureString*, прочитайте файл `activity.json`, который помещен в ту же папку, что и ваш .EXE, десериализуйте JSON, а затем получите доступ к свойству JSON (extendedProperties => [propertyName] => value).

## <a name="compare-v2-custom-activity-and-version-1-custom-dotnet-activity"></a><a name="compare-v2-v1"></a>Сравнение настраиваемого действия версии 2 и (настраиваемого) действия DotNet версии 1

В фабрике данных Azure версии 1 вы реализуете (настраиваемое) действие DotNet, создавая проект библиотеки классов .NET с классом, реализующим `Execute` метод `IDotNetActivity` интерфейса. Связанные службы, наборы данных и расширенные свойства, входящие в полезные данные JSON для (настраиваемого) действия DotNet, передавались в метод выполнения в виде строго типизированных объектов. Дополнительные сведения о поведении версии 1 см. в разделе [Использование настраиваемых действий в конвейере фабрики данных Azure](v1/data-factory-use-custom-activities.md). В связи с этой реализацией код действия DotNet версии 1 предназначен для платформа .NET Framework 4.5.2. Действие DotNet версии 1 также должно выполняться на узлах пула пакетной службы Azure на основе Windows.

В настраиваемом действии фабрики данных Azure версии 2 не требуется реализовывать интерфейс .NET. Теперь все команды, сценарии и пользовательский код можно компилировать и выполнять в виде исполняемого файла. Чтобы настроить эту реализацию, необходимо указать свойство `Command` вместе со свойством `folderPath`. Настраиваемое действие передает исполняемый файл и его зависимости в каталог `folderpath` и выполняет указанную команду.

Связанные службы, наборы данных (определенные в referenceObjects) и расширенные свойства, входящие в полезные данные JSON для настраиваемого действия фабрики данных версии 2, доступны для исполняемого файла в виде JSON-файлов. Вы можете обратиться к нужным свойствам с помощью сериализатора JSON, как показано в предыдущем примере кода SampleApp.exe.

Изменения, внесенные в настраиваемое действие фабрики данных версии 2, позволяют запрограммировать логику на любом языке и выполнять этот код в любых операционных системах Windows и Linux, которые поддерживает пакетная служба Azure.

В следующей таблице описаны различия между настраиваемым действием фабрики данных версии 2 и (настраиваемым) действием DotNet фабрики данных версии 1.

|Различия      | Настраиваемое действие      | (Настраиваемое) действие DotNet версии 1      |
| ---- | ---- | ---- |
|Способ определения настраиваемой логики      |Путем предоставления исполняемого файла.      |Путем реализации библиотеки DLL .NET      |
|Среда выполнения настраиваемой логики      |Windows или Linux      |Windows (платформа .NET Framework 4.5.2)      |
|Выполнение скриптов      |Поддерживает выполнение сценариев напрямую (например, "cmd /c echo hello world" на виртуальной машине Windows)      |Требуется реализация в библиотеке DLL .NET      |
|Обязательный набор данных      |Необязательно      |Является обязательным для создания цепочки действий и передачи данных      |
|Передача данных из действия в настраиваемую логику      |С помощью ReferenceObjects (LinkedServices и Datasets) и ExtendedProperties (пользовательские свойства)      |С помощью ExtendedProperties (пользовательские свойства), входных и выходных наборов данных      |
|Извлечение данных в настраиваемой логике      |Анализ файлов activity.json, linkedServices.json и datasets.json, которые хранятся в одной папке с исполняемым файлом.      |С помощью пакета SDK для .NET (.NET Frame 4.5.2)      |
|Ведение журнала      |Запись непосредственно в STDOUT      |Реализация средства ведения журнала в библиотеке DLL .NET      |

Если у вас есть код .NET, написанный для действия DotNet версии 1, необходимо изменить код, чтобы он работал с текущей версией настраиваемого действия. Для этого следует изменить этот код в соответствии со следующими общими рекомендациями.

  - Измените проект из библиотеки классов .NET на консольное приложение.
  - Запустите приложение с помощью метода `Main`. Метод `Execute` интерфейса `IDotNetActivity` больше не требуется.
  - Считайте и проанализируйте связанные службы, наборы данных и действия с помощью сериализатора JSON, а не в качестве строго типизированных объектов. Передайте значения требуемых свойств в логику основного настраиваемого кода. В качестве примера воспользуйтесь представленным выше кодом SampleApp.exe.
  - Объект средства ведения журнала больше не поддерживается. Выходные данные исполняемого файла можно вывести в консоль и сохранить в файл stdout.txt.
  - Пакет NuGet Microsoft.Azure.Management.DataFactories больше не требуется.
  - Скомпилируйте код, передайте исполняемый файл и зависимости в службу хранилища Azure и укажите путь к ним в свойстве `folderPath`.

Полный пример того, как повторно создать комплексную библиотеку DLL и конвейер, которые описаны в статье [Использование настраиваемых действий в конвейере фабрики данных Azure](./v1/data-factory-use-custom-activities.md) для фабрики данных версии 1, в формате настраиваемого действия фабрики данных версии 2: [пример настраиваемого действия фабрики данных](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ADFv2CustomActivitySample).

## <a name="auto-scaling-of-azure-batch"></a>Автомасштабирование пакетной службы Azure

Можно также создать пул пакетной службы Azure с использованием функции **автомасштабирования** . Например, можно создать пул пакетной службы Azure с нулем выделенных виртуальных машин и формулой автоматического масштабирования на основе числа ожидающих задач.

Приведенный здесь пример формулы обеспечивает следующее поведение: при создании пула он изначально содержит одну виртуальную машину. Метрика $PendingTasks определяет количество задач в состоянии выполнения и активном состоянии (в очереди). Формула находит среднее число ожидающих выполнения задач за последние 180 секунд и соответствующим образом задает значение TargetDedicated. Благодаря этому значение TargetDedicated никогда не превысит 25 виртуальных машин. Таким образом, по мере поступления новых задач пул автоматически увеличивается, а по мере их завершения виртуальные машины высвобождаются по одной, и функция автоматического масштабирования уменьшает пул. Значения startingNumberOfVMs и maxNumberofVMs можно настроить в соответствии со своими потребностями.

Формула автоматического масштабирования:

```
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
```

Дополнительные сведения см. в статье [Автоматическое масштабирование вычислительных узлов в пуле пакетной службы Azure](../batch/batch-automatic-scaling.md).

Если в пуле используется [autoScaleEvaluationInterval](/rest/api/batchservice/pool/enableautoscale)(значение по умолчанию), пакетной службе может потребоваться 15–30 минут на подготовку виртуальной машины перед выполнением настраиваемого действия. Если пул использует другое значение autoScaleEvaluationInterval, пакетная служба может затрачивать autoScaleEvaluationInterval плюс 10 минут.

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами:

* [Действие U-SQL](transform-data-using-data-lake-analytics.md)
* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие Pig](transform-data-using-hadoop-pig.md)
* [Действие MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Действие потоковой передачи Hadoop](transform-data-using-hadoop-streaming.md)
* [Действие Spark](transform-data-using-spark.md)
* [Действие выполнения пакета Машинное обучение Azure Studio (классическая модель)](transform-data-using-machine-learning.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)