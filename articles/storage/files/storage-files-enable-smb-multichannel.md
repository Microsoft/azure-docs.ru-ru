---
title: Включить многоканальный SMB
description: Узнайте, как включить многоканальный протокол SMB в общих файловых ресурсах Azure Premium.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 11/16/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 2f867fa6d4b7e1d864a85106b5d957a53d38eb76
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101732554"
---
# <a name="enable-smb-multichannel-on-a-filestorage-account-preview"></a>Включение многоканального SMB в учетной записи Филестораже (Предварительная версия) 

Учетные записи Azure Филестораже поддерживают многоканальный протокол SMB (Предварительная версия), что повышает производительность клиента SMB 3. x путем установки нескольких сетевых подключений к общим файловым ресурсам уровня "Премиум". В этой статье приводятся пошаговые инструкции по включению многоканального SMB в существующей учетной записи хранения. Подробные сведения о многоканальном SMB для файлов Azure см. в статье многоканальная производительность SMB.

## <a name="limitations"></a>Ограничения

[!INCLUDE [storage-files-smb-multi-channel-restrictions](../../../includes/storage-files-smb-multi-channel-restrictions.md)]

### <a name="regional-availability"></a>Доступность по регионам

[!INCLUDE [storage-files-smb-multi-channel-regions](../../../includes/storage-files-smb-multi-channel-regions.md)]

## <a name="prerequisites"></a>Предварительные требования

- [Создайте учетную запись филестораже](./storage-how-to-create-file-share.md).
- Если вы планируете использовать модуль Azure PowerShell, [установите версию модуля 3.0.1-Preview](https://www.powershellgallery.com/packages/Az.Storage/3.0.1-preview).

## <a name="getting-started"></a>Начало работы

Откройте окно PowerShell и войдите в подписку Azure. После этого вы можете зарегистрироваться для просмотра многоканального протокола SMB с помощью следующих команд.

```azurepowershell
Connect-AzAccount
# Setting your active subscription to the one you want to register for the preview. 
# Replace the <subscription-id> placeholder with your subscription id. 
$context = Get-AzSubscription -SubscriptionId <your-subscription-id> 
Set-AzContext $context

Register-AzProviderFeature -FeatureName AllowSMBMultichannel -ProviderNamespace Microsoft.Storage 
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage 
```

> [!NOTE]
> Регистрация может занять до часа.

### <a name="verify-that-feature-registration-is-complete"></a>Убедитесь, что регистрация компонента завершена.

Так как включение функции в учетной записи хранения может занять до часа, можно использовать следующую команду, чтобы проверить, зарегистрирована для вашей подписки:

```azurepowershell
Get-AzProviderFeature -FeatureName AllowSMBMultichannel -ProviderNamespace Microsoft.Storage
```


## <a name="enable-smb-multichannel"></a>Включение SMB Multichannel 
После создания учетной записи Филестораже можно выполнить инструкции по обновлению параметров многоканального SMB для вашей учетной записи хранения.

# <a name="portal"></a>[Портал](#tab/azure-portal)
1. Войдите в портал Azure и перейдите в учетную запись хранения Филестораже, для которой требуется настроить многоканальный SMB.
1. Выберите **Общие папки** в разделе **Файловая служба**, а затем выберите **Параметры общей папки**.
1. **Переключите** **многоканальный SMB** (или **Отключите** его для отключения) и нажмите кнопку **сохранить**.

:::image type="content" source="media/storage-files-enable-smb-multichannel/enable-smb-multichannel-on-storage-account.png" alt-text="Снимок экрана учетной записи хранения, многоканальный протокол SMB включен.":::

Если параметр "многоканальный SMB" не отображается в разделе " **Параметры общей папки** " или при обновлении конфигурации возникает ошибка "не удалось обновить параметр", убедитесь, что подписка зарегистрирована, а ваша учетная запись находится в одном из [поддерживаемых регионов](#regional-availability) с поддерживаемым типом учетной записи и репликацией.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы включить многоканальный протокол SMB с помощью модуля Azure PowerShell, необходимо [установить версию модуля 3.0.1-Preview](https://www.powershellgallery.com/packages/Az.Storage/3.0.1-preview) .

`$resourceGroupName` `$storageAccountName` Перед выполнением этих команд PowerShell задайте переменные, а также учетную запись хранения и группу ресурсов.

```azurepowershell
# Enable SMB Multichannel on the premium storage account that's in one of the supported regions
Update-AzStorageFileServiceProperty -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName -EnableSmbMultichannel $true 
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Azure CLI пока не поддерживает настройку многоканального SMB. См. инструкции по настройке многоканального SMB в учетной записи хранения в инструкциях портала.

---

> [!NOTE]
> Любые изменения параметров конфигурации многоканального протокола SMB будут применены ко всем общим папкам в учетной записи хранения. Однако необходимо будет повторно подключить общую папку на клиенте, чтобы изменения вступили в силу.


## <a name="next-steps"></a>Дальнейшие действия 

- Повторно [Подключите файловый ресурс](storage-how-to-use-files-windows.md) , чтобы воспользоваться преимуществами многоканального SMB.
- [Устраните все проблемы, связанные с многоканальным SMB](storage-troubleshooting-files-performance.md#smb-multichannel-option-not-visible-under-file-share-settings).
- Дополнительные сведения об улучшениях см. в статье [многоканальная производительность SMB](storage-files-smb-multichannel-performance.md) .
 - Дополнительные сведения о функции многоканального SMB в Windows см. в статье [Управление SMB мулитчаннел](/azure-stack/hci/manage/manage-smb-multichannel).