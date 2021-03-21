---
title: Семантика Рунтокомплетион в Service Fabric
description: Описывает семантику Рунтокомплетион в Service Fabric.
author: shsha-msft
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: shsha
ms.openlocfilehash: 6f2f6aa4380fcf6909957118bf682275350ce68c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96000272"
---
# <a name="runtocompletion"></a>RunToCompletion

Начиная с версии 7,1 Service Fabric поддерживает семантику **рунтокомплетион** для [контейнеров][containers-introduction-link] и [гостевых исполняемых][guest-executables-introduction-link] приложений. Эти семантики позволяют приложениям и службам, которые завершают задачу и завершать работу, в отличие от всегда запуска приложений и служб.

Прежде чем продолжить работу с этой статьей, мы рекомендуем ознакомиться с [моделью Service Fabric приложения][application-model-link] и [моделью размещения Service Fabric][hosting-model-link].

> [!NOTE]
> Семантика Рунтокомплетион в настоящее время не поддерживается для служб, написанных с помощью модели программирования [Reliable Services][reliable-services-link] .
 
## <a name="runtocompletion-semantics-and-specification"></a>Семантика и спецификация Рунтокомплетион
Семантику Рунтокомплетион можно указать как **ExecutionPolicy** при [импорте ServiceManifest][application-and-service-manifests-link]. Указанная политика наследуется всеми содержащиеся, составляющими ServiceManifest. В следующем фрагменте кода ApplicationManifest.xml приведен пример.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="RunToCompletionServicePackage" ServiceManifestVersion="1.0"/>
  <Policies>
    <ExecutionPolicy Type="RunToCompletion" Restart="OnFailure"/>
  </Policies>
</ServiceManifestImport>
```
**ExecutionPolicy** допускает следующие два атрибута:
* **Тип:** **рунтокомплетион** в настоящее время является единственным допустимым значением для этого атрибута.
* **Перезапустить:** Этот атрибут указывает политику перезапуска, которая применяется к содержащиеся, включающей в себя ServicePackage, при сбое. CodePackage выход с **ненулевым кодом завершения** считается неудачным. Допустимыми значениями для этого атрибута являются **onFailure** и **Never** — значение по умолчанию — **onFailure** .

Если для политики перезапуска задано значение **onFailure**, то при сбое любого CodePackage **(ненулевой код выхода)** он перезапускается с использованием обратных компромиссов между повторными сбоями. Если политика перезапуска имеет значение **Never**, то в случае сбоя какого-либо CodePackage состояние развертывания DeployedServicePackage помечается как **Failed** , но другие содержащиеся могут продолжать выполнение. Если все содержащиеся, включающие в себя выполнение ServicePackage для успешного завершения **(код выхода 0)**, состояние развертывания DeployedServicePackage помечается как **рантокомплетион**. 

## <a name="complete-example-using-runtocompletion-semantics"></a>Полный пример с использованием семантики Рунтокомплетион

Рассмотрим полный пример с использованием семантики Рунтокомплетион.

> [!IMPORTANT]
> В следующем примере предполагается знание создания [приложений контейнера Windows с помощью Service Fabric и DOCKER][containers-getting-started-link].
>
> Этот пример ссылается на mcr.microsoft.com/windows/nanoserver:1809. Контейнеры Windows Server совместимы не со всеми версиями ОС узла. Дополнительные сведения см. в разделе [Совместимость версий контейнеров Windows](/virtualization/windowscontainers/deploy-containers/version-compatibility).

В следующем ServiceManifest.xml описывается ServicePackage, состоящий из двух содержащиеся, которые представляют контейнеры. *RunToCompletionCodePackage1* просто записывает сообщение в **stdout** и завершает работу. *RunToCompletionCodePackage2* проверяет связь с адресом замыкания на себя через некоторое время, а затем завершается кодом выхода **0**, **1** или **2**.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ServiceManifest Name="WindowsRunToCompletionServicePackage" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Description>Windows RunToCompletion Service</Description>
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="WindowsRunToCompletionServiceType"  UseImplicitHost="true"/>
  </ServiceTypes>
  <CodePackage Name="RunToCompletionCodePackage1" Version="1.0">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/c,echo Hi from RunToCompletionCodePackage1 &amp;&amp; exit 0</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>

  <CodePackage Name="RunToCompletionCodePackage2" Version="1.0">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/v,/c,ping 127.0.0.1 &amp;&amp; set /a exitCode=%random% % 3 &amp;&amp; exit !exitCode!</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>
</ServiceManifest>
```

В следующем ApplicationManifest.xml описывается приложение, основанное на ServiceManifest.xml, описанном выше. Он указывает **рунтокомплетион** **ExecutionPolicy** для *виндовсрунтокомплетионсервицепаккаже* с политикой перезапуска **onFailure**. После активации *виндовсрунтокомплетионсервицепаккаже* будет запущено его составное содержащиеся. *RunToCompletionCodePackage1* должен успешно выйти при первой активации. Однако *RunToCompletionCodePackage2* может завершиться неудачно **(ненулевой код выхода)**. в этом случае он будет перезапущен, так как политика перезапуска находится в **onFailure**.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApplicationManifest ApplicationTypeName="WindowsRunToCompletionApplicationType" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <Description>Windows RunToCompletion Application</Description>

  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="WindowsRunToCompletionServicePackage" ServiceManifestVersion="1.0"/>
    <Policies>
      <ExecutionPolicy Type="RunToCompletion" Restart="OnFailure"/>
    </Policies>
  </ServiceManifestImport>

  <DefaultServices>
    <Service Name="WindowsRunToCompletionService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="WindowsRunToCompletionServiceType" InstanceCount="1">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```
## <a name="querying-deployment-status-of-a-deployedservicepackage"></a>Запрос состояния развертывания DeployedServicePackage
Состояние развертывания DeployedServicePackage можно запрашивать из PowerShell с помощью [Get-сервицефабрикдеплойедсервицепаккаже][deployed-service-package-link] или C# с помощью [FabricClient][fabric-client-link] API [жетдеплойедсервицепаккажелистасинк (строка, URI, строка)][deployed-service-package-fabricclient-link] .

## <a name="considerations-when-using-runtocompletion-semantics"></a>Рекомендации по использованию семантики Рунтокомплетион

Для текущей поддержки Рунтокомплетион следует отметить следующие моменты.
* Эти семантики поддерживаются только для [контейнеров][containers-introduction-link] и [гостевых исполняемых][guest-executables-introduction-link] приложений.
* Сценарии обновления для приложений с семантикой Рунтокомплетион не допускаются. При необходимости пользователи должны удалять и повторно создавать такие приложения.
* События отработки отказа могут привести к повторному выполнению содержащиеся после успешного завершения, на том же узле или на других узлах кластера. Примерами событий отработки отказа являются перезапуски узлов и обновление среды выполнения Service Fabric на узле.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях.

* [Service Fabric и контейнеры.][containers-introduction-link]
* [Service Fabric и гостевые исполняемые файлы.][guest-executables-introduction-link]

<!-- Links -->
[containers-introduction-link]: service-fabric-containers-overview.md
[containers-getting-started-link]: service-fabric-get-started-containers.md
[guest-executables-introduction-link]: service-fabric-guest-executables-introduction.md
[reliable-services-link]: service-fabric-reliable-services-introduction.md
[application-model-link]: service-fabric-application-model.md
[hosting-model-link]: service-fabric-hosting-model.md
[application-and-service-manifests-link]: service-fabric-application-and-service-manifests.md
[setup-entry-point-link]: service-fabric-run-script-at-service-startup.md
[deployed-service-package-working-with-link]: service-fabric-hosting-model.md#work-with-a-deployed-service-package
[deployed-code-package-link]: /powershell/module/servicefabric/get-servicefabricdeployedcodepackage
[deployed-service-package-link]: /powershell/module/servicefabric/get-servicefabricdeployedservicepackage
[fabric-client-link]: /dotnet/api/system.fabric.fabricclient
[deployed-service-package-fabricclient-link]: /dotnet/api/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync
