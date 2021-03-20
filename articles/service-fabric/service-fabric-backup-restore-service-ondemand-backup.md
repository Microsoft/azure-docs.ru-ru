---
title: Резервное копирование по запросу в Azure Service Fabric
description: Использование функции резервного копирования и восстановления в Service Fabric для резервного копирования данных приложения по мере необходимости.
author: aagup
ms.topic: conceptual
ms.date: 10/30/2018
ms.author: aagup
ms.openlocfilehash: d7986c8cd8d0714215c7b4dc57170be346e627ed
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98928043"
---
# <a name="on-demand-backup-in-azure-service-fabric"></a>Резервное копирование по запросу в Azure Service Fabric

Можно создать резервную копию данных Reliable Services с отслеживанием состояния и данных Reliable Actors на случай аварии или потери данных.

Azure Service Fabric предоставляет функции для [периодического резервного копирования данных](service-fabric-backuprestoreservice-quickstart-azurecluster.md) и резервного копирования данных по мере необходимости. Резервное копирование по запросу полезно, так как оно защищает от / _повреждения данных_ потери данных из-за запланированных изменений в базовой службе или ее среде.

Функции резервного копирования по запросу позволяют записать сведения о состоянии служб перед любой выполняющейся вручную операцией, которая относится к этой службе или к ее среде. Например, когда вы вносите изменения в двоичные файлы службы при ее обновлении или переходе на более раннюю версию. В этом случае резервное копирование по запросу защищает данные от повреждения из-за ошибок в коде приложения.
## <a name="prerequisites"></a>Предварительные условия

- Установите модуль Microsoft. ServiceFabric. PowerShell. HTTP (Предварительная версия) для выполнения вызовов конфигурации.

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

> [!NOTE]
> Если версия PowerShellGet меньше 1.6.0, необходимо обновить, чтобы добавить поддержку флага *-AllowPrerelease* :
>
> `Install-Module -Name PowerShellGet -Force`

- Убедитесь, что кластер подключен с помощью `Connect-SFCluster` команды перед выполнением любого запроса конфигурации с помощью модуля Microsoft. ServiceFabric. PowerShell. http.

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```


## <a name="triggering-on-demand-backup"></a>Запуск резервного копирования по запросу

Для резервного копирования по запросу требуются сведения о хранилище, чтобы отправлять файлы резервных копий. Расположение хранилища для резервного копирования данных можно указать в политике периодического резервного копирования или в запросе на резервное копирование по запросу.

### <a name="on-demand-backup-to-storage-specified-by-a-periodic-backup-policy"></a>Резервное копирование по запросу в хранилище, указанное в политике периодического резервного копирования

Можно настроить политику периодического резервного копирования таким образом, чтобы использовалась секция службы Reliable Services с отслеживанием состояния или Reliable Actors для дополнительного резервного копирования по запросу в хранилище.

Описанный ниже случай — это продолжение сценария в разделе о [включении периодического резервного копирования для службы Reliable Services с отслеживанием состояния и Reliable Actors](service-fabric-backuprestoreservice-quickstart-azurecluster.md#enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors). В этом случае вы настраиваете политику резервного копирования для использования секции, и резервное копирование выполняется в Службе хранилища Azure с установленной частотой.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell с использованием модуля Microsoft. ServiceFabric. PowerShell. http

```powershell

Backup-SFPartition -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22' 

```

#### <a name="rest-call-using-powershell"></a>Вызов функции RESTful с помощью PowerShell

Используйте API [BackupPartition](/rest/api/servicefabric/sfclient-api-backuppartition), чтобы инициировать резервное копирование по запросу для идентификатора секции `974bd92a-b395-4631-8a7f-53bd4ae9cf22`.

```powershell
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Backup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

Используйте API [GetBackupProgress](/rest/api/servicefabric/sfclient-api-getpartitionbackupprogress), чтобы включить отслеживание для [хода резервного копирования по запросу](service-fabric-backup-restore-service-ondemand-backup.md#tracking-on-demand-backup-progress).

### <a name="on-demand-backup-to-specified-storage"></a>Резервное копирование по запросу в указанное хранилище

Вы можете создать запрос на резервное копирование для секции службы Reliable Services с отслеживанием состояния или Reliable Actors. Предоставьте сведения о хранилище как часть запроса на резервное копирование по запросу.


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell с использованием модуля Microsoft. ServiceFabric. PowerShell. http

```powershell

Backup-SFPartition -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22' -AzureBlobStore -ConnectionString  'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container'

```

#### <a name="rest-call-using-powershell"></a>Вызов функции RESTful с помощью PowerShell

Используйте API [BackupPartition](/rest/api/servicefabric/sfclient-api-backuppartition), чтобы инициировать резервное копирование по запросу для идентификатора секции `974bd92a-b395-4631-8a7f-53bd4ae9cf22`. Включите следующие сведения о Службе хранилища Azure:

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$OnDemandBackupRequest = @{
    BackupStorage = $StorageInfo
}

$body = (ConvertTo-Json $OnDemandBackupRequest)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Backup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

Можно настроить отслеживание для [хода резервного копирования по запросу](service-fabric-backup-restore-service-ondemand-backup.md#tracking-on-demand-backup-progress) с помощью API [GetBackupProgress](/rest/api/servicefabric/sfclient-api-getpartitionbackupprogress).

### <a name="using-service-fabric-explorer"></a>Использование Service Fabric Explorer
Убедитесь, что в параметрах Service Fabric Explorer включен расширенный режим.
1. Выберите нужные секции и щелкните действия. 
2. Выберите триггер резервное копирование секций и введите сведения для Azure:

    ![Триггер резервного копирования секций][0]

    или общая папка:

    ![Активировать общую папку резервных копий секций][1]

## <a name="tracking-on-demand-backup-progress"></a>Отслеживание хода выполнения резервного копирования по запросу

Секция Reliable Services с отслеживанием состояния или Reliable Actors может единовременно принять только один запрос на резервное копирование по запросу. Следующий запрос будет приниматься только после выполнения текущего.

Можно одновременно выполнять несколько запросов на резервное копирование по запросу в разных секциях.


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell с использованием модуля Microsoft. ServiceFabric. PowerShell. http

```powershell

Get-SFPartitionBackupProgress -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22'

```
#### <a name="rest-call-using-powershell"></a>Вызов функции RESTful с помощью PowerShell

```powershell
$url = "https://mysfcluster-backup.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/GetBackupProgress?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3' 
$backupResponse = (ConvertFrom-Json $response.Content) 
$backupResponse
```

Возможные состояния запросов на резервное копирование по запросу:

- **Принято**: резервная копия для секции запущена и выполняется.
  ```
  BackupState             : Accepted
  TimeStampUtc            : 0001-01-01T00:00:00Z
  BackupId                : 00000000-0000-0000-0000-000000000000
  BackupLocation          :
  EpochOfLastBackupRecord :
  LsnOfLastBackupRecord   : 0
  FailureError            :
  ```
- **Успех**, **сбой** или **время ожидания**. запрошенное резервное копирование по запросу можно выполнить в любом из следующих состояний:
  - **Успешно**. состояние _успешного_ резервного копирования указывает на успешное резервное копирование состояния секции. В ответе в этом случае возвращаются параметры _BackupEpoch_ и _BackupLSN_ для секции, а также время в формате UTC.
    ```
    BackupState             : Success
    TimeStampUtc            : 2018-11-21T20:00:01Z
    BackupId                : 5d64b697-6acd-45a4-adbd-3d75e0078081
    BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-11-21 20.00.01.zip
    EpochOfLastBackupRecord : @{DataLossNumber=131873018908156893; ConfigurationNumber=8589934592}
    LsnOfLastBackupRecord   : 36
    FailureError            :
    ```
  - **Сбой**. Состояние резервного копирования с _ошибками_ указывает на сбой во время резервного копирования состояния секции. Причины сбоя указаны в ответе на запрос.
    ```
    BackupState             : Failure
    TimeStampUtc            : 0001-01-01T00:00:00Z
    BackupId                : 00000000-0000-0000-0000-000000000000
    BackupLocation          :
    EpochOfLastBackupRecord :
    LsnOfLastBackupRecord   : 0
    FailureError            : @{Code=FABRIC_E_BACKUPCOPIER_UNEXPECTED_ERROR; Message=An error occurred during this operation.  Please check the trace logs for more details.}
    ```
  - **Время ожидания**: время _ожидания_ резервного копирования указывает, что резервное копирование состояния секции не удалось создать в течение заданного времени. Время ожидания по умолчанию составляет 10 минут. В этом случае инициируйте новый запрос на резервное копирование с увеличенным значением [BackupTimeout](/rest/api/servicefabric/sfclient-api-backuppartition#backuptimeout).
    ```
    BackupState             : Timeout
    TimeStampUtc            : 0001-01-01T00:00:00Z
    BackupId                : 00000000-0000-0000-0000-000000000000
    BackupLocation          :
    EpochOfLastBackupRecord :
    LsnOfLastBackupRecord   : 0
    FailureError            : @{Code=FABRIC_E_TIMEOUT; Message=The request of backup has timed out.}
    ```

## <a name="next-steps"></a>Дальнейшие действия

- [Основные сведения о настройке периодического резервного копирования](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [Справочник по REST API BackupRestore](/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/trigger-partition-backup.png
[1]: ./media/service-fabric-backuprestoreservice/trigger-backup-fileshare.png
