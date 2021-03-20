---
title: Примеры манифеста приложения контейнера Service Fabric Azure
description: Узнайте, как настроить параметры манифестов приложений и служб для многоконтейнерного приложения Service Fabric.
author: peterpogorski
ms.topic: conceptual
ms.date: 06/08/2018
ms.author: pepogors
ms.openlocfilehash: 5efdbd17db20e69ace33b379ddbb99b2c4a20e69
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84701172"
---
# <a name="multi-container-application-and-service-manifest-examples"></a>Примеры манифестов многоконтейнерных приложений и служб
Ниже приведены примеры манифестов приложений и служб для многоконтейнерного приложения Service Fabric. Цель этих примеров — показать, какие параметры являются доступными и как их использовать. Эти манифесты приложений и служб основаны на манифестах [контейнера Windows Server 2016](https://github.com/Azure-Samples/service-fabric-containers/tree/master/Windows).

Показаны следующие функции:

|манифеста|Компоненты|
|---|---|
|[Манифест приложения](#application-manifest)| [Переопределение переменных среды](service-fabric-get-started-containers.md#configure-and-set-environment-variables), [настройка сопоставления порта контейнера с узлом](service-fabric-get-started-containers.md#configure-container-port-to-host-port-mapping-and-container-to-container-discovery), [настройка проверки подлинности в реестре контейнеров](service-fabric-get-started-containers.md#configure-container-repository-authentication), [управление ресурсами](service-fabric-resource-governance.md), [установка режима изоляции](service-fabric-get-started-containers.md#configure-isolation-mode), [указание образов контейнеров конкретной сборки ОС](service-fabric-get-started-containers.md#specify-os-build-specific-container-images)| 
|[Манифест службы FrontEndService](#frontendservice-service-manifest)| [Настройка переменных среды](service-fabric-get-started-containers.md#configure-and-set-environment-variables), [настройка конечной точки](service-fabric-get-started-containers.md#configure-communication), передача команд в контейнер, [импорт сертификата в контейнер](service-fabric-securing-containers.md)| 
|[Манифест службы BackEndService](#backendservice-service-manifest)|[Настройка переменных среды](service-fabric-get-started-containers.md#configure-and-set-environment-variables), [настройка конечной точки](service-fabric-get-started-containers.md#configure-communication), [настройка драйвера тома](service-fabric-containers-volume-logging-drivers.md)| 

Дополнительные сведения о конкретных XML-элементах см. в разделах [Элементы манифеста приложения](#application-manifest-elements), [Элементы манифеста службы FrontEndService](#frontendservice-service-manifest-elements) и [Элементы манифеста службы BackEndService](#backendservice-service-manifest-elements).

## <a name="application-manifest"></a>Манифест приложения

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="Container.ApplicationType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="BackEndService_InstanceCount" DefaultValue="-1" />
    <Parameter Name="FrontEndService_InstanceCount" DefaultValue="-1" />
    <Parameter Name="CpuCores" DefaultValue="2" />
    <Parameter Name="BlockIOWeight" DefaultValue="200" />
    <Parameter Name="MaximumIOBandwidth" DefaultValue="1024" />
    <Parameter Name="MemoryReservationInMB" DefaultValue="1024" />
    <Parameter Name="MemorySwapInMB" DefaultValue="4084"/>
    <Parameter Name="MaximumIOps" DefaultValue="20"/>
    <Parameter Name="MemoryFront" DefaultValue="4084" />
    <Parameter Name="MemoryBack" DefaultValue="2048" />
    <Parameter Name="CertThumbprint" DefaultValue=""/>
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="BackEndServicePkg" ServiceManifestVersion="1.0.0" />    
    
    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[MemoryFront]"/>

      <!-- Set resource governance at the code package level. -->
      <ResourceGovernancePolicy CodePackageRef="Code" CpuPercent="10" MemoryInMB="[MemoryFront]" BlockIOWeight="[BlockIOWeight]" MaximumIOBandwidth="[MaximumIOBandwidth]" MaximumIOps="[MaximumIOps]" MemoryReservationInMB="[MemoryReservationInMB]" MemorySwapInMB="[MemorySwapInMB]"/>
      
      <!-- Policies for activating container hosts. -->
      <ContainerHostPolicies CodePackageRef="Code" Isolation="process">
        
        <!-- Credentials for the repository hosting the container image.-->
        <RepositoryCredentials AccountName="sfsamples" Password="ENCRYPTED-PASSWORD" PasswordEncrypted="true"/>
        
        <!-- This binds the port the container is listening on (8905 in this sample) to an endpoint resource named "BackEndServiceTypeEndpoint", which is defined in the service manifest.  -->
        <PortBinding ContainerPort="8905" EndpointRef="BackEndServiceTypeEndpoint"/>
        
        <!-- Configure the Azure Files volume plugin.  Bind the source folder on the host VM or a remote share to the destination folder within the running container. -->
        <Volume Source="azfiles" Destination="c:\VolumeTest\Data" Driver="sfazurefile">
          <!-- Driver options to be passed to driver. The Azure Files volume plugin supports the following driver options:
            shareName (the Azure Files file share that provides the volume for the container), storageAccountName (the Azure storage account
            that contains the Azure Files file share), storageAccountKey (Access key for the Azure storage account that contains the Azure Files file share).
            These three driver options are required. -->
          <DriverOption Name="shareName" Value="" />
          <DriverOption Name="storageAccountName" Value="MY-STORAGE-ACCOUNT-NAME" />
          <DriverOption Name="storageAccountKey" Value="MY-STORAGE-ACCOUNT-KEY" />
        </Volume>
        
        <!-- Windows Server containers may not be compatible across different versions of the OS.  You can specify multiple OS images per container and tag 
        them with the build versions of the OS. Get the build version of the OS by running "winver" at a Windows command prompt. -->
        <ImageOverrides>
          <!-- If the underlying OS is build version 16299 (Windows Server version 1709), Service Fabric picks the container image tagged with Os="16299". -->
          <Image Name="sfsamples.azurecr.io/sfsamples/servicefabricbackendservice_1709" Os="16299" />
          
          <!-- An untagged container image is assumed to work across all versions of the OS and overrides the image specified in the service manifest. -->
          <Image Name="sfsamples.azurecr.io/sfsamples/servicefabricbackendservice_default" />          
        </ImageOverrides>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>

  <!-- Policies to be applied to the imported service manifest. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="FrontEndServicePkg" ServiceManifestVersion="1.0.0" />
    
    <!-- This enables you to provide different values for environment variables when creating a FrontEndService
         Theses environment variables are declared in the FrontEndServiceType service manifest-->
    <EnvironmentOverrides CodePackageRef="Code">
      <EnvironmentVariable Name="BackendServiceName" Value="Container.Application/BackEndService"/>
      <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
      <EnvironmentVariable Name="IsContainer" Value="true"/>
    </EnvironmentOverrides>
    
    <!-- This policy maps the  port of the container (80) to the endpoint declared in the service, 
         FrontEndServiceTypeEndpoint which is exposed as port 80 on the host-->    
    <Policies>

      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[MemoryBack]"/>

      <!-- Policies for activating container hosts. -->
      <ContainerHostPolicies CodePackageRef="Code" Isolation="process">

        <!-- Credentials for the repository hosting the container image.-->
        <RepositoryCredentials AccountName="sfsamples" Password="ENCRYPTED-PASSWORD" PasswordEncrypted="true"/>

        <!-- Binds an endpoint resource (declared in the service manifest) to the exposed container port. -->
        <PortBinding ContainerPort="80" EndpointRef="FrontEndServiceTypeEndpoint"/>

        <!-- Import a certificate into the container.  The certificate must be installed in the LocalMachine store of all the cluster nodes.
          When the application starts, the runtime reads the certificate and generates a PFX file and password (on Windows) or a PEM file (on Linux).
          The PFX file and password are accessible in the container using the Certificates_ServicePackageName_CodePackageName_CertName_PFX and 
          Certificates_ServicePackageName_CodePackageName_CertName_Password environment variables. The PEM file is accessible in the container using the 
          Certificates_ServicePackageName_CodePackageName_CertName_PEM and Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey environment variables.-->
        <CertificateRef Name="MyCert1" X509StoreName="My" X509FindValue="[CertThumbprint]" />

        <!-- If the certificate is already in PFX or PEM form, you can create a data package inside your application and reference that certificate here. -->
        <CertificateRef Name="MyCert2" DataPackageRef="Data" DataPackageVersion="1.0.0" RelativePath="MyCert2.PFX" Password="ENCRYPTED-PASSWORD" IsPasswordEncrypted="true"/>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>
  
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
        
    <Service Name="FrontEndService" >
      <StatelessService ServiceTypeName="FrontEndServiceType" InstanceCount="[FrontEndService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
        <Service Name="BackEndService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="BackEndServiceType" InstanceCount="[BackEndService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

## <a name="frontendservice-service-manifest"></a>Манифест службы FrontEndService

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="FrontEndServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="FrontEndServiceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ContainerHost>
        <!--The repo and image on https://hub.docker.com or Azure Container Registry. -->
        <ImageName>sfsamples.azurecr.io/sfsamples/servicefabricfrontendservice:v1</ImageName>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container or exe.  These variables are overridden in the application manifest. -->
    <EnvironmentVariables>
      <EnvironmentVariable Name="BackendServiceName" Value=""/>
      <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
      <EnvironmentVariable Name="IsContainer" Value=""/>
    </EnvironmentVariables>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />
  
  <!-- Data package is the contents of the Data directory under PackageRoot that contains an 
       independently-updateable and versioned static data that's consumed by the process at runtime. -->
  <DataPackage Name="Data" Version="1.0.0"/>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. For a guest executable is used to register with the NamingService at its REST endpoint
           with http scheme and port 80 -->
      <Endpoint Name="FrontEndServiceTypeEndpoint" UriScheme="http" Port="80"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="backendservice-service-manifest"></a>Манифест службы BackEndService

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="BackEndServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="BackEndServiceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ContainerHost>
        <!--The repo and image on https://hub.docker.com or Azure Container Registry. -->
        <ImageName>sfsamples.azurecr.io/sfsamples/servicefabricbackendservice:v1</ImageName>
        
        <!-- Pass comma delimited commands to your container. -->
        <Commands> dotnet, myproc.dll, 5 </Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container. These variables are overridden in the application manifest. -->
    <EnvironmentVariables>
      <EnvironmentVariable Name="IsContainer" Value="true"/>
    </EnvironmentVariables>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the host port on which to 
           listen. For a guest executable is used to register with the NamingService at its REST endpoint
           with http scheme. In this case since no port is specified, one is created and assigned dynamically
           to the service. This dynamically assigned host port is mapped to the container port (8905 in this sample),
            which was specified in the application manifest.-->
      <Endpoint Name="BackEndServiceTypeEndpoint" UriScheme="http" />
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

### <a name="policies-element"></a>Элемент Policies
Описывает политики (привязка конечной точки, совместный доступ к пакетам, запуск от имени и безопасный доступ) для применения импортированного манифеста службы. Дополнительные сведения см. в разделе об [элементе Policies](service-fabric-service-model-schema-elements.md#PoliciesElementServiceManifestImportPoliciesTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="servicepackageresourcegovernancepolicy-element"></a>Элемент ServicePackageResourceGovernancePolicy
Определяет политику управления ресурсами, которая применяется на уровне всего пакета службы. Дополнительные сведения см. в разделе [Элемент ServicePackageResourceGovernancePolicy](service-fabric-service-model-schema-elements.md#ServicePackageResourceGovernancePolicyElementServicePackageResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInServicePackageTypecomplexType).

### <a name="resourcegovernancepolicy-element"></a>Элемент ResourceGovernancePolicy
Указывает ограничения для ресурсов для пакета кода. Дополнительные сведения см. в разделе [Элемент ResourceGovernancePolicy](service-fabric-service-model-schema-elements.md#ResourceGovernancePolicyElementResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelementDefinedInDigestedEndpointelement).

### <a name="containerhostpolicies-element"></a>Элемент ContainerHostPolicies
Задает политики для активации узлов контейнеров. Дополнительные сведения см. в разделе [Элемент ContainerHostPolicies](service-fabric-service-model-schema-elements.md#ContainerHostPoliciesElementContainerHostPoliciesTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="repositorycredentials-element"></a>Элемент RepositoryCredentials
Учетные данные репозитория образов контейнеров для получения образов. Дополнительные сведения см. в разделе [Элемент RepositoryCredentials](service-fabric-service-model-schema-elements.md#RepositoryCredentialsElementRepositoryCredentialsTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="portbinding-element"></a>Элемент PortBinding
Указывает, какой ресурс конечной точки следует привязать к предоставленному порту контейнера. Дополнительные сведения см. в разделе [Элемент PortBinding](service-fabric-service-model-schema-elements.md#PortBindingElementPortBindingTypeComplexTypeDefinedInServicePackageContainerPolicyTypecomplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="volume-element"></a>Элемент Volume
Указывает том для привязки к контейнеру. Дополнительные сведения см. в разделе [Элемент Volume](service-fabric-service-model-schema-elements.md#VolumeElementContainerVolumeTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="driveroption-element"></a>Элемент DriverOption
Параметры драйвера, которые необходимо передать. Дополнительные сведения см. в разделе [Элемент DriverOption](service-fabric-service-model-schema-elements.md#DriverOptionElementDriverOptionTypeComplexTypeDefinedInContainerLoggingDriverTypecomplexTypeDefinedInContainerVolumeTypecomplexType).

### <a name="imageoverrides-element"></a>Элемент ImageOverrides
Контейнеры Windows Server могут быть несовместимы в разных версиях ОС.  Можно указать несколько образов операционной системы для контейнера и пометить их с помощью тега версии сборки операционной системы. Получить версию сборки операционной системы можно, выполнив в командной строке Windows команду winver. Если используется базовая ОС сборки 16299 (версия Windows Server 1709), Service Fabric выбирает образ контейнера с отметкой Os="16299". Предполагается, что образ контейнера без отметки будет работать во всех версиях операционной системы и он имеет приоритет над образом, указанным в манифесте службы. Дополнительные сведения см. в разделе [Элемент ImageOverrides](service-fabric-service-model-schema-elements.md#ImageOverridesElementImageOverridesTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="image-element"></a>Элемент Image
Образ контейнера, соответствующий номеру версии сборки ОС, которая будет запущена. Если атрибут Os не указан, предполагается, что образ контейнера работает во всех версиях операционной системы и он имеет приоритет над образом, указанным в манифесте службы. Дополнительные сведения см. в разделе [Элемент Image](service-fabric-service-model-schema-elements.md#ImageElementImageTypeComplexTypeDefinedInImageOverridesTypecomplexType).

### <a name="environmentoverrides-element"></a>Элемент EnvironmentOverrides
 Дополнительные сведения см. в разделе [Элемент EnvironmentOverrides](service-fabric-service-model-schema-elements.md#EnvironmentOverridesElementEnvironmentOverridesTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="environmentvariable-element"></a>Элемент EnvironmentVariable
Переменная среды. Дополнительные сведения см. в разделе [Элемент EnvironmentVariable](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableOverrideTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexType).

### <a name="certificateref-element"></a>Элемент CertificateRef
Указывает сведения о сертификате X509, который должен быть предоставлен в среде контейнера. Этот сертификат должен быть установлен в хранилище LocalMachine на всех узлах в кластере.
При запуске приложения в среде выполнения считываются сертификаты, создается файл PFX и пароль (в Windows) или файл PEM (в Linux).
Файл PFX и пароль, доступны в контейнере с помощью переменных среды Certificates_ServicePackageName_CodePackageName_CertName_PFX и Certificates_ServicePackageName_CodePackageName_CertName_Password. Файл PEM доступен в контейнере с помощью переменных среды Certificates_ServicePackageName_CodePackageName_CertName_PEM и Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey. Дополнительные сведения см. в разделе [Элемент CertificateRef](service-fabric-service-model-schema-elements.md#CertificateRefElementContainerCertificateTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="defaultservices-element"></a>Элемент DefaultServices
Декларирует экземпляры служб, которые создаются автоматически при создании экземпляра приложения в соответствии с его типом. Дополнительные сведения см. в разделе [Элемент DefaultServices](service-fabric-service-model-schema-elements.md#DefaultServicesElementDefaultServicesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInApplicationInstanceTypecomplexType).

### <a name="service-element"></a>Элемент Service
Объявляет службу, которая автоматически будет создана при создании экземпляра приложения. Дополнительные сведения см. в разделе [Элемент Service](service-fabric-service-model-schema-elements.md#ServiceElementanonymouscomplexTypeComplexTypeDefinedInDefaultServicesTypecomplexType).

### <a name="statelessservice-element"></a>Элемент StatelessService
Определяет службу без отслеживания состояния. Дополнительные сведения см. в разделе [Элемент StatelessService](service-fabric-service-model-schema-elements.md#StatelessServiceElementStatelessServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement).


## <a name="frontendservice-service-manifest-elements"></a>Элементы манифеста службы FrontEndService
### <a name="servicemanifest-element"></a>Элемент ServiceManifest
Декларативно описывает тип службы и его версию. В нем указан независимо обновляемый код, конфигурация и пакеты данных, из которых состоит пакет службы, для поддержки одного или нескольких типов служб. Указаны также ресурсы, параметры диагностики и метаданные службы, такие как тип службы, свойства работоспособности, а также метрики балансировки нагрузки. Дополнительные сведения см. в разделе [Элемент ServiceManifest](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType).

### <a name="servicetypes-element"></a>Элемент ServiceTypes
Указывает, какие типы служб поддерживаются пакетами CodePackage в этом манифесте. При создании экземпляра службы в соответствии с одним из этих типов службы все пакеты кода, объявленные в этом манифесте, активируются путем запуска соответствующих точек входа. Типы служб декларируются на уровне манифеста, а не на уровне пакета кода. Дополнительные сведения см. в разделе об [элементе ServiceTypes](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="statelessservicetype-element"></a>Элемент StatelessServiceType
Описывает тип службы без отслеживания состояния. Дополнительные сведения см. в разделе [Элемент StatelessServiceType](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType).

### <a name="codepackage-element"></a>Элемент CodePackage
Описывает пакеты кода, которые поддерживают тип определенной службы. При создании экземпляра службы в соответствии с одним из этих типов службы все пакеты кода, объявленные в этом манифесте, активируются путем запуска соответствующих точек входа. Запущенные вследствие этого процессы должны зарегистрировать поддерживаемые типы служб во время выполнения. При наличии нескольких пакетов кода они все активируются при поиске системой любого из задекларированных типов служб. Дополнительные сведения см. в разделе [Элемент CodePackage](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="entrypoint-element"></a>Элемент EntryPoint
Исполняемый файл, указанный в точке входа EntryPoint, обычно является узлом службы, запускаемым на длительный срок. Наличие отдельной точки входа настройки позволяет избежать необходимости в выполнении узла службы с расширенными правами в течение длительного срока. Исполняемый файл, указанный в точке входа EntryPoint, запускается после успешного выхода из точки SetupEntryPoint. Возникающий вследствие этого процесс отслеживается и перезапускается (снова начиная с точки входа SetupEntryPoint), даже если произошло непредвиденное завершение его работы или сбой. Дополнительные сведения см. в разделе [Элемент EntryPoint](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="containerhost-element"></a>Элемент ContainerHost
 Дополнительные сведения см. в разделе [Элемент ContainerHost](service-fabric-service-model-schema-elements.md#ContainerHostElementContainerHostEntryPointTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType).

### <a name="imagename-element"></a>Элемент ImageName
Репозиторий и образ в [https://hub.docker.com](https://hub.docker.com) или реестре контейнеров Azure. Дополнительные сведения см. в разделе [Элемент ImageName](service-fabric-service-model-schema-elements.md#ImageNameElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType).

### <a name="environmentvariables-element"></a>Элемент EnvironmentVariables
Передает переменные среды в контейнер или EXE-файл.  Дополнительные сведения см. в разделе [Элемент EnvironmentVariables](service-fabric-service-model-schema-elements.md#EnvironmentVariablesElementEnvironmentVariablesTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="environmentvariable-element"></a>Элемент EnvironmentVariable
Переменная среды. Дополнительные сведения см. в разделе [Элемент EnvironmentVariable](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableOverrideTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexType).

### <a name="configpackage-element"></a>Элемент ConfigPackage
Объявляет папку с именем, указанным в атрибуте Name, которая содержит файл Settings.xml. Этот файл содержит разделы заданных пользователем параметров пар "ключ-значение", которые могут считываться процессом во время выполнения. Во время обновления при изменении одного только атрибута version для ConfigPackage перезапуск процесса не выполняется. Вместо этого при помощи обратного вызова в процесс передается уведомление о том, что параметры конфигурации изменились, поэтому они были перезагружены в динамическом режиме. Дополнительные сведения см. в разделе [Элемент ConfigPackage](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement).

### <a name="datapackage-element"></a>Элемент DataPackage
Объявляет папку с именем, указанным в атрибуте Name, которая содержит файлы статических данных. Service Fabric будет перезапускать все EXE-файлы и DLLHOST-файлы, указанные в пакетах поддержки и размещения, когда обновляется любой из пакетов данных, перечисленных в манифесте службы. Дополнительные сведения см. в разделе [Элемент DataPackage](service-fabric-service-model-schema-elements.md#DataPackageElementDataPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedDataPackageelement).

### <a name="resources-element"></a>Элемент Resources
Описывает ресурсы, используемые этой службой, которые могут быть объявлены без изменения скомпилированного кода и изменены при развертывании службы. Доступ к этим ресурсам осуществляется с помощью разделов Principals и Policies манифеста приложения. Дополнительные сведения см. в разделе [Элемент Resources](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="endpoints-element"></a>Элемент Endpoints
Определяет конечные точки службы. Дополнительные сведения см. в разделе об [элементе Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType).

### <a name="endpoint-element"></a>Элемент Endpoint
Дополнительные сведения см. в разделе [Элемент Endpoint (тип EndpointOverrideType)](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).


## <a name="backendservice-service-manifest-elements"></a>Элементы манифеста службы BackEndService
### <a name="servicemanifest-element"></a>Элемент ServiceManifest
Декларативно описывает тип службы и его версию. В нем указан независимо обновляемый код, конфигурация и пакеты данных, из которых состоит пакет службы, для поддержки одного или нескольких типов служб. Указаны также ресурсы, параметры диагностики и метаданные службы, такие как тип службы, свойства работоспособности, а также метрики балансировки нагрузки. Дополнительные сведения см. в разделе [Элемент ServiceManifest](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType).

### <a name="servicetypes-element"></a>Элемент ServiceTypes
Указывает, какие типы служб поддерживаются пакетами CodePackage в этом манифесте. При создании экземпляра службы в соответствии с одним из этих типов службы все пакеты кода, объявленные в этом манифесте, активируются путем запуска соответствующих точек входа. Типы служб декларируются на уровне манифеста, а не на уровне пакета кода. Дополнительные сведения см. в разделе об [элементе ServiceTypes](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="statelessservicetype-element"></a>Элемент StatelessServiceType
Описывает тип службы без отслеживания состояния. Дополнительные сведения см. в разделе [Элемент StatelessServiceType](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType).

### <a name="codepackage-element"></a>Элемент CodePackage
Описывает пакеты кода, которые поддерживают тип определенной службы. При создании экземпляра службы в соответствии с одним из этих типов службы все пакеты кода, объявленные в этом манифесте, активируются путем запуска соответствующих точек входа. Запущенные вследствие этого процессы должны зарегистрировать поддерживаемые типы служб во время выполнения. При наличии нескольких пакетов кода они все активируются при поиске системой любого из задекларированных типов служб. Дополнительные сведения см. в разделе [Элемент CodePackage](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="entrypoint-element"></a>Элемент EntryPoint
Исполняемый файл, указанный в точке входа EntryPoint, обычно является узлом службы, запускаемым на длительный срок. Наличие отдельной точки входа настройки позволяет избежать необходимости в выполнении узла службы с расширенными правами в течение длительного срока. Исполняемый файл, указанный в точке входа EntryPoint, запускается после успешного выхода из точки SetupEntryPoint. Возникающий вследствие этого процесс отслеживается и перезапускается (снова начиная с точки входа SetupEntryPoint), даже если произошло непредвиденное завершение его работы или сбой. Дополнительные сведения см. в разделе [Элемент EntryPoint](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="containerhost-element"></a>Элемент ContainerHost
Дополнительные сведения см. в разделе [Элемент ContainerHost](service-fabric-service-model-schema-elements.md#ContainerHostElementContainerHostEntryPointTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType).

### <a name="imagename-element"></a>Элемент ImageName
Репозиторий и образ в [https://hub.docker.com](https://hub.docker.com) или реестре контейнеров Azure. Дополнительные сведения см. в разделе [Элемент ImageName](service-fabric-service-model-schema-elements.md#ImageNameElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType).

### <a name="commands-element"></a>Элемент Commands
Передает разделенный запятыми список команд контейнеру. Дополнительные сведения см. в разделе [Элемент Commands](service-fabric-service-model-schema-elements.md#CommandsElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType).

### <a name="environmentvariables-element"></a>Элемент EnvironmentVariables
Передает переменные среды в контейнер или EXE-файл.  Дополнительные сведения см. в разделе [Элемент EnvironmentVariables](service-fabric-service-model-schema-elements.md#EnvironmentVariablesElementEnvironmentVariablesTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="environmentvariable-element"></a>Элемент EnvironmentVariable
Переменная среды. Дополнительные сведения см. в разделе [Элемент EnvironmentVariable](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableOverrideTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexType).

### <a name="configpackage-element"></a>Элемент ConfigPackage
Объявляет папку с именем, указанным в атрибуте Name, которая содержит файл Settings.xml. Этот файл содержит разделы заданных пользователем параметров пар "ключ-значение", которые могут считываться процессом во время выполнения. Во время обновления при изменении одного только атрибута version для ConfigPackage перезапуск процесса не выполняется. Вместо этого при помощи обратного вызова в процесс передается уведомление о том, что параметры конфигурации изменились, поэтому они были перезагружены в динамическом режиме. Дополнительные сведения см. в разделе [Элемент ConfigPackage](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement).

### <a name="resources-element"></a>Элемент Resources
Описывает ресурсы, используемые этой службой, которые могут быть объявлены без изменения скомпилированного кода и изменены при развертывании службы. Доступ к этим ресурсам осуществляется с помощью разделов Principals и Policies манифеста приложения. Дополнительные сведения см. в разделе [Элемент Resources](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="endpoints-element"></a>Элемент Endpoints
Определяет конечные точки службы. Дополнительные сведения см. в разделе об [элементе Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType).

### <a name="endpoint-element"></a>Элемент Endpoint
 Дополнительные сведения см. в разделе [Элемент Endpoint (тип EndpointOverrideType)](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).

