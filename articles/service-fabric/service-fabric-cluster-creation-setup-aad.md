---
title: Настройка Azure Active Directory для проверки подлинности клиента
description: Сведения о настройке Azure Active Directory (Azure AD) для проверки подлинности клиентов для кластеров Service Fabric.
ms.topic: conceptual
ms.date: 6/28/2019
ms.openlocfilehash: a1f89e144f9cef12c5bff87befb00a88bad8d7d9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102215975"
---
# <a name="set-up-azure-active-directory-for-client-authentication"></a>Настройка Azure Active Directory для проверки подлинности клиента

Для кластеров, работающих в Azure, рекомендуется использовать Azure Active Directory (Azure AD), чтобы обеспечить безопасность доступа к конечным точкам управления. В этой статье описывается, как настроить проверку подлинности клиентов для Service Fabric кластера в Azure AD.

В этой статье термин "приложение" будет использоваться для обращения к [Azure Active Directoryным приложениям](../active-directory/develop/developer-glossary.md#client-application), а не Service Fabricным приложениям. При необходимости это различие будет установлено. Azure AD позволяет организациям (известным как клиенты) управлять доступом пользователей к приложениям.

Кластеры Service Fabric предлагают несколько точек входа для управления функциями кластеров, включая веб-интерфейс [Service Fabric Explorer][service-fabric-visualizing-your-cluster] и [Visual Studio][service-fabric-manage-application-in-visual-studio]. В результате вы создадите два приложения Azure AD для управления доступом к кластеру: одно веб-приложение и одно собственное приложение. После создания приложений вы назначите пользователям роли "только для чтения" и "Администраторы".

> [!NOTE]
> В Linux перед созданием кластера необходимо выполнить следующие действия. В Windows вы также можете [настроить аутентификацию Azure AD для существующего кластера](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/Security/Configure%20Azure%20Active%20Directory%20Authentication%20for%20Existing%20Cluster.md).

> [!NOTE]
> Это [известная ситуация](https://github.com/microsoft/service-fabric/issues/399) , из-за которой приложения и узлы в кластерах с поддержкой AAD на платформе Linux не могут быть просмотрены на портале Azure.



## <a name="prerequisites"></a>Предварительные условия
В этой статье предполагается, что клиент уже создан. Если это не так, обратитесь к статье [Как получить клиент Azure Active Directory][active-directory-howto-tenant].

Чтобы упростить некоторые шаги по настройке Azure AD с кластером Service Fabric, мы создали набор сценариев Windows PowerShell.

1. [Клонировать репозиторий](https://github.com/Azure-Samples/service-fabric-aad-helpers) на компьютер.
2. Убедитесь, что для установленных скриптов установлены [все необходимые компоненты](https://github.com/Azure-Samples/service-fabric-aad-helpers#getting-started) .

## <a name="create-azure-ad-applications-and-assign-users-to-roles"></a>Создание приложения Azure AD и назначение ролей пользователям

Мы будем использовать эти сценарии для создания двух приложений Azure AD для управления доступом к кластеру: одно веб-приложение и одно собственное приложение. Создав приложения для представления кластера, вы создадите пользователей для [ролей, поддерживаемых Service Fabric](service-fabric-cluster-security-roles.md): только для чтения и для администратора.

Запустите `SetupApplications.ps1` и укажите идентификатор клиента, имя кластера и URL-адрес ответа веб-приложения в качестве параметров.  Укажите также имена пользователей и пароли для пользователей. Пример:

```powershell
$Configobj = .\SetupApplications.ps1 -TenantId '0e3d2646-78b3-4711-b8be-74a381d9890c' -ClusterName 'mysftestcluster' -WebApplicationReplyUrl 'https://mysftestcluster.eastus.cloudapp.azure.com:19080/Explorer/index.html' -AddResourceAccess
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestUser' -Password 'P@ssword!123'
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestAdmin' -Password 'P@ssword!123' -IsAdmin
```

> [!NOTE]
> Для национальных облаков (например, Azure для государственных организаций, Azure для Китая и Azure для Германии) также необходимо указать параметр `-Location`.

Идентификатор *TenantId* можно найти, выполнив команду PowerShell `Get-AzureSubscription` . После выполнения этой команды для каждой подписки отображается TenantId.

Значение *ClusterName* используется в качестве префикса для приложений Azure AD, создаваемых скриптом. Точное совпадение с именем реального кластера не требуется. Оно предназначено только для упрощения сопоставления артефактов Azure AD с кластером Service Fabric, с которым они используются.

*WebApplicationReplyUrl* является конечной точкой по умолчанию, которую Azure AD возвращает пользователям после завершения ими входа. Задайте эту конечную точку в качестве конечной точки Service Fabric Explorer кластера. Если вы создаете приложения Azure AD для представления существующего кластера, убедитесь, что этот URL-адрес соответствует конечной точке существующего кластера. Если вы создаете приложения для нового кластера, спланируйте конечную точку кластера и убедитесь, что не используете конечную точку существующего кластера. По умолчанию конечной точкой Service Fabric Explorer является:

https://&lt;cluster_domain&gt;:19080/Explorer

Вам будет предложено войти в учетную запись с правами администратора для клиента Azure AD. После входа сценарий начнет создавать веб-приложение и собственное приложение для представления кластера Service Fabric. Если посмотреть на список приложений клиента на [портале Azure][azure-portal], вы увидите две новых записи:

   * *имя_кластера*\_Кластер
   * *имя_кластера*\_Клиент

Сценарий печатает JSON, необходимый для шаблона Azure Resource Manager, при [создании кластера с поддержкой AAD](service-fabric-cluster-creation-create-template.md#add-azure-ad-configuration-to-use-azure-ad-for-client-access), поэтому рекомендуется открыть окно PowerShell.

```json
"azureActiveDirectory": {
  "tenantId":"<guid>",
  "clusterApplication":"<guid>",
  "clientApplication":"<guid>"
},
```

## <a name="troubleshooting-help-in-setting-up-azure-active-directory"></a>Устранение неполадок при настройке Azure Active Directory
Настройка и использование Azure AD может оказаться сложной задачей, поэтому воспользуйтесь следующими советами для отладки проблемы.

### <a name="service-fabric-explorer-prompts-you-to-select-a-certificate"></a>Service Fabric Explorer предлагает выбрать сертификат
#### <a name="problem"></a>Проблема
После успешного входа в Azure AD в Service Fabric Explorer браузер отображает домашнюю страницу, но на экране отображается предложение выбрать сертификат.

![Диалоговое окно сертификата SFX][sfx-select-certificate-dialog]

#### <a name="reason"></a>Причина
Пользователю не назначена роль в приложении кластера Azure AD. Таким образом, аутентификация Azure AD в кластере Service Fabric завершается ошибкой. Service Fabric Explorer воспользуется проверкой подлинности на основе сертификата.

#### <a name="solution"></a>Решение
Следуйте инструкциям по настройке Azure AD и назначению ролей пользователей. Кроме того, рекомендуется включить параметр "Для доступа к приложению требуется назначение пользователей", как делает сценарий `SetupApplications.ps1`.

### <a name="connection-with-powershell-fails-with-an-error-the-specified-credentials-are-invalid"></a>Подключение с помощью PowerShell завершается ошибкой: "Указанные учетные данные недействительны".
#### <a name="problem"></a>Проблема
После успешного входа в Azure AD при использовании PowerShell для подключения к кластеру с помощью режима безопасности AzureActiveDirectory возникла ошибка, подключение завершается с ошибкой: "Указанные учетные данные недействительны".

#### <a name="solution"></a>Решение
Решение этой проблемы такое же, как и в предыдущем случае.

### <a name="service-fabric-explorer-returns-a-failure-when-you-sign-in-aadsts50011"></a>Service Fabric Explorer при входе возвращает ошибку "AADSTS50011"
#### <a name="problem"></a>Проблема
При попытке входа на страницу Azure AD в Service Fabric Explorer выдается ошибка: "AADSTS50011: адрес ответа &lt;URL-адрес&gt; не совпадает с адресами ответа, настроенными для приложения: &lt;guid&gt;".

![Несовпадение адреса ответа в Service Fabric Explorer][sfx-reply-address-not-match]

#### <a name="reason"></a>Причина
Приложение кластера (веб-приложение), представляющее Service Fabric Explorer, пытается выполнить аутентификацию в Azure AD и в составе запроса предоставляет URL-адрес ответа для перенаправления. URL-адрес не указан в списке **URL-АДРЕС ОТВЕТА** приложения Azure AD.

#### <a name="solution"></a>Решение
На странице регистрации приложений Azure AD для кластера выберите **Проверка подлинности** и в разделе **URI перенаправления** добавьте URL-адрес Service Fabric Explorer в список. Сохраните изменения.

![URL-адрес ответа веб – приложения][web-application-reply-url]

### <a name="connecting-to-the-cluster-using-azure-ad-authentication-via-powershell-gives-an-error-when-you-sign-in-aadsts50011"></a>При подключении к кластеру с использованием проверки подлинности Azure AD с помощью PowerShell возникает ошибка при входе в систему: "AADSTS50011"
#### <a name="problem"></a>Проблема
При попытке подключения к кластеру Service Fabric с помощью Azure AD через PowerShell страница входа возвращает ошибку: "AADSTS50011: URL-адрес ответа, указанный в запросе, не совпадает с URL-адреса ответа, настроенного для приложения: &lt; GUID &gt; ".

#### <a name="reason"></a>Причина
Как и в предыдущих выпусках, PowerShell пытается выполнить проверку подлинности в Azure AD, которая предоставляет URL-адрес перенаправления, не указанный в списке **URL-адресов ответа** приложения Azure AD.  

#### <a name="solution"></a>Решение
Используйте тот же процесс, что и в предыдущем выпуске, но URL-адрес должен иметь значение `urn:ietf:wg:oauth:2.0:oob` , специальное перенаправление для проверки подлинности в командной строке.

### <a name="connect-the-cluster-by-using-azure-ad-authentication-via-powershell"></a>Подключение к кластеру с помощью аутентификации Azure AD с использованием PowerShell
Чтобы подключить кластер Service Fabric, воспользуйтесь следующим примером команды PowerShell:

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <endpoint> -KeepAliveIntervalInSec 10 -AzureActiveDirectory -ServerCertThumbprint <thumbprint>
```

Дополнительные сведения о командлете Connect-ServiceFabricCluster см. в [этой статье](/powershell/module/servicefabric/connect-servicefabriccluster).

### <a name="can-i-reuse-the-same-azure-ad-tenant-in-multiple-clusters"></a>Можно ли повторно использовать один и тот же клиент Azure AD в нескольких кластерах?
Да. Не забудьте добавить URL-адрес Service Fabric Explorer в (веб-)приложение кластера. В противном случае — Service Fabric Explorer не работает.

### <a name="why-do-i-still-need-a-server-certificate-while-azure-ad-is-enabled"></a>Почему мне все еще нужен сертификат сервера при включенном Azure AD?
FabricClient и FabricGateway выполняют взаимную аутентификацию. Во время проверки подлинности Azure AD интеграция Azure AD предоставляет серверу удостоверение клиента, а сертификат сервера используется клиентом для проверки подлинности сервера. Дополнительные сведения о сертификатах Service Fabric см. в разделе [Сертификаты X.509 и Service Fabric][x509-certificates-and-service-fabric].

## <a name="next-steps"></a>Дальнейшие действия
После настройки приложений Azure Active Directory и ролей для пользователей [настройте и разверните кластер](service-fabric-cluster-creation-via-arm.md).


<!-- Links -->
[azure-CLI]:https://docs.microsoft.com/cli/azure/get-started-with-azure-cli
[azure-portal]: https://portal.azure.com/
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[active-directory-howto-tenant]:../active-directory/develop/quickstart-create-new-tenant.md
[service-fabric-visualizing-your-cluster]: service-fabric-visualizing-your-cluster.md
[service-fabric-manage-application-in-visual-studio]: service-fabric-manage-application-in-visual-studio.md
[sf-aad-ps-script-download]:http://servicefabricsdkstorage.blob.core.windows.net/publicrelease/MicrosoftAzureServiceFabric-AADHelpers.zip
[x509-certificates-and-service-fabric]: service-fabric-cluster-security.md#x509-certificates-and-service-fabric

<!-- Images -->
[sfx-select-certificate-dialog]: ./media/service-fabric-cluster-creation-setup-aad/sfx-select-certificate-dialog.png
[sfx-reply-address-not-match]: ./media/service-fabric-cluster-creation-setup-aad/sfx-reply-address-not-match.png
[web-application-reply-url]: ./media/service-fabric-cluster-creation-setup-aad/web-application-reply-url.png
