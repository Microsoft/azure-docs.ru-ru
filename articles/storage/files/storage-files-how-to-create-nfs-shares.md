---
title: Создание общего ресурса NFS — файлы Azure (Предварительная версия)
description: Узнайте, как создать файловый ресурс Azure, который можно подключить с помощью протокола сетевой файловой системы.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 01/22/2021
ms.author: rogarana
ms.subservice: files
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: b085b9991175d8cd43e2dac0db80c5af4e703c34
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102521243"
---
# <a name="how-to-create-an-nfs-share"></a>Создание общей папки NFS
Файловые ресурсы Azure — это полностью управляемые общие файловые ресурсы, которые находятся в облаке. В этой статье рассматривается создание общей папки, использующей протокол NFS. Дополнительные сведения об этих протоколах см. в статье [протоколы файловых ресурсов Azure](storage-files-compare-protocols.md).

## <a name="limitations"></a>Ограничения
[!INCLUDE [files-nfs-limitations](../../../includes/files-nfs-limitations.md)]

### <a name="regional-availability"></a>Доступность по регионам
[!INCLUDE [files-nfs-regional-availability](../../../includes/files-nfs-regional-availability.md)]

## <a name="prerequisites"></a>Предварительные требования
- Доступ к общим папкам NFS можно получить только из доверенных сетей. Подключения к общему ресурсу NFS должны исходить из одного из следующих источников:
    - Либо [Создайте закрытую конечную точку](storage-files-networking-endpoints.md#create-a-private-endpoint) (рекомендуется), либо [Ограничьте доступ к общедоступной конечной точке](storage-files-networking-endpoints.md#restrict-public-endpoint-access).
    - [Настройте VPN-подключение типа "точка — сеть" (P2S) в Linux для использования с файлами Azure](storage-files-configure-p2s-vpn-linux.md).
    - [Настройте VPN типа "сеть — сеть" для использования с файлами Azure](storage-files-configure-s2s-vpn.md).
    - Настройте [ExpressRoute](../../expressroute/expressroute-introduction.md).

- Если вы хотите использовать Azure CLI, [установите последнюю версию](/cli/azure/install-azure-cli).

## <a name="register-the-nfs-41-protocol"></a>Регистрация протокола NFS 4,1
Если вы используете модуль Azure PowerShell или Azure CLI, зарегистрируйте функцию с помощью следующих команд:

# <a name="portal"></a>[Портал](#tab/azure-portal)
Чтобы зарегистрировать компонент NFS 4,1 для службы файлов Azure, используйте либо Azure PowerShell, либо Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```azurepowershell
# Connect your PowerShell session to your Azure account, if you have not already done so.
Connect-AzAccount

# Set the actively selected subscription, if you have not already done so.
$subscriptionId = "<yourSubscriptionIDHere>"
$context = Get-AzSubscription -SubscriptionId $subscriptionId
Set-AzContext $context

# Register the NFS 4.1 feature with Azure Files to enable the preview.
Register-AzProviderFeature `
    -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowNfsFileShares 
    
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
# Connect your Azure CLI to your Azure account, if you have not already done so.
az login

# Provide the subscription ID for the subscription where you would like to 
# register the feature
subscriptionId="<yourSubscriptionIDHere>"

az feature register \
    --name AllowNfsFileShares \
    --namespace Microsoft.Storage \
    --subscription $subscriptionId

az provider register \
    --namespace Microsoft.Storage
```

---

Утверждение регистрации может занять до часа. Чтобы убедиться, что регистрация завершена, используйте следующие команды:

# <a name="portal"></a>[Портал](#tab/azure-portal)
Чтобы проверить регистрацию функции NFS 4,1 для службы файлов Azure, используйте либо Azure PowerShell, либо Azure CLI. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```azurepowershell
Get-AzProviderFeature `
    -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowNfsFileShares
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az feature show \
    --name AllowNfsFileShares \
    --namespace Microsoft.Storage \
    --subscription $subscriptionId
```

---

## <a name="create-a-filestorage-storage-account"></a>Создание учетной записи хранения Филестораже
В настоящее время общие папки NFS 4,1 доступны только в качестве файловых ресурсов уровня "Премиум". Чтобы развернуть файловый ресурс Premium с поддержкой протокола NFS 4,1, необходимо сначала создать учетную запись хранения Филестораже. Учетная запись хранения — это объект верхнего уровня в Azure, который представляет общий пул хранилища, который можно использовать для развертывания нескольких файловых ресурсов Azure.

# <a name="portal"></a>[Портал](#tab/azure-portal)
Чтобы создать учетную запись хранения Филестораже, перейдите к портал Azure.

1. В портал Azure выберите **учетные записи хранения** в меню слева.

    ![портал Azure главной страницы выберите учетную запись хранения](media/storage-how-to-create-premium-fileshare/azure-portal-storage-accounts.png)

2. В появившемся окне **Учетные записи хранения** выберите **добавить**.
3. Выберите подписку, в которой будет создана учетная запись хранения.
4. Выберите группу ресурсов, в которой будет создана учетная запись хранения.

5. Далее введите имя своей учетной записи хранения. Выбранное вами имя должно быть уникальным в Azure. Также имя должно содержать от 3 до 24 символов и может состоять только из цифр и строчных букв.
6. Выберите расположение учетной записи хранения или используйте расположение по умолчанию.
7. Для **повышения производительности** выберите **Premium**.

    В раскрывающемся списке **тип учетной записи** выберите значение **Premium** для **филестораже** , чтобы стать доступным.

8. Выберите **тип учетной записи** и щелкните **филестораже**.
9. Оставьте для параметра **репликация** значение по умолчанию **локально избыточное хранилище (LRS)**.

    ![Создание учетной записи хранения для файлового ресурса уровня "Премиум"](media/storage-how-to-create-premium-fileshare/create-filestorage-account.png)

10. Выберите **Просмотр и создание**, чтобы просмотреть настройки учетной записи хранения и создать учетную запись.
11. Выберите **Создать**.

После создания ресурса учетной записи хранения перейдите к нему.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Чтобы создать учетную запись хранения Филестораже, откройте командную строку PowerShell и выполните следующие команды, не заменяя `<resource-group>` и `<storage-account>` указав соответствующие значения для вашей среды.

```powershell
$resourceGroupName = "<resource-group>"
$storageAccountName = "<storage-account>"
$location = "westus2"

$storageAccount = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -SkuName Premium_LRS `
    -Location $location `
    -Kind FileStorage
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Чтобы создать учетную запись хранения Филестораже, откройте терминал и выполните следующие команды, не заменяя `<resource-group>` и `<storage-account>` указав соответствующие значения для вашей среды.

```azurecli-interactive
resourceGroup="<resource-group>"
storageAccount="<storage-account>"
location="westus2"

az storage account create \
    --resource-group $resourceGroup \
    --name $storageAccount \
    --location $location \
    --sku Premium_LRS \
    --kind FileStorage
```
---

## <a name="create-an-nfs-share"></a>Создание общей папки NFS

# <a name="portal"></a>[Портал](#tab/azure-portal)

Теперь, когда вы создали учетную запись Филестораже и настроили сеть, можно создать файловый ресурс NFS. Процесс аналогичен созданию общего ресурса SMB. при создании общей папки выбирается **NFS** вместо **SMB** .

1. Перейдите к своей учетной записи хранения и выберите **Общие папки**.
1. Выберите **+ Общая папка** , чтобы создать новый файловый ресурс.
1. Назовите общую папку и выберите подготовленную емкость.
1. В качестве **протокола** выберите **NFS (Предварительная версия)**.
1. Для выбора **корневого Squash** сделайте выбор.

    - Root Squash (по умолчанию) — доступ для удаленного суперпользователя (root) сопоставляется с UID (65534) и GID (65534).
    - Без корневого Squash — удаленный суперпользователь (root) получает доступ от имени привилегированного пользователя.
    - Все Squash — все пользователи сопоставлены с UID (65534) и GID (65534).
    
1. Нажмите кнопку **создания**.

    :::image type="content" source="media/storage-files-how-to-create-mount-nfs-shares/create-nfs-file-share.png" alt-text="Снимок экрана: колонка создания файлового ресурса":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Убедитесь, что платформа .NET Framework установлена. См. раздел [Download платформа .NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).
 
1. Убедитесь, что установленная версия PowerShell `5.1` или более поздняя, с помощью следующей команды.    

   ```powershell
   echo $PSVersionTable.PSVersion.ToString() 
   ```
    
   Сведения об обновлении версии PowerShell см. в разделе [обновление существующих Windows PowerShell](/powershell/scripting/install/installing-windows-powershell#upgrading-existing-windows-powershell) .
    
1. Установите последнюю версию модуля PowershellGet.

   ```powershell
   install-Module PowerShellGet –Repository PSGallery –Force  
   ```

1. Закройте, а затем снова откройте консоль PowerShell.

1. Установите модуль просмотра **AZ. Storage** предварительной версии **2.5.2-Preview**.

   ```powershell
   Install-Module Az.Storage -Repository PsGallery -RequiredVersion 2.5.2-preview -AllowClobber -AllowPrerelease -Force  
   ```

   Дополнительные сведения об установке модулей PowerShell см. [в статье Установка модуля Azure PowerShell](/powershell/azure/install-az-ps) .
   
1. Чтобы создать файловый ресурс уровня "Премиум" с помощью модуля Azure PowerShell, используйте командлет [New-азрмсторажешаре](/powershell/module/az.storage/new-azrmstorageshare) .

    > [!NOTE]
    > Счета за файловые ресурсы уровня "Премиум" выставляются с помощью подготовленной модели. Подготовленный размер общей папки указывается `QuotaGiB` ниже. Дополнительные сведения см. [в разделе Основные сведения о подготовленной модели](understanding-billing.md#provisioned-model) и [странице цен на службы файлов Azure](https://azure.microsoft.com/pricing/details/storage/files/).

    ```powershell
    New-AzRmStorageShare `
        -StorageAccount $storageAccount `
        -Name myshare `
        -EnabledProtocol NFS `
        -RootSquash RootSquash `
        -QuotaGiB 1024
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Чтобы создать общую папку уровня "Премиум" с помощью Azure CLI, используйте команду [AZ Storage Share Create](/cli/azure/storage/share-rm) .

> [!NOTE]
> Счета за файловые ресурсы уровня "Премиум" выставляются с помощью подготовленной модели. Подготовленный размер общей папки указывается `quota` ниже. Дополнительные сведения см. [в разделе Основные сведения о подготовленной модели](understanding-billing.md#provisioned-model) и [странице цен на службы файлов Azure](https://azure.microsoft.com/pricing/details/storage/files/).

```azurecli-interactive
az storage share-rm create \
    --resource-group $resourceGroup \
    --storage-account $storageAccount \
    --name "myshare" \
    --enabled-protocol NFS \
    --root-squash RootSquash \
    --quota 1024
```
---

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда вы создали общий ресурс NFS, для его использования необходимо подключить его к клиенту Linux. Дополнительные сведения см. [в разделе Подключение общего ресурса NFS](storage-files-how-to-mount-nfs-shares.md).

Если возникнут проблемы, см. раздел [Устранение неполадок файловых ресурсов NFS Azure](storage-troubleshooting-files-nfs.md).