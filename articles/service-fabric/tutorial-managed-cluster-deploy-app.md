---
title: Развертывание приложения в управляемом кластере Service Fabric с помощью PowerShell (предварительная версия)
description: Из этого учебника вы узнаете, как подключаться к управляемому кластеру Service Fabric и разворачивать приложение с помощью PowerShell.
ms.topic: tutorial
ms.date: 09/28/2020
ms.openlocfilehash: 36a91d2852bcda5f958441b48ef4721d6ccc83c4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91410255"
---
# <a name="tutorial-deploy-an-app-to-a-service-fabric-managed-cluster-preview"></a>Руководство по развертыванию приложения в управляемом кластере Service Fabric (предварительная версия)

В этой серии учебников мы обсудим следующие темы:

> [!div class="checklist"]
> * [Развертывание управляемого кластера Service Fabric](tutorial-managed-cluster-deploy.md).
> * [Масштабирование управляемого кластера Service Fabric](tutorial-managed-cluster-scale.md).
> * [Добавление и удаление узлов в управляемом кластере Service Fabric](tutorial-managed-cluster-add-remove-node-type.md).
> * Развертывание приложения в управляемом кластере Service Fabric.

В этой части серии учебников описывается:

> [!div class="checklist"]
> * Подключение к управляемому кластеру Service Fabric.
> * Отправка приложения в кластер.
> * Создание экземпляра приложения в кластере.
> * Удаление приложения из кластера.

## <a name="prerequisites"></a>Предварительные требования

* [Управляемый кластер Service Fabric (см. статью *Развертывание управляемого кластера*](tutorial-managed-cluster-deploy.md)).

## <a name="connect-to-your-cluster"></a>Подключение к кластеру

Для подключения к кластеру потребуется отпечаток сертификата кластера. Это значение можно найти в выходных данных свойства кластера развертывания ресурса, или запросив свойства кластера в существующем ресурсе.

Следующую команду можно использовать для создания запроса к ресурсу кластера, чтобы получить отпечаток сертификата кластера.

```powershell
$serverThumbprint = (Get-AzResource -ResourceId /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.ServiceFabric/managedclusters/mysfcluster).Properties.clusterCertificateThumbprint
```

Получив отпечаток сертификата кластера, можно подключаться к кластеру.

```powershell
$connectionEndpoint = "mysfcluster.eastus2.cloudapp.azure.com:19000"
Connect-ServiceFabricCluster -ConnectionEndpoint $connectionEndpoint -KeepAliveIntervalInSec 10 `
      -X509Credential `
      -ServerCertThumbprint $serverThumbprint  `
      -FindType FindByThumbprint `
      -FindValue $clientThumbprint `
      -StoreLocation CurrentUser `
      -StoreName My

```

### <a name="upload-an-application-package"></a>Отправка пакета приложения

В этом руководстве мы будем использовать пример [приложения для голосования Service Fabric](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/tree/voting-sample-no-reverse-proxy). Дополнительные сведения о развертывании приложений Service Fabric с помощью PowerShell см. в статье [Развертывание и удаление приложений Service Fabric](service-fabric-deploy-remove-applications.md).

> [!NOTE]
> В предварительной версии управляемого кластера Service Fabric функция публикации приложения непосредственно из Visual Studio не поддерживается.

Поэтому вам сначала необходимо [упаковать приложение для развертывания](service-fabric-package-apps.md). Для целей этого учебника вам понадобится следовать инструкциям по упаковке приложения из Visual Studio. Запишите путь к упакованному приложению, поскольку он будет использоваться для указанного ниже пути.

После создания пакета приложения его можно передать в кластер. Обновите значение `$path`, чтобы указать путь к пакету приложения, и выполните следующую команду:

```powershell
$path = "C:\Users\<user>\Documents\service-fabric-dotnet-quickstart\Voting\pkg\Debug"
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -CompressPackage
Register-ServiceFabricApplicationType -ApplicationPathInImageStore Debug
```

### <a name="create-an-application"></a>Создание приложения

Чтобы создать экземпляр приложения, можно использовать любую зарегистрированную версию типа приложения. Для этого выполните командлет New-ServiceFabricApplication. Имя приложения должно начинаться со схемы fabric: и быть уникальным для каждого экземпляра приложения. Если в манифесте приложения для конкретного его типа были определены службы по умолчанию, то они также будут созданы.

```powershell
New-ServiceFabricApplication fabric:/Voting VotingType 1.0.0
```

После завершения этой операции вы увидите, что экземпляры приложения выполняются в Service Fabric Explorer.

### <a name="remove-an-application"></a>Удаление приложения

Если экземпляр приложения больше не нужен, его можно навсегда удалить по имени с помощью командлета `Remove-ServiceFabricApplication`. Выполнение этого командлета также приводит к автоматическому удалению всех служб приложения и окончательному удалению состояния службы.

```powershell
Remove-ServiceFabricApplication fabric:/Voting
```

## <a name="next-steps"></a>Дальнейшие действия

На этом шаге мы узнали, как развернуть приложение в управляемом кластере Service Fabric. Дополнительные сведения об управляемых кластерах Service Fabric см. в статье:

> [!div class="nextstepaction"]
> [Распространенные вопросы об управляемых кластерах Service Fabric](faq-managed-cluster.md)
