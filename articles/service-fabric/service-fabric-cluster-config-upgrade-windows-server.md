---
title: Обновление конфигурации изолированного кластера
description: Узнайте, как обновить конфигурацию, которую использует изолированный кластер Service Fabric.
ms.topic: conceptual
ms.date: 11/09/2018
ms.openlocfilehash: 3cb1d40f5b32415588d3fd5a108967cfb4c0e534
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91842620"
---
# <a name="upgrade-the-configuration-of-a-standalone-cluster"></a>Обновление конфигурации изолированного кластера 

Для любой современной системы возможность обновления является ключом к успеху вашего продукта в долгосрочной перспективе. Кластер Azure Service Fabric — это ресурс, владельцем которого вы являетесь. Из этой статьи вы узнаете, как обновить параметры конфигурации для изолированного кластера Service Fabric.

## <a name="customize-cluster-settings-in-the-clusterconfigjson-file"></a>Настройка параметров кластера в файле ClusterConfig.json
Автономные кластеры настраиваются с помощью *ClusterConfig.jsв* файле. Дополнительные сведения о различных параметрах см. в статье [Параметры конфигурации для изолированного кластера Windows](service-fabric-cluster-manifest.md).

Добавить, обновить или удалить параметры можно в `fabricSettings` разделе [Свойства кластера](./service-fabric-cluster-manifest.md#cluster-properties) раздела *ClusterConfig.json*. 

Например, следующий JSON добавляет новый параметр *MaxDiskQuotaInMB* в разделе *Diagnostics* в строке `fabricSettings`:

```json
      {
        "name": "Diagnostics",
        "parameters": [
          {
            "name": "MaxDiskQuotaInMB",
            "value": "65536"
          }
        ]
      }
```

Изменив параметры в файле ClusterConfig.json, [протестируйте конфигурацию кластера](#test-the-cluster-configuration), после чего [примените новые параметры](#upgrade-the-cluster-configuration), чтобы обновить конфигурацию. 

## <a name="test-the-cluster-configuration"></a>Тестирование конфигурации кластера
Прежде чем начинать обновление конфигурации, можно протестировать новый JSON-файл конфигурации кластера, выполнив следующий скрипт PowerShell в изолированном пакете:

```powershell
TestConfiguration.ps1 -ClusterConfigFilePath <Path to the new Configuration File> -OldClusterConfigFilePath <Path to the old Configuration File>
```

Также можно выполнить этот скрипт:

```powershell
TestConfiguration.ps1 -ClusterConfigFilePath <Path to the new Configuration File> -OldClusterConfigFilePath <Path to the old Configuration File> -FabricRuntimePackagePath <Path to the .cab file which you want to test the configuration against>
```

Невозможно обновить некоторые конфигурации, например конечные точки, имя кластера, IP-адрес узла и т. д. Новый JSON конфигурации кластера проверяется по старому и выдает ошибки в окне PowerShell при возникновении проблемы.

## <a name="upgrade-the-cluster-configuration"></a>Обновление конфигурации кластера
Чтобы обновить конфигурацию кластера, выполните команду [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade). Обновление конфигурации обрабатывает домен обновления.

```powershell
Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
```

## <a name="upgrade-cluster-certificate-configuration"></a>Обновление конфигурации сертификата кластера
Сертификат кластера используется для аутентификации между узлами кластера. Смену сертификата следует выполнять с особой осторожностью, так как в случае сбоя заблокируется связь между узлами кластера.

Поддерживаются четыре способа:  

* Однократное обновление сертификата. Путь обновления: ''сертификат А (основной) -> сертификат B (основной) -> сертификат C (основной) ->…''.

* Двойное обновление сертификата. Путь обновления ''сертификат А (основной) -> сертификат А (основной) и В (дополнительный) -> сертификат В (основной) -> сертификат В (основной) и С (дополнительный) -> сертификат C (основной)->…''.

* Тип обновления сертификатов: сертификат на основе отпечатка <-> сертификаты на основе конфигурации CommonName. Например, отпечаток сертификата A (основной) и отпечаток B (дополнительный) -> CommonName сертификата C.

* Обновление отпечатка издателя сертификата. Путь обновления "общее имя сертификата — A, отпечаток издателя — IT1 (основной) > общее имя сертификата — A, отпечаток издателя — IT1, IT2 (основной) > общее имя сертификата — A, отпечаток издателя — IT2 (основной)".


## <a name="next-steps"></a>Дальнейшие действия
* Узнайте, как настроить некоторые [параметры кластера Service Fabric](service-fabric-cluster-fabric-settings.md).
* Ознакомьтесь с концепцией [масштабирования кластера](service-fabric-cluster-scale-in-out.md).
* Узнайте об [обновлениях приложений](service-fabric-application-upgrade.md).

<!--Image references-->
[getfabversions]: ./media/service-fabric-cluster-upgrade-windows-server/getfabversions.PNG
