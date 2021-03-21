---
title: Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault управляемом HSM (Предварительная версия)
titleSuffix: Azure Storage
description: Узнайте, как настроить шифрование службы хранилища Azure с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault управляемый модуль HSM (Предварительная версия) с использованием Azure CLI.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/05/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: f9be9272a898ad48f3553d4c5e48952e1fcdde81
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102218644"
---
# <a name="configure-encryption-with-customer-managed-keys-stored-in-azure-key-vault-managed-hsm-preview"></a>Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault управляемом HSM (Предварительная версия)

Служба хранилища Azure шифрует все данные в неактивных учетных записях хранения. По умолчанию данные шифруются с помощью ключей, управляемых корпорацией Майкрософт. Для дополнительного контроля над ключами шифрования можно управлять собственными ключами. Ключи, управляемые клиентом, должны храниться в Azure Key Vault или Key Vault управляемой аппаратной модели безопасности (HSM) (в предварительной версии). Управляемый модуль HSM Azure Key Vault — это проверенный HSM-модуль FIPS 140-2 уровня 3.

В этой статье показано, как настроить шифрование с использованием управляемых клиентом ключей, хранящихся в управляемом модуле HSM, с помощью Azure CLI. Сведения о настройке шифрования с помощью управляемых клиентом ключей, хранящихся в хранилище ключей, см. в статье [Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault](customer-managed-keys-configure-key-vault.md).

> [!IMPORTANT]
>
> Шифрование с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault управляемый модуль HSM, в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с [дополнительными условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не включены в общедоступную версию.
>
> Управляемый модуль HSM Azure Key Vault и Azure Key Vault поддерживают те же интерфейсы API и интерфейсы управления для конфигурации.

## <a name="assign-an-identity-to-the-storage-account"></a>Назначение удостоверения учетной записи хранения

Во-первых, назначьте учетной записи хранения управляемое удостоверение, назначенное системой. Вы будете использовать это управляемое удостоверение, чтобы предоставить учетной записи хранения разрешения на доступ к управляемому HSM. Дополнительные сведения о назначенных системой удостоверениях см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../../active-directory/managed-identities-azure-resources/overview.md).

Чтобы назначить управляемое удостоверение с помощью Azure CLI, вызовите команду [AZ Storage учетная запись Update](/cli/azure/storage/account#az-storage-account-update). Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями:

```azurecli
az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

## <a name="assign-a-role-to-the-storage-account-for-access-to-the-managed-hsm"></a>Назначение роли учетной записи хранения для доступа к управляемому HSM-модулю

Затем назначьте управляемому удостоверению учетной записи хранения **управляемую роль шифрования HSM** , чтобы у учетной записи хранения были разрешения на доступ к управляемому HSM. Корпорация Майкрософт рекомендует ограничить назначение ролей уровнем отдельного ключа, чтобы предоставить минимально возможные привилегии для управляемого удостоверения.

Чтобы создать назначение ролей для учетной записи хранения, вызовите команду [AZ Key Vault назначение ролей создать](/cli/azure/role/assignment#az_role_assignment_create). Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями.
  
```azurecli
storage_account_principal = $(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)

az keyvault role assignment create \
    --hsm-name <hsm-name> \
    --role "Managed HSM Crypto Service Encryption" \
    --assignee $storage_account_principal \
    --scope /keys/<key-name>
```

## <a name="configure-encryption-with-a-key-in-the-managed-hsm"></a>Настройка шифрования с помощью ключа в управляемом модуле HSM

Наконец, настройте шифрование службы хранилища Azure с помощью ключей, управляемых клиентом, чтобы использовать ключ, хранящийся в управляемом HSM. Поддерживаемые типы ключей включают ключи RSA-HSM размером 2048, 3072 и 4096. Установите Azure CLI 2.12.0 или более поздней версии, чтобы настроить шифрование для использования управляемого клиентом ключа в управляемом HSM. Дополнительные сведения см. в статье [Установка Azure CLI](/cli/azure/install-azure-cli).

Чтобы автоматически обновить ключ для управляемого клиентом ключа, не указывайте версию ключа при настройке шифрования с помощью управляемых клиентом ключей для учетной записи хранения. Чтобы обновить параметры шифрования учетной записи хранения, вызовите команду [AZ Storage Account Update](/cli/azure/storage/account#az_storage_account_update) , как показано в следующем примере. Включите и задайте для параметра значение, `--encryption-key-source parameter` `Microsoft.Keyvault` чтобы включить управляемые клиентом ключи для учетной записи. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями.

```azurecli
hsmurl = $(az keyvault show \
    --hsm-name <hsm-name> \
    --query properties.hsmUri \
    --output tsv)

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $hsmurl
```

Чтобы вручную обновить версию для ключа, управляемого клиентом, включите версию ключа при настройке шифрования для учетной записи хранения.

```azurecli-interactive
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $hsmurl
```

При ручном обновлении версии ключа необходимо обновить параметры шифрования учетной записи хранения, чтобы использовать новую версию. Сначала запросите универсальный код ресурса (URI) хранилища ключей, вызвав команду [AZ keyvault показывать](/cli/azure/keyvault#az-keyvault-show), а для версии key [— вызов AZ keyvault Key List-Versions](/cli/azure/keyvault/key#az-keyvault-key-list-versions). Затем вызовите команду [AZ Storage Account Update](/cli/azure/storage/account#az-storage-account-update) , чтобы обновить параметры шифрования учетной записи хранения для использования новой версии ключа, как показано в предыдущем примере.

## <a name="next-steps"></a>Дальнейшие действия

- [Шифрование службы хранилища Azure для неактивных данных](storage-service-encryption.md)
- [Управляемые клиентом ключи для шифрования службы хранилища Azure](customer-managed-keys-overview.md)
