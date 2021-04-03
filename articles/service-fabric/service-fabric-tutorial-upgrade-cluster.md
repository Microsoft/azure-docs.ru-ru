---
title: Обновление среды выполнения Service Fabric в Azure
description: Из этого руководства вы узнаете, как обновить среду выполнения кластера Service Fabric, размещенного в Azure, с помощью PowerShell.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: 23b3aabf8e991e512ef9a5c07d725c3084ea7f83
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "86244741"
---
# <a name="tutorial-upgrade-the-runtime-of-a-service-fabric-cluster-in-azure"></a>Руководство по Обновление среды выполнения кластера Service Fabric в Azure

Это руководство является четвертой частью цикла. В нем показано, как обновить среду выполнения Service Fabric в кластере Azure Service Fabric. Эта часть руководства предназначена для кластеров Service Fabric, работающих в Azure, и не относится к изолированным кластерам Service Fabric.

> [!WARNING]
> Для изучения этой части руководства требуется PowerShell. Обновление среды выполнения кластера еще не поддерживается инструментами Azure CLI. Кроме того, кластер можно обновить на портале. Дополнительные сведения см. в статье [Обновление кластера Azure Service Fabric](service-fabric-cluster-upgrade.md).

Если в кластере уже выполняется последняя версия среды выполнения Service Fabric, выполнять этот шаг не требуется. Однако эту статью можно использовать для установки любой поддерживаемой среды выполнения в кластере Azure Service Fabric.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Чтение версии кластера.
> * Настройка версии кластера.

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * создание безопасного [кластера Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) в Azure с помощью шаблона;
> * [Мониторинг кластера](service-fabric-tutorial-monitor-cluster.md)
> * [свертывание и развертывание кластера](service-fabric-tutorial-scale-cluster.md);
> * обновление среды выполнения кластера;
> * [Удаление кластера](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Установите [Azure PowerShell](/powershell/azure/install-az-ps) или [Azure CLI](/cli/azure/install-azure-cli).
* Создание защищенного [кластера Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) в Azure
* Настройте среду разработки Windows. [Установите Visual Studio 2019](https://www.visualstudio.com), а также рабочие нагрузки **Разработка для Azure**, **ASP.NET и разработка веб-приложений** и **Кроссплатформенная разработка .NET Core**.  Теперь настройте [среду разработки .NET](service-fabric-get-started.md).

### <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в учетную запись Azure и выберите подписку, прежде чем выполнять команды Azure.

```powershell
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>
```

## <a name="get-the-runtime-version"></a>Получение версии среды выполнения

После подключения к Azure и выбора подписки, содержащей кластер Service Fabric, можно узнать версию среды выполнения кластера.

```powershell
Get-AzServiceFabricCluster -ResourceGroupName SFCLUSTERTUTORIALGROUP -Name aztestcluster `
    | Select-Object ClusterCodeVersion
```

Можно также просто получить список всех кластеров в подписке с помощью следующего примера.

```powershell
Get-AzServiceFabricCluster | Select-Object Name, ClusterCodeVersion
```

Обратите внимание на значение **ClusterCodeVersion**. Оно будут использовано в следующем разделе.

## <a name="upgrade-the-runtime"></a>Обновление среды выполнения

Используйте значение **ClusterCodeVersion** из предыдущего раздела в командлете `Get-ServiceFabricRuntimeUpgradeVersion`, чтобы выяснить, какие целевые версии доступны для обновления. Этот командлет может быть выполнен только на компьютере, подключенном к Интернету. Например, если нужно узнать, до каких версий среды выполнения можно обновить версию `5.7.198.9494`, используйте следующую команду.

```powershell
Get-ServiceFabricRuntimeUpgradeVersion -BaseVersion "5.7.198.9494"
```

Имея список версий, можно указать кластеру Azure Service Fabric обновить среду выполнения. Например, если версия `6.0.219.9494` доступна для обновления, выполните следующую команду, чтобы обновить кластер.

```powershell
Set-AzServiceFabricUpgradeType -ResourceGroupName SFCLUSTERTUTORIALGROUP `
                                    -Name aztestcluster `
                                    -UpgradeMode Manual `
                                    -Version "6.0.219.9494"
```

> [!IMPORTANT]
> Обновление среды выполнения кластера может занять длительное время. PowerShell блокируется, пока выполняется обновление. Можно использовать другой сеанс PowerShell, чтобы проверить состояние обновления.

Состояние обновления можно отслеживать с помощью PowerShell или командной строки Azure Service Fabric (sfctl).

Сначала подключитесь к кластеру с помощью TLS/SSL-сертификата, созданного по инструкциями из первой части этого руководства. Используйте командлет `Connect-ServiceFabricCluster` или `sfctl cluster upgrade-status`.

```powershell
$endpoint = "<mycluster>.southcentralus.cloudapp.azure.com:19000"
$thumbprint = "63EB5BA4BC2A3BADC42CA6F93D6F45E5AD98A1E4"

Connect-ServiceFabricCluster -ConnectionEndpoint $endpoint `
                             -KeepAliveIntervalInSec 10 `
                             -X509Credential -ServerCertThumbprint $thumbprint `
                             -FindType FindByThumbprint -FindValue $thumbprint `
                             -StoreLocation CurrentUser -StoreName My
```

```console
sfctl cluster select --endpoint https://aztestcluster.southcentralus.cloudapp.azure.com:19080 \
--pem ./aztestcluster201709151446.pem --no-verify
```

Затем используйте командлет `Get-ServiceFabricClusterUpgrade` или `sfctl cluster upgrade-status` для отображения состояния. Ниже приведен примерный вид результата.

```powershell
Get-ServiceFabricClusterUpgrade

TargetCodeVersion                          : 6.0.219.9494
TargetConfigVersion                        : 3
StartTimestampUtc                          : 11/28/2017 3:09:48 AM
UpgradeState                               : RollingForwardPending
UpgradeDuration                            : 00:09:00
CurrentUpgradeDomainDuration               : 00:09:00
NextUpgradeDomain                          : 1
UpgradeDomainsStatus                       : { "0" = "Completed";
                                             "1" = "Pending";
                                             "2" = "Pending";
                                             "3" = "Pending";
                                             "4" = "Pending" }
UpgradeKind                                : Rolling
RollingUpgradeMode                         : Monitored
FailureAction                              : Rollback
ForceRestart                               : False
UpgradeReplicaSetCheckTimeout              : 37201.09:59:01
HealthCheckWaitDuration                    : 00:05:00
HealthCheckStableDuration                  : 00:05:00
HealthCheckRetryTimeout                    : 00:45:00
UpgradeDomainTimeout                       : 02:00:00
UpgradeTimeout                             : 12:00:00
ConsiderWarningAsError                     : False
MaxPercentUnhealthyApplications            : 0
MaxPercentUnhealthyNodes                   : 100
ApplicationTypeHealthPolicyMap             : {}
EnableDeltaHealthEvaluation                : True
MaxPercentDeltaUnhealthyNodes              : 0
MaxPercentUpgradeDomainDeltaUnhealthyNodes : 0
ApplicationHealthPolicyMap                 : {}
```

```console
sfctl cluster upgrade-status

{
  "codeVersion": "6.0.219.9494",
  "configVersion": "3",

... item cut to save space ...

  },
  "upgradeDomains": [
    {
      "name": "0",
      "state": "Completed"
    },
    {
      "name": "1",
      "state": "Pending"
    },
    {
      "name": "2",
      "state": "Pending"
    },
    {
      "name": "3",
      "state": "Pending"
    },
    {
      "name": "4",
      "state": "Pending"
    }
  ],
  "upgradeDurationInMilliseconds": "PT1H2M4.63889S",
  "upgradeState": "RollingForwardPending"
}
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Получение версии среды выполнения кластера.
> * Обновление среды выполнения кластера.
> * Отслеживание обновления.

Перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Удаление кластера](service-fabric-tutorial-delete-cluster.md)
