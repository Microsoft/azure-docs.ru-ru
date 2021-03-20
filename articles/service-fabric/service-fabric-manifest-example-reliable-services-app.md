---
title: Примеры манифеста приложения надежных служб
description: Узнайте, как настроить параметры манифестов приложений и служб для приложения Service Fabric Reliable Services.
author: peterpogorski
ms.topic: conceptual
ms.date: 06/11/2018
ms.author: pepogors
ms.openlocfilehash: f40e54f5260f827f0b18c833d23d1f57b5ebc3a3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84701104"
---
# <a name="reliable-services-application-and-service-manifest-examples"></a>Примеры манифестов приложений Reliable Services и служб
Ниже приведены примеры манифестов приложений и служб для приложения Service Fabric с внешним веб-интерфейсном ASP.NET Core и серверной частью с отслеживанием состояния. Цель этих примеров — показать, какие параметры являются доступными и как их использовать. Эти манифесты приложений и служб основаны на манифестах [в кратком руководстве по Service Fabric для .NET](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/).

Показаны следующие функции:

|манифеста|Компоненты|
|---|---|
|[Манифест приложения](#application-manifest)| [Управление ресурсами](service-fabric-resource-governance.md), [запуск службы с использованием учетной записи локального администратора](service-fabric-application-runas-security.md), [применение политики по умолчанию для всех пакетов кода службы](service-fabric-application-runas-security.md#apply-a-default-policy-to-all-service-code-packages), [создание субъектов пользователей и групп](service-fabric-application-runas-security.md), совместное использование пакета данных экземплярами служб, [переопределение конечных точек службы](service-fabric-service-manifest-resources.md#overriding-endpoints-in-servicemanifestxml)| 
|Манифест службы FrontEndService| [Выполнение сценария при запуске службы](service-fabric-run-script-at-service-startup.md), [определение конечной точки HTTPS](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#define-an-https-endpoint-in-the-service-manifest) | 
|Манифест службы BackEndService| [Объявление пакета конфигурации](service-fabric-application-and-service-manifests.md), [объявление пакета данных](service-fabric-application-and-service-manifests.md), [настройка конечной точки](service-fabric-service-manifest-resources.md)| 

Дополнительные сведения о конкретных XML-элементах см. в разделах [Элементы манифеста приложения](#application-manifest-elements), [Элементы манифеста службы VotingWeb](#votingweb-service-manifest-elements) и [Элементы манифеста службы VotingData](#votingdata-service-manifest-elements).

## <a name="application-manifest"></a>Манифест приложения

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="VotingData_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingData_PartitionCount" DefaultValue="1" />
    <Parameter Name="VotingData_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingWeb_InstanceCount" DefaultValue="-1" />
    <Parameter Name="CpuCores" DefaultValue="2" />
    <Parameter Name="Memory" DefaultValue="4084" />
    <Parameter Name="BlockIOWeight" DefaultValue="200" />
    <Parameter Name="MaximumIOBandwidth" DefaultValue="1024" />
    <Parameter Name="MemoryReservationInMB" DefaultValue="1024" />
    <Parameter Name="MemorySwapInMB" DefaultValue="4084"/>
    <Parameter Name="Port" DefaultValue="8081" />
    <Parameter Name="Protocol" DefaultValue="tcp" />
    <Parameter Name="Type" DefaultValue="internal" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingDataPkg" ServiceManifestVersion="1.0.0" />
    <!-- Override endpoints declared in the service manifest. -->
    <ResourceOverrides>
      <Endpoints>
        <Endpoint Name="DataEndpoint" Port="[Port]" Protocol="[Protocol]" Type="[Type]" />
      </Endpoints>
    </ResourceOverrides>

    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      
      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[Memory]"/>

      <!-- Set resource governance at the code package level. -->
      <ResourceGovernancePolicy CodePackageRef="Code" CpuPercent="10" MemoryInMB="[Memory]" BlockIOWeight="[BlockIOWeight]" 
                                MaximumIOBandwidth="[MaximumIOBandwidth]" MaximumIOps="[MaximumIOps]" MemoryReservationInMB="[MemoryReservationInMB]" 
                                MemorySwapInMB="[MemorySwapInMB]"/>

      <!-- Share the data package across multiple instances of the VotingData service-->
      <PackageSharingPolicy PackageRef="Data"/>

      <!-- Give read rights on the "DataEndpoint" endpoint to the Customer2 account.-->
      <SecurityAccessPolicy GrantRights="Read" PrincipalRef="Customer2" ResourceRef="DataEndpoint" ResourceType="Endpoint"/>         
    </Policies>
  </ServiceManifestImport>
  
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="1.0.0" />
    
    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      <!-- Run the setup entry point (defined in the imported service manifest) as the SetupAdminUser account 
      (declared in the following Principals section). -->
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
      
    </Policies>

  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="VotingData">
      <StatefulService ServiceTypeName="VotingDataType" TargetReplicaSetSize="[VotingData_TargetReplicaSetSize]" MinReplicaSetSize="[VotingData_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[VotingData_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    <Service Name="VotingWeb" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="VotingWebType" InstanceCount="[VotingWeb_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <!-- Define users and groups required to run the services and access resources.  Principals are used in the Policies section(s). -->
  <Principals>
    <!-- Declare a set of groups as security principals, which can be referenced in policies. Groups are useful if there are multiple users 
    for different service entry points and they need to have certain common privileges that are available at the group level. -->
    <Groups>
      <!-- Create a group that has administrator privileges. -->
      <Group Name="LocalAdminGroup">
        <Membership>
          <SystemGroup Name="Administrators" />
        </Membership>
      </Group>
    </Groups>
    <Users>
      <!-- Declare a user and add the user to the Administrators system group. The SetupAdminUser account is used to run the 
      setup entry point of the VotingWebPkg code package (described in the preceding Policies section).-->
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
      <!-- Create a user. Local user accounts are created on the machines where the application is deployed. By default, these accounts 
      do not have the same names as those specified here. Instead, they are dynamically generated and have random passwords. -->
      <User Name="Customer1" >
        <MemberOf>
          <!-- Add the user to the local administrators group.-->
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <!-- Create a user as a local user with the specified account name and password. Local user accounts are created on the machines 
      where the application is deployed. -->
      <User Name="Customer2" AccountType="LocalUser" AccountName="Customer1" Password="MyPassword">
        <MemberOf>
          <!-- Add the user to the local administrators group.-->
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <!-- Create a user as NetworkService. -->
      <User Name="MyDefaultAccount" AccountType="NetworkService" />      
    </Users>
  </Principals>
  <!-- Policies applied at the application level. -->
  <Policies>
    <!-- Specify a default user account for all code packages that don’t have a specific RunAsPolicy defined in 
    the ServiceManifestImport section(s). -->
    <DefaultRunAsPolicy UserRef="MyDefaultAccount" />
    
  </Policies>
</ApplicationManifest>

```

## <a name="votingweb-service-manifest"></a>Манифест службы VotingWeb

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType. 
         This name must match the string used in RegisterServiceType call in Program.cs. -->
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <!-- A privileged entry point that by default runs with the same credentials as Service Fabric (typically the NetworkService account) before 
    any other entry point. Use the setup entry point to set system environment variables, give the account running the service (NETWORK SERVICE, by default) 
    access to a certificate's private key, or perform other setup tasks. In the application manifest, you can change the security permissions to run the startup script 
    under a local system account or an administrator account.  -->
    <SetupEntryPoint>
      <ExeHost>
        <!-- The setup script to run. -->
        <Program>Setup.bat</Program>
        <!-- Pass arguments to the script when it runs.-->
        <Arguments>MyValue</Arguments>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
        <!-- Warning! Do not use console redirection in a production application, only use it for local development and debugging. Redirects console output from the startup
        script to an output file in the application folder called "log" on the cluster node where the application is deployed and run. Also set the number of output files
        to retain and the maximum file size (in KB). -->
        <ConsoleRedirection FileRetentionCount="10" FileMaxSizeInKb="20480"/>
      </ExeHost>
    </SetupEntryPoint>
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- Configure a HTTPS endpoint on port 443. This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>

```

## <a name="votingdata-service-manifest"></a>Манифест службы VotingData

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingDataPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType. 
         This name must match the string used in RegisterServiceType call in Program.cs. -->
    <StatefulServiceType ServiceTypeName="VotingDataType"  HasPersistedState="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingData.exe</Program>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Declares a folder, named by the Name attribute, under PackageRoot that contains a Settings.xml file. This file contains sections of user-defined, 
  key-value pair settings that the process can read back at run time. During an upgrade, if only the ConfigPackage version has changed, 
  then the running process is not restarted. Instead, a callback notifies the process that configuration settings have changed so they can be reloaded dynamically. -->
  <ConfigPackage Name="Config" Version="1.0.0" />
  
  <!-- Declares a folder, named by the Name attribute, under PackageRoot which contains static data files to be consumed by the process at run time. -->
  <DataPackage Name="Data" Version="1.0.0"/>

  <Resources>
    <Endpoints>
      <!-- Define an internal (used for intra-application communication) TCP endpoint. Since no port is specified, one is created and assigned dynamically
           to the service.-->
      <Endpoint Name="DataEndpoint" Protocol="tcp" Type="Internal" />
    </Endpoints>
  </Resources>

</ServiceManifest>

```

## <a name="application-manifest-elements"></a>Элементы манифеста приложения
### <a name="applicationmanifest-element"></a>Элемент ApplicationManifest
Декларативно описывает тип приложения и его версию. Один или несколько манифестов составных служб указываются для составления типа приложения. Параметры конфигурации составных служб могут быть переопределены с помощью параметризованных настроек приложения. Службы по умолчанию, шаблоны служб, субъекты, политики, диагностическая настройка и сертификаты также могут быть объявлены на уровне приложения. Дополнительные сведения см. в разделе об [элементе ApplicationManifest](service-fabric-service-model-schema-elements.md#ApplicationManifestElementApplicationManifestTypeComplexType).

### <a name="parameters-element"></a>Элемент Parameters
Объявляет параметры, используемые в этом манифесте приложения. Значения этих параметров могут быть предоставлены при создании экземпляра приложения и могут использоваться для переопределения параметров конфигурации приложения или службы. Дополнительные сведения см. в разделе [Элемент Parameters (указывается в ApplicationManifestType)](service-fabric-service-model-schema-elements.md#ParametersElementanonymouscomplexTypeComplexTypeDefinedInApplicationManifestTypecomplexType).

### <a name="parameter-element"></a>Элемент Parameter
Параметр приложения для использования в данном манифесте. Значение параметра может быть изменено во время создания экземпляра приложения. Если значение не задано, используется значение по умолчанию. Дополнительные сведения см. в разделе [Элемент Parameter (указывается в Parameters)](service-fabric-service-model-schema-elements.md#ParameterElementanonymouscomplexTypeComplexTypeDefinedInParameterselement).

### <a name="servicemanifestimport-element"></a>Элемент ServiceManifestImport
Импортирует манифест службы, созданный разработчиком службы. Манифест службы должен быть импортирован для каждой составной службы в приложении. Для манифеста службы могут быть объявлены политики и переопределения параметров. Дополнительные сведения см. в разделе [Элемент ServiceManifestImport](service-fabric-service-model-schema-elements.md#ServiceManifestImportElementanonymouscomplexTypeComplexTypeDefinedInApplicationManifestTypecomplexType).

### <a name="servicemanifestref-element"></a>Элемент ServiceManifestRef
Импортирует манифест службы по ссылке. В настоящее время файл манифеста службы (ServiceManifest.xml) должен присутствовать в пакете сборки. Дополнительные сведения см. в разделе [Элемент ServiceManifestRef](service-fabric-service-model-schema-elements.md#ServiceManifestRefElementServiceManifestRefTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="resourceoverrides-element"></a>Элемент ResourceOverrides
Указывает переопределения ресурсов для конечных точек, объявленных в ресурсах манифеста служб. Дополнительные сведения см. в разделе [Элемент ResourceOverrides](service-fabric-service-model-schema-elements.md#ResourceOverridesElementResourceOverridesTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="endpoints-element"></a>Элемент Endpoints
Конечные точки для переопределения. Дополнительные сведения см. в разделе об [элементе Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourceOverridesTypecomplexType).

### <a name="endpoint-element"></a>Элемент Endpoint
Конечная точка для переопределения, объявленная в манифесте службы. Дополнительные сведения см. в разделе [Элемент Endpoint (тип EndpointOverrideType)](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).

### <a name="policies-element"></a>Элемент Policies
Описывает политики (привязка конечной точки, совместный доступ к пакетам, запуск от имени и безопасный доступ) для применения импортированного манифеста службы. Дополнительные сведения см. в разделе об [элементе Policies](service-fabric-service-model-schema-elements.md#PoliciesElementServiceManifestImportPoliciesTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="servicepackageresourcegovernancepolicy-element"></a>Элемент ServicePackageResourceGovernancePolicy
Определяет политику управления ресурсами, которая применяется на уровне всего пакета службы. Дополнительные сведения см. в разделе [Элемент ServicePackageResourceGovernancePolicy](service-fabric-service-model-schema-elements.md#ServicePackageResourceGovernancePolicyElementServicePackageResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInServicePackageTypecomplexType).

### <a name="resourcegovernancepolicy-element"></a>Элемент ResourceGovernancePolicy
Указывает ограничения ресурсов для элемента CodePackage. Дополнительные сведения см. в разделе [Элемент ResourceGovernancePolicy](service-fabric-service-model-schema-elements.md#ResourceGovernancePolicyElementResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelementDefinedInDigestedEndpointelement).

### <a name="packagesharingpolicy-element"></a>Элемент PackageSharingPolicy
Указывает, что пакет кода, конфигурации или данных должен совместно использоваться в экземплярах служб одного типа. Дополнительные сведения см. в разделе [Элемент PackageSharingPolicy](service-fabric-service-model-schema-elements.md#PackageSharingPolicyElementPackageSharingPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexType).

### <a name="securityaccesspolicy-element"></a>Элемент SecurityAccessPolicy
Предоставляет разрешение на доступ к субъекту в ресурсе (например, к конечной точке), определенному в манифесте службы. Как правило, это полезно для контроля и ограничения доступа служб к различным ресурсам, чтобы свести к минимуму угрозы безопасности. Это особенно важно в тех случаях, когда приложение создается на основе коллекции служб из marketplace, которые создали разные разработчики. Дополнительные сведения см. в разделе [Элемент SecurityAccessPolicy](service-fabric-service-model-schema-elements.md#SecurityAccessPolicyElementSecurityAccessPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInSecurityAccessPolicieselementDefinedInDigestedEndpointelement).

### <a name="runaspolicy-element"></a>Элемент RunAsPolicy
Указывает учетную запись локального пользователя или локальной системы, в которой будет выполняться пакет служебного кода. Учетные записи домена поддерживаются в развертываниях Windows Server, в которых находится Azure Active Directory. По умолчанию приложения выполняются под учетной записью, используемой процессом Fabric.exe. Приложения также могут выполняться с другими учетными записями, которые должны быть объявлены в разделе Principals. Если к службе применяется политика запуска от имени, а манифест службы объявляет в качестве ресурсов конечные точки, использующие протокол HTTP, вам нужно указать SecurityAccessPolicy. Это гарантирует, что к выделенным для этих конечных точек портам будут правильно применены списки управления доступом для той учетной записи, от имени которой выполняется служба. Для конечной точки HTTPS необходимо также определить EndpointBindingPolicy, чтобы указать имя сертификата, который будет возвращен клиенту. Дополнительные сведения см. в разделе [Элемент RunAsPolicy](service-fabric-service-model-schema-elements.md#RunAsPolicyElementRunAsPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="defaultservices-element"></a>Элемент DefaultServices
Декларирует экземпляры служб, которые создаются автоматически при создании экземпляра приложения в соответствии с его типом. Дополнительные сведения см. в разделе [Элемент DefaultServices](service-fabric-service-model-schema-elements.md#DefaultServicesElementDefaultServicesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInApplicationInstanceTypecomplexType).

### <a name="service-element"></a>Элемент Service
Объявляет службу, которая автоматически будет создана при создании экземпляра приложения. Дополнительные сведения см. в разделе [Элемент Service](service-fabric-service-model-schema-elements.md#ServiceElementanonymouscomplexTypeComplexTypeDefinedInDefaultServicesTypecomplexType).

### <a name="statefulservice-element"></a>Элемент StatefulService
Определяет службу с отслеживанием состояния. Дополнительные сведения см. в разделе [Элемент StatefulService](service-fabric-service-model-schema-elements.md#StatefulServiceElementStatefulServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement).

### <a name="statelessservice-element"></a>Элемент StatelessService
Определяет службу без отслеживания состояния. Дополнительные сведения см. в разделе [Элемент StatelessService](service-fabric-service-model-schema-elements.md#StatelessServiceElementStatelessServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement).

### <a name="principals-element"></a>Элемент Principals
Описывает субъекты безопасности (пользователи, группы), необходимые для запуска служб и защищенных ресурсов в приложении. На субъекты можно ссылаться в разделах политики. Дополнительные сведения см. в разделе [Элемент Principals](service-fabric-service-model-schema-elements.md#PrincipalsElementSecurityPrincipalsTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInEnvironmentTypecomplexType).

### <a name="groups-element"></a>Элемент Groups
Объявляет набор групп в качестве субъектов безопасности, на которые можно ссылаться в политиках. Элемент Groups может быть полезен, если для разных точек входа службы есть несколько пользователей и у них должны быть определены общие права, доступные на уровне группы. Дополнительные сведения см. в разделе [Элемент Groups](service-fabric-service-model-schema-elements.md#GroupsElementanonymouscomplexTypeComplexTypeDefinedInSecurityPrincipalsTypecomplexType).

### <a name="group-element"></a>Элемент Group
Объявляет группу в качестве субъекта безопасности, на который можно ссылаться в политиках. Дополнительные сведения см. в разделе [Элемент Group (определен в MemberOf)](service-fabric-service-model-schema-elements.md#GroupElementanonymouscomplexTypeComplexTypeDefinedInGroupselement).

### <a name="membership-element"></a>Элемент Membership
 Дополнительные сведения см. в разделе [Элемент Membership](service-fabric-service-model-schema-elements.md#MembershipElementanonymouscomplexTypeComplexTypeDefinedInGroupelement).

### <a name="systemgroup-element"></a>Элемент SystemGroup
 Дополнительные сведения см. в разделе [Элемент SystemGroup (определен в MemberOf)](service-fabric-service-model-schema-elements.md#SystemGroupElementanonymouscomplexTypeComplexTypeDefinedInMembershipelement).

### <a name="users-element"></a>Элемент Users
Объявляет набор пользователей в качестве субъектов безопасности, на которых можно ссылаться в политиках. Дополнительные сведения см. в разделе [Элемент Users](service-fabric-service-model-schema-elements.md#UsersElementanonymouscomplexTypeComplexTypeDefinedInSecurityPrincipalsTypecomplexType).

### <a name="user-element"></a>Элемент User
Объявляет пользователя в качестве субъекта безопасности, на которого можно ссылаться в политиках. Дополнительные сведения см. в разделе [Элемент User](service-fabric-service-model-schema-elements.md#UserElementanonymouscomplexTypeComplexTypeDefinedInUserselement).

### <a name="memberof-element"></a>Элемент MemberOf
Элемент Users можно добавлять в любую группу членов, чтобы он мог унаследовать все свойства и параметры безопасности этой группы. Такую группу можно использовать для защиты внешних ресурсов, доступ к которым требуется разным службам или одной и той же службе на разных компьютерах. Дополнительные сведения см. в разделе [Элемент MemberOf](service-fabric-service-model-schema-elements.md#MemberOfElementanonymouscomplexTypeComplexTypeDefinedInUserelement).

### <a name="systemgroup-element"></a>Элемент SystemGroup
Системная группа, в которую требуется добавить пользователя.  Эту группу необходимо определить в разделе Groups. Дополнительные сведения см. в разделе [Элемент SystemGroup (определен в MemberOf)](service-fabric-service-model-schema-elements.md#SystemGroupElementanonymouscomplexTypeComplexTypeDefinedInMemberOfelement).

### <a name="group-element"></a>Элемент Group
Группа, в которую требуется добавить пользователя.  Ее необходимо определить в разделе Groups. Дополнительные сведения см. в разделе [Элемент Group (определен в MemberOf)](service-fabric-service-model-schema-elements.md#GroupElementanonymouscomplexTypeComplexTypeDefinedInMemberOfelement).

### <a name="policies-element"></a>Элемент Policies
Описывает политики (сбор журналов, запуск от имени по умолчанию, работоспособность и доступ к безопасности) для применения на уровне приложения. Дополнительные сведения см. в разделе об [элементе Policies](service-fabric-service-model-schema-elements.md#PoliciesElementApplicationPoliciesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInEnvironmentTypecomplexType).

### <a name="defaultrunaspolicy-element"></a>Элемент DefaultRunAsPolicy
Указывает учетную запись пользователя по умолчанию для всех пакетов кода службы, для которых не определен RunAsPolicy в разделе ServiceManifestImport. Дополнительные сведения см. в разделе [Элемент DefaultRunAsPolicy](service-fabric-service-model-schema-elements.md#DefaultRunAsPolicyElementanonymouscomplexTypeComplexTypeDefinedInApplicationPoliciesTypecomplexType).




## <a name="votingweb-service-manifest-elements"></a>Элементы манифеста службы VotingWeb
### <a name="servicemanifest-element"></a>Элемент ServiceManifest
Декларативно описывает тип службы и его версию. В нем указан независимо обновляемый код, конфигурация и пакеты данных, из которых состоит пакет службы, для поддержки одного или нескольких типов служб. Указаны также ресурсы, параметры диагностики и метаданные службы, такие как тип службы, свойства работоспособности, а также метрики балансировки нагрузки. Дополнительные сведения см. в разделе [Элемент ServiceManifest](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType).

### <a name="servicetypes-element"></a>Элемент ServiceTypes
Указывает, какие типы служб поддерживаются пакетами CodePackage в этом манифесте. При создании экземпляра службы в соответствии с одним из этих типов службы все пакеты кода, объявленные в этом манифесте, активируются путем запуска соответствующих точек входа. Типы служб декларируются на уровне манифеста, а не на уровне пакета кода. Дополнительные сведения см. в разделе об [элементе ServiceTypes](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="statelessservicetype-element"></a>Элемент StatelessServiceType
Описывает тип службы без отслеживания состояния. Дополнительные сведения см. в разделе [Элемент StatelessServiceType](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType).

### <a name="codepackage-element"></a>Элемент CodePackage
Описывает пакеты кода, которые поддерживают тип определенной службы. При создании экземпляра службы в соответствии с одним из этих типов службы все пакеты кода, объявленные в этом манифесте, активируются путем запуска соответствующих точек входа. Запущенные вследствие этого процессы должны зарегистрировать поддерживаемые типы служб во время выполнения. При наличии нескольких пакетов кода они все активируются при поиске системой любого из задекларированных типов служб. Дополнительные сведения см. в разделе [Элемент CodePackage](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="setupentrypoint-element"></a>Элемент SetupEntryPoint
Это привилегированная точка входа, которая по умолчанию запускается с теми же учетными данными, что и Service Fabric (обычно это учетная запись NETWORKSERVICE), перед тем как будут запущены любые другие точки входа. Исполняемый файл, указанный в точке входа EntryPoint, обычно является узлом службы, запускаемым на длительный срок. Наличие отдельной точки входа настройки позволяет избежать необходимости в выполнении узла службы с расширенными правами в течение длительного срока. Дополнительные сведения см. в разделе [Элемент SetupEntryPoint](service-fabric-service-model-schema-elements.md#SetupEntryPointElementanonymouscomplexTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="exehost-element"></a>Элемент ExeHost
 Дополнительные сведения см. в разделе [Элемент ExeHost](service-fabric-service-model-schema-elements.md#ExeHostElementExeHostEntryPointTypeComplexTypeDefinedInSetupEntryPointelement).

### <a name="program-element"></a>Элемент Program
Имя исполняемого файла.  Например, MySetup.bat или MyServiceHost.exe. Дополнительные сведения см. в разделе [Элемент Program](service-fabric-service-model-schema-elements.md#ProgramElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="arguments-element"></a>Элемент Arguments
 Дополнительные сведения см. в разделе [Элемент Arguments](service-fabric-service-model-schema-elements.md#ArgumentsElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="workingfolder-element"></a>Элемент WorkingFolder
Рабочая папка для процесса находится в пакете кода на узле кластера, в котором развернуто приложение. Можно задать три значения: Work (по умолчанию), CodePackage или CodeBase. CodeBase указывает, что в качестве рабочей папки установлен каталог, в котором в пакете кода определен EXE-файл. CodePackage задает рабочую папку, которая будет корнем пакета кода независимо от того, определен ли EXE-файл в каталоге пакета кода. Work задает в качестве рабочей уникальную папку, созданную на узле.  Эта папка является одинаковой для всего экземпляра приложения. По умолчанию в качестве рабочей папки всех процессов в приложении задана рабочая папка приложения. Здесь процессы могут записывать данные. Записывать данные в пакет кода или базу кода не рекомендуется, так как эти папки могут совместно использоваться в различных экземплярах приложений и могут быть удалены. Дополнительные сведения см. в разделе [Элемент WorkingFolder](service-fabric-service-model-schema-elements.md#WorkingFolderElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="consoleredirection-element"></a>Элемент ConsoleRedirection

> [!WARNING]
> Не используйте перенаправление консоли в рабочем приложении. Его рекомендуется использовать только при локальной разработке и отладке. Этот элемент перенаправляет выходные данные консоли из сценария запуска в выходной файл в папке приложения с именем "log" на том узле кластера, где развертывается и выполняется приложение. Дополнительные сведения см. в разделе [Элемент ConsoleRedirection](service-fabric-service-model-schema-elements.md#ConsoleRedirectionElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="entrypoint-element"></a>Элемент EntryPoint
Исполняемый файл, указанный в точке входа EntryPoint, обычно является узлом службы, запускаемым на длительный срок. Наличие отдельной точки входа настройки позволяет избежать необходимости в выполнении узла службы с расширенными правами в течение длительного срока. Исполняемый файл, указанный в точке входа EntryPoint, запускается после успешного выхода из точки SetupEntryPoint. Возникающий вследствие этого процесс отслеживается и перезапускается (снова начиная с точки входа SetupEntryPoint), даже если произошло непредвиденное завершение его работы или сбой. Дополнительные сведения см. в разделе [Элемент EntryPoint](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="exehost-element"></a>Элемент ExeHost
 Дополнительные сведения см. в разделе [Элемент ExeHost](service-fabric-service-model-schema-elements.md#ExeHostElementanonymouscomplexTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType).

### <a name="configpackage-element"></a>Элемент ConfigPackage
Объявляет в разделе PackageRoot папку с именем, указанным в атрибуте Name, которая содержит файл Settings.xml. Этот файл содержит разделы заданных пользователем параметров пар "ключ-значение", которые могут считываться процессом во время выполнения. Во время обновления при изменении одного только атрибута version для ConfigPackage перезапуск процесса не выполняется. Вместо этого при помощи обратного вызова в процесс передается уведомление о том, что параметры конфигурации изменились, поэтому они были перезагружены в динамическом режиме. Дополнительные сведения см. в разделе [Элемент ConfigPackage](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement).

### <a name="resources-element"></a>Элемент Resources
Описывает ресурсы, используемые этой службой, которые могут быть объявлены без изменения скомпилированного кода и изменены при развертывании службы. Доступ к этим ресурсам осуществляется с помощью разделов Principals и Policies манифеста приложения. Дополнительные сведения см. в разделе [Элемент Resources](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="endpoints-element"></a>Элемент Endpoints
Определяет конечные точки службы. Дополнительные сведения см. в разделе об [элементе Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType).

### <a name="endpoint-element"></a>Элемент Endpoint
Конечная точка для переопределения, объявленная в манифесте службы. Дополнительные сведения см. в разделе [Элемент Endpoint (тип EndpointOverrideType)](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).



## <a name="votingdata-service-manifest-elements"></a>Элементы манифеста службы VotingData
### <a name="servicemanifest-element"></a>Элемент ServiceManifest
Декларативно описывает тип службы и его версию. В нем указан независимо обновляемый код, конфигурация и пакеты данных, из которых состоит пакет службы, для поддержки одного или нескольких типов служб. Указаны также ресурсы, параметры диагностики и метаданные службы, такие как тип службы, свойства работоспособности, а также метрики балансировки нагрузки. Дополнительные сведения см. в разделе [Элемент ServiceManifest](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType).

### <a name="servicetypes-element"></a>Элемент ServiceTypes
Указывает, какие типы служб поддерживаются пакетами CodePackage в этом манифесте. При создании экземпляра службы в соответствии с одним из этих типов службы все пакеты кода, объявленные в этом манифесте, активируются путем запуска соответствующих точек входа. Типы служб декларируются на уровне манифеста, а не на уровне пакета кода. Дополнительные сведения см. в разделе об [элементе ServiceTypes](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="statefulservicetype-element"></a>Элемент StatefulServiceType
Описывает тип службы с отслеживанием состояния. Дополнительные сведения см. в разделе [Элемент StatefulServiceType](service-fabric-service-model-schema-elements.md#StatefulServiceTypeElementStatefulServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType).

### <a name="codepackage-element"></a>Элемент CodePackage
Описывает пакеты кода, которые поддерживают тип определенной службы. При создании экземпляра службы в соответствии с одним из этих типов службы все пакеты кода, объявленные в этом манифесте, активируются путем запуска соответствующих точек входа. Запущенные вследствие этого процессы должны зарегистрировать поддерживаемые типы служб во время выполнения. При наличии нескольких пакетов кода они все активируются при поиске системой любого из задекларированных типов служб. Дополнительные сведения см. в разделе [Элемент CodePackage](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="entrypoint-element"></a>Элемент EntryPoint
Исполняемый файл, указанный в точке входа EntryPoint, обычно является узлом службы, запускаемым на длительный срок. Наличие отдельной точки входа настройки позволяет избежать необходимости в выполнении узла службы с расширенными правами в течение длительного срока. Исполняемый файл, указанный в точке входа EntryPoint, запускается после успешного выхода из точки SetupEntryPoint. Возникающий вследствие этого процесс отслеживается и перезапускается (снова начиная с точки входа SetupEntryPoint), даже если произошло непредвиденное завершение его работы или сбой. Дополнительные сведения см. в разделе [Элемент EntryPoint](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="exehost-element"></a>Элемент ExeHost
 Дополнительные сведения см. в разделе [Элемент ExeHost](service-fabric-service-model-schema-elements.md#ExeHostElementanonymouscomplexTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType).

### <a name="program-element"></a>Элемент Program
Имя исполняемого файла.  Например, MySetup.bat или MyServiceHost.exe. Дополнительные сведения см. в разделе [Элемент Program](service-fabric-service-model-schema-elements.md#ProgramElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="workingfolder-element"></a>Элемент WorkingFolder
Рабочая папка для процесса находится в пакете кода на узле кластера, в котором развернуто приложение. Можно задать три значения: Work (по умолчанию), CodePackage или CodeBase. CodeBase указывает, что в качестве рабочей папки установлен каталог, в котором в пакете кода определен EXE-файл. CodePackage задает рабочую папку, которая будет корнем пакета кода независимо от того, определен ли EXE-файл в каталоге пакета кода. Work задает в качестве рабочей уникальную папку, созданную на узле.  Эта папка является одинаковой для всего экземпляра приложения. По умолчанию в качестве рабочей папки всех процессов в приложении задана рабочая папка приложения. Здесь процессы могут записывать данные. Записывать данные в пакет кода или базу кода не рекомендуется, так как эти папки могут совместно использоваться в различных экземплярах приложений и могут быть удалены. Дополнительные сведения см. в разделе [Элемент WorkingFolder](service-fabric-service-model-schema-elements.md#WorkingFolderElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="configpackage-element"></a>Элемент ConfigPackage
Объявляет в разделе PackageRoot папку с именем, указанным в атрибуте Name, которая содержит файл Settings.xml. Этот файл содержит разделы заданных пользователем параметров пар "ключ-значение", которые могут считываться процессом во время выполнения. Во время обновления при изменении одного только атрибута version для ConfigPackage перезапуск процесса не выполняется. Вместо этого при помощи обратного вызова в процесс передается уведомление о том, что параметры конфигурации изменились, поэтому они были перезагружены в динамическом режиме. Дополнительные сведения см. в разделе [Элемент ConfigPackage](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement).

### <a name="datapackage-element"></a>Элемент DataPackage
Объявляет в разделе PackageRoot папку с именем, указанным в атрибуте Name, которая содержит статические файлы данных, обрабатываемые процессом во время выполнения. Service Fabric будет перезапускать все EXE-файлы и DLLHOST-файлы, указанные в пакетах поддержки и размещения, когда обновляется любой из пакетов данных, перечисленных в манифесте службы. Дополнительные сведения см. в разделе [Элемент DataPackage](service-fabric-service-model-schema-elements.md#DataPackageElementDataPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedDataPackageelement).

### <a name="resources-element"></a>Элемент Resources
Описывает ресурсы, используемые этой службой, которые могут быть объявлены без изменения скомпилированного кода и изменены при развертывании службы. Доступ к этим ресурсам осуществляется с помощью разделов Principals и Policies манифеста приложения. Дополнительные сведения см. в разделе [Элемент Resources](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="endpoints-element"></a>Элемент Endpoints
Определяет конечные точки службы. Дополнительные сведения см. в разделе об [элементе Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType).

### <a name="endpoint-element"></a>Элемент Endpoint
Конечная точка для переопределения, объявленная в манифесте службы. Дополнительные сведения см. в разделе [Элемент Endpoint (тип EndpointOverrideType)](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).

