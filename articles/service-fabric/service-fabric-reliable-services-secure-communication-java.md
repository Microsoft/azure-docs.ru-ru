---
title: Безопасная связь удаленного взаимодействия со службой Java
description: Узнайте, как защитить удаленное взаимодействие со службой для служб Reliable Services на Java, запущенных в кластере Azure Service Fabric.
author: PavanKunapareddyMSFT
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-java
ms.author: pakunapa
ms.openlocfilehash: 01a64fbcfef9f56abb0e1aa6cf7f5d821dd3763b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87325718"
---
# <a name="secure-service-remoting-communications-in-a-java-service"></a>Безопасное удаленное взаимодействие со службой для службы Java
> [!div class="op_single_selector"]
> * [C# в Windows](service-fabric-reliable-services-secure-communication.md)
> * [Java в Linux](service-fabric-reliable-services-secure-communication-java.md)
>
>

Безопасность — один из самых важных аспектов взаимодействия. Платформа приложений Reliable Services предоставляет несколько готовых стеков взаимодействия и средств, которыми можно воспользоваться для повышения безопасности. В этой статье объясняется, как повысить безопасность при использовании удаленного взаимодействия со службой для службы Java. Мы будем использовать существующий [пример](service-fabric-reliable-services-communication-remoting-java.md), в котором описывается настройка удаленного взаимодействия для Reliable Services, написанных на Java. 

Для защиты службы при использовании удаленного взаимодействия со службами Java выполните следующие действия:

1. Создайте интерфейс `HelloWorldStateless`, определяющий методы, которые будут доступны для удаленного вызова процедур в службе. Служба также будет использовать прослушиватель `FabricTransportServiceRemotingListener`, который объявляется в пакете `microsoft.serviceFabric.services.remoting.fabricTransport.runtime`. Это реализация `CommunicationListener` , которая предоставляет возможности удаленного взаимодействия.

    ```java
    public interface HelloWorldStateless extends Service {
        CompletableFuture<String> getHelloWorld();
    }

    class HelloWorldStatelessImpl extends StatelessService implements HelloWorldStateless {
        @Override
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this);
            }));
        return listeners;
        }

        public CompletableFuture<String> getHelloWorld() {
            return CompletableFuture.completedFuture("Hello World!");
        }
    }
    ```
2. Добавьте параметры прослушивателя и учетные данные безопасности.

    Убедитесь, что сертификат, который вы хотите использовать для защиты взаимодействия со службой, установлен на всех узлах в кластере. Для служб, выполняемых на Linux, сертификат должен находиться в PEM-файле. Это может быть файл `.pem` с сертификатом и закрытым ключом или файл `.crt` с сертификатом и файл `.key` с закрытым ключом. Дополнительные сведения см. в разделе [Расположение и формат сертификатов X.509 в узлах Linux](./service-fabric-configure-certificates-linux.md#location-and-format-of-x509-certificates-on-linux-nodes).
    
    Существует два способа указания параметров прослушивателя и учетных данных безопасности:

   1. Предоставьте их с помощью [пакета конфигурации](service-fabric-application-and-service-manifests.md):

       Добавьте именованный раздел `TransportSettings` в файл settings.xml.

       ```xml
       <!--Section name should always end with "TransportSettings".-->
       <!--Here we are using a prefix "HelloWorldStateless".-->
        <Section Name="HelloWorldStatelessTransportSettings">
            <Parameter Name="MaxMessageSize" Value="10000000" />
            <Parameter Name="SecurityCredentialsType" Value="X509_2" />
            <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
            <Parameter Name="CertificateProtectionLevel" Value="EncryptandSign" />
            <Parameter Name="CertificateRemoteThumbprints" Value="BD1C71E248B8C6834C151174DECDBDC02DE1D954" />
        </Section>

       ```

       В этом случае метод `createServiceInstanceListeners` будет выглядеть следующим образом:

       ```java
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this, FabricTransportRemotingListenerSettings.loadFrom(HelloWorldStatelessTransportSettings));
            }));
            return listeners;
        }
       ```

        При добавлении раздела `TransportSettings` в файл settings.xml без префикса `FabricTransportListenerSettings` по умолчанию загрузит все параметры из этого раздела.

        ```xml
        <!--"TransportSettings" section without any prefix.-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        В этом случае метод `CreateServiceInstanceListeners` будет выглядеть следующим образом:

        ```java
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this);
            }));
            return listeners;
        }
       ```
3. При вызове методов для защищенной службы с помощью стека удаленного взаимодействия вместо использования класса `microsoft.serviceFabric.services.remoting.client.ServiceProxyBase` для создания прокси-службы используйте `microsoft.serviceFabric.services.remoting.client.FabricServiceProxyFactory`.

    Если клиентский код выполняется как часть службы, можно загрузить `FabricTransportSettings` из файла settings.xml. Создайте раздел TransportSettings по аналогии с кодом службы, как показано выше. Внесите следующие изменения в клиентский код:

    ```java

    FabricServiceProxyFactory serviceProxyFactory = new FabricServiceProxyFactory(c -> {
            return new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.loadFrom("TransportPrefixTransportSettings"), null, null, null, null);
        }, null)

    HelloWorldStateless client = serviceProxyFactory.createServiceProxy(HelloWorldStateless.class,
        new URI("fabric:/MyApplication/MyHelloWorldService"));

    CompletableFuture<String> message = client.getHelloWorld();

    ```
