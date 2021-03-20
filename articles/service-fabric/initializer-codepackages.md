---
title: Инициализатор содержащиеся в Service Fabric
description: Описывает инициализатор содержащиеся в Service Fabric.
author: shsha-msft
ms.topic: conceptual
ms.date: 03/10/2020
ms.author: shsha
ms.openlocfilehash: 3be079b97c2660437344f88203fdda06cc6d6740
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86258980"
---
# <a name="initializer-codepackages"></a>Initializer Code Packages

Начиная с версии 7,1, Service Fabric поддерживает **инициализаторы содержащиеся** для [контейнеров][containers-introduction-link] и [гостевых исполняемых][guest-executables-introduction-link] приложений. Инициализатор содержащиеся предоставляет возможность выполнять инициализации в области ServicePackage до начала выполнения других содержащиеся. Их связь с ServicePackage является аналогом [SetupEntryPoint][setup-entry-point-link] для CodePackage.

Прежде чем продолжить работу с этой статьей, мы рекомендуем ознакомиться с [моделью Service Fabric приложения][application-model-link] и [моделью размещения Service Fabric][hosting-model-link].

> [!NOTE]
> Инициализаторы содержащиеся в настоящее время не поддерживаются для служб, написанных с помощью модели программирования [Reliable Services][reliable-services-link] .
 
## <a name="semantics"></a>Семантика

Для успешного выполнения ожидается CodePackage инициализатора **(код выхода 0)**. Сбой CodePackage инициализатора перезапускается до успешного завершения. Несколько содержащиеся инициализатора разрешены и выполняются для **успешного завершения**, **последовательно** **в указанном порядке** до начала выполнения других содержащиеся в ServicePackage.

## <a name="specifying-initializer-codepackages"></a>Указание содержащиеся инициализатора
Можно пометить CodePackage как инициализатор, задав атрибуту **инициализатора** **значение true** в ServiceManifest. При наличии нескольких содержащиеся инициализатора их порядок выполнения можно указать с помощью атрибута **ексекордер** . **Ексекордер** должно быть неотрицательным целым числом и допустимым только для инициализатора содержащиеся. Сначала выполняются инициализаторы содержащиеся с более низкими значениями **ексекордер** . Если **ексекордер** не задан для инициализатора CodePackage, предполагается значение по умолчанию 0. Не задан относительный порядок выполнения инициализатора содержащиеся с тем же значением **ексекордер** .

Следующий фрагмент ServiceManifest описывает три содержащиеся, которые помечены как инициализаторы. При активации ServicePackage выполняется сначала *InitCodePackage0* , так как он имеет наименьшее значение **ексекордер**. При успешном завершении (код выхода 0) для *InitCodePackage0* выполняется *InitCodePackage1* . Наконец, при успешном завершении *InitCodePackage1* выполняется *ворклоадкодепаккаже* .

```xml
<CodePackage Name="InitCodePackage0" Version="1.0" Initializer="true" ExecOrder="0">
  ...
</CodePackage>

<CodePackage Name="InitCodePackage1" Version="1.0" Initializer="true" ExecOrder="1">
  ...
</CodePackage>

<CodePackage Name="WorkloadCodePackage" Version="1.0">
  ...
</CodePackage>
```
## <a name="complete-example-using-initializer-codepackages"></a>Полный пример с использованием инициализатора содержащиеся

Рассмотрим полный пример с помощью инициализатора содержащиеся.

> [!IMPORTANT]
> В следующем примере предполагается знание создания [приложений контейнера Windows с помощью Service Fabric и DOCKER][containers-getting-started-link].
>
> Этот пример ссылается на mcr.microsoft.com/windows/nanoserver:1809. Контейнеры Windows Server совместимы не со всеми версиями ОС узла. Дополнительные сведения см. в разделе [Совместимость версий контейнеров Windows](/virtualization/windowscontainers/deploy-containers/version-compatibility).

Следующий ServiceManifest.xml строится на основе фрагмента ServiceManifest, описанного ранее. *InitCodePackage0*, *InitCodePackage1* и *ворклоадкодепаккаже* — содержащиеся, которые представляют контейнеры. После активации сначала выполняется *InitCodePackage0* . Он записывает сообщение в файл и завершает работу. Затем выполняется *InitCodePackage1* , который также записывает сообщение в файл и завершает работу. Наконец, *ворклоадкодепаккаже* начинает выполнение. Он также записывает сообщение в файл, выводит содержимое файла в **stdout** , а затем выполняет проверку связи бессрочно.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ServiceManifest Name="WindowsInitCodePackageServicePackage" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Description>Windows Init CodePackage Service</Description>
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="WindowsInitCodePackageServiceType"  UseImplicitHost="true"/>
  </ServiceTypes>
  <CodePackage Name="InitCodePackage0" Version="1.0" Initializer="true" ExecOrder="0">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/c,echo Hi from InitCodePackage0. &gt; C:\WorkspaceOnContainer\log.txt</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>

  <CodePackage Name="InitCodePackage1" Version="1.0" Initializer="true" ExecOrder="1">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/c,echo Hi from InitCodePackage1. &gt;&gt; C:\WorkspaceOnContainer\log.txt</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>

  <CodePackage Name="WorkloadCodePackage" Version="1.0">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/c,echo Hi from WorkloadCodePackage. &gt;&gt; C:\WorkspaceOnContainer\log.txt &amp;&amp; type C:\WorkspaceOnContainer\log.txt &amp;&amp; ping -t 127.0.0.1 &gt; nul</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>
</ServiceManifest>
```

В следующем ApplicationManifest.xml описывается приложение, основанное на ServiceManifest.xml, описанном выше. Обратите внимание, что он задает один и тот же **том** для всех контейнеров, т. е. **к:\воркспацеонхост** подключается по адресу **к:\воркспацеонконтаинер** во всех трех контейнерах. В результате все контейнеры записываются в один и тот же файл журнала в том порядке, в котором они были активированы.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApplicationManifest ApplicationTypeName="WindowsInitCodePackageApplicationType" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <Description>Windows Init CodePackage Application</Description>

  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="WindowsInitCodePackageServicePackage" ServiceManifestVersion="1.0"/>
    <Policies>
      <ContainerHostPolicies CodePackageRef="InitCodePackage0" ContainersRetentionCount="2" RunInteractive="true">
        <Volume Source="C:\WorkspaceOnHost" Destination="C:\WorkspaceOnContainer" IsReadOnly="false" />
      </ContainerHostPolicies>

     <ContainerHostPolicies CodePackageRef="InitCodePackage1" ContainersRetentionCount="2" RunInteractive="true">
        <Volume Source="C:\WorkspaceOnHost" Destination="C:\WorkspaceOnContainer" IsReadOnly="false" />
      </ContainerHostPolicies>

      <ContainerHostPolicies CodePackageRef="WorkloadCodePackage" ContainersRetentionCount="2" RunInteractive="true">
        <Volume Source="C:\WorkspaceOnHost" Destination="C:\WorkspaceOnContainer" IsReadOnly="false" />
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>

  <DefaultServices>
    <Service Name="WindowsInitCodePackageService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="WindowsInitCodePackageServiceType" InstanceCount="1">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```
После успешной активации ServicePackage содержимое **C:\WorkspaceOnHost\log.txt** должно быть следующим.

```console
C:\Users\test>type C:\WorkspaceOnHost\log.txt
Hi from InitCodePackage0.
Hi from InitCodePackage1.
Hi from WorkloadCodePackage.
```

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
[setup-entry-point-link]: service-fabric-run-script-at-service-startup.md
