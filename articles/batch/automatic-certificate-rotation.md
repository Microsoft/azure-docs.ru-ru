---
title: Включение автоматического смены сертификатов в пуле пакетной службы
description: Вы можете создать пул пакетной службы с управляемым удостоверением и сертификатом, который будет автоматически продлен.
ms.topic: conceptual
ms.date: 03/23/2021
ms.custom: references_regions
ms.openlocfilehash: e8bea49b2980deb8f20258ab7ea5619ece8cd2bf
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104962575"
---
# <a name="enable-automatic-certificate-rotation-in-a-batch-pool"></a>Включение автоматического смены сертификатов в пуле пакетной службы

 Вы можете создать пул пакетной службы с сертификатом, который будет автоматически продлен. Для этого необходимо создать пул с [назначенным пользователем управляемым удостоверением](managed-identity-pools.md) , который будет иметь доступ к сертификату в [Azure Key Vault](../key-vault/general/overview.md).

> [!IMPORTANT]
> Поддержка пулов пакетной службы Azure с назначенными пользователем управляемыми удостоверениями в настоящее время доступна в общедоступной предварительной версии для следующих регионов: Западная часть США 2, Юго-Центральный регион США, восток США, US Gov (Аризона) и US Gov (Вирджиния).
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="create-a-user-assigned-identity"></a>Создание назначаемого пользователем удостоверения

Сначала [Создайте управляемое удостоверение, назначаемое пользователем](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity) в том же клиенте, что и учетная запись пакетной службы. Это управляемое удостоверение не обязательно должно находиться в той же группе ресурсов или даже в той же подписке.

Обязательно запишите **идентификатор клиента** управляемого удостоверения, назначенного пользователем. Это значение понадобится позже.

:::image type="content" source="media/automatic-certificate-rotation/client-id.png" alt-text="Снимок экрана, показывающий идентификатор клиента управляемого удостоверения, назначенного пользователем в портал Azure.":::

## <a name="create-your-certificate"></a>Создание сертификата

Затем необходимо создать сертификат и добавить его в Azure Key Vault. Если вы еще не создали хранилище ключей, это необходимо сделать первым. Инструкции см. [в разделе Краткое руководство. Установка и извлечение сертификата из Azure Key Vault с помощью портал Azure](../key-vault/certificates/quick-create-portal.md).

При создании сертификата не забудьте задать для параметра **действие времени жизни** значение автоматическое продление и укажите число дней, по истечении которого сертификат должен продлить.

:::image type="content" source="media/automatic-certificate-rotation/certificate.png" alt-text="Снимок экрана с экраном создания сертификата в портал Azure.":::

После создания сертификата запишите его **идентификатор секрета**. Это значение понадобится позже.

:::image type="content" source="media/automatic-certificate-rotation/secret-identifier.png" alt-text="Снимок экрана, показывающий секретный идентификатор сертификата.":::

## <a name="add-an-access-policy-in-azure-key-vault"></a>Добавление политики доступа в Azure Key Vault

В хранилище ключей назначьте политику доступа Key Vault, которая позволяет назначенному пользователем управляемому удостоверению получать доступ к секретам и сертификатам. Подробные инструкции см. [в разделе Назначение политики доступа Key Vault с помощью портал Azure](../key-vault/general/assign-access-policy-portal.md).

## <a name="create-a-batch-pool-with-a-user-assigned-managed-identity"></a>Создание пула пакетной службы с управляемым удостоверением, назначенным пользователем

Создайте пул пакетной службы с управляемым удостоверением с помощью [библиотеки управления .NET пакетной](/dotnet/api/overview/azure/batch#management-library)службы. Дополнительные сведения см. [в статье Настройка управляемых удостоверений в пулах пакетной](managed-identity-pools.md)службы.

В следующем примере для создания пула используется REST API управления пакетной службой. Не забудьте использовать **идентификатор секрета** сертификата для `observedCertificates` **идентификатора клиента** управляемого удостоверения для `msiClientId` , заменив приведенные ниже примеры данных.

Универсальный код ресурса (URI) REST API

```http
PUT https://management.azure.com/subscriptions/<subscriptionid>/resourceGroups/<resourcegroupName>/providers/Microsoft.Batch/batchAccounts/<batchaccountname>/pools/<poolname>?api-version=2021-01-01
```

Текст запроса

```json
{
    "name": "test2",
    "type": "Microsoft.Batch/batchAccounts/pools",
    "properties": {
        "vmSize": "STANDARD_DS2_V2",
        "taskSchedulingPolicy": {
            "nodeFillType": "Pack"
        },
        "deploymentConfiguration": {
            "virtualMachineConfiguration": {
                "imageReference": {
                    "publisher": "canonical",
                    "offer": "ubuntuserver",
                    "sku": "18.04-lts",
                    "version": "latest"
                },
                "nodeAgentSkuId": "batch.node.ubuntu 18.04",
                "extensions": [
                    {
                        "name": "KVExtensions",
                        "type": "KeyVaultForLinux",
                        "publisher": "Microsoft.Azure.KeyVault",
                        "typeHandlerVersion": "1.0",
                        "autoUpgradeMinorVersion": true,
                        "settings": {
                            "secretsManagementSettings": {
                                "pollingIntervalInS": "300",
                                "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault",
                                "requireInitialSync": true,
                                "observedCertificates": [
                                    "https://testkvwestus2s.vault.azure.net/secrets/authcertforumatesting/8f5f3f491afd48cb99286ba2aacd39af"
                                ]
                            },
                            "authenticationSettings": {
                                "msiEndpoint": "http://169.254.169.254/metadata/identity",
                                "msiClientId": "b9f6dd56-d2d6-4967-99d7-8062d56fd84c"
                            }  }, "protectedSettings":{}
                    }                ]            }
        },
        "scaleSettings": {
            "fixedScale": {
                "targetDedicatedNodes": 1,
                "resizeTimeout": "PT15M"
            }
        },
    },
    "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "/subscriptions/042998e4-36dc-4b7d-8ce3-a7a2c4877d33/resourceGroups/ACR/providers/Microsoft.ManagedIdentity/userAssignedIdentities/testumaforpools": {}
        }
    }
}

```

## <a name="validate-the-certificate"></a>Проверка сертификата

Чтобы убедиться, что сертификат успешно развернут, войдите на узел COMPUTE. Вы должны увидеть результат, аналогичный приведенному ниже:

```
root@74773db5fe1b42ab9a4b6cf679d929da000000:/var/lib/waagent/Microsoft.Azure.KeyVault.KeyVaultForLinux-1.0.1363.13/status# cat 1.status
[{"status":{"code":0,"formattedMessage":{"lang":"en","message":"Successfully started Key Vault extension service. 2021-03-03T23:12:23Z"},"operation":"Service start.","status":"success"},"timestampUTC":"2021-03-03T23:12:23Z","version":"1.0"}]root@74773db5fe1b42ab9a4b6cf679d929da000000:/var/lib/waagent/Microsoft.Azure.KeyVault.KeyVaultForLinux-1.0.1363.13/status#
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md).
- Узнайте, как использовать [управляемые клиентом ключи с удостоверениями, управляемыми пользователем](batch-customer-managed-key.md).
