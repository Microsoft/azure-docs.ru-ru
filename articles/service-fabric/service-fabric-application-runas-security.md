---
title: Запуск службы под управлением системы и локальной учетной записи безопасности
description: Узнайте о том, как запустить приложение Service Fabric с использованием системных и локальных учетных записей системы безопасности.  Создайте субъекты безопасности и примените политику запуска от имени для защищенного выполнения служб.
ms.topic: conceptual
ms.date: 03/29/2018
ms.openlocfilehash: 53212f8636602705899834b6db1d3f0d80b5fe4f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75610134"
---
# <a name="run-a-service-as-a-local-user-account-or-local-system-account"></a>Запуск службы с использованием локальной пользовательской или системной учетной записи
Платформа Azure Service Fabric помогает защищать приложения, работающие в кластере под разными учетными записями. По умолчанию приложения Service Fabric выполняются под учетной записью, используемой процессом Fabric.exe. Также Service Fabric позволяет запускать приложения от имени локального пользователя или с использованием учетной записи системы. Поддерживаются следующие типы учетных записей локальной системы: **LocalUser**, **NetworkService**, **LocalService** и **LocalSystem**.  При запуске Service Fabric в изолированном кластере Windows службу можно запустить с помощью [учетных записей домена Active Directory](service-fabric-run-service-as-ad-user-or-group.md) или [групповых управляемых учетных записей службы](service-fabric-run-service-as-gmsa.md).

В манифесте приложения в разделе **Principals** указываются учетные записи пользователей, которые используются для запуска служб или защиты ресурсов. Также вы можете определять и создавать группы пользователей, чтобы совместно управлять несколькими пользователями. Это полезно, если для разных точек входа службы созданы несколько пользователей с некоторыми общими привилегиями, доступными на уровне группы.  Эти пользователи указываются в политике запуска от имени, которая применяется к одной определенной службе или ко всем службам в приложении. 

По умолчанию политика запуска от имени применяется к главной точке входа.  Вы также можете применить политику запуска от имени к точке входа установки, если нужно [выполнить ряд операций с высоким уровнем привилегий, используя системную учетную запись](service-fabric-run-script-at-service-startup.md), или к обеим этим точкам входа сразу.  

> [!NOTE] 
> Если применить политику запуска от имени к службе, манифест которой объявляет ресурсы конечной точки с протоколом HTTP, необходимо указать **SecurityAccessPolicy**.  Дополнительные сведения см. в статье [Назначение политики безопасности доступа для конечных точек HTTP и HTTPS](service-fabric-assign-policy-to-endpoint.md). 
>

## <a name="run-a-service-as-a-local-user"></a>Запуск службы от имени локального пользователя
Можно создать локального пользователя и использовать его для защиты службы в приложении. Если в разделе Principals манифеста приложения указан тип учетной записи **LocalUser** , платформа Service Fabric создает учетные записи локальных пользователей на компьютерах с развернутым приложением. По умолчанию эти учетные записи не соответствуют именам, указанным в манифесте приложения (например, *Customer3* в следующем примере манифеста приложения). Вместо этого они создаются динамически и имеют случайные пароли.

В разделе **RunAsPolicy** для **ServiceManifestImport** укажите учетную запись из раздела **Principals**, с использованием которой нужно запустить пакет кода службы.  В следующем примере показано, как создать локального пользователя и применить политику запуска от имени к главной точке входа.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer3" EntryPointType="Main" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>    
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="Customer3" />
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="create-a-local-user-group"></a>Создание локальной группы пользователей
Вы можете создавать группы пользователей и добавлять в них пользователей. Это полезно, если для разных точек входа службы есть несколько пользователей и у них должны быть определены общие права, доступные на уровне группы. В следующем примере манифеста приложения представлена локальная группа с именем *LocalAdminGroup* и привилегиями администратора. В нее входят два пользователя: *Customer1* и *Customer2*. В разделе **ServiceManifestImport** применена политика запуска от имени, которая позволяет запустить пакет кода *Stateful1Pkg* от имени *Customer2*.  Другая политика запуска от имени применяется для запуска пакета кода *Web1Pkg* от имени *Customer1*.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Stateful1_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="Stateful1_PartitionCount" DefaultValue="1" />
    <Parameter Name="Stateful1_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer2" EntryPointType="Main"/>
    </Policies>
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer1" EntryPointType="Main"/>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="Stateful1" ServicePackageActivationMode="ExclusiveProcess">
      <StatefulService ServiceTypeName="Stateful1Type" TargetReplicaSetSize="[Stateful1_TargetReplicaSetSize]" MinReplicaSetSize="[Stateful1_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[Stateful1_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
      </StatefulService>
    </Service>
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Groups>
      <Group Name="LocalAdminGroup">
        <Membership>
          <SystemGroup Name="Administrators" />
        </Membership>
      </Group>
    </Groups>
    <Users>
      <User Name="Customer1">
        <MemberOf>
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <User Name="Customer2">
        <MemberOf>
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="apply-a-default-policy-to-all-service-code-packages"></a>Применение политики по умолчанию ко всем пакетам кода службы
С помощью раздела **DefaultRunAsPolicy** можно указать учетную запись пользователя по умолчанию для всех пакетов кода, для которых не определен раздел **RunAsPolicy**. Иногда большую часть пакетов кода, указанных в используемых приложением манифестах служб, нужно запускать от имени одной и той же учетной записи пользователя. В таком случае в приложении можно определить стандартную политику запуска от имени этой учетной записи пользователя. В следующем примере устанавливаются такие правила: если для пакета кода не указана политика **RunAsPolicy**, он будет запускаться от имени пользователя **MyDefaultAccount**, указанного в разделе Principals.  Поддерживаются следующие типы учетных записей: LocalUser, NetworkService, LocalSystem и LocalService.  Если вы используете локального пользователя или локальную службу, укажите для них имя и пароль учетной записи.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    
  </ServiceManifestImport>
  <DefaultServices>    
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="MyDefaultAccount" AccountType="NetworkService" />      
    </Users>
  </Principals>
  <Policies>
    <DefaultRunAsPolicy UserRef="MyDefaultAccount" />
  </Policies>
</ApplicationManifest>
```

## <a name="debug-a-code-package-locally-using-console-redirection"></a>Локальная отладка пакета кода с помощью перенаправления консоли
Иногда для отладки нужно просмотреть выходные данные консоли, в которой выполняется служба. С помощью манифеста службы вы можете задать для точки входа политику перенаправления консоли, которая записывает выходные данные в файл. Выходной файл сохраняется в папку приложения с именем **log** на том узле кластера, где развертывается и выполняется приложение. 

> [!WARNING]
> Никогда не используйте политику перенаправления консоли для приложений в рабочей среде, так как она может повлиять на отработку отказа приложения. Используйте ее *только* для локальной разработки и отладки.  
> 
> 

В приведенном ниже примере манифеста службы показано, как включить перенаправление консоли с помощью значения FileRetentionCount.

```xml
<CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
        <ConsoleRedirection FileRetentionCount="10"/>
      </ExeHost>
    </EntryPoint>
</CodePackage>

```

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>Дальнейшие действия
* [Сведения о модели приложения](service-fabric-application-model.md)
* [Указание ресурсов в манифесте службы](service-fabric-service-manifest-resources.md)
* [Развертывание приложения](service-fabric-deploy-remove-applications.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png
