---
title: Ограничение сетевого доступа к ресурсам PaaS — Azure CLI
description: Из этой статьи вы узнаете, как ограничить сетевой доступ к ресурсам Azure, таким как служба хранилища Azure и служба "База данных SQL Azure", с помощью конечных точек служб для виртуальной сети и Azure CLI.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure-services
ms.date: 03/14/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 5c23259a9cb1f3c409ad437a1e91151b5f401129
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946290"
---
# <a name="restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-the-azure-cli"></a>Ограничение сетевого доступа к ресурсам PaaS посредством конечных точек служб для виртуальной сети с помощью Azure CLI

Конечные точки служб для виртуальной сети позволяют ограничить сетевой доступ к некоторым ресурсам службы Azure определенной подсетью виртуальной сети. Можно также запретить доступ к ресурсам через Интернет. Конечные точки службы предоставляют прямое подключение из виртуальной сети к поддерживаемым службам Azure. Это позволяет использовать закрытый диапазон адресов виртуальной сети для доступа к службам Azure. Трафик, поступающий к ресурсам Azure через конечные точки службы, всегда остается в магистральной сети Microsoft Azure. Вы узнаете, как выполнять следующие задачи:

* Создание виртуальной сети с одной подсетью.
* Добавление подсети и включение конечной точки службы.
* Создание ресурса Azure и разрешение сетевого доступа к нему только из подсети.
* Развертывание виртуальной машины в каждой подсети.
* Подтверждение прав доступа к ресурсу из подсети.
* Подтверждение запрета доступа к ресурсу из подсети и Интернета.

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
  --subnet-name Public \
  --subnet-prefix 10.0.0.0/24
```

## <a name="enable-a-service-endpoint"></a>Включение конечной точки службы 

Вы можете включить конечные точки службы только для служб, поддерживающих эту функцию. Просмотрите, какие службы с поддержкой конечных точек службы доступны в расположении Azure, выполнив команду [az network vnet list-endpoint-services](/cli/azure/network/vnet). В следующем примере возвращается список служб с поддержкой конечных точек службы, доступных в регионе *eastus*. Список возвращаемых служб со временем будет увеличиваться, так как поддержка конечных точек службы будет реализовываться во все большем числе служб Azure.

```azurecli-interactive
az network vnet list-endpoint-services \
  --location eastus \
  --out table
``` 

Создайте дополнительную подсеть в виртуальной сети командой [az network vnet subnet create](/cli/azure/network/vnet/subnet). В этом примере в подсети создается конечная точка службы для службы *Microsoft.Storage*. 

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.1.0/24 \
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

## <a name="restrict-network-access-to-a-resource"></a>Ограничение сетевого доступа к ресурсу

Действия, необходимые для ограничения сетевого доступа к ресурсам, созданным с помощью служб Azure, использующих конечные точки службы, отличаются в зависимости службы. Ознакомьтесь с документацией по отдельным службам, чтобы получить точные инструкции для них. В оставшейся части этой статьи в качестве примера приведены инструкции по ограничению сетевого доступа для учетной записи хранения Azure.

### <a name="create-a-storage-account"></a>Создание учетной записи хранения

Создайте учетную запись хранения Azure с помощью команды [az storage account create](/cli/azure/storage/account). Замените `<replace-with-your-unique-storage-account-name>` именем, которое является уникальным для всех расположений Azure, содержащим только цифры и строчные буквы (длиной от 3 до 24 знаков).

```azurecli-interactive
storageAcctName="<replace-with-your-unique-storage-account-name>"

az storage account create \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2
```

После создания учетной записи хранения передайте ее строку подключения в переменную, выполнив команду [az storage account show-connection-string](/cli/azure/storage/account). Строка подключения используется для создания файлового ресурса на более позднем этапе.

```azurecli-interactive
saConnectionString=$(az storage account show-connection-string \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)
```

<a name="account-key"></a>Просмотрите содержимое переменной и запишите значение параметра **AccountKey** из полученных выходных данных, так как он будет использоваться позже.

```azurecli-interactive
echo $saConnectionString
```

### <a name="create-a-file-share-in-the-storage-account"></a>Создание файлового ресурса в учетной записи хранения

Создайте файловый ресурс в учетной записи хранения командой [az storage share create](/cli/azure/storage/share). Позже этот файловый ресурс будет подключен для подтверждения сетевого доступа к нему.

```azurecli-interactive
az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString > /dev/null
```

### <a name="deny-all-network-access-to-a-storage-account"></a>Запрет любого сетевого доступа к учетной записи хранения

По умолчанию учетные записи хранения принимают сетевые подключения клиентов в любой сети. Чтобы ограничить доступ выбранными сетями, измените действие по умолчанию на *Запретить*, выполнив команду [az storage account update](/cli/azure/storage/account). После запрещения сетевого доступа учетная запись хранения не будет доступна из любой сети.

```azurecli-interactive
az storage account update \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --default-action Deny
```

### <a name="enable-network-access-from-a-subnet"></a>Включение сетевого доступа из подсети

Разрешите сетевой доступ к учетной записи хранения из подсети *Private*, выполнив команду [az storage account network-rule add](/cli/azure/storage/account/network-rule).

```azurecli-interactive
az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName \
  --vnet-name myVirtualNetwork \
  --subnet Private
```
## <a name="create-virtual-machines"></a>Создание виртуальных машин

Чтобы проверить сетевой доступ к учетной записи хранения, разверните виртуальную машину в каждой подсети.

### <a name="create-the-first-virtual-machine"></a>Создание первой виртуальной машины

Создайте виртуальную машину в подсети *Public* с помощью команды [az vm create](/cli/azure/vm). Также команда создает ключи SSH, если они не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPublic \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Public \
  --generate-ssh-keys
```

Создание виртуальной машины занимает несколько минут. После создания виртуальной машины в Azure CLI отображаются примерно такие данные: 

```azurecli 
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmPublic",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Запишите значение **publicIpAddress** из полученных выходных данных. На следующем шаге этот адрес потребуется для доступа к виртуальной машине из Интернета.

### <a name="create-the-second-virtual-machine"></a>Создание второй виртуальной машины

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

## <a name="confirm-access-to-storage-account"></a>Подтверждение прав доступа к учетной записи хранения

Подключитесь к виртуальной машине *myVmPrivate* по протоколу SSH. Замените *\<publicIpAddress>* общедоступным IP-адресом виртуальной машины *myVmPrivate* .

```bash 
ssh <publicIpAddress>
```

Создайте папку для точки подключения.

```bash
sudo mkdir /mnt/MyAzureFileShare
```

Подключите файловый ресурс Azure к созданному каталогу. Прежде чем выполнить следующую команду, замените `<storage-account-name>` именем учетной записи и `<storage-account-key>` ключом, которые вы получили при [создании учетной записи хранения](#create-a-storage-account).

```bash
sudo mount --types cifs //<storage-account-name>.file.core.windows.net/my-file-share /mnt/MyAzureFileShare --options vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

Отобразится строка запроса `user@myVmPrivate:~$`. Файловый ресурс Azure успешно подключен к папке */mnt/MyAzureFileShare*.

Убедитесь, что виртуальная машина запрещает любые исходящие подключения с любых других общедоступных IP-адресов.

```bash
ping bing.com -c 4
```

Ответы не отобразятся, так как группа безопасности сети, связанная с подсетью *Private*, не разрешает исходящий доступ к общедоступным IP-адресам, отличным от адреса, назначенного службе хранилища Azure.

Закройте сеанс SSH с виртуальной машиной *myVmPrivate*.

## <a name="confirm-access-is-denied-to-storage-account"></a>Подтверждение запрета доступа к учетной записи хранения

Создайте сеанс SSH с виртуальной машиной *myVmPublic* при помощи приведенной ниже команды. Замените `<publicIpAddress>` общедоступным IP-адресом виртуальной машины *myVmPublic*. 

```bash 
ssh <publicIpAddress>
```

Создайте каталог для точки подключения.

```bash
sudo mkdir /mnt/MyAzureFileShare
```

Попытайтесь подключить файловый ресурс Azure к созданному каталогу. В этой статье предполагается, что вы развернули последнюю версию Ubuntu. При использовании более ранних версий Ubuntu обратитесь к разделу [Использование файлов Azure в Linux](../storage/files/storage-how-to-use-files-linux.md?toc=%2fazure%2fvirtual-network%2ftoc.json), чтобы получить дополнительные инструкции по подключению файловых ресурсов. Перед выполнением следующей команды замените `<storage-account-name>` именем учетной записи и `<storage-account-key>` ключом, полученным в разделе [Создание учетной записи хранения](#create-a-storage-account).

```bash
sudo mount --types cifs //storage-account-name>.file.core.windows.net/my-file-share /mnt/MyAzureFileShare --options vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

Будет отказано в доступе, и отобразится сообщение об ошибке `mount error(13): Permission denied`, так как виртуальная машина *myVmPublic* развернута в подсети *Public*. Для подсети *Public* не включена конечная точка службы для службы хранилища Azure, и учетная запись хранения предназначена только для сетевого доступа из подсети *Private*, а не *Public*.

Закройте сеанс SSH с виртуальной машиной *myVmPublic*.

Используя свой компьютер, попытайтесь просмотреть файловые ресурсы в учетной записи хранения, выполнив команду [az storage share list](/cli/azure/storage/share). Замените `<account-name>` и `<account-key>` именем и ключом учетной записи хранения, полученными при [создании учетной записи хранения](#create-a-storage-account).

```azurecli-interactive
az storage share list \
  --account-name <account-name> \
  --account-key <account-key>
```

Будет отказано в доступе, и появится сообщение об ошибке *This request is not authorized to perform this operation* (Данный запрос не авторизован для выполнения этой операции), так как ваш компьютер не находится в подсети *Private* виртуальной сети *MyVirtualNetwork*.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполнив команду [az group delete](/cli/azure).

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этой статьи вы включили конечную точку службы для подсети виртуальной сети. Вы узнали, что конечные точки службы можно включать для ресурсов, развернутых несколькими службами Azure. Вы создали учетную запись хранения Azure и ограничили сетевой доступ к учетной записи хранения ресурсами одной подсети в виртуальной сети. См. дополнительные сведения о [конечных точках службы](virtual-network-service-endpoints-overview.md) и [управляемых подсетях](virtual-network-manage-subnet.md).

Если в вашей учетной записи имеется несколько виртуальных сетей, можно соединить две виртуальные сети, чтобы ресурсы в каждой из них могли взаимодействовать друг с другом. Дополнительные сведения см. в статье о [подключении к виртуальным сетям](tutorial-connect-virtual-networks-cli.md).
