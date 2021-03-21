---
title: Импорт сертификатов в контейнер
description: Узнайте, как импортировать файлы сертификатов в службу контейнеров Service Fabric.
ms.topic: conceptual
ms.date: 2/23/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 219882a3f7f6db665f1ec311098ef53464773b71
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92313699"
---
# <a name="import-a-certificate-file-into-a-container-running-on-service-fabric"></a>Импорт файла сертификата в контейнер под управлением Service Fabric

> [!NOTE]
> Для кластеров Service Fabric, работающих в Azure, рекомендуется использовать [управляемое удостоверение приложения Service Fabric](./concepts-managed-identity.md) для инициализации сертификатов приложений в контейнере. Управляемое удостоверение обеспечивает изоляцию секретов и сертификатов на уровне обслуживания и позволяет подготовить сертификат приложения в рамках рабочего процесса приложения, а не рабочего процесса инфраструктуры. Механизм Цертификатереф будет нерекомендуемым в будущем выпуске.

Службы контейнеров можно защитить с помощью сертификата. Service Fabric предоставляет для служб в контейнере механизм, который обеспечивает доступ к сертификату, установленному на узлах кластера Windows или Linux (версии 5.7 или выше). Этот сертификат необходимо установить в хранилище сертификатов LocalMachine на всех узлах кластера. Необходимо, чтобы закрытый ключ, соответствующий сертификату, был доступным и экспортируемым в Windows. Сведения о сертификате указываются в манифесте приложения после тега `ContainerHostPolicies`, как показано в следующем фрагменте кода:

```xml
  <ContainerHostPolicies CodePackageRef="NodeContainerService.Code">
    <CertificateRef Name="MyCert1" X509StoreName="My" X509FindValue="[Thumbprint1]"/>
    <CertificateRef Name="MyCert2" X509FindValue="[Thumbprint2]"/>
 ```

При запуске приложения для кластеров Windows среда выполнения экспортирует каждый указанный сертификат и соответствующий ему закрытый ключ в PFX-файл, защищенный случайно сгенерированным паролем. Доступ к PFX-файлу и паролю в контейнере можно получить с помощью следующих переменных среды: 

* Certificates_ServicePackageName_CodePackageName_CertName_PFX
* Certificates_ServicePackageName_CodePackageName_CertName_Password

В кластерах Linux сертификаты (PEM) копируются из хранилища, заданного параметром X509StoreName, в контейнер. Ниже перечислены соответствующие переменные среды в Linux:

* Certificates_ServicePackageName_CodePackageName_CertName_PEM;
* Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey.

Обратите внимание, что `PEM` `PrivateKey` файл и содержит сертификат и незашифрованный закрытый ключ.

Кроме того, если у вас уже есть сертификаты в требуемом формате и вам нужно обращаться к ним внутри контейнера, можно создать пакет данных внутри пакета приложения и указать в манифесте приложения следующее:

```xml
<ContainerHostPolicies CodePackageRef="NodeContainerService.Code">
  <CertificateRef Name="MyCert1" DataPackageRef="[DataPackageName]" DataPackageVersion="[Version]" RelativePath="[Relative Path to certificate inside DataPackage]" Password="[password]" IsPasswordEncrypted="[true/false]"/>
 ```

Импорт файлов сертификатов в контейнер выполняется с помощью службы контейнеров или процесса контейнера. Для импорта сертификата можно использовать сценарии `setupentrypoint.sh` или пользовательский исполняемый код в процессе контейнера. Ниже приведен пример кода C# для импорта PFX-файла:

```csharp
string certificateFilePath = Environment.GetEnvironmentVariable("Certificates_MyServicePackage_NodeContainerService.Code_MyCert1_PFX");
string passwordFilePath = Environment.GetEnvironmentVariable("Certificates_MyServicePackage_NodeContainerService.Code_MyCert1_Password");
X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
string password = File.ReadAllLines(passwordFilePath, Encoding.Default)[0];
password = password.Replace("\0", string.Empty);
X509Certificate2 cert = new X509Certificate2(certificateFilePath, password, X509KeyStorageFlags.MachineKeySet | X509KeyStorageFlags.PersistKeySet);
store.Open(OpenFlags.ReadWrite);
store.Add(cert);
store.Close();
```
Этот PFX-файл сертификата можно использовать для аутентификации приложения или службы, а также для безопасного обмена данными с другими службами. По умолчанию список управления доступом применяется к файлам только для SYSTEM. Вы можете применять список управления доступом к этим файлам для других учетных записей, если это нужно для работы службы.

Теперь ознакомьтесь со следующими статьями:

* [Развертывание контейнера Windows в Service Fabric на платформе Windows Server 2016](service-fabric-get-started-containers.md)
* [Развертывание контейнера Docker в Service Fabric на платформе Linux](service-fabric-get-started-containers-linux.md)