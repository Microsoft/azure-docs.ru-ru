---
title: Запуск службы Service Fabric Azure в качестве пользователя или группы AD
description: Узнайте, как для запускать службу как пользователи или группу Active Directory в автономном кластере Service Fabric.
ms.topic: conceptual
ms.date: 03/29/2018
ms.openlocfilehash: d4a7afc2ddb0f39014a7cf0fd006d7fe23673a95
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91840733"
---
# <a name="run-a-service-as-an-active-directory-user-or-group"></a>Запуск службы как группы или пользователя Active Directory
В автономном кластере Windows Server вы можете запустить службу в качестве пользователя или группы Active Directory с помощью политики запуска от имени.  По умолчанию приложения Service Fabric выполняются под учетной записью, используемой процессом Fabric.exe. Запуск приложений в разных учетных записях позволяет изолировать друг от друга выполняемые приложения, даже если они запущены в общей среде. Обратите внимание, что в вашем домене используется локальная служба Active Directory, а не Azure Active Directory (Azure AD).  Кроме того, можно запустить службу в [групповой управляемой учетной записи службы (gMSA)](service-fabric-run-service-as-gmsa.md).

С помощью пользователей или групп домена вы можете обращаться к другим ресурсам в домене (например, к общим файловым ресурсам), для которых предоставлены разрешения на доступ.

В следующем примере представлена учетная запись пользователя Active Directory с именем *TestUser* и паролем, зашифрованным с помощью сертификата с именем *MyCert*. Можно использовать команду PowerShell `Invoke-ServiceFabricEncryptText` для создания секретного зашифрованного текста. Дополнительные сведения см. в статье [Управление секретами в приложениях Service Fabric](service-fabric-application-secret-management.md).

На локальном компьютере следует развернуть закрытый ключ сертификата, что позволит расшифровывать на нем пароль с использованием внешних средств (в Azure для этого используется Azure Resource Manager). Тогда Service Fabric сможет расшифровать секретный код при развертывании пакета службы на компьютере и использовать его вместе с именем пользователя для проверки подлинности в Active Directory.

```xml
<Principals>
  <Users>
    <User Name="TestUser" AccountType="DomainUser" AccountName="Domain\User" Password="[Put encrypted password here using MyCert certificate]" PasswordEncrypted="true" />
  </Users>
</Principals>
<Policies>
  <DefaultRunAsPolicy UserRef="TestUser" />
  <SecurityAccessPolicies>
    <SecurityAccessPolicy ResourceRef="MyCert" PrincipalRef="TestUser" GrantRights="Full" ResourceType="Certificate" />
  </SecurityAccessPolicies>
</Policies>
<Certificates>
```

> [!NOTE] 
> Если применить политику запуска от имени к службе, манифест которой объявляет ресурсы конечной точки с протоколом HTTP, необходимо также указать **SecurityAccessPolicy**.  Дополнительные сведения см. в статье [Назначение политики безопасности доступа для конечных точек HTTP и HTTPS](service-fabric-assign-policy-to-endpoint.md). 
>

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
Теперь ознакомьтесь со следующими статьями:
* [Сведения о модели приложения](service-fabric-application-model.md)
* [Указание ресурсов в манифесте службы](service-fabric-service-manifest-resources.md)
* [Развертывание приложения](service-fabric-deploy-remove-applications.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png
