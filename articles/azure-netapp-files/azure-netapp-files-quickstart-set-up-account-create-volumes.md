---
title: Краткое руководство. Настройка Azure NetApp Files и тома NFS
description: Краткое руководство. Описание быстрой настройки Azure NetApp Files и создания тома.
author: b-juche
ms.author: b-juche
ms.service: azure-netapp-files
ms.workload: storage
ms.topic: quickstart
ms.date: 09/22/2020
ms.custom: devx-track-azurecli, subject-armqs
ms.openlocfilehash: e31a1cef427062723adf4b45bd47cd8009630128
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94888816"
---
# <a name="quickstart-set-up-azure-netapp-files-and-create-an-nfs-volume"></a>Краткое руководство. Настройка Azure NetApp Files и создание тома NFS

В этой статье описывается, как быстро настроить Azure NetApp Files и создать том.

В этом кратком руководстве описывается настройка следующих элементов:

- регистрации для использования Azure NetApp Files и поставщика ресурсов NetApp;
- учетной записи NetApp;
- пула емкости;
- тома NFS для Azure NetApp Files.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="before-you-begin"></a>Перед началом

> [!IMPORTANT]
> Необходимо иметь доступ к службе Azure NetApp Files. Запросить доступ к службе можно на [странице заявки на доступ к Azure NetApp Files](https://aka.ms/azurenetappfiles).  Перед продолжением необходимо дождаться официального подтверждения по электронной почте от команды Azure NetApp Files.

---

## <a name="register-for-azure-netapp-files-and-netapp-resource-provider"></a>Регистрация для использования Azure NetApp Files и поставщика ресурсов NetApp

> [!NOTE]
> Процесс регистрации может занять некоторое время.
>

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы выполнить регистрацию с помощью портала, откройте сеанс Cloud Shell, как указано выше, и выполните следующие действия на интерфейсе командной строки Azure:

[!INCLUDE [azure-netapp-files-cloudshell-include](../../includes/azure-netapp-files-azure-cloud-shell-window.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для работы с этой статьей требуется модуль Azure PowerShell Az 2.6.0 или более поздней версии. Чтобы узнать, какая версия используется сейчас, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить установку или обновление, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При желании вместо сеанса PowerShell можно использовать консоль Cloud Shell.

1. В командной строке PowerShell (или сеансе Cloud Shell PowerShell) укажите подписку, которая была утверждена для Azure NetApp Files:
    ```powershell-interactive
    Select-AzSubscription -Subscription <subscriptionId>
    ```

2. Зарегистрируйте поставщик ресурсов Azure.
    ```powershell-interactive
    Register-AzResourceProvider -ProviderNamespace Microsoft.NetApp
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Подготовьте среду к работе с Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [azure-netapp-files-cloudshell-include](../../includes/azure-netapp-files-azure-cloud-shell-window.md)]

# <a name="template"></a>[Шаблон](#tab/template)

Отсутствует.

Для регистрации Azure NetApp Files и поставщика ресурсов NetApp используйте портал Azure, PowerShell или Azure CLI.

Дополнительные сведения см. в статье [Регистрация в службе Azure NetApp Files](azure-netapp-files-register.md).

---

## <a name="create-a-netapp-account"></a>Создание учетной записи NetApp

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. В поле поиска на портале Azure введите **Azure NetApp Files**, затем выберите **Azure NetApp Files** в появившемся списке.

      ![Выбор Azure NetApp Files](../media/azure-netapp-files/azure-netapp-files-select-azure-netapp-files.png)

2. Нажмите кнопку **+ Добавить** для создания новой учетной записи NetApp.

     ![Создание учетной записи NetApp](../media/azure-netapp-files/azure-netapp-files-create-new-netapp-account.png)

3. В окне "New NetApp Account" (Новая учетная запись NetApp) укажите следующую информацию.
   1. Введите имя учетной записи **myaccount1**.
   2. Выберите свою подписку.
   3. Выберите **Создать**, чтобы создать группу ресурсов. Введите имя группы ресурсов **myRG1**. Нажмите кнопку **ОК**.
   4. Выберите расположение учетной записи.

      ![Окно создания учетной записи NetApp](../media/azure-netapp-files/azure-netapp-files-new-account-window.png)

      ![Окно группы ресурсов](../media/azure-netapp-files/azure-netapp-files-resource-group-window.png)

4. Нажмите кнопку **Создать**, чтобы создать учетную запись NetApp.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Определите некоторые переменные, чтобы мы могли ссылаться на них во всех остальных примерах:

    ```powershell-interactive
    $resourceGroup = "myRG1"
    $location = "eastus"
    $anfAccountName = "myaccount1"
    ```

    > [!NOTE]
    > Список поддерживаемых регионов см. в статье [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all).
    > Чтобы получить имя региона, которое поддерживается нашими средствами командной строки, используйте `Get-AzLocation | select Location`
    >

1. Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

    ```powershell-interactive
    New-AzResourceGroup -Name $resourceGroup -Location $location
    ```

2. Создайте учетную запись Azure NetApp Files с помощью команды [New-AzNetAppFilesAccount](/powershell/module/az.netappfiles/New-AzNetAppFilesAccount):

    ```powershell-interactive
    New-AzNetAppFilesAccount -ResourceGroupName $resourceGroup -Location $location -Name $anfAccountName
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Определите некоторые переменные, чтобы мы могли ссылаться на них во всех остальных примерах:

    ```azurecli-interactive
    RESOURCE_GROUP="myRG1"
    LOCATION="eastus"
    ANF_ACCOUNT_NAME="myaccount1"
    ```

    > [!NOTE]
    > Список поддерживаемых регионов см. в статье [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all).
    > Чтобы получить имя региона, которое поддерживается нашими средствами командной строки, используйте `az account list-locations --query "[].{Region:name}" --out table`
    >

2. Создайте новую группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create):

    ```azurecli-interactive
    az group create \
        --name $RESOURCE_GROUP \
        --location $LOCATION
    ```

3. Создайте учетную запись Azure NetApp Files с помощью команды [az netappfiles account create](/cli/azure/netappfiles/account#az-netappfiles-account-create):

    ```azurecli-interactive
    az netappfiles account create \
        --resource-group $RESOURCE_GROUP \
        --location $LOCATION \
        --account-name $ANF_ACCOUNT_NAME
    ```

# <a name="template"></a>[Шаблон](#tab/template)

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

В следующем фрагменте кода показано, как создать учетную запись NetApp в шаблоне Azure Resource Manager (ARM) с помощью ресурса [Microsoft.NetApp/netAppAccounts](/azure/templates/microsoft.netapp/netappaccounts). Чтобы запустить код, скачайте [полный шаблон ARM](https://github.com/Azure/azure-quickstart-templates/blob/master/101-anf-nfs-volume/azuredeploy.json) из нашего репозитория GitHub.

:::code language="json" source="~/quickstart-templates/101-anf-nfs-volume/azuredeploy.json" range="177-183":::

<!-- Block begins with "type": "Microsoft.NetApp/netAppAccounts", -->

---

## <a name="set-up-a-capacity-pool"></a>Настройка пула емкости

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. В колонке управления Azure NetApp Files выберите свою учетную запись NetApp (**myaccount1**).

    ![Выбор учетной записи NetApp](../media/azure-netapp-files/azure-netapp-files-select-netapp-account.png)

2. В колонке управления Azure NetApp Files для своей учетной записи NetApp щелкните **Пулы емкости**.

    ![Выбор "Пулы емкости"](../media/azure-netapp-files/azure-netapp-files-click-capacity-pools.png)

3. Щелкните **+ Добавить пулы**.

    ![Выбор "+ Добавить пулы"](../media/azure-netapp-files/azure-netapp-files-new-capacity-pool.png)

4. Укажите сведения о пуле емкости.
    * Введите имя пула **mypool1**.
    * Выберите уровень обслуживания **Премиум**.
    * Укажите размер пула **4 (Тиб)** .
    * Используйте тип качества обслуживания **Авто**.

5. Нажмите кнопку **Создать**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Определение новых переменных для дальнейшего использования

    ```powershell-interactive
    $poolName = "mypool1"
    $poolSizeBytes = 4398046511104 # 4TiB
    $serviceLevel = "Premium" # Valid values are Standard, Premium and Ultra
    ```

1. Создайте новый пул емкости с помощью [New-AzNetAppFilesPool](/powershell/module/az.netappfiles/new-aznetappfilespool)

    ```powershell-interactive
    New-AzNetAppFilesPool -ResourceGroupName $resourceGroup -Location $location -AccountName $anfAccountName -Name $poolName -PoolSize $poolSizeBytes -ServiceLevel $serviceLevel
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Определение новых переменных для дальнейшего использования

    ```azurecli-interactive
    POOL_NAME="mypool1"
    POOL_SIZE_TiB=4 # Size in Azure CLI needs to be in TiB unit (minimum 4 TiB)
    SERVICE_LEVEL="Premium" # Valid values are Standard, Premium and Ultra
    ```

2. Создайте новый пул емкости с помощью [az netappfiles pool create](/cli/azure/netappfiles/pool#az-netappfiles-pool-create)

    ```azurecli-interactive
    az netappfiles pool create \
        --resource-group $RESOURCE_GROUP \
        --location $LOCATION \
        --account-name $ANF_ACCOUNT_NAME \
        --pool-name $POOL_NAME \
        --size $POOL_SIZE_TiB \
        --service-level $SERVICE_LEVEL
    ```

# <a name="template"></a>[Шаблон](#tab/template)

<!-- [!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)] -->

В следующем фрагменте кода показано, как создать пул емкости в шаблоне Azure Resource Manager (ARM) с помощью ресурса [Microsoft.NetApp/netAppAccounts/capacityPools](/azure/templates/microsoft.netapp/netappaccounts/capacitypools). Чтобы запустить код, скачайте [полный шаблон ARM](https://github.com/Azure/azure-quickstart-templates/blob/master/101-anf-nfs-volume/azuredeploy.json) из нашего репозитория GitHub.

:::code language="json" source="~/quickstart-templates/101-anf-nfs-volume/azuredeploy.json" range="184-196":::

<!-- LN 185, block begins with  "type": "Microsoft.NetApp/netAppAccounts/capacityPools", -->

---

## <a name="create-nfs-volume-for-azure-netapp-files"></a>Создание тома NFS для Azure NetApp Files

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. В колонке управления Azure NetApp Files для своей учетной записи NetApp щелкните **Тома**.

    ![Выбор "Тома"](../media/azure-netapp-files/azure-netapp-files-click-volumes.png)

2. Щелкните **+ Add volume** (+ Добавить том).

    ![Выбор "+ Добавить том"](../media/azure-netapp-files/azure-netapp-files-click-add-volumes.png)

3. В окне "Создать том" укажите сведения о томе.
   1. Введите имя тома **myvol1**.
   2. Выберите пул емкости (**mypool1**).
   3. Используйте значение квоты по умолчанию.
   4. В разделе "Виртуальная сеть" щелкните **Создать**, чтобы создать виртуальную сеть Azure.  Затем укажите следующие сведения.
       * Введите имя виртуальной сети **myvnet1**.
       * Укажите диапазон адресов для параметра, например 10.7.0.0/16.
       * Введите имя подсети **myANFsubnet**.
       * Укажите диапазон адресов подсети, например 10.7.0.0/24. Невозможно использовать выделенную подсеть совместно с другими ресурсами.
       * Для делегирования подсети выберите **Microsoft.NetApp/volumes**.
       * Нажмите кнопку **ОК**, чтобы создать виртуальную сеть.
   5. В разделе "Подсеть" выберите созданную виртуальную сеть (**myvnet1**) в качестве делегируемой подсети.

      ![Окно создания тома](../media/azure-netapp-files/azure-netapp-files-create-volume-window.png)

      ![Окно создания виртуальной сети](../media/azure-netapp-files/azure-netapp-files-create-virtual-network-window.png)

4. Щелкните **Протокол**, а затем выполните приведенные ниже действия.
    * Выберите **NFS** в качестве типа протокола тома.
    * Укажите путь к файлу (**myfilepath1**), который будет использоваться для создания пути экспорта для этого тома.
    * Выберите версию NFS (**NFSv3** или **NFSv4.1**) для тома.
      Сведения о версиях NFS см. в разделах [Сonsiderations](azure-netapp-files-create-volumes.md#considerations) (Рекомендации) и [Best practice](azure-netapp-files-create-volumes.md#best-practice) (Лучшая методика).

    ![Чтобы сразу же начать работу, выберите протокол NFS](../media/azure-netapp-files/azure-netapp-files-quickstart-protocol-nfs.png)

5. Щелкните **Review + create** (Просмотреть и создать).

    ![Окно просмотра и создания](../media/azure-netapp-files/azure-netapp-files-review-and-create-window.png)

6. Ознакомьтесь с информацией о томе, а затем щелкните **Создать**.
    Созданный том появится в колонке "Тома".

    ![Созданный том](../media/azure-netapp-files/azure-netapp-files-create-volume-created.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Создайте делегирование подсети для "Microsoft.NetApp/volumes" с помощью команды [New-AzDelegation](/powershell/module/az.network/new-azdelegation).

    ```powershell-interactive
    $anfDelegation = New-AzDelegation -Name ([guid]::NewGuid().Guid) -ServiceName "Microsoft.NetApp/volumes"
    ```

2. Создайте конфигурацию подсети с помощью команды [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig).

    ```powershell-interactive
    $subnet = New-AzVirtualNetworkSubnetConfig -Name "myANFSubnet" -AddressPrefix "10.7.0.0/24" -Delegation $anfDelegation
    ```

3. Создайте виртуальную сеть с помощью команды [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

    ```powershell-interactive
    $vnet = New-AzVirtualNetwork -Name "myvnet1" -ResourceGroupName $resourceGroup -Location $location -AddressPrefix "10.7.0.0/16" -Subnet $subnet
    ```

4. Создайте том с помощью команды [New-AzNetAppFilesVolume](/powershell/module/az.netappfiles/new-aznetappfilesvolume).

    ```powershell-interactive
    $volumeSizeBytes = 1099511627776 # 100GiB
    $subnetId = $vnet.Subnets[0].Id

    New-AzNetAppFilesVolume -ResourceGroupName $resourceGroup `
        -Location $location `
        -AccountName $anfAccountName `
        -PoolName $poolName `
        -Name "myvol1" `
        -UsageThreshold $volumeSizeBytes `
        -SubnetId $subnetId `
        -CreationToken "myfilepath1" `
        -ServiceLevel $serviceLevel `
        -ProtocolType NFSv3
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Определение некоторых переменных для последующего использования.

    ```azurecli-interactive
    VNET_NAME="myvnet1"
    SUBNET_NAME="myANFSubnet"
    ```

1. Создайте виртуальную сеть без подсети с помощью команды [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create).

    ```azurecli-interactive
    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --location $LOCATION \
        --address-prefix "10.7.0.0/16"

    ```

2. Создайте делегированную подсеть с помощью команды [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create).

    ```azurecli-interactive
    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_NAME \
        --address-prefixes "10.7.0.0/24" \
        --delegations "Microsoft.NetApp/volumes"
    ```

3. Создайте том с помощью команды [az netappfiles volume create](/cli/azure/netappfiles/volume#az-netappfiles-volume-create).

    ```azurecli-interactive
    VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP --name $VNET_NAME --query "id" -o tsv)
    SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP --vnet-name $VNET_NAME --name $SUBNET_NAME --query "id" -o tsv)
    VOLUME_SIZE_GiB=100 # 100 GiB
    UNIQUE_FILE_PATH="myfilepath2" # Please note that creation token needs to be unique within subscription and region

    az netappfiles volume create \
        --resource-group $RESOURCE_GROUP \
        --location $LOCATION \
        --account-name $ANF_ACCOUNT_NAME \
        --pool-name $POOL_NAME \
        --name "myvol1" \
        --service-level $SERVICE_LEVEL \
        --vnet $VNET_ID \
        --subnet $SUBNET_ID \
        --usage-threshold $VOLUME_SIZE_GiB \
        --file-path $UNIQUE_FILE_PATH \
        --protocol-types "NFSv3"
    ```

# <a name="template"></a>[Шаблон](#tab/template)

<!-- [!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)] -->

В следующих фрагментах кода показано, как настроить виртуальную сеть и создать том Azure NetApp Files в шаблоне Azure Resource Manager (ARM). Для настройки виртуальной сети требуется ресурс [Microsoft.Network/virtualNetworks](/azure/templates/Microsoft.Network/virtualNetworks). Для создания тома используется ресурс [Microsoft.NetApp/netAppAccounts/capacityPools/volumes](/azure/templates/microsoft.netapp/netappaccounts/capacitypools/volumes). Чтобы запустить код, скачайте [полный шаблон ARM](https://github.com/Azure/azure-quickstart-templates/blob/master/101-anf-nfs-volume/azuredeploy.json) из нашего репозитория GitHub.

:::code language="json" source="~/quickstart-templates/101-anf-nfs-volume/azuredeploy.json" range="148-176":::

<!-- Block begins with  "type": "Microsoft.Network/virtualNetworks", -->

:::code language="json" source="~/quickstart-templates/101-anf-nfs-volume/azuredeploy.json" range="197-229":::

<!-- Block begins with  "type": "Microsoft.NetApp/netAppAccounts/capacityPools/volumes", -->

---

## <a name="clean-up-resources"></a>Очистка ресурсов

# <a name="portal"></a>[Портал](#tab/azure-portal)

По окончании группу ресурсов можно при необходимости удалить. Удаление группы ресурсов — необратимая операция.

> [!IMPORTANT]
> Все ресурсы в группе ресурсов будут окончательно удалены, и отменить эту операцию невозможно.

1. В поле поиска на портале Azure введите **Azure NetApp Files**, затем выберите **Azure NetApp Files** в появившемся списке.

2. В списке подписок щелкните группу ресурсов (myRG1), которую необходимо удалить.

    ![Переход к группе ресурсов](../media/azure-netapp-files/azure-netapp-files-azure-navigate-to-resource-groups.png)


3. На странице группы ресурсов выберите команду **Удалить группу ресурсов**.

    ![Снимок экрана, на котором выделена кнопка "Удалить группу ресурсов".](../media/azure-netapp-files/azure-netapp-files-azure-delete-resource-group.png)

    Откроется окно с предупреждением о ресурсах, которые будут удалены вместе с группой ресурсов.

4. Введите имя группы ресурсов (myRG1) для подтверждения того, что вы хотите окончательно удалить группу ресурсов и все входящие в нее ресурсы, а затем нажмите кнопку **Удалить**.

    ![Подтверждение удаления группы ресурсов](../media/azure-netapp-files/azure-netapp-files-azure-confirm-resource-group-deletion.png )

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

По окончании группу ресурсов можно при необходимости удалить. Удаление группы ресурсов — необратимая операция.

> [!IMPORTANT]
> Все ресурсы в группе ресурсов будут окончательно удалены, и отменить эту операцию невозможно.

1. Удалите группу ресурсов с помощью команды [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

    ```powershell-interactive
    Remove-AzResourceGroup -Name $resourceGroup
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

По окончании группу ресурсов можно при необходимости удалить. Удаление группы ресурсов — необратимая операция.

> [!IMPORTANT]
> Все ресурсы в группе ресурсов будут окончательно удалены, и отменить эту операцию невозможно.

1. Удалите группу ресурсов с помощью команды [az group delete](/cli/azure/group#az-group-delete).

    ```azurecli-interactive
    az group delete \
        --name $RESOURCE_GROUP
    ```

# <a name="template"></a>[Шаблон](#tab/template)

Отсутствует.

Чтобы удалить группу ресурсов, используйте портал Azure, PowerShell или Azure CLI.

---

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Иерархия хранилища Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md)

> [!div class="nextstepaction"]
> [Уровни обслуживания для Azure NetApp Files](azure-netapp-files-service-levels.md)

> [!div class="nextstepaction"]
> [Создание тома NFS](azure-netapp-files-create-volumes.md)
