---
title: Указание конечных точек службы Service Fabric
description: В этой статье поясняется, как описать ресурсы конечной точки в манифесте служб, включая настройку конечных точек HTTPS.
ms.topic: conceptual
ms.date: 09/16/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: 0ed5a4aa8993f52d42b97288cd143e6114ff36ff
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97033312"
---
# <a name="specify-resources-in-a-service-manifest"></a>Указание ресурсов в манифесте службы
## <a name="overview"></a>Обзор
Service Fabric приложения и службы определяются, а управление версиями осуществляется с помощью файлов манифеста. Обзор ServiceManifest.xml и ApplicationManifest.xml более высокого уровня см. в разделе [Service Fabric манифесты приложения и службы](service-fabric-application-and-service-manifests.md).

Манифест службы позволяет объявлять или изменять ресурсы, используемые службой, без изменения скомпилированного кода. Service Fabric поддерживает настройку ресурсов конечных точек для службы. Доступ к ресурсам, указанным в манифесте служб, можно контролировать в манифесте приложения с помощью элемента SecurityGroup. Объявление ресурсов позволяет изменять их при развертывании, т. е. службе не нужно внедрять новый механизм настройки. Определение схемы для файла ServiceManifest.xml устанавливается вместе с Service Fabricным пакетом SDK и инструментами для *C:\Program Files\Microsoft SDKs\Service Fabric\schemas\ServiceFabricServiceModel.xsd* и описывается в [документации по схеме ServiceFabricServiceModel. xsd](service-fabric-service-model-schema.md).

## <a name="endpoints"></a>Конечные точки
Если ресурс конечной точки определен в манифесте службы, Service Fabric назначает порты из диапазона зарезервированных портов приложений, если порт не указан явным образом. Например, рассмотрим конечную точку *ServiceEndpoint1* , которая указана во фрагменте кода манифеста, приведенном после абзаца. Кроме того, службы также могут запрашивать наличие в ресурсе конкретного порта. Репликам службы, которые выполняются на различных узлах кластера, можно назначить разные номера портов, а реплики службы, выполняющиеся на одном и том же узле, будут совместно используют один порт. Реплики службы при необходимости могут использовать эти порты для репликации и прослушивания клиентских запросов.

При активации службы, указывающей конечную точку HTTPS, Service Fabric установит запись управления доступом для порта, привязать указанный сертификат сервера к порту, а также предоставит удостоверение, которое служба выполняет как разрешения для закрытого ключа сертификата. Поток активации вызывается при каждом запуске Service Fabric или при изменении объявления сертификата приложения с помощью обновления. Сертификат конечной точки также будет отслеживаться для изменений и продлений, а разрешения будут периодически повторно применены по мере необходимости.

После завершения работы службы Service Fabric очистит запись управления доступом к конечной точке и удалит привязку к сертификату. Однако все разрешения, примененные к закрытому ключу сертификата, не будут очищены.

> [!WARNING] 
> При проектировании статические порты не должны пересекаться с диапазоном портов приложений, указанным в ClusterManifest. Если указан статический порт, назначьте его за пределами диапазона портов приложения, в противном случае это приведет к конфликтам портов. При использовании выпуска 6.5 CU2 мы выпустим **предупреждение о работоспособности** , когда обнаружите такой конфликт, но разрешите, что развертывание продолжит синхронизироваться с поставленным поведением 6,5. Однако мы можем не допустить развертывания приложения из следующих основных выпусков.
>
> В выпуске 7,0 мы выпустим **предупреждение о работоспособности** при обнаружении использования диапазона портов приложения сверх хостингконфиг:: аппликатионпортексхаустсрешолдперцентаже (по умолчанию 80%).
>

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="ServiceEndpoint1" Protocol="http"/>
    <Endpoint Name="ServiceEndpoint2" Protocol="http" Port="80"/>
    <Endpoint Name="ServiceEndpoint3" Protocol="https"/>
  </Endpoints>
</Resources>
```

Если в одном пакете службы несколько пакетов кода, на пакет кода также нужно добавить ссылку в разделе **Конечные точки**.  Например, если **ServiceEndpoint2a** и **ServiceEndpoint2b** являются конечными точками из одного и того же пакета службы и ссылаются на разные пакеты кода, пакет кода, который соответствует каждой конечной точке, определяется, как показано ниже:

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="ServiceEndpoint2a" Protocol="http" Port="802" CodePackageRef="Code1"/>
    <Endpoint Name="ServiceEndpoint2b" Protocol="http" Port="801" CodePackageRef="Code2"/>
  </Endpoints>
</Resources>
```

Дополнительные сведения о создании ссылок на конечные точки из файла параметров пакета конфигурации (settings.xml) см. в статье [Настройка надежных служб с отслеживанием состояния](service-fabric-reliable-services-configuration.md).

## <a name="example-specifying-an-http-endpoint-for-your-service"></a>Пример. Указание конечной точки HTTP для службы
Следующий манифест служб в элементе &lt;Resources&gt; определяет один ресурс конечной точки TCP и два ресурса конечной точки HTTP.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="Stateful1Pkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         This name must match the string used in the RegisterServiceType call in Program.cs. -->
    <StatefulServiceType ServiceTypeName="Stateful1Type" HasPersistedState="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>Stateful1.exe</Program>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an
       independently updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port number on which to
           listen. Note that if your service is partitioned, this port is shared with
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="ServiceEndpoint1" Protocol="http"/>
      <Endpoint Name="ServiceEndpoint2" Protocol="http" Port="80"/>
      <Endpoint Name="ServiceEndpoint3" Protocol="https"/>
      <Endpoint Name="ServiceEndpoint4" Protocol="https" Port="14023"/>

      <!-- This endpoint is used by the replicator for replicating the state of your service.
           This endpoint is configured through the ReplicatorSettings config section in the Settings.xml
           file under the ConfigPackage. -->
      <Endpoint Name="ReplicatorEndpoint" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="example-specifying-an-https-endpoint-for-your-service"></a>Пример. Указание конечной точки HTTPS для службы
Протокол HTTPS обеспечивает аутентификацию сервера, а также используется для шифрования данных, передаваемых между клиентом сервером. Чтобы включить протокол HTTPS в службе Service Fabric, укажите его в разделе *Ресурсы > Конечные точки > Конечная точка* манифеста служб, как показано выше для конечной точки *ServiceEndpoint3*.

> [!NOTE]
> Протокол службы невозможно изменить при обновлении приложения. Если изменить его во время обновления, то это будет считаться критическим изменением.
> 

> [!WARNING] 
> При использовании HTTPS не используйте тот же порт и сертификат для разных экземпляров службы (независимо от приложения), развернутых на одном узле. Обновление двух разных служб, использующих один порт в разных экземплярах приложения, приведет к сбою обновлений. Дополнительные сведения. см в статье [Обновление нескольких приложений с помощью конечных точек HTTPS](service-fabric-application-upgrade.md#upgrading-multiple-applications-with-https-endpoints).
>

Ниже приведен пример ApplicationManifest, демонстрирующий конфигурацию, необходимую для конечной точки HTTPS. Сертификат сервера или конечной точки может быть объявлен по отпечатку или общему имени субъекта, и должно быть указано значение. Ендпоинтреф — это ссылка на Ендпоинтресаурце в ServiceManifest, протокол которой должен быть установлен в протокол HTTPS. Можно добавить несколько элементов Endpointcertificate.  

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="Application1Type"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="Stateful1_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="Stateful1_PartitionCount" DefaultValue="1" />
    <Parameter Name="Stateful1_TargetReplicaSetSize" DefaultValue="3" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion
       should match the Name and Version attributes of the ServiceManifest element defined in the
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <EndpointBindingPolicy CertificateRef="SslCertByTP" EndpointRef="ServiceEndpoint3"/>
      <EndpointBindingPolicy CertificateRef="SslCertByCN" EndpointRef="ServiceEndpoint4"/>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types when an instance of this
         application type is created. You can also create one or more instances of service type by using the
         Service Fabric PowerShell module.

         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="Stateful1">
      <StatefulService ServiceTypeName="Stateful1Type" TargetReplicaSetSize="[Stateful1_TargetReplicaSetSize]" MinReplicaSetSize="[Stateful1_ ]">
        <UniformInt64Partition PartitionCount="[Stateful1_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
      </StatefulService>
    </Service>
  </DefaultServices>
  <Certificates>
    <EndpointCertificate Name="SslCertByTP" X509FindValue="FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF F0" X509StoreName="MY" />  
    <EndpointCertificate Name="SslCertByCN" X509FindType="FindBySubjectName" X509FindValue="ServiceFabric-EndpointCertificateBinding-Test" X509StoreName="MY" />  
  </Certificates>
</ApplicationManifest>
```

Для кластеров Linux **MY** сохраняет значения по умолчанию в папку **/var/lib/sfcerts**.

Пример полного приложения, использующего конечную точку HTTPS, см. [в разделе Добавление конечной точки HTTPS в интерфейсную службу ASP.NET Core интерфейса веб-API с помощью Kestrel](./service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#define-an-https-endpoint-in-the-service-manifest).

## <a name="port-acling-for-http-endpoints"></a>Порт адресам для конечных точек HTTP
Service Fabric будет автоматически указывать конечные точки HTTP (S) ACL, указанные по умолчанию. Он **не** будет выполнять автоматическое адресам, если с конечной точкой не связана [SecurityAccessPolicy](service-fabric-assign-policy-to-endpoint.md) , а Service Fabric настроен для запуска с использованием учетной записи с правами администратора.

## <a name="overriding-endpoints-in-servicemanifestxml"></a>Переопределение конечных точек в файле ServiceManifest.xml

В ApplicationManifest добавьте раздел ResourceOverrides, который будет разноситься в одном уровне с разделом ConfigOverrides. В этом разделе можно задать параметры переопределения для раздела конечных точек в разделе ресурсов, указанном в манифесте служб. Переопределение конечных точек поддерживается в среде выполнения 5.7.217 и пакете SDK 2.7.217 и более поздних версий.

Чтобы переопределить конечную точку в ServiceManifest с помощью ApplicationParameters, измените ApplicationManifest следующим образом:

Добавьте новый подраздел ResourceOverrides в раздел ServiceManifestImport.

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <ResourceOverrides>
      <Endpoints>
        <Endpoint Name="ServiceEndpoint" Port="[Port]" Protocol="[Protocol]" Type="[Type]" />
        <Endpoint Name="ServiceEndpoint1" Port="[Port1]" Protocol="[Protocol1] "/>
      </Endpoints>
    </ResourceOverrides>
        <Policies>
           <EndpointBindingPolicy CertificateRef="SslCertByTP" EndpointRef="ServiceEndpoint"/>
        </Policies>
  </ServiceManifestImport>
```

В раздел Parameters добавьте следующее:

```xml
  <Parameters>
    <Parameter Name="Port" DefaultValue="" />
    <Parameter Name="Protocol" DefaultValue="" />
    <Parameter Name="Type" DefaultValue="" />
    <Parameter Name="Port1" DefaultValue="" />
    <Parameter Name="Protocol1" DefaultValue="" />
  </Parameters>
```

При развертывании приложения можно передать эти значения как ApplicationParameters.  Пример:

```powershell
PS C:\> New-ServiceFabricApplication -ApplicationName fabric:/myapp -ApplicationTypeName "AppType" -ApplicationTypeVersion "1.0.0" -ApplicationParameter @{Port='1001'; Protocol='https'; Type='Input'; Port1='2001'; Protocol='http'}
```

Примечание. Если значение, указанное для данного Аппликатионпараметер, является пустым, мы вернемся к значению по умолчанию, указанному в ServiceManifest для соответствующего элемента EndPointName.

Пример:

Допустим, в ServiceManifest заданы следующие значения:

```xml
  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpoint1" Protocol="tcp"/>
    </Endpoints>
  </Resources>
```

Предположим, что значение Port1 и Protocol1 для параметров приложения равно null или пусто. Порт будет принят ServiceFabric, а протокол будет TCP.

Предположим, что вы задали неверное значение. Скажем, для Port вы указали строковое значение "foo" вместо int.  New-ServiceFabricApplication команда завершится ошибкой: `The override parameter with name 'ServiceEndpoint1' attribute 'Port1' in section 'ResourceOverrides' is invalid. The value specified is 'Foo' and required is 'int'.`

## <a name="next-steps"></a>Дальнейшие действия

В этой статье объясняется, как определить конечные точки в манифесте службы Service Fabric. Более подробные примеры см. в следующих статьях:

> [!div class="nextstepaction"]
> [Примеры манифестов приложений и служб](service-fabric-manifest-examples.md)

Пошаговое руководство по упаковке и развертыванию существующего приложения в кластере Service Fabric см. в следующих статьях:

> [!div class="nextstepaction"]
> [Упаковка и развертывание существующего исполняемого файла вручную](service-fabric-deploy-existing-app.md)