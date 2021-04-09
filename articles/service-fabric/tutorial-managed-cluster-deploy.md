---
title: Развертывание управляемого кластера Service Fabric (предварительная версия)
description: Из этого учебника вы узнаете, как развернуть управляемый кластер Service Fabric для тестирования.
ms.topic: tutorial
ms.date: 08/27/2020
ms.custom: references_regions
ms.openlocfilehash: fd54e6f8c5272c3a8fe0e0162ca2e57968b24642
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98791716"
---
# <a name="tutorial-deploy-a-service-fabric-managed-cluster-preview"></a>Руководство по развертыванию управляемого кластера Service Fabric (предварительная версия)

В этой серии учебников мы обсудим следующие темы:

> [!div class="checklist"]
> * Развертывание управляемого кластера Service Fabric. 
> * [Масштабирование управляемого кластера Service Fabric](tutorial-managed-cluster-scale.md).
> * [Добавление и удаление узлов в управляемом кластере Service Fabric](tutorial-managed-cluster-add-remove-node-type.md).
> * [Развертывание приложения в управляемом кластере Service Fabric](tutorial-managed-cluster-deploy-app.md).

В этой части серии учебников описывается:

> [!div class="checklist"]
> * Подключение к учетной записи Azure
> * Создание группы ресурсов
> * Развертывание управляемого кластера Service Fabric.
> * Добавление типа первичного узла в кластер.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Установите [пакет SDK для Service Fabric и модуль PowerShell](service-fabric-get-started.md).

* Установите [Azure PowerShell версии 4.7.0](/powershell/azure/release-notes-azureps#azservicefabric) (или более поздней).

## <a name="connect-to-your-azure-account"></a>Подключение к учетной записи Azure

Замените `<your-subscription>` строкой подписки для вашей учетной записи Azure и подключитесь.

```powershell
Login-AzAccount
Set-AzContext -SubscriptionId <your-subscription>

```

## <a name="create-a-new-resource-group"></a>Создание группы ресурсов

Затем создайте группу ресурсов для управляемого кластера Service Fabric, заменив `<your-rg>` и `<location>` именем и расположением нужной группы.

> [!NOTE]
> К регионам с поддержкой общедоступной предварительной версии можно отнести `centraluseuap`, `eastus2euap`, `eastasia`, `northeurope`, `westcentralus`, а также `eastus2`.

```powershell
$resourceGroup = "myResourceGroup"
$location = "EastUS2"

New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="deploy-a-service-fabric-managed-cluster"></a>Развертывание управляемого кластера Service Fabric.

### <a name="create-a-service-fabric-managed-cluster"></a>Создание управляемого кластера Service Fabric

На этом шаге вы создадите управляемый кластер Service Fabric с помощью команды PowerShell New-AzServiceFabricManagedCluster. В следующем примере в группе ресурсов myResourceGroup создается кластер с именем myCluster. Указанная группа ресурсов была создана на предыдущем шаге в регионе eastus2.

На этом этапе укажите собственные значения для таких параметров:

* **Имя кластера**. Введите уникальное имя кластера, например *mysfcluster*.
* **Пароль администратора**. Введите пароль администратора, который будет использоваться для RDP на базовых виртуальных машинах в кластере.
* **Отпечаток сертификата клиента**. Укажите отпечаток сертификата клиента, который хотите использовать для доступа к кластеру. Если у вас нет сертификата, выполните действия по [настройке и получению сертификата](../key-vault/certificates/quick-create-portal.md), чтобы создать самозаверяющий сертификат.
* **SKU кластера**. Укажите [тип управляемого кластера Service Fabric](overview-managed-cluster.md#service-fabric-managed-cluster-skus) для развертывания. Кластеры с *базовыми* SKU предназначены только для тестовых развертываний, и их нельзя использовать для добавления или удаления типов узлов.

```powershell
$clusterName = "<unique cluster name>"
$password = "Password4321!@#" | ConvertTo-SecureString -AsPlainText -Force
$clientThumbprint = "<certificate thumbprint>"
$clusterSku = "Standard"

New-AzServiceFabricManagedCluster -ResourceGroupName $resourceGroup -Location $location -ClusterName $clusterName -ClientCertThumbprint $clientThumbprint -ClientCertIsAdmin -AdminPassword $password -Sku $clusterSKU -Verbose
```

### <a name="add-a-primary-node-type-to-the-service-fabric-managed-cluster"></a>Добавление типа первичного узла в управляемый кластер Service Fabric

На этом шаге вы добавите тип первичного узла в только что созданный кластер. У каждого кластера Service Fabric должен быть по крайней мере один тип первичного узла.

На этом этапе укажите собственные значения для таких параметров:

* **Имя типа узла**. Введите уникальное имя для типа узла, добавляемого в кластер, например "NT1".

> [!NOTE]
> Если добавляемый тип узла является первым или единственным типом узла в кластере, необходимо использовать свойство "Первичный".

```powershell
$nodeType1Name = "NT1"

New-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -Name $nodeType1Name -Primary -InstanceCount 5
```

Выполнение этой команды может занять несколько минут.

## <a name="validate-the-deployment"></a>Проверка развертывания

### <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

После завершения развертывания найдите значение Service Fabric Explorer на странице обзора ресурсов управляемого кластера Service Fabric на портале. При появлении запроса о сертификате используйте сертификат, для которого в команде PowerShell был предоставлен отпечаток клиента.

## <a name="next-steps"></a>Дальнейшие действия

На этом шаге мы создали и развернули наш первый управляемый кластер Service Fabric. Дополнительные сведения о масштабировании кластера см. в статье:

> [!div class="nextstepaction"]
> [Масштабирование управляемого кластера Service Fabric](tutorial-managed-cluster-scale.md)
