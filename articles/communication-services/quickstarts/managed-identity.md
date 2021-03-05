---
title: Использование управляемых удостоверений в службах связи (.NET)
titleSuffix: An Azure Communication Services quickstart
description: Управляемые удостоверения позволяют авторизовать доступ к службам связи Azure из приложений, работающих на виртуальных машинах Azure, в приложениях функций и других ресурсах.
services: azure-communication-services
author: stefang931
ms.service: azure-communication-services
ms.topic: how-to
ms.date: 12/04/2020
ms.author: gistefan
ms.reviewer: mikben
ms.openlocfilehash: ee691d4809a68a0ba60f60a2240b76a1e53104bc
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102171579"
---
# <a name="use-managed-identities-net"></a>Использование управляемых удостоверений (.NET)

Приступая к работе со службами связи Azure с помощью управляемых удостоверений в .NET. Администрирование служб Communication Services и клиентские библиотеки SMS поддерживают проверку подлинности Azure Active Directory (Azure AD) с помощью [управляемых удостоверений для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md).

В этом кратком руководстве показано, как авторизовать доступ к клиентским библиотекам администрирования и SMS из среды Azure, которая поддерживает управляемые удостоверения. В нем также описывается тестирование кода в среде разработки.

## <a name="prerequisites"></a>Предварительные требования

 - Учетная запись Azure с активной подпиской. [Создать учетную запись бесплатно](https://azure.microsoft.com/free)
 - Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](./create-communication-resource.md?pivots=platform-azp&tabs=windows)

## <a name="setting-up"></a>Настройка

### <a name="enable-managed-identities-on-a-virtual-machine-or-app-service"></a>Включение управляемых удостоверений на виртуальной машине или в службе приложений

Управляемые удостоверения должны быть включены в ресурсах Azure, для которых выполняется авторизация. Сведения о включении управляемых удостоверений для ресурсов Azure см. в одной из следующих статей.

- [Портал Azure](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Шаблон Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Клиентские библиотеки Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)
- [Службы приложений](../../app-service/overview-managed-identity.md)

#### <a name="assign-azure-roles-with-the-azure-portal"></a>Назначение ролей Azure с помощью портал Azure

1. Перейдите на портал Azure.
1. Перейдите к ресурсу службы связи Azure.
1. Перейдите к меню управления доступом (IAM) — > + Add-> добавить назначение ролей.
1. Выберите роль участник (это единственная поддерживаемая роль).
1. Выберите "управляемое удостоверение, назначенное пользователем" (или "управляемое системой удостоверение"), а затем выберите нужное удостоверение. Сохраните выбранные элементы.

![Управляемая роль идентификации](media/managed-identity-assign-role.png)

#### <a name="assign-azure-roles-with-powershell"></a>Назначение ролей Azure с помощью PowerShell

Сведения о назначении ролей и разрешений с помощью PowerShell см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure PowerShell](../../../articles/role-based-access-control/role-assignments-powershell.md)

## <a name="add-managed-identity-to-your-communication-services-solution"></a>Добавление управляемого удостоверения в решение служб связи

### <a name="install-the-client-library-packages"></a>Установка пакетов клиентской библиотеки

```console
dotnet add package Azure.Communication.Identity
dotnet add package Azure.Communication.Configuration
dotnet add package Azure.Communication.Sms
dotnet add package Azure.Identity
```

### <a name="use-the-client-library-packages"></a>Использование пакетов клиентских библиотек

Добавьте следующие `using` директивы в код для использования Azure Identity и клиентских библиотек службы хранилища Azure.

```csharp
using Azure.Identity;
using Azure.Communication.Identity;
using Azure.Communication.Configuration;
using Azure.Communication.Sms;
```

В приведенных ниже примерах используется [дефаултазурекредентиал](/dotnet/api/azure.identity.defaultazurecredential). Эти учетные данные подходят для сред рабочей среды и разработки.

### <a name="create-an-identity-and-issue-a-token"></a>Создание удостоверения и выдача маркера

В следующем примере кода показано, как создать объект клиента службы с токенами Azure Active Directory, а затем использовать клиент для выдаче маркера для нового пользователя:

```csharp
     public async Task<Response<CommunicationUserToken>> CreateIdentityAndIssueTokenAsync(Uri resourceEndpoint) 
     {
          TokenCredential credential = new DefaultAzureCredential();
     
          var client = new CommunicationIdentityClient(resourceEndpoint, credential);
          var identityResponse = await client.CreateUserAsync();
     
          var tokenResponse = await client.IssueTokenAsync(identity, scopes: new [] { CommunicationTokenScope.VoIP });

          return tokenResponse;
     }
```

### <a name="send-an-sms-with-azure-active-directory-tokens"></a>Отправка SMS с маркерами Azure Active Directory

В следующем примере кода показано, как создать объект клиента службы с маркерами Azure Active Directory, а затем использовать клиент для отправки SMS-сообщения:

```csharp

     public async Task SendSmsAsync(Uri resourceEndpoint, PhoneNumber from, PhoneNumber to, string message)
     {
          TokenCredential credential = new DefaultAzureCredential();
     
          SmsClient smsClient = new SmsClient(resourceEndpoint, credential);
          smsClient.Send(
               from: from,
               to: to,
               message: message,
               new SendSmsOptions { EnableDeliveryReport = true } // optional
          );
     }
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о проверке подлинности](../concepts/authentication.md)

Возможно, вы также захотите выполнить такие задачи:

- [Дополнительные сведения об управлении доступом на основе ролей в Azure](../../../articles/role-based-access-control/index.yml)
- [Дополнительные сведения о библиотеке удостоверений Azure для .NET](/dotnet/api/overview/azure/identity-readme)
- [Создание маркеров доступа пользователей](../quickstarts/access-tokens.md)
- [Отправка SMS](../quickstarts/telephony-sms/send.md)
- [Дополнительные сведения об SMS](../concepts/telephony-sms/concepts.md)