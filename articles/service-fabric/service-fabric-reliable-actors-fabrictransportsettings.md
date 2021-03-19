---
title: Изменить параметры FabricTransport
description: Узнайте, как настроить параметры связи с Service Fabric субъекта Azure для различных конфигураций субъектов.
author: suchiagicha
ms.topic: conceptual
ms.date: 04/20/2017
ms.author: pepogors
ms.custom: devx-track-csharp
ms.openlocfilehash: e3c20d86bd29d60eca328a44ab5d5d600bbf4da4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89010942"
---
# <a name="configure-fabrictransport-settings-for-reliable-actors"></a>Настройка параметров FabricTransport для Reliable Actors

Ниже приведены параметры, которые можно настраивать.
- C#: [FabricTransportRemotingSettings](/java/api/microsoft.servicefabric.services.remoting.fabrictransport.fabrictransportremotingsettings)
- Java: [FabricTransportRemotingSettings](/java/api/microsoft.servicefabric.services.remoting.fabrictransport.fabrictransportremotingsettings).

Конфигурацию FabricTransport по умолчанию можно изменить одним из следующих способов.

## <a name="assembly-attribute"></a>Атрибут сборки

Атрибут [FabricTransportActorRemotingProvider](/dotnet/api/microsoft.servicefabric.actors.remoting.fabrictransport.fabrictransportactorremotingproviderattribute) необходимо применить в клиенте субъекта и в сборках службы субъектов.

В следующем примере показано, как изменить значение по умолчанию параметра OperationTimeout в конфигурации FabricTransport.

  ```csharp
    using Microsoft.ServiceFabric.Actors.Remoting.FabricTransport;
    [assembly:FabricTransportActorRemotingProvider(OperationTimeoutInSeconds = 600)]
   ```

   Во втором примере показано, как изменить значения по умолчанию параметров MaxMessageSize и OperationTimeoutInSeconds в конфигурации FabricTransport.

  ```csharp
    using Microsoft.ServiceFabric.Actors.Remoting.FabricTransport;
    [assembly:FabricTransportActorRemotingProvider(OperationTimeoutInSeconds = 600,MaxMessageSize = 134217728)]
   ```

## <a name="config-package"></a>Пакет конфигурации

Для изменения конфигурации по умолчанию можно использовать [пакет конфигурации](service-fabric-application-and-service-manifests.md).

> [!IMPORTANT]
> На узлах Linux сертификаты должны иметь формат PEM. Дополнительные сведения о поиске и настройке сертификатов для Linux см. в разделе [Настройка сертификатов на Linux](./service-fabric-configure-certificates-linux.md). 
> 

### <a name="configure-fabrictransport-settings-for-the-actor-service"></a>Настройка параметров FabricTransport для службы субъектов

Добавьте раздел TransportSettings в файл settings.xml.

По умолчанию код субъекта для параметра SectionName выглядит следующим образом: &lt;имя_субъекта&gt;TransportSettings. Если соответствующее значение не найдено, выполняется поиск значения TransportSettings.

  ```xml
  <Section Name="MyActorServiceTransportSettings">
       <Parameter Name="MaxMessageSize" Value="10000000" />
       <Parameter Name="OperationTimeoutInSeconds" Value="300" />
       <Parameter Name="SecurityCredentialsType" Value="X509" />
       <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
       <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
       <Parameter Name="CertificateRemoteThumbprints" Value="b3449b018d0f6839a2c5d62b5b6c6ac822b6f662" />
       <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
       <Parameter Name="CertificateStoreName" Value="My" />
       <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
       <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
   </Section>
  ```

### <a name="configure-fabrictransport-settings-for-the-actor-client-assembly"></a>Настройка параметров FabricTransport для сборки службы субъектов

Если клиент не выполняется как часть службы, можно создать файл &lt;имя_EXE-файла_клиента&gt;.settings.xml в том же каталоге, в котором находится EXE-файл клиента. Затем добавьте раздел TransportSettings в этом файле. Для параметра SectionName задайте значение TransportSettings.

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
    <Section Name="TransportSettings">
      <Parameter Name="SecurityCredentialsType" Value="X509" />
      <Parameter Name="OperationTimeoutInSeconds" Value="300" />
      <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
      <Parameter Name="CertificateFindValue" Value="b3449b018d0f6839a2c5d62b5b6c6ac822b6f662" />
      <Parameter Name="CertificateRemoteThumbprints" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
      <Parameter Name="OperationTimeoutInSeconds" Value="300" />
      <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
      <Parameter Name="CertificateStoreName" Value="My" />
      <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
      <Parameter Name="CertificateRemoteCommonNames" Value="WinFabric-Test-SAN1-Alice" />
    </Section>
  </Settings>
   ```

* Настройка параметров FabricTransport для защиты клиента и службы субъекта с помощью дополнительного сертификата.
  Добавить данные дополнительного сертификата можно, добавив параметр CertificateFindValuebySecondary.
  Ниже приведен пример TransportSettings для прослушивателя.

  ```xml
  <Section Name="TransportSettings">
  <Parameter Name="SecurityCredentialsType" Value="X509" />
  <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
  <Parameter Name="CertificateFindValue" Value="b3449b018d0f6839a2c5d62b5b6c6ac822b6f662" />
  <Parameter Name="CertificateFindValuebySecondary" Value="h9449b018d0f6839a2c5d62b5b6c6ac822b6f690" />
  <Parameter Name="CertificateRemoteThumbprints" Value="4FEF3950642138446CC364A396E1E881DB76B48C,a9449b018d0f6839a2c5d62b5b6c6ac822b6f667" />
  <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
  <Parameter Name="CertificateStoreName" Value="My" />
  <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
  </Section>
   ```
   Ниже приведен пример TransportSettings для клиента.

  ```xml
  <Section Name="TransportSettings">
  <Parameter Name="SecurityCredentialsType" Value="X509" />
  <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
  <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
  <Parameter Name="CertificateFindValuebySecondary" Value="a9449b018d0f6839a2c5d62b5b6c6ac822b6f667" />
  <Parameter Name="CertificateRemoteThumbprints" Value="b3449b018d0f6839a2c5d62b5b6c6ac822b6f662,h9449b018d0f6839a2c5d62b5b6c6ac822b6f690" />
  <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
  <Parameter Name="CertificateStoreName" Value="My" />
  <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
  </Section>
   ```
  * Настройка параметров FabricTransport для защиты службы или клиента субъекта с помощью имени субъекта.
    Пользователь должен предоставить findType в качестве FindBySubjectName. Добавьте значения CertificateIssuerThumbprints и CertificateRemoteCommonNames.
    Ниже приведен пример TransportSettings для прослушивателя.

    ```xml
    <Section Name="TransportSettings">
    <Parameter Name="SecurityCredentialsType" Value="X509" />
    <Parameter Name="CertificateFindType" Value="FindBySubjectName" />
    <Parameter Name="CertificateFindValue" Value="CN = WinFabric-Test-SAN1-Alice" />
    <Parameter Name="CertificateIssuerThumbprints" Value="b3449b018d0f6839a2c5d62b5b6c6ac822b6f662" />
    <Parameter Name="CertificateRemoteCommonNames" Value="WinFabric-Test-SAN1-Bob" />
    <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
    <Parameter Name="CertificateStoreName" Value="My" />
    <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
    </Section>
    ```
    Ниже приведен пример TransportSettings для клиента.

  ```xml
   <Section Name="TransportSettings">
  <Parameter Name="SecurityCredentialsType" Value="X509" />
  <Parameter Name="CertificateFindType" Value="FindBySubjectName" />
  <Parameter Name="CertificateFindValue" Value="CN = WinFabric-Test-SAN1-Bob" />
  <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
  <Parameter Name="CertificateStoreName" Value="My" />
  <Parameter Name="CertificateRemoteCommonNames" Value="WinFabric-Test-SAN1-Alice" />
  <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
  </Section>
   ```
