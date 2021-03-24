---
title: Настройка управляемых удостоверений в пулах пакетной службы
description: Узнайте, как включить управляемые пользователем удостоверения в пулах пакетной службы и как использовать управляемые удостоверения в узлах.
ms.topic: conceptual
ms.date: 03/23/2021
ms.custom: references_regions
ms.openlocfilehash: 7fab213ac1545c0bff9b74bc46504717b6038e8e
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104950167"
---
# <a name="configure-managed-identities-in-batch-pools"></a>Настройка управляемых удостоверений в пулах пакетной службы

[Управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md) устраняют необходимость в управлении учетными данными разработчиками, предоставляя удостоверение для ресурса Azure в Azure Active Directory (Azure AD) и используя его для получения маркеров Azure Active Directory (Azure AD).

В этом разделе объясняется, как включить назначенные пользователем управляемые удостоверения в пулах пакетной службы и как использовать управляемые удостоверения на узлах.

> [!IMPORTANT]
> Поддержка пулов пакетной службы Azure с назначенными пользователем управляемыми удостоверениями в настоящее время доступна в общедоступной предварительной версии для следующих регионов: Западная часть США 2, Юго-Центральный регион США, восток США, US Gov (Аризона) и US Gov (Вирджиния).
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="create-a-user-assigned-identity"></a>Создание назначаемого пользователем удостоверения

Сначала [Создайте управляемое удостоверение, назначаемое пользователем](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity) в том же клиенте, что и учетная запись пакетной службы. Это управляемое удостоверение не обязательно должно находиться в той же группе ресурсов или даже в той же подписке.

## <a name="create-a-batch-pool-with-user-assigned-managed-identities"></a>Создание пула пакетной службы с управляемыми удостоверениями, назначенными пользователем

После создания одного или нескольких назначенных пользователем управляемых удостоверений можно создать пул пакетной службы с этим управляемым удостоверением с помощью [библиотеки управления .NET пакетной](/dotnet/api/overview/azure/batch#management-library)службы.

> [!IMPORTANT]
> Для использования управляемых удостоверений необходимо настроить пулы с помощью [конфигурации виртуальной машины](nodes-and-pools.md#virtual-machine-configuration) .

```csharp
var poolParameters = new Pool(name: "yourPoolName")
    {
        VmSize = "standard_d1_v2",
        ScaleSettings = new ScaleSettings
        {
            FixedScale = new FixedScaleSettings
            {
                TargetDedicatedNodes = 1
            }
        },
        DeploymentConfiguration = new DeploymentConfiguration
        {
            VirtualMachineConfiguration = new VirtualMachineConfiguration(
                new ImageReference(
                    "Canonical",
                    "UbuntuServer",
                    "18.04-LTS",
                    "latest"),
                "batch.node.ubuntu 18.04")
        };
        Identity = new BatchPoolIdentity
        {
            Type = PoolIdentityType.UserAssigned,
            UserAssignedIdentities = new Dictionary<string, BatchPoolIdentityUserAssignedIdentitiesValue>
            {
                ["Your Identity Resource Id"] =
                    new BatchPoolIdentityUserAssignedIdentitiesValue()
            }
        }
    };

var pool = await managementClient.Pool.CreateWithHttpMessagesAsync(
    poolName:"yourPoolName",
    resourceGroupName: "yourResourceGroupName",
    accountName: "yourAccountName",
    parameters: poolParameters,
    cancellationToken: default(CancellationToken)).ConfigureAwait(false);    
```

> [!NOTE]
> Создание пулов с управляемыми удостоверениями в настоящее время не поддерживается [клиентской библиотекой .NET для пакетной](/dotnet/api/overview/azure/batch#client-library)службы.

## <a name="use-user-assigned-managed-identities-in-batch-nodes"></a>Использование назначенных пользователем управляемых удостоверений в узлах пакетной службы

После создания пулов назначаемые пользователем управляемые удостоверения могут получать доступ к узлам пула через Secure Shell (SSH) или удаленный рабочий стол (RDP). Вы также можете настроить задачи таким образом, чтобы управляемые удостоверения могли напрямую обращаться к [ресурсам Azure, поддерживающим управляемые удостоверения](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md).

В узлах пакетной службы вы можете получить токены управляемых удостоверений и использовать их для проверки подлинности с помощью аутентификации Azure AD через [службу метаданных экземпляра Azure](../virtual-machines/windows/instance-metadata-service.md).

Для Windows сценарий PowerShell, получающий маркер доступа для проверки подлинности:

```powershell
$Response = Invoke-RestMethod -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource={Resource App Id Url}' -Method GET -Headers @{Metadata="true"} 
```

Для Linux скрипт Bash:

```bash
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource={Resource App Id Url}' -H Metadata:true
```

Дополнительные сведения см. в статье [Использование управляемых удостоверений для ресурсов Azure на виртуальной машине Azure для получения маркера доступа](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md).
- Узнайте, как использовать [управляемые клиентом ключи с удостоверениями, управляемыми пользователем](batch-customer-managed-key.md).
- Узнайте, как [включить автоматическое вращение сертификатов в пуле пакетной службы](automatic-certificate-rotation.md).
