---
title: Включение двойного шифрования на дисках, управляемых Azure CLIами неактивных дисков
description: Включите двойное шифрование неактивных данных для управляемых дисков с помощью Azure CLI.
author: roygara
ms.date: 08/24/2020
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 266ba4a6fbe0607fd09f86f5cd01addfa60252aa
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102558327"
---
# <a name="use-the-azure-cli-to-enable-double-encryption-at-rest-for-managed-disks"></a>Использование Azure CLI для включения двойного шифрования при хранении для управляемых дисков

Хранилище дисков Azure поддерживает двойное шифрование при хранении для управляемых дисков. Концептуальные сведения о двойном шифровании неактивных данных и других типах шифрования дисков см. в разделе [двойное шифрование при](../disk-encryption.md#double-encryption-at-rest) хранении статьи о шифровании диска.

## <a name="prerequisites"></a>Предварительные условия

Установите последнюю [Azure CLI](/cli/azure/install-az-cli2) и войдите в учетную запись Azure с помощью команды [AZ login](/cli/azure/reference-index).

## <a name="getting-started"></a>Начало работы

1. Создайте экземпляр Azure Key Vault и ключ шифрования.

    При создании экземпляра Key Vault необходимо включить обратимое удаление и защиту от удаления. Обратимое удаление гарантирует, что Key Vault будет хранить удаленный ключ в течение заданного срока (по умолчанию 90 дней). Защита от удаления гарантирует, что удаленный ключ нельзя удалить навсегда до истечения срока хранения. Эти параметры защищают от потери данных из-за случайного удаления. Эти параметры являются обязательными при использовании Key Vault для шифрования управляемых дисков.

    
    ```azurecli
    subscriptionId=yourSubscriptionID
    rgName=yourResourceGroupName
    location=westcentralus
    keyVaultName=yourKeyVaultName
    keyName=yourKeyName
    diskEncryptionSetName=yourDiskEncryptionSetName
    diskName=yourDiskName

    az account set --subscription $subscriptionId

    az keyvault create -n $keyVaultName -g $rgName -l $location --enable-purge-protection true --enable-soft-delete true

    az keyvault key create --vault-name $keyVaultName -n $keyName --protection software
    ```

1.    Создайте Дискенкриптионсет с encryptionType, установленным как Енкриптионатрествисплатформандкустомеркэйс. Используйте API версии **2020-05-01** в шаблоне Azure Resource Manager (ARM). 
    
        ```azurecli
        az deployment group create -g $rgName \
       --template-uri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/DoubleEncryption/CreateDiskEncryptionSetForDoubleEncryption.json" \
        --parameters "diskEncryptionSetName=$diskEncryptionSetName" "encryptionType=EncryptionAtRestWithPlatformAndCustomerKeys" "keyVaultId=$keyVaultId" "keyVaultKeyUrl=$keyVaultKeyUrl" "region=$location"
        ```

1.    Предоставьте ресурсу DiskEncryptionSet доступ к хранилищу ключей. 

        > [!NOTE]
        > Создание удостоверения DiskEncryptionSet в Azure Active Directory может занять несколько минут. Если при выполнении следующей команды появляется сообщение об ошибке вида "Не удается найти объект Active Directory", подождите несколько минут и повторите попытку.

        ```azurecli
        desIdentity=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [identity.principalId] -o tsv)

        az keyvault set-policy -n $keyVaultName -g $rgName --object-id $desIdentity --key-permissions wrapkey unwrapkey get
        ```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы создали и настроили эти ресурсы, их можно использовать для защиты управляемых дисков. Следующие ссылки содержат примеры сценариев, каждый из которых имеет соответствующий сценарий, который можно использовать для защиты управляемых дисков.

- [Примеры шаблонов Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/DoubleEncryption)
- [Использование управляемых клиентом ключей с шифрованием на стороне сервера. примеры](disks-enable-customer-managed-keys-cli.md#examples)
