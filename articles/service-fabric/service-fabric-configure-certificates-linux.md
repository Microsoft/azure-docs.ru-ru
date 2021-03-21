---
title: Настройка сертификатов для приложений в Linux
description: Настройка сертификатов для приложения с помощью среды выполнения Service Fabric в кластере Linux
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: pepogors
ms.openlocfilehash: 70f9cc38d84681f68c10882889214648a4dd2624
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98785572"
---
# <a name="certificates-and-security-on-linux-clusters"></a>Сертификаты и безопасность в кластерах Linux

Данная статья содержит информацию о настройке сертификатов X.509 в кластерах Linux.

## <a name="location-and-format-of-x509-certificates-on-linux-nodes"></a>Расположение и формат сертификатов X.509 в узлах Linux

Service Fabric обычно ожидает, что сертификаты X.509 будут присутствовать в каталоге */var/lib/sfcerts* в узлах кластера Linux. Это относится к сертификатам кластера, сертификатам клиентов и т. д. В некоторых случаях для сертификатов можно указать расположение, отличное от папки *var/lib/сфцертс* . Например, при использовании служб Reliable Services, созданных на основе пакета SDK Service Fabric для Java, можно указать для некоторых сертификатов конкретного приложения другое расположение с помощью пакета конфигурации (Settings.xml). Дополнительные сведения см. в разделе [Сертификаты, на которые имеются ссылки в пакете конфигурации (Settings.xml)](#certificates-referenced-in-the-configuration-package-settingsxml).

Для кластеров Linux платформа Service Fabric ожидает, что сертификаты должны быть либо PEM-файлом, содержащим сертификат и закрытый ключ, либо CRT-файлом, который содержит сертификат, и KEY-файлом, который содержит закрытый ключ. Все файлы должны быть в формате PEM. 

Если вы устанавливаете сертификат из Azure Key Vault с помощью [шаблона Resource Manager](./service-fabric-cluster-creation-create-template.md) или команд [PowerShell](/powershell/module/az.servicefabric/), то сертификат установится в правильном формате в каталог */var/lib/sfcerts* на каждом узле. Если вы устанавливаете сертификат другим способом, то вам следует убедиться в его правильной установке на узлах кластера.

## <a name="certificates-referenced-in-the-application-manifest"></a>Сертификаты, на которые имеются ссылки в манифесте приложения

Сертификаты, определенные в манифесте приложения, например в элементе [**SecretsCertificate**](./service-fabric-service-model-schema-elements.md#secretscertificate-element) или [**EndpointCertificate**](./service-fabric-service-model-schema-elements.md#endpointcertificate-element), должны присутствовать в каталоге */var/lib/sfcerts*. Элементы, используемые для указания сертификатов в манифесте приложения, не принимают атрибут path, поэтому сертификаты должны присутствовать в каталоге по умолчанию. Эти элементы принимают необязательный атрибут **X509StoreName**. По умолчанию используется значение My, что указывает на каталог */var/lib/sfcerts* в узлах Linux. Любое другое значение в кластере Linux не определено. Мы рекомендуем пропустить атрибут **X509StoreName** для приложений, которые выполняются в кластерах Linux. 

## <a name="certificates-referenced-in-the-configuration-package-settingsxml"></a>Сертификаты, на которые имеются ссылки в пакете конфигурации (Settings.xml)

Для некоторых служб вы можете настроить сертификат X.509 в [пакете конфигурации](./service-fabric-application-and-service-manifests.md) (по умолчанию — Settings.xml). Например, в случае, когда вы объявляете сертификаты, используемые для защиты каналов RPC для служб Reliable Services, созданных на основе пакетов SDK Service Fabric для .NET Core или Java. Есть два способа добавить ссылки на сертификаты в пакете конфигурации. Поддержка этих способов отличается в пакетах SDK для .NET Core и Java.

### <a name="using-x509-securitycredentialstype"></a>Использование SecurityCredentialsType X509

При использовании пакетов SDK для .NET или Java вы можете указать **X509** для **SecurityCredentialsType**. Соответствует `X509Credentials` типу (.NET Java[](/previous-versions/azure/reference/mt124925(v=azure.100)) / [)](/java/api/system.fabric.x509credentials) `SecurityCredentials` ([.NET](/previous-versions/azure/reference/mt124894(v=azure.100)) / [Java](/java/api/system.fabric.securitycredentials)).

Ссылка **X509** определяет местонахождение сертификата в хранилище сертификатов. В следующем коде XML показаны параметры, используемые для указания расположения сертификата:

```xml
    <Parameter Name="SecurityCredentialsType" Value="X509" />
    <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
    <Parameter Name="CertificateStoreName" Value="My" />
```

Для службы, работающей в Linux, **хранилище LocalMachine** / **My** указывает на расположение по умолчанию для сертификатов, каталог */Вар/либ/сфцертс* . Для Linux любые другие сочетания **CertificateStoreLocation** и **CertificateStoreName** не определены. 

Всегда указывайте **LocalMachine** для параметра **CertificateStoreLocation**. Нет необходимости указывать параметр **CertificateStoreName**, так как по умолчанию используется значение My. Со ссылкой **X509** файлы сертификатов должны размещаться в каталоге */var/lib/sfcerts* в узле кластера.  

В следующем коде XML показан раздел **TransportSettings** на основе этого стиля:

```xml
<Section Name="HelloWorldStatefulTransportSettings">
    <Parameter Name="MaxMessageSize" Value="10000000" />
    <Parameter Name="SecurityCredentialsType" Value="X509" />
    <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
    <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
    <Parameter Name="CertificateRemoteThumbprints" Value="9FEF3950642138446CC364A396E1E881DB76B483" />
    <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
    <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
    <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
</Section>
```

### <a name="using-x509_2-securitycredentialstype"></a>Использование SecurityCredentialsType X509_2

С помощью пакета SDK для Java вы можете указать **X509_2** для **SecurityCredentialsType**. Это соответствует типу `X509Credentials2` ([Java](/java/api/system.fabric.x509credentials2)) `SecurityCredentials` ([Java](/java/api/system.fabric.securitycredentials)). 

Со ссылкой **X509_2** вы указываете параметр path, чтобы определить местонахождение сертификата в каталоге, отличном от */var/lib/sfcerts*.  В следующем коде XML показаны параметры, используемые для указания расположения сертификата: 

```xml
     <Parameter Name="SecurityCredentialsType" Value="X509_2" />
     <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
```

В следующем коде XML показан раздел **TransportSettings** , основанный на этом стиле.

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

> [!NOTE]
> В предыдущем коде XML сертификат указывается как CRT-файл. Это означает, что в том же расположении находится и KEY-файл, содержащий закрытый ключ.

## <a name="configure-a-reliable-services-app-to-run-on-linux-clusters"></a>Настройка приложения Reliable Services для запуска на кластерах Linux

Пакеты SDK Service Fabric позволяют взаимодействовать с API среды выполнения Service Fabric для использования платформы. При запуске любого приложения, использующего эту функциональность на защищенных кластерах Linux, необходимо настроить приложение с сертификатом, который можно использовать для проверки с помощью среды выполнения Service Fabric. Такая конфигурация требуется приложениям, которые содержат службы Reliable Service в Service Fabric, написанные с помощью пакетов SDK для .NET Core или Java. 

Чтобы настроить приложение, добавьте элемент [**SecretsCertificate**](./service-fabric-service-model-schema-elements.md#secretscertificate-element) под тег **Certificates**, который находится под тегом **ApplicationManifest** в файле *ApplicationManifest.xml*. В следующем коде XML показан сертификат, на который содержится ссылка по отпечатку: 

```xml
   <Certificates>
       <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="0A00AA0AAAA0AAA00A000000A0AA00A0AAAA00" />
   </Certificates>   
```

Вы можете ссылаться на сертификат кластера или сертификат, устанавливаемый на каждый узел кластера. В Linux файлы сертификатов должны присутствовать в каталоге */var/lib/sfcerts*. Дополнительные сведения см. в разделе [Расположение и формат сертификатов X.509 в узлах Linux](#location-and-format-of-x509-certificates-on-linux-nodes).
