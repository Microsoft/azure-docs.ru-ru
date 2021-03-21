---
title: Ограничение утечка данных в службе хранилища Azure Azure CLI
description: Из этой статьи вы узнаете, как ограничить и ограничить утечка данных виртуальной сети ресурсами хранилища Azure с помощью политик конечной точки службы виртуальной сети, используя Azure CLI.
services: virtual-network
documentationcenter: virtual-network
author: rdhillon
manager: ''
editor: ''
tags: azure-resource-manager
Customer intent: I want only specific Azure Storage account to be allowed access from a virtual network subnet.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: rdhillon
ms.custom: ''
ms.openlocfilehash: 9ce1e320a93a834a938ce95f3931d885d2214faa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98216875"
---
# <a name="manage-data-exfiltration-to-azure-storage-accounts-with-virtual-network-service-endpoint-policies-using-the-azure-cli"></a>Управление данными утечка в учетные записи хранения Azure с помощью политик конечной точки службы виртуальной сети с использованием Azure CLI

Политики конечной точки службы виртуальной сети позволяют применять управление доступом к учетным записям хранения Azure из виртуальной сети через конечные точки службы. Это ключ к обеспечению безопасности рабочих нагрузок, управлению учетными записями хранения и разрешениим утечка данных.
Вы узнаете, как выполнять следующие задачи:

* Создайте виртуальную сеть и добавьте подсеть.
* Включите конечную точку службы для хранилища Azure.
* Создайте две учетные записи хранения Azure и разрешите сетевой доступ к ней из созданной выше подсети.
* Создайте политику конечной точки службы, чтобы разрешить доступ только к одной из учетных записей хранения.
* Разверните виртуальную машину (ВМ) в подсети.
* Подтвердите доступ к разрешенной учетной записи хранения из подсети.
* Убедитесь, что доступ запрещен учетной записи хранения, неразрешенной из подсети.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-virtual-network"></a>Создание виртуальной сети

Перед созданием виртуальной сети необходимо создать для нее группу ресурсов и другие компоненты, указанные в этой статье. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create \
  --name myResourceGroup \
  --location eastus
```

Создайте виртуальную сеть с одной подсетью при помощи команды [az network vnet create](/cli/azure/network/vnet).

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Private \
  --subnet-prefix 10.0.0.0/24
```

## <a name="enable-a-service-endpoint"></a>Включение конечной точки службы 

В этом примере конечная точка службы для *Microsoft. Storage* создается для подсети *Private*: 

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.0.0/24 \
  --service-endpoints Microsoft.Storage
```

## <a name="restrict-network-access-for-a-subnet"></a>Ограничение сетевого доступа для подсети

Создайте группу безопасности сети с помощью команды [az network nsg create](/cli/azure/network/nsg). В следующем примере создается группа безопасности сети *myNsgPrivate*.

```azurecli-interactive
az network nsg create \
  --resource-group myResourceGroup \
  --name myNsgPrivate
```

Чтобы привязать подсеть *Private* к этой группе безопасности сети, выполните команду [az network vnet subnet update](/cli/azure/network/vnet/subnet). В следующем примере подсеть *Private* привязывается к группе безопасности сети *myNsgPrivate*.

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --name Private \
  --resource-group myResourceGroup \
  --network-security-group myNsgPrivate
```

Создайте правила безопасности командой [az network nsg rule create](/cli/azure/network/nsg/rule). Приведенное ниже правило разрешает исходящий доступ к общедоступным IP-адресам, назначенным службе хранилища Azure. 

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Allow-Storage-All \
  --access Allow \
  --protocol "*" \
  --direction Outbound \
  --priority 100 \
  --source-address-prefix "VirtualNetwork" \
  --source-port-range "*" \
  --destination-address-prefix "Storage" \
  --destination-port-range "*"
```

Каждая группа безопасности сети содержит несколько [правил безопасности по умолчанию](./network-security-groups-overview.md#default-security-rules). Следующее правило переопределяет правило безопасности по умолчанию, разрешающее исходящий доступ ко всем общедоступным IP-адресам. `destination-address-prefix "Internet"`Параметр запрещает исходящий доступ ко всем общедоступным IP-адресам. Предыдущее правило переопределяет это правило ввиду более высокого приоритета. Оно предоставляет доступ к общедоступным IP-адресам службы хранилища Azure.

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Deny-Internet-All \
  --access Deny \
  --protocol "*" \
  --direction Outbound \
  --priority 110 \
  --source-address-prefix "VirtualNetwork" \
  --source-port-range "*" \
  --destination-address-prefix "Internet" \
  --destination-port-range "*"
```

Следующее правило разрешает входящий трафик SSH в подсеть из любого места. Это правило переопределяет правило безопасности по умолчанию, запрещающее весь входящий трафик из Интернета. Для подсети разрешено использование SSH, чтобы можно было протестировать подключение на более позднем этапе.

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Allow-SSH-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 120 \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "VirtualNetwork" \
  --destination-port-range "22"
```

## <a name="restrict-network-access-to-azure-storage-accounts"></a>Ограничение сетевого доступа к учетным записям хранения Azure

В этом разделе перечислены действия по ограничению сетевого доступа для учетной записи хранения Azure из заданной подсети в виртуальной сети с помощью конечной точки службы.

### <a name="create-a-storage-account"></a>Создание учетной записи хранения

Создайте две учетные записи хранения Azure с помощью команды [AZ Storage Account Create](/cli/azure/storage/account).

```azurecli-interactive
storageAcctName1="allowedstorageacc"

az storage account create \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2

storageAcctName2="notallowedstorageacc"

az storage account create \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2
```

После создания учетных записей хранения извлеките строку подключения для учетных записей хранения в переменную с помощью команды [AZ Storage Account показывать-Connection-String](/cli/azure/storage/account). Строка подключения используется для создания файлового ресурса на более позднем этапе.

```azurecli-interactive
saConnectionString1=$(az storage account show-connection-string \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)

saConnectionString2=$(az storage account show-connection-string \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)
```

<a name="account-key"></a>Просмотрите содержимое переменной и запишите значение параметра **AccountKey** из полученных выходных данных, так как он будет использоваться позже.

```azurecli-interactive
echo $saConnectionString1

echo $saConnectionString2
```

### <a name="create-a-file-share-in-the-storage-account"></a>Создание файлового ресурса в учетной записи хранения

Создайте файловый ресурс в учетной записи хранения командой [az storage share create](/cli/azure/storage/share). Позже этот файловый ресурс будет подключен для подтверждения сетевого доступа к нему.

```azurecli-interactive
az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString1 > /dev/null

az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString2 > /dev/null
```

### <a name="deny-all-network-access-to-the-storage-account"></a>Запретить весь сетевой доступ к учетной записи хранения

По умолчанию учетные записи хранения принимают сетевые подключения клиентов в любой сети. Чтобы ограничить доступ выбранными сетями, измените действие по умолчанию на *Запретить*, выполнив команду [az storage account update](/cli/azure/storage/account). После запрещения сетевого доступа учетная запись хранения не будет доступна из любой сети.

```azurecli-interactive
az storage account update \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --default-action Deny

az storage account update \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --default-action Deny
```

### <a name="enable-network-access-from-virtual-network-subnet"></a>Включить сетевой доступ из подсети виртуальной сети

Разрешите сетевой доступ к учетной записи хранения из подсети *Private*, выполнив команду [az storage account network-rule add](/cli/azure/storage/account/network-rule).

```azurecli-interactive
az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName1 \
  --vnet-name myVirtualNetwork \
  --subnet Private

az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName2 \
  --vnet-name myVirtualNetwork \
  --subnet Private
```

## <a name="apply-policy-to-allow-access-to-valid-storage-account"></a>Применение политики для разрешения доступа к допустимой учетной записи хранения

Политики конечной точки службы Azure доступны только в службе хранилища Azure. Поэтому мы будем включать конечную точку службы для *Microsoft. Storage* в этой подсети для этого примера установки.

Политики конечной точки службы применяются к конечным точкам службы. Начнем с создания политики конечной точки службы. Затем мы создадим определения политик в этой политике для того, чтобы учетные записи хранения Azure были утверждены для этой подсети.

Создание политики конечной точки службы.

```azurecli-interactive
az network service-endpoint policy create \
  --resource-group myResourceGroup \
  --name mysepolicy \
  --location eastus
```

Сохраните URI ресурса для разрешенной учетной записи хранения в переменной. Перед выполнением приведенной ниже команды замените *\<your-subscription-id>* фактическим ЗНАЧЕНИЕМ идентификатора подписки.

```azurecli-interactive
$serviceResourceId="/subscriptions/<your-subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/allowedstorageacc"
```

Создание & Добавление определения политики для разрешения указанной выше учетной записи хранения Azure в политику конечной точки службы

```azurecli-interactive
az network service-endpoint policy-definition create \
  --resource-group myResourceGroup \
  --policy-name mysepolicy \
  --name mypolicydefinition \
  --service "Microsoft.Storage" \
  --service-resources $serviceResourceId
```

И обновите подсеть виртуальной сети, чтобы связать ее с политикой конечной точки службы, созданной на предыдущем шаге.

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --service-endpoints Microsoft.Storage \
  --service-endpoint-policy mysepolicy
```

## <a name="validate-access-restriction-to-azure-storage-accounts"></a>Проверка ограничения доступа для учетных записей хранения Azure

### <a name="create-the-virtual-machine"></a>Создание виртуальной машины

Чтобы проверить сетевой доступ к учетной записи хранения, разверните виртуальную машину в подсети.

Создайте виртуальную машину в *частной* подсети с помощью команды [AZ VM Create](/cli/azure/vm). Также команда создает ключи SSH, если они не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPrivate \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Private \
  --generate-ssh-keys
```

Создание виртуальной машины занимает несколько минут. После создания запишите значение **publicIpAddress** из полученных выходных данных. На следующем шаге этот адрес потребуется для доступа к виртуальной машине из Интернета.

### <a name="confirm-access-to-storage-account"></a>Подтверждение прав доступа к учетной записи хранения

Подключитесь к виртуальной машине *myVmPrivate* по протоколу SSH. Замените *\<publicIpAddress>* общедоступным IP-адресом виртуальной машины *myVmPrivate* .

```bash 
ssh <publicIpAddress>
```

Создайте папку для точки подключения.

```bash
sudo mkdir /mnt/MyAzureFileShare1
```

Подключите файловый ресурс Azure к созданному каталогу. Перед выполнением приведенной ниже команды замените *\<storage-account-key>* значением *AccountKey* с **$saConnectionString 1**.

```bash
sudo mount --types cifs //allowedstorageacc.file.core.windows.net/my-file-share /mnt/MyAzureFileShare1 --options vers=3.0,username=allowedstorageacc,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

Отобразится строка запроса `user@myVmPrivate:~$`. Файловый ресурс Azure успешно подключен к папке */mnt/MyAzureFileShare*.

### <a name="confirm-access-is-denied-to-storage-account"></a>Подтверждение запрета доступа к учетной записи хранения

В той же *myVmPrivate* виртуальной машины создайте каталог для точки подключения:

```bash
sudo mkdir /mnt/MyAzureFileShare2
```

Попытайтесь подключить общую папку Azure из учетной записи хранения *ноталловедсторажеакк* к созданному каталогу. В этой статье предполагается, что вы развернули последнюю версию Ubuntu. При использовании более ранних версий Ubuntu обратитесь к разделу [Использование файлов Azure в Linux](../storage/files/storage-how-to-use-files-linux.md?toc=%2fazure%2fvirtual-network%2ftoc.json), чтобы получить дополнительные инструкции по подключению файловых ресурсов. 

Перед выполнением приведенной ниже команды замените *\<storage-account-key>* значением *AccountKey* из **$saConnectionString 2**.

```bash
sudo mount --types cifs //notallowedstorageacc.file.core.windows.net/my-file-share /mnt/MyAzureFileShare2 --options vers=3.0,username=notallowedstorageacc,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

Отказано в доступе, и появляется `mount error(13): Permission denied` сообщение об ошибке, так как эта учетная запись хранения не входит в список разрешений политики конечной точки службы, применяемой к подсети. 

Закройте сеанс SSH с виртуальной машиной *myVmPublic*.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполнив команду [az group delete](/cli/azure).

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы применили политику конечной точки службы к службе хранилища Azure для конечной точки службы виртуальной сети Azure. Вы создали учетные записи хранения Azure и ограниченный сетевой доступ только к определенным учетным записям хранения (и, таким же, отклонили другие) из подсети виртуальной сети. Дополнительные сведения о политиках конечной точки службы см. в статье [Общие сведения о политиках конечных точек служб](virtual-network-service-endpoint-policies-overview.md).