---
title: Настройка gMSA для служб контейнеров Azure Service Fabric
description: Узнайте, как настроить групповую управляемую учетную запись службы (gMSA) для контейнера, работающего в Azure Service Fabric.
ms.topic: conceptual
ms.date: 03/20/2019
ms.openlocfilehash: d34b4c6e11628b6a4843f8a9077ebf69c9e023fe
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86260881"
---
# <a name="set-up-gmsa-for-windows-containers-running-on-service-fabric"></a>Настройка учетных записей gMSA для контейнеров Windows, запущенных в Service Fabric

Чтобы настроить gMSA, необходимо разместить файл спецификаций учетных данных (`credspec`) на всех узлах в кластере. Вы можете скопировать файл на все узлы, используя расширение виртуальной машины.  Файл `credspec` должен содержать информацию об учетной записи gMSA. Дополнительные сведения о `credspec` файле см. в разделе [Создание спецификации учетных данных](/virtualization/windowscontainers/manage-containers/manage-serviceaccounts#create-a-credential-spec). Спецификация учетных данных и `Hostname` тег указываются в манифесте приложения. Тег `Hostname` должен соответствовать имени учетной записи gMSA, которая используется для запуска контейнера.  Тег `Hostname` позволяет контейнеру автоматически проходить проверку подлинности в других службах в домене с помощью проверки подлинности Kerberos.  В следующем фрагменте кода показан пример с указанием тегов `Hostname` и `credspec` в манифесте приложения.

```xml
<Policies>
  <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" Hostname="gMSAAccountName">
    <SecurityOption Value="credentialspec=file://WebApplication1.json"/>
  </ContainerHostPolicies>
</Policies>
```
Теперь ознакомьтесь со следующими статьями:

* [Развертывание контейнера Windows в Service Fabric на платформе Windows Server 2016](service-fabric-get-started-containers.md)
* [Развертывание контейнера Docker в Service Fabric на платформе Linux](service-fabric-get-started-containers-linux.md)
