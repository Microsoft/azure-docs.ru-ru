---
title: Параметризация файлов конфигурации в Azure Service Fabric
description: Узнайте, как параметризовать файлы конфигурации в Service Fabric, полезный способ управления несколькими средами.
ms.topic: conceptual
ms.date: 10/09/2018
ms.openlocfilehash: ca376230c427c47e839b2dee96e8daa83ccedf15
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96576763"
---
# <a name="how-to-parameterize-configuration-files-in-service-fabric"></a>Как параметризовать файлы конфигурации в Service Fabric

В этой статье описывается, как параметризовать файл конфигурации в Service Fabric.  Дополнительные сведения об основных концепциях см. в разделе [Управление приложениями для нескольких сред](service-fabric-manage-multiple-environment-app-configuration.md).

## <a name="procedure-for-parameterizing-configuration-files"></a>Процедура параметризации файлов конфигурации

В этом примере с помощью параметров в развертывании приложения переопределяется значение конфигурации.

1. Откройте файл *\<MyService>\PackageRoot\Config\Settings.xml* в проекте службы.
1. Задайте имя и значение параметра конфигурации, например кэш размером 25, добавив следующий XML:

   ```xml
    <Section Name="MyConfigSection">
      <Parameter Name="CacheSize" Value="25" />
    </Section>
   ```

1. Сохраните файл и закройте его.
1. Откройте файл *\<MyApplication>\ApplicationPackageRoot\ApplicationManifest.xml* .
1. В файле ApplicationManifest.xml объявите параметр и значение по умолчанию в элементе `Parameters`.  Рекомендуется задавать имя параметра с содержанием имени службы (например, "MyService").

   ```xml
    <Parameters>
      <Parameter Name="MyService_CacheSize" DefaultValue="80" />
    </Parameters>
   ```
1. В `ServiceManifestImport` разделе файла ApplicationManifest.xml добавьте `ConfigOverrides` `ConfigOverride` элемент и, ссылающийся на пакет конфигурации, раздел и параметр.

   ```xml
    <ConfigOverrides>
      <ConfigOverride Name="Config">
          <Settings>
            <Section Name="MyConfigSection">
                <Parameter Name="CacheSize" Value="[MyService_CacheSize]" />
            </Section>
          </Settings>
      </ConfigOverride>
    </ConfigOverrides>
   ```

> [!NOTE]
> В случае добавления ConfigOverride служба Service Fabric всегда выбирает параметры приложения или значение по умолчанию, указанное в манифесте приложения.
>
>

## <a name="next-steps"></a>Дальнейшие действия
Сведения о других возможностях управления приложениями, доступными в Visual Studio, см. в статье [Использование Visual Studio для упрощения создания приложений Service Fabric и управления ими](service-fabric-manage-application-in-visual-studio.md).
