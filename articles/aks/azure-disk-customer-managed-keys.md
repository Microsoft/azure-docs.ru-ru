---
title: Использование ключа, управляемого клиентом, для шифрования дисков Azure в службе Kubernetes Azure (AKS)
description: Применяйте собственные ключи (BYOK), чтобы зашифровать диски AKS OS и данных.
services: container-service
ms.topic: article
ms.date: 09/01/2020
ms.openlocfilehash: 4b1c311132cc812ccb2bbbc95c4b7414b108008c
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102499209"
---
# <a name="bring-your-own-keys-byok-with-azure-disks-in-azure-kubernetes-service-aks"></a>Перенесите собственные ключи (BYOK) с помощью дисков Azure в службе Kubernetes Azure (AKS)

Служба хранилища Azure шифрует все данные в неактивных учетных записях хранения. По умолчанию данные шифруются с помощью ключей, управляемых корпорацией Майкрософт. Для дополнительного контроля над ключами шифрования можно предоставить ключи, управляемые клиентом, для шифрования неактивных данных как для дисков операционной системы, так и для диска с данными для кластеров AKS. Дополнительные сведения о ключах, управляемых клиентом, в [Linux][customer-managed-keys-linux] и [Windows][customer-managed-keys-windows].

## <a name="limitations"></a>Ограничения
* Поддержка шифрования дисков данных ограничена кластерами AKS, работающими под управлением Kubernetes версии 1,17 и выше.
* Шифрование ОС и диска данных с ключами, управляемыми клиентом, можно включить только при создании кластера AKS.

## <a name="prerequisites"></a>Предварительные требования
* Необходимо включить обратимое удаление и очистку для *Azure Key Vault* при использовании Key Vault для шифрования управляемых дисков.
* Требуется Azure CLI версии 2.11.1 или более поздней.

## <a name="create-an-azure-key-vault-instance"></a>Создание экземпляра Azure Key Vault

Используйте экземпляр Azure Key Vault для хранения ключей.  При необходимости можно использовать портал Azure для [настройки ключей, управляемых клиентом, с помощью Azure Key Vault][byok-azure-portal]

Создайте новую *группу ресурсов*, а затем создайте новый экземпляр *Key Vault* и включите защиту с обратимым удалением и очисткой.  Убедитесь, что для каждой команды используются одни и те же имена регионов и групп ресурсов.

```azurecli-interactive
# Optionally retrieve Azure region short names for use on upcoming commands
az account list-locations
```

```azurecli-interactive
# Create new resource group in a supported Azure region
az group create -l myAzureRegionName -n myResourceGroup

# Create an Azure Key Vault resource in a supported Azure region
az keyvault create -n myKeyVaultName -g myResourceGroup -l myAzureRegionName  --enable-purge-protection true --enable-soft-delete true
```

## <a name="create-an-instance-of-a-diskencryptionset"></a>Создание экземпляра Дискенкриптионсет

Замените *микэйваултнаме* именем хранилища ключей.  Кроме того, потребуется *ключ* , хранящийся в Azure Key Vault, чтобы выполнить следующие действия.  Сохраните существующий ключ в Key Vault, созданном на предыдущих шагах, или [Создайте новый ключ][key-vault-generate] и замените *микэйнаме* ниже именем своего ключа.
    
```azurecli-interactive
# Retrieve the Key Vault Id and store it in a variable
keyVaultId=$(az keyvault show --name myKeyVaultName --query "[id]" -o tsv)

# Retrieve the Key Vault key URL and store it in a variable
keyVaultKeyUrl=$(az keyvault key show --vault-name myKeyVaultName  --name myKeyName  --query "[key.kid]" -o tsv)

# Create a DiskEncryptionSet
az disk-encryption-set create -n myDiskEncryptionSetName  -l myAzureRegionName  -g myResourceGroup --source-vault $keyVaultId --key-url $keyVaultKeyUrl 
```

## <a name="grant-the-diskencryptionset-access-to-key-vault"></a>Предоставление Дискенкриптионсет доступа к хранилищу ключей

Используйте Дискенкриптионсет и группы ресурсов, созданные на предыдущих шагах, и предоставьте ресурсу Дискенкриптионсет доступ к Azure Key Vault.

```azurecli-interactive
# Retrieve the DiskEncryptionSet value and set a variable
desIdentity=$(az disk-encryption-set show -n myDiskEncryptionSetName  -g myResourceGroup --query "[identity.principalId]" -o tsv)

# Update security policy settings
az keyvault set-policy -n myKeyVaultName -g myResourceGroup --object-id $desIdentity --key-permissions wrapkey unwrapkey get
```

## <a name="create-a-new-aks-cluster-and-encrypt-the-os-disk"></a>Создание нового кластера AKS и шифрование диска ОС

Создайте **новую группу ресурсов** и кластер AKS, а затем используйте ключ для шифрования диска операционной системы. Ключи, управляемые клиентом, поддерживаются только в Kubernetes версиях более 1,17. 

> [!IMPORTANT]
> Убедитесь, что вы создали новую группу ресурсов для кластера AKS.

```azurecli-interactive
# Retrieve the DiskEncryptionSet value and set a variable
diskEncryptionSetId=$(az disk-encryption-set show -n mydiskEncryptionSetName -g myResourceGroup --query "[id]" -o tsv)

# Create a resource group for the AKS cluster
az group create -n myResourceGroup -l myAzureRegionName

# Create the AKS cluster
az aks create -n myAKSCluster -g myResourceGroup --node-osdisk-diskencryptionset-id $diskEncryptionSetId --kubernetes-version KUBERNETES_VERSION --generate-ssh-keys
```

При добавлении новых пулов узлов в кластер, созданный выше, для шифрования диска операционной системы используется управляемый клиентом ключ, указанный во время создания.

## <a name="encrypt-your-aks-cluster-data-diskoptional"></a>Шифрование диска данных кластера AKS (необязательно)
Ключ шифрования диска ОС будет использоваться для шифрования диска данных, если ключ не предоставлен для диска данных из v 1.17.2, а также можно зашифровать диски данных AKS с помощью других ключей.

> [!IMPORTANT]
> Убедитесь, что у вас есть правильные учетные данные AKS. Управляемому удостоверению потребуется доступ участника к группе ресурсов, в которой развернута дискенкриптионсет. В противном случае появится сообщение об ошибке с предложением о том, что у управляемого удостоверения нет разрешений.

```azurecli-interactive
# Retrieve your Azure Subscription Id from id property as shown below
az account list
```

```
someuser@Azure:~$ az account list
[
  {
    "cloudName": "AzureCloud",
    "id": "666e66d8-1e43-4136-be25-f25bb5de5893",
    "isDefault": true,
    "name": "MyAzureSubscription",
    "state": "Enabled",
    "tenantId": "3ebbdf90-2069-4529-a1ab-7bdcb24df7cd",
    "user": {
      "cloudShellID": true,
      "name": "someuser@azure.com",
      "type": "user"
    }
  }
]
```

Создайте файл с именем **byok-Azure-Disk. YAML** , содержащий следующие сведения.  Замените Мязуресубскриптионид, myResourceGroup и Мидискенкрптионсетнаме своими значениями и примените YAML.  Обязательно используйте группу ресурсов, в которой развернута Дискенкриптионсет.  Если вы используете Azure Cloud Shell, этот файл можно создать с помощью VI или Nano, как при работе в виртуальной или физической системе:

```
kind: StorageClass
apiVersion: storage.k8s.io/v1  
metadata:
  name: hdd
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: Standard_LRS
  kind: managed
  diskEncryptionSetID: "/subscriptions/{myAzureSubscriptionId}/resourceGroups/{myResourceGroup}/providers/Microsoft.Compute/diskEncryptionSets/{myDiskEncryptionSetName}"
```
Затем запустите это развертывание в кластере AKS:
```azurecli-interactive
# Get credentials
az aks get-credentials --name myAksCluster --resource-group myResourceGroup --output table

# Update cluster
kubectl apply -f byok-azure-disk.yaml
```

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с рекомендациями [по безопасности кластера AKS][best-practices-security]

<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: ./operator-best-practices-cluster-security.md
[byok-azure-portal]: ../storage/common/customer-managed-keys-configure-key-vault.md
[customer-managed-keys-windows]: ../virtual-machines/disk-encryption.md#customer-managed-keys
[customer-managed-keys-linux]: ../virtual-machines/disk-encryption.md#customer-managed-keys
[key-vault-generate]: ../key-vault/general/manage-with-cli2.md
[supported-regions]: ../virtual-machines/disk-encryption.md#supported-regions