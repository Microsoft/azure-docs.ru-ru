---
title: Добавление узлов в автономный кластер Service Fabric под управлением Windows Server или удаление узлов из него
description: Узнайте, как добавлять узлы в кластер Azure Service Fabric или удалять их из него на физическом или виртуальном компьютере под управлением Windows Server, расположенном в локальной системе или в любом облаке.
ms.topic: conceptual
ms.date: 11/02/2017
ms.openlocfilehash: 26945b4785a0591d997139f2427b0ae6b59fa742
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98790602"
---
# <a name="add-or-remove-nodes-to-a-standalone-service-fabric-cluster-running-on-windows-server"></a>Добавление узлов в автономный кластер Service Fabric под управлением Windows Server или удаление узлов из него
После [создания автономного кластера Service Fabric на компьютерах под Windows Server](service-fabric-cluster-creation-for-windows-server.md), потребности вашего бизнеса могут измениться, и вам потребуется добавить или удалить узлы в кластере, как описано в этой статье.

> [!NOTE]
> Функции добавления и удаления узлов не поддерживаются в кластерах локальной разработки.

## <a name="add-nodes-to-your-cluster"></a>Добавление узлов в кластер

1. Подготовьте ВИРТУАЛЬную машину или компьютер, которые требуется добавить в кластер, выполнив действия, описанные в разделе [планирование и подготовка развертывания Service Fabric кластера](service-fabric-cluster-standalone-deployment-preparation.md).

2. Выясните, в какой домен сбоя и домен обновления вы собираетесь добавить эту виртуальную машину или компьютер.

   Если для защиты кластера используются сертификаты, предполагается, что сертификаты будут установлены в локальные хранилища сертификатов при подготовке узла к присоединению к кластеру. Аналоговый параметр применяется при использовании других форм безопасности.

3. Подключитесь к виртуальной машине или компьютеру, который нужно добавить в кластер, с помощью удаленного рабочего стола.

4. Скопируйте или [Скачайте автономный пакет для Service Fabric для Windows Server](https://go.microsoft.com/fwlink/?LinkId=730690) на виртуальную машину или компьютер и распакуйте пакет.

5. Запустите PowerShell с повышенными привилегиями и перейдите в расположение распакованного пакета.

6. Запустите скрипт *AddNode.ps1*, указав параметры, описывающие новый узел, который будет добавлен. В следующем примере добавляется новый узел с именем VM5 с типом NodeType0 и IP-адресом 182.17.34.52, в ДО1 и демон:/DC1/R0. `ExistingClusterConnectionEndPoint` — Это конечная точка подключения для узла, уже существующего в существующем кластере, который может быть IP-адресом *любого* узла в кластере. 

   Небезопасные (создание прототипов):

   ```
   .\AddNode.ps1 -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClientConnectionEndpoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain fd:/dc1/r0 -AcceptEULA
   ```

   Безопасность (на основе сертификата):

   ```  
   $CertThumbprint= "***********************"
    
   .\AddNode.ps1 -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClientConnectionEndpoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain fd:/dc1/r0 -X509Credential -ServerCertThumbprint $CertThumbprint  -AcceptEULA

   ```

   После завершения выполнения скрипта можно проверить, добавлен ли новый узел, выполнив командлет [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode) .

7. Чтобы обеспечить согласованность на различных узлах в кластере, нужно обновить конфигурацию. Выполните команду [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration) , чтобы получить последний файл конфигурации и добавить добавленный узел в раздел "узлы". Также рекомендуется всегда иметь доступ к последней конфигурации кластера, если потребуется повторное развертывание кластера с такой же конфигурацией.

   ```
    {
        "nodeName": "vm5",
        "iPAddress": "182.17.34.52",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD1"
    }
   ```

8. Выполните команду [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade), чтобы начать обновление.

   ```
   Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
   ```

   Ход выполнения обновления можно отслеживать с помощью Service Fabric Explorer. Кроме того, можно выполнить команду [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade).

### <a name="add-nodes-to-clusters-configured-with-windows-security-using-gmsa"></a>Добавление узлов в кластеры, настроенные с помощью Windows Security, с использованием gMSA
Для кластеров, настроенных с помощью групповой управляемой учетной записи службы (https://technet.microsoft.com/library/hh831782.aspx)), можно добавить новый узел, используя обновление конфигурации.
1. Выполните команду [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration) на любом из имеющихся узлов, чтобы получить самый последний файл конфигурации, и добавьте сведения о новом узле, который нужно добавить в раздел узлов. Убедитесь, что новый узел входит в ту же учетную запись, которой управляет группа. Этой учетной записи нужно назначить роль "Администратор" на всех компьютерах.

    ```
        {
            "nodeName": "vm5",
            "iPAddress": "182.17.34.52",
            "nodeTypeRef": "NodeType0",
            "faultDomain": "fd:/dc1/r0",
            "upgradeDomain": "UD1"
        }
    ```
2. Выполните команду [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade), чтобы начать обновление.

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
    ```
    Ход выполнения обновления можно отслеживать с помощью Service Fabric Explorer. Кроме того, с этой же целью можно выполнить команду [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade).

### <a name="add-node-types-to-your-cluster"></a>Добавление типов узлов в кластер
Чтобы добавить новый тип узла, измените конфигурацию, чтобы добавить новый тип узла в разделе NodeTypes в области Properties, и начните обновление конфигурации, используя [ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade). После завершения обновления в кластер можно добавить новые узлы этого типа.

## <a name="remove-nodes-from-your-cluster"></a>Удаление узлов из кластера
Вы можете удалить узел из кластера в ходе обновления конфигурации следующим образом:

1. Выполните команду [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration), чтобы получить последний файл конфигурации, и *удалите* узел из раздела узлов.
Добавьте параметр NodesToBeRemoved в раздел Setup в разделе FabricSettings. Значение должно быть списком имен узлов, разделенных запятыми, для узлов, которые необходимо удалить.

    ```
         "fabricSettings": [
            {
            "name": "Setup",
            "parameters": [
                {
                "name": "FabricDataRoot",
                "value": "C:\\ProgramData\\SF"
                },
                {
                "name": "FabricLogRoot",
                "value": "C:\\ProgramData\\SF\\Log"
                },
                {
                "name": "NodesToBeRemoved",
                "value": "vm0, vm1"
                }
            ]
            }
        ]
    ```
2. Выполните команду [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade), чтобы начать обновление.

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>

    ```
    Ход выполнения обновления можно отслеживать с помощью Service Fabric Explorer. Кроме того, можно выполнить команду [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade).

> [!NOTE]
> При удалении узлов может выполниться несколько обновлений. Некоторые узлы будут отмечены тегом `IsSeedNode=”true”`. Их можно определить, запросив манифест кластера с использованием `Get-ServiceFabricClusterManifest`. Удаление таких узлов может занимать больше времени, чем других, так как в таких сценариях начальные узлы придется перемещать. Кластер должен поддерживать как минимум 3 первичных узла.
> 
> 

### <a name="remove-node-types-from-your-cluster"></a>Удаление типов узлов из кластера
Прежде чем удалить тип узла, проверьте, нет ли узлов, которые ссылаются на этот тип. Перед удалением соответствующего типа узла удалите эти узлы. После удаления всех соответствующих узлов NodeType можно удалить из конфигурации кластера и начать обновление конфигурации с помощью [ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade).


### <a name="replace-primary-nodes-of-your-cluster"></a>Замена основных узлов кластера
Замену основных узлов следует выполнять последовательно, вместо того чтобы удалять, а затем добавлять их массово.


## <a name="next-steps"></a>Дальнейшие действия
* [Параметры конфигурации для автономного кластера Windows](service-fabric-cluster-manifest.md)
* [Защита автономного кластера под управлением Windows с помощью сертификатов](service-fabric-windows-cluster-x509-security.md)
* [Create a three node standalone Service Fabric cluster with Azure virtual machines running Windows Server (Создание автономного кластера Service Fabric с тремя узлами на виртуальных машинах Azure под управлением Windows)](./service-fabric-cluster-creation-via-arm.md)
