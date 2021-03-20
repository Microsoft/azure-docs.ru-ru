---
title: Обновление версии изолированного кластера
description: Обновление кода Azure Service Fabric в автономном кластере Service Fabric.
ms.topic: conceptual
ms.date: 11/09/2018
ms.openlocfilehash: 5831fcffe718e4966c3444a29d35c5bbd55e5f57
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91842943"
---
# <a name="upgrade-the-service-fabric-version-that-runs-on-your-cluster"></a>Изменение версии Service Fabric в кластере 

Для любой современной системы возможность обновления является ключом к успеху вашего продукта в долгосрочной перспективе. Кластер Azure Service Fabric — это ресурс, владельцем которого вы являетесь. В этой статье описано, как обновить версию Service Fabric в автономном или частном кластере.

> [!NOTE]
> Кластер должен всегда работать под управлением поддерживаемой версии Service Fabric. Когда корпорация Майкрософт объявляет о выпуске новой версии Service Fabric, для предыдущей версии определяется срок завершения жизненного цикла. Этот срок составляет по меньшей мере 60 дней с даты объявления. О доступности новых выпусков сообщается в [блоге группы разработчиков Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric), после чего вы можете их использовать.
>
>

Кластер можно обновить до новой версии, только если используется конфигурация узла с настройками рабочей среды, где каждый узел Service Fabric выделяется для отдельного физического или виртуального устройства. Если несколько узлов Service Fabric выполняется на отдельном физическом компьютере или виртуальной машине в кластере разработки, то нужно удалить такой кластер и создать его заново с использованием новой версии.

Обновить кластер до последней или поддерживаемой версии Service Fabric можно с помощью двух разных рабочих процессов. Один используется для кластеров с возможностью подключения для автоматического скачивания последней версии. Другой рабочий процесс используется для кластеров без возможности подключения для скачивания последней версии Service Fabric.

## <a name="enable-auto-upgrade-of-the-service-fabric-version-of-your-cluster"></a>Включение автоматического обновления версии Service Fabric для кластера
Чтобы настроить кластер для скачивания обновлений Service Fabric после того, как корпорация Майкрософт выпускает новую версию, присвойте параметру кластера `fabricClusterAutoupgradeEnabled` значение *true*. Чтобы выбрать поддерживаемую версию Service Fabric для кластера вручную, присвойте параметру кластера `fabricClusterAutoupgradeEnabled` значение *false*.

## <a name="upgrade-clusters-that-have-connectivity-to-download-the-latest-code-and-configuration"></a>Обновление кластеров с возможностью подключения для скачивания последней версии кода и конфигурации
Выполните приведенные ниже действия, чтобы обновить до поддерживаемой версии кластер с возможностью интернет-подключения к [Центру загрузки Майкрософт](https://download.microsoft.com).

Для кластеров с возможностью подключения к [Центру загрузки Майкрософт](https://download.microsoft.com) корпорация Майкрософт рекомендует периодически проверять наличие новых версий Service Fabric.

Когда выходит новая версия Service Fabric, пакет скачивается в кластер и начинается подготовка к обновлению. Наряду с информированием клиентов о доступности новой версии система выдает явное предупреждение о состоянии работоспособности кластера следующего содержания:

"Поддержка текущей версии кластера [номер версии] заканчивается [дата]".

После запуска последней версии кластера это предупреждение исчезнет.

Когда вы увидите предупреждение о работоспособности кластера, обновите этот кластер:

1. Подключитесь к кластеру с любой виртуальной машины, на которой есть доступ администратора ко всем ВМ, перечисленным в качестве узлов в файле конфигурации кластера. Компьютер, на котором выполняется этот скрипт, может и не входить в кластер.

    ```powershell
    ###### connect to the secure cluster using certs
    $ClusterName= "mysecurecluster.something.com:19000"
    $CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7FG2D630F8F3"
    Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
        -X509Credential `
        -ServerCertThumbprint $CertThumbprint  `
        -FindType FindByThumbprint `
        -FindValue $CertThumbprint `
        -StoreLocation CurrentUser `
        -StoreName My
    ```

2. Получите список версий Service Fabric, до которых можно выполнить обновление.

    ```powershell
    ###### Get the list of available Service Fabric versions
    Get-ServiceFabricRegisteredClusterCodeVersion
    ```

    Должен отобразиться результат следующего вида.

    ![Получение версий Service Fabric][getfabversions]
3. Запустите обновление кластера до доступной версии с помощью команды Windows PowerShell [Start-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/start-servicefabricclusterupgrade).

    ```powershell
    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

    ###### Here is a filled-out example

    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback
    ```
   За ходом обновления можно следить в Service Fabric Explorer. Также можно выполнить следующую команду PowerShell:

    ```powershell
    Get-ServiceFabricClusterUpgrade
    ```

    Если требования политик работоспособности кластера не соблюдаются, выполняется откат обновления. О том, как указать пользовательские политики работоспособности для команды Start-ServiceFabricClusterUpgrade, можно узнать из [соответствующей документации](/powershell/module/servicefabric/start-servicefabricclusterupgrade).

    Устранив проблемы, которые привели к откату, запустите обновление снова, выполнив уже описанные действия.

## <a name="upgrade-clusters-that-have-no-connectivity-to-download-the-latest-code-and-configuration"></a>Обновление кластеров *без возможности подключения* для скачивания последней версии кода и конфигурации
Выполните следующие действия, чтобы обновить до поддерживаемой версии кластер без возможности интернет-подключения к [Центру загрузки Майкрософт](https://download.microsoft.com).

> [!NOTE]
> Если вы используете кластер без возможности подключения к Интернету, вам нужно будет просматривать [блог команды разработчиков Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric), чтобы узнавать о новых выпусках. Система не будет показывать предупреждения о работоспособности кластера, чтобы оповещать вас о новых выпусках.  
>
>

### <a name="auto-provisioning-vs-manual-provisioning"></a>Автоматическая подготовка и подготовка вручную
Чтобы включить автоматические операции скачивания и регистрации для получения последней версии кода, настройте службу обновления Service Fabric. Инструкции см. в файле *Tools\ServiceFabricUpdateService.zip\Readme_InstructionsAndHowTos.txt* в [изолированном пакете](service-fabric-cluster-standalone-package-contents.md).

Для выполнения операций вручную сделайте следующее.

Прежде чем начать обновление конфигурации, измените конфигурацию кластера, присвоив следующему свойству значение *false*:

```json
"fabricClusterAutoupgradeEnabled": false,
```

См. дополнительные сведения об использовании [команды PowerShell Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade). Обязательно обновите параметр clusterConfigurationVersion в JSON, прежде чем запускать обновление конфигурации.

```powershell
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
```

### <a name="cluster-upgrade-workflow"></a>Рабочий процесс обновления кластера

1. Выполните командлет [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade) на одном из узлов в кластере и запишите значение *TargetCodeVersion*.

2. Выполните следующую команду с компьютера, подключенного к Интернету, чтобы получить список всех версий, совместимых с обновлением, для текущей версии и скачайте соответствующий пакет с помощью связанных ссылок для загрузки.

    ```powershell
    ###### Get list of all upgrade compatible packages  
    Get-ServiceFabricRuntimeUpgradeVersion -BaseVersion <TargetCodeVersion as noted in Step 1> 
    ```

3. Подключитесь к кластеру с любой виртуальной машины, на которой есть доступ администратора ко всем ВМ, перечисленным в качестве узлов в файле конфигурации кластера. Компьютер, на котором выполняется этот скрипт, может и не входить в кластер.

    ```powershell
    ###### Get the list of available Service Fabric versions
    Copy-ServiceFabricClusterPackage -Code -CodePackagePath <name of the .cab file including the path to it> -ImageStoreConnectionString "fabric:ImageStore"

    ###### Here is a filled-out example
    Copy-ServiceFabricClusterPackage -Code -CodePackagePath .\MicrosoftAzureServiceFabric.5.3.301.9590.cab -ImageStoreConnectionString "fabric:ImageStore"
    ```
4. Скопируйте скачанный пакет в хранилище образов кластера.

5. Зарегистрируйте скопированный пакет.

    ```powershell
    ###### Get the list of available Service Fabric versions
    Register-ServiceFabricClusterPackage -Code -CodePackagePath <name of the .cab file>

    ###### Here is a filled-out example
    Register-ServiceFabricClusterPackage -Code -CodePackagePath MicrosoftAzureServiceFabric.5.3.301.9590.cab
    ```
6. Запустите обновление кластера до доступной версии.

    ```powershell
    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

    ###### Here is a filled-out example
    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback
    ```
    За ходом обновления можно следить в Service Fabric Explorer. Можно также выполнить следующую команду PowerShell:

    ```powershell
    Get-ServiceFabricClusterUpgrade
    ```

    Если требования политик работоспособности кластера не соблюдаются, выполняется откат обновления. О том, как указать пользовательские политики работоспособности для команды Start-ServiceFabricClusterUpgrade, можно узнать из [соответствующей документации](/powershell/module/servicefabric/start-servicefabricclusterupgrade).

    Устранив проблемы, которые привели к откату, запустите обновление снова, выполнив уже описанные действия.

## <a name="next-steps"></a>Дальнейшие действия
* [Обновление конфигурации изолированного кластера](service-fabric-cluster-config-upgrade-windows-server.md)
* [Настройка параметров кластера Service Fabric](service-fabric-cluster-fabric-settings.md).
* [Масштабирование кластера](service-fabric-cluster-scale-in-out.md).

<!--Image references-->
[getfabversions]: ./media/service-fabric-cluster-upgrade-windows-server/getfabversions.PNG
