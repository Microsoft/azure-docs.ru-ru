---
title: 'Безопасная связь службы удаленного взаимодействия с C #'
description: Узнайте, как защитить удаленное взаимодействие со службой для служб Reliable Services на C#, запущенных в кластере Azure Service Fabric.
author: suchiagicha
ms.topic: conceptual
ms.date: 04/20/2017
ms.author: pepogors
ms.openlocfilehash: ba68df53f1f21b9ff360772fe1a60c93c8df74d3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86252221"
---
# <a name="secure-service-remoting-communications-in-a-c-service"></a>Безопасное удаленное взаимодействие со службой для службы C#
> [!div class="op_single_selector"]
> * [C# в Windows](service-fabric-reliable-services-secure-communication.md)
> * [Java в Linux](service-fabric-reliable-services-secure-communication-java.md)
>
>

Безопасность — один из самых важных аспектов взаимодействия. Платформа приложений Reliable Services предоставляет несколько готовых стеков взаимодействия и средств, которыми можно воспользоваться для повышения безопасности. В этой статье объясняется, как повысить безопасность при использовании удаленного взаимодействия со службой для службы C#. Мы будем использовать существующий [пример](service-fabric-reliable-services-communication-remoting.md), в котором описывается настройка удаленного взаимодействия для Reliable Services, написанных на C#. 

Для защиты службы при использовании удаленного взаимодействия со службами C# выполните следующие действия:

1. Создайте интерфейс `IHelloWorldStateful`, определяющий методы, которые будут доступны для удаленного вызова процедур в службе. Служба также будет использовать прослушиватель `FabricTransportServiceRemotingListener`, который объявляется в пространстве имен `Microsoft.ServiceFabric.Services.Remoting.FabricTransport.Runtime`. Это реализация `ICommunicationListener` , которая предоставляет возможности удаленного взаимодействия.

    ```csharp
    public interface IHelloWorldStateful : IService
    {
        Task<string> GetHelloWorld();
    }

    internal class HelloWorldStateful : StatefulService, IHelloWorldStateful
    {
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]{
                    new ServiceReplicaListener(
                        (context) => new FabricTransportServiceRemotingListener(context,this))};
        }

        public Task<string> GetHelloWorld()
        {
            return Task.FromResult("Hello World!");
        }
    }
    ```
2. Добавьте параметры прослушивателя и учетные данные безопасности.

    Убедитесь, что сертификат, который вы хотите использовать для защиты взаимодействия со службой, установлен на всех узлах в кластере. 
    
    > [!NOTE]
    > На узлах Linux сертификат должен представлять собой PEM-файлы в каталоге */var/lib/sfcerts*. Дополнительные сведения см. в разделе [Расположение и формат сертификатов X.509 в узлах Linux](./service-fabric-configure-certificates-linux.md#location-and-format-of-x509-certificates-on-linux-nodes). 

    Существует два способа указания параметров прослушивателя и учетных данных безопасности:

   1. Предоставьте их непосредственно в коде службы:

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           FabricTransportRemotingListenerSettings  listenerSettings = new FabricTransportRemotingListenerSettings
           {
               MaxMessageSize = 10000000,
               SecurityCredentials = GetSecurityCredentials()
           };
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(context,this,listenerSettings))
           };
       }

       private static SecurityCredentials GetSecurityCredentials()
       {
           // Provide certificate details.
           var x509Credentials = new X509Credentials
           {
               FindType = X509FindType.FindByThumbprint,
               FindValue = "4FEF3950642138446CC364A396E1E881DB76B48C",
               StoreLocation = StoreLocation.LocalMachine,
               StoreName = "My",
               ProtectionLevel = ProtectionLevel.EncryptAndSign
           };
           x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
           x509Credentials.RemoteCertThumbprints.Add("9FEF3950642138446CC364A396E1E881DB76B483");
           return x509Credentials;
       }
       ```
   2. Предоставьте их с помощью [пакета конфигурации](service-fabric-application-and-service-manifests.md):

       Добавьте именованный раздел `TransportSettings` в файл settings.xml.

       ```xml
       <Section Name="HelloWorldStatefulTransportSettings">
           <Parameter Name="MaxMessageSize" Value="10000000" />
           <Parameter Name="SecurityCredentialsType" Value="X509" />
           <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
           <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
           <Parameter Name="CertificateRemoteThumbprints" Value="9FEF3950642138446CC364A396E1E881DB76B483" />
           <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
           <Parameter Name="CertificateStoreName" Value="My" />
           <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
           <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
       </Section>
       ```

       В этом случае метод `CreateServiceReplicaListeners` будет выглядеть следующим образом:

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(
                       context,this,FabricTransportRemotingListenerSettings .LoadFrom("HelloWorldStatefulTransportSettings")))
           };
       }
       ```

        При добавлении раздела `TransportSettings` в файл settings.xml `FabricTransportRemotingListenerSettings` по умолчанию загрузит все параметры из этого раздела.

        ```xml
        <!--"TransportSettings" section .-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        В этом случае метод `CreateServiceReplicaListeners` будет выглядеть следующим образом:

        ```csharp
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]
            {
                return new[]{
                        new ServiceReplicaListener(
                            (context) => new FabricTransportServiceRemotingListener(context,this))};
            };
        }
        ```
3. При вызове методов для защищенной службы с помощью стека удаленного взаимодействия вместо использования класса `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` для создания прокси-службы используйте `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxyFactory`. Передайте параметр `FabricTransportRemotingSettings`, который содержит объект `SecurityCredentials`.

    ```csharp

    var x509Credentials = new X509Credentials
    {
        FindType = X509FindType.FindByThumbprint,
        FindValue = "9FEF3950642138446CC364A396E1E881DB76B483",
        StoreLocation = StoreLocation.LocalMachine,
        StoreName = "My",
        ProtectionLevel = ProtectionLevel.EncryptAndSign
    };
    x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
    x509Credentials.RemoteCertThumbprints.Add("4FEF3950642138446CC364A396E1E881DB76B48C");

    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        SecurityCredentials = x509Credentials,
    };

    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(transportSettings));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    Если клиентский код выполняется как часть службы, можно загрузить `FabricTransportRemotingSettings` из файла settings.xml. Создайте раздел HelloWorldClientTransportSettings по аналогии с кодом службы, как показано выше. Внесите следующие изменения в клиентский код:

    ```csharp
    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.LoadFrom("HelloWorldClientTransportSettings")));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    Если клиент не выполняется как часть службы, можно создать файл client_name.settings.xml в том же каталоге, в котором находится файл client_name.exe. Затем создайте раздел TransportSettings в этом файле.

    Как и для службы, если в файл клиента settings.xml/client_name.settings.xml добавить раздел `TransportSettings`, то `FabricTransportRemotingSettings` по умолчанию загрузит все параметры из этого раздела.

    В этом случае приведенный выше код еще более упрощается.  

    ```csharp

    IHelloWorldStateful client = ServiceProxy.Create<IHelloWorldStateful>(
                 new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```


Теперь ознакомьтесь со статьей [Веб-API с OWIN в модели Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md).
