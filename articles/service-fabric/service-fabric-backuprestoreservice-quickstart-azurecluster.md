---
title: Периодическое резервное копирование и восстановление в Azure Service Fabric
description: Использование функции периодического резервного копирования и восстановления Service Fabric для включения периодического резервного копирования данных приложения.
ms.topic: conceptual
ms.date: 5/24/2019
ms.openlocfilehash: 42097b50277e78b3f0e8f5e61a2bf70cc08dbc02
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103198715"
---
# <a name="periodic-backup-and-restore-in-an-azure-service-fabric-cluster"></a>Периодическое резервное копирование и восстановление в кластере Azure Service Fabric
> [!div class="op_single_selector"]
> * [Кластеры в Azure](service-fabric-backuprestoreservice-quickstart-azurecluster.md) 
> * [Автономные кластеры](service-fabric-backuprestoreservice-quickstart-standalonecluster.md)
> 

Service Fabric — это платформа распределенных систем, которая упрощает разработку надежных распределенных облачных приложений на основе микрослужб и управление ими. Она позволяет запускать микрослужбы с отслеживанием и без отслеживания состояния. Службы с отслеживанием состояния могут поддерживать изменчивое, авторитетное состояние за пределами запроса и ответа или полной транзакции. Если служба с отслеживанием состояния отключена в течение длительного времени или теряет информацию вследствие аварии, может возникнуть необходимость восстановления до недавней резервной копии состояния для дальнейшего использования службы.

Service Fabric реплицирует состояние на нескольких узлах, обеспечивая высокую доступность службы. Даже в случае сбоя одного узла в кластере служба будет оставаться доступной. Однако в некоторых случаях все же желательно обеспечить устойчивость данных службы к более масштабным сбоям.
 
Например, службе может потребоваться создать резервную копию своих данных для защиты в следующих ситуациях:
- в случае полной потери всего кластера Service Fabric.
- Полной потери большей части реплик секций служб.
- Если из-за административных ошибок произошло случайное удаление или повреждение состояния. Например, администратор с достаточными привилегиями ошибочно удалил службу.
- Если из-за ошибок в службе происходит повреждение данных. Например, это может произойти, если при обновлении кода службы начинается запись поврежденных данных в надежную коллекцию. В этом случае может потребоваться вернуть код и данные в предыдущее состояние.
- При автономной обработке данных. Автономная обработка данных для бизнес-аналитики, которая происходит отдельно от службы, создающей данные, может быть очень удобной.

Service Fabric предоставляет встроенный API для выполнения [резервного копирования и восстановления](service-fabric-reliable-services-backup-restore.md) на момент во времени. Разработчики приложений могут использовать эти API для периодического резервного копирования состояния службы. Кроме того, если администраторы службы хотят активировать резервное копирование из-за пределов службы в определенный момент время, например перед обновлением приложения, разработчики должны предоставить возможность получения резервной копии (и восстановления) в виде API этой службы. Поддержка резервных копий влечет за собой дополнительные расходы. Например, вы можете создать пять добавочных резервных копий с интервалом в полчаса, а затем — полную резервную копию. После создания полной резервной копии можно удалить предыдущие добавочные резервные копии. Такой подход требует написания дополнительного кода, что приводит к дополнительным затратам при разработке приложений.

Служба резервного копирования и восстановления в Service Fabric обеспечивает простое автоматическое резервное копирование данных, хранящихся в службах с отслеживанием состояния. Периодическое резервное копирование данных приложения является основой для защиты от потери данных и недоступности службы. Service Fabric предоставляет дополнительную службу резервного копирования и восстановления, которая позволяет настраивать периодическое резервное копирование надежных служб с отслеживанием состояния (включая службы субъекта) без необходимости написания дополнительного кода. Она также упрощает восстановление ранее сделанных резервных копий. 


Service Fabric предоставляет набор API для использования функций, связанных с возможностью периодического резервного копирования и восстановления.

- Планирование периодического резервного копирования надежных служб с отслеживанием состояния и Reliable Actors с поддержкой передачи резервных копий во внешние места хранения. Поддерживаемые места хранения
    - Хранилище Azure
    - Файловый ресурс (в локальной среде)
- Перечисление резервных копий.
- Активация нерегламентированного резервного копирования секции
- Восстановление секции с помощью предыдущей резервной копии.
- Временная остановка резервного копирования.
- Управление хранением резервных копий (предстоящих).

## <a name="prerequisites"></a>Предварительные требования
* Service Fabric кластер с структурой версии 6,4 или более поздней. Инструкции по созданию кластера Service Fabric с помощью шаблона ресурсов Azure см. в [этой статье](service-fabric-cluster-creation-via-arm.md).
* Сертификат X.509 для шифрования секретов, необходимых для подключения к хранилищу резервных копий. Сведения о получении или создании сертификата X.509 см. в статье [Создание кластера Service Fabric в Azure с помощью Azure Resource Manager](service-fabric-cluster-creation-via-arm.md).
* Надежное приложения Service Fabric с отслеживанием состояния, созданное с помощью пакета SDK Service Fabric версии 3.0 или выше. Для приложений, предназначенных для .NET Core 2,0, приложение должно быть создано с помощью Service Fabric пакета SDK версии 3,1 или более поздней.
* Создайте учетную запись хранения Azure для хранения резервных копий приложения.
* Установите модуль Microsoft. ServiceFabric. PowerShell. HTTP (Предварительная версия) для выполнения вызовов конфигурации.

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

> [!NOTE]
> Если версия PowerShellGet меньше 1.6.0, необходимо обновить, чтобы добавить поддержку флага *-AllowPrerelease* :
>
> `Install-Module -Name PowerShellGet -Force`

* Убедитесь, что кластер подключен с помощью `Connect-SFCluster` команды перед выполнением любого запроса конфигурации с помощью модуля Microsoft. ServiceFabric. PowerShell. http.

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```

## <a name="enabling-backup-and-restore-service"></a>Включение резервного копирования и восстановления службы

### <a name="using-azure-portal"></a>Использование портала Azure

Флажок включить в `Include backup restore service` разделе `+ Show optional settings` на `Cluster Configuration` вкладке.

![Включение службы восстановления резервных копий с помощью портала][1]


### <a name="using-azure-resource-manager-template"></a>Использование шаблона Azure Resource Manager
Сначала необходимо включить _службу резервного копирования и восстановления_ в кластере. Получите шаблон для кластера, который требуется развернуть. Вы можете использовать [примеры шаблонов](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype) или создать шаблон Resource Manager. Включите _службу резервного копирования и восстановления_ следующим образом:

1. Убедитесь, что `apiversion` для ресурса задано значение, а в противном **`2018-02-01`** `Microsoft.ServiceFabric/clusters` случае обновите его, как показано в следующем фрагменте кода:

    ```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. Включите _службу резервного копирования и восстановления_, добавив следующий раздел `addonFeatures` после раздела `properties`, как показано во фрагменте кода ниже: 

    ```json
        "properties": {
            ...
            "addonFeatures":  ["BackupRestoreService"],
            "fabricSettings": [ ... ]
            ...
        }

    ```
3. Настройте сертификат X.509 для шифрования учетных данных. Это необходимо, чтобы перед сохранением зашифровать учетные данные, предоставленные для подключения к хранилищу. Настройте сертификат шифрования, добавив следующий раздел `BackupRestoreService` после раздела `fabricSettings`, как показано во фрагменте кода ниже: 

    ```json
    "properties": {
        ...
        "addonFeatures": ["BackupRestoreService"],
        "fabricSettings": [{
            "name": "BackupRestoreService",
            "parameters":  [{
                "name": "SecretEncryptionCertThumbprint",
                "value": "[Thumbprint]"
            }]
        }
        ...
    }
    ```

4. После обновления шаблона кластера примените указанные выше изменения и дождитесь завершения развертывания или обновления. По завершении _служба резервного копирования и восстановления_ запустится в кластере. Uri этой службы — `fabric:/System/BackupRestoreService`. Она может быть расположена в разделе системных служб в обозревателе Service Fabric. 

## <a name="enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors"></a>Включение периодического резервного копирования для надежной службы с отслеживанием состояния и Reliable Actors
Давайте рассмотрим шаги ниже, чтобы включить периодическое резервное копирование для надежной службы с отслеживанием состояния, а также службы Reliable Actors. Предполагается следующее:
- Кластер был настроен с помощью сертификата безопасности X.509 со _службой резервного копирования и восстановления_.
- Надежная служба с отслеживанием состояния развернута в кластере. В этом кратком руководстве Uri приложения — `fabric:/SampleApp`, а Uri службы с отслеживанием состояния, относящейся к этому приложению, — `fabric:/SampleApp/MyStatefulService`. Эта служба развертывается с одним разделом с идентификатором `974bd92a-b395-4631-8a7f-53bd4ae9cf22`.
- Сертификат клиента с ролью администратора устанавливается в _My_ (_Personal_) имени хранилища сертификатов _CurrentUser_ на компьютере, откуда будут вызываться сценарии. В этом примере в качестве отпечатка этого сертификата используется `1b7ebe2174649c45474a4819dafae956712c31d3`. Дополнительные сведения о сертификатах клиента см. в статье [Контроль доступа на основе ролей для клиентов Service Fabric](service-fabric-cluster-security-roles.md).

### <a name="create-backup-policy"></a>Создание политики архивации

Первым шагом является создание политики резервного копирования. Она определяет расписание резервного копирования, целевое хранилище для данных резервного копирования, имя политики и максимальное число добавочных резервных копий, которые необходимо разрешить до запуска полного резервного копирования, а также политику хранения для хранилища резервных копий. 

В качестве хранилища резервных копий используйте созданную ранее учетную запись хранения Azure. Контейнер `backup-container` настроен для хранения резервных копий. Если контейнер с таким именем не существует, он создается во время отправки резервных копий. Укажите в параметре `ConnectionString` действительную строку подключения к учетной записи хранения Azure. Вместо `account-name` укажите имя учетной записи хранения, а вместо `account-key` — ее ключ.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell с использованием модуля Microsoft. ServiceFabric. PowerShell. http

Выполните следующие командлеты PowerShell для создания новой политики архивации. Вместо `account-name` укажите имя учетной записи хранения, а вместо `account-key` — ее ключ.

```powershell

New-SFBackupPolicy -Name 'BackupPolicy1' -AutoRestoreOnDataLoss $true -MaxIncrementalBackups 20 -FrequencyBased -Interval 00:15:00 -AzureBlobStore -ConnectionString 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container' -Basic -RetentionDuration '10.00:00:00'

```

#### <a name="rest-call-using-powershell"></a>Вызов функции RESTful с помощью PowerShell

Выполните следующий сценарий PowerShell, чтобы вызвать требуемый REST API для создания политики. Вместо `account-name` укажите имя учетной записи хранения, а вместо `account-key` — ее ключ.

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$ScheduleInfo = @{
    Interval = 'PT15M'
    ScheduleKind = 'FrequencyBased'
}

$RetentionPolicy = @{ 
    RetentionPolicyType = 'Basic'
    RetentionDuration =  'P10D'
}

$BackupPolicy = @{
    Name = 'BackupPolicy1'
    MaxIncrementalBackups = 20
    Schedule = $ScheduleInfo
    Storage = $StorageInfo
    RetentionPolicy = $RetentionPolicy
}

$body = (ConvertTo-Json $BackupPolicy)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/BackupRestore/BackupPolicies/$/Create?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

```

#### <a name="using-service-fabric-explorer"></a>Использование Service Fabric Explorer

1. В Service Fabric Explorer перейдите на вкладку резервные копии и выберите действия > создать политику архивации.

    ![Создать политику архивации][6]

2. Заполните информацию. Для кластеров Azure следует выбрать Азуреблобсторе.

    ![Создание политики резервного копирования хранилище BLOB-объектов Azure][7]

### <a name="enable-periodic-backup"></a>Включение периодического резервного копирования
После определения политики архивации для соответствия требованиям защиты данных приложения необходимо связать ее с приложением. В зависимости от требований политику резервного копирования можно связать с приложением, службой или секцией.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell с использованием модуля Microsoft. ServiceFabric. PowerShell. http

```powershell

Enable-SFApplicationBackup -ApplicationId 'SampleApp' -BackupPolicyName 'BackupPolicy1'

```
#### <a name="rest-call-using-powershell"></a>Вызов функции RESTful с помощью PowerShell

Выполните следующий сценарий PowerShell для вызова необходимого REST API, чтобы связать политику резервного копирования `BackupPolicy1`, созданную на предыдущем шаге, с приложением `SampleApp`.

```powershell
$BackupPolicyReference = @{
    BackupPolicyName = 'BackupPolicy1'
}

$body = (ConvertTo-Json $BackupPolicyReference)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Applications/SampleApp/$/EnableBackup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
``` 

#### <a name="using-service-fabric-explorer"></a>Использование Service Fabric Explorer
Убедитесь, что включен [Расширенный режим](service-fabric-visualizing-your-cluster.md#backup-and-restore) для Service Fabric Explorer 

1. Выберите приложение и перейдите к действию. Щелкните Включить/обновить резервную копию приложения.

    ![Включить резервное копирование приложения][3]

2. Наконец, выберите нужную политику и щелкните включить резервное копирование.

    ![Выбор политики][4]


### <a name="verify-that-periodic-backups-are-working"></a>Проверка работоспособности периодического резервного копирования

После включения резервного копирования на уровне приложения все секции, принадлежащие надежным службам с отслеживанием состояния и службами Reliable Actors в рамках приложения, будут периодически получать резервные копии согласно соответствующей политике архивации. 

![Событие работоспособности резервных копий секции][0]

### <a name="list-backups"></a>Перечисление резервных копий

Резервные копии, связанные со всеми секциями, принадлежащими надежным службам с отслеживанием состояния и службам Reliable Actors приложения, можно перечислить с помощью API _GetBackups_. Резервные копии могут быть перечислены для приложения, службы или секции.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell с использованием модуля Microsoft. ServiceFabric. PowerShell. http

```powershell
    
Get-SFApplicationBackupList -ApplicationId WordCount
```

#### <a name="rest-call-using-powershell"></a>Вызов функции RESTful с помощью PowerShell

Выполните следующий сценарий PowerShell, чтобы вызвать API HTTP для перечисления резервных копий, созданных для всех секций внутри приложения `SampleApp`.

```powershell
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Applications/SampleApp/$/GetBackups?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

$BackupPoints = (ConvertFrom-Json $response.Content)
$BackupPoints.Items
```

Пример выходных данных описанного выше процесса.

```
BackupId                : b9577400-1131-4f88-b309-2bb1e943322c
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 20.55.16.zip
BackupType              : Full
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3334
CreationTimeUtc         : 2018-04-06T20:55:16Z
FailureError            : 

BackupId                : b0035075-b327-41a5-a58f-3ea94b68faa4
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3552
CreationTimeUtc         : 2018-04-06T21:10:27Z
FailureError            : 

BackupId                : 69436834-c810-4163-9386-a7a800f78359
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.25.36.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3764
CreationTimeUtc         : 2018-04-06T21:25:36Z
FailureError            : 
```

#### <a name="using-service-fabric-explorer"></a>Использование Service Fabric Explorer

Чтобы просмотреть резервные копии в Service Fabric Explorer, перейдите к разделу и выберите вкладку резервные копии.

![Перечислить резервные копии][5]

## <a name="limitation-caveats"></a>Ограничения и предупреждения
- Командлеты PowerShell Service Fabric находятся в режиме предварительного просмотра.
- Отсутствие поддержки кластеров Service Fabric в Linux.

## <a name="next-steps"></a>Дальнейшие действия
- [Основные сведения о настройке периодического резервного копирования](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [Backup restore REST API reference](/rest/api/servicefabric/sfclient-index-backuprestore) (Справочник по REST API службы резервного копирования и восстановления)

[0]: ./media/service-fabric-backuprestoreservice/partition-backedup-health-event-azure.png
[1]: ./media/service-fabric-backuprestoreservice/enable-backup-restore-service-with-portal.png
[3]: ./media/service-fabric-backuprestoreservice/enable-app-backup.png
[4]: ./media/service-fabric-backuprestoreservice/enable-application-backup.png
[5]: ./media/service-fabric-backuprestoreservice/backup-enumeration.png
[6]: ./media/service-fabric-backuprestoreservice/create-bp.png
[7]: ./media/service-fabric-backuprestoreservice/creation-bp.png
