---
title: Обновление служб мультимедиа v3 после получения ключей доступа к хранилищу | Документация Майкрософт
description: В этой статье содержатся инструкции по обновлению служб мультимедиа версии 3 после последовательного ввода ключей доступа к хранилищу.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/22/2021
ms.author: inhenkel
ms.openlocfilehash: 76ba1b848b033c5a26e81be18bef23a2b4715a7e
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105572481"
---
# <a name="update-media-services-v3-after-rolling-storage-access-keys"></a>Обновление служб мультимедиа v3 после последовательного доступа к хранилищу

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

При создании новой учетной записи служб мультимедиа Azure (AMS) вам будет предложено выбрать учетную запись хранения Azure.  В учетную запись служб мультимедиа можно добавить несколько учетных записей хранения. В этой статье показано, как чередовать ключи к хранилищу данных. В нем также показано, как добавлять учетные записи хранения в учетную запись служб мультимедиа.

Для выполнения действий, описанных в этой статье, следует использовать [интерфейсы api Azure Resource Manager](/rest/api/media/operations/azure-media-services-rest-api-reference) и [PowerShell](/powershell/module/az.media).  Дополнительные сведения см. [в статье Управление ресурсами Azure с помощью PowerShell и диспетчер ресурсов](../../azure-resource-manager/management/manage-resource-groups-powershell.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="storage-access-key-generation"></a>Создание ключа доступа к хранилищу

При создании новой учетной записи хранения Azure создает ключи доступа к хранилищу 2 512-bit, которые используются для проверки подлинности доступа к вашей учетной записи хранения. Чтобы обеспечить безопасность подключений к хранилищу, периодически повторно создавайте и вращайте ключ доступа к хранилищу. Предоставляются два ключа доступа (первичная и вторичная), позволяющие поддерживать подключения к учетной записи хранения, используя один ключ доступа при повторном создании другого ключа доступа. Этот процесс называется "оборот ключей доступа".

Службы мультимедиа зависят от предоставленного ключа к хранилищу данных. В частности, от ключа доступа к хранилищу зависят указатели, используемые для потоковой передачи или скачивания ресурсов-контейнеров. При создании учетной записи AMS по умолчанию используется зависимость от основного ключа доступа к хранилищу. Однако в качестве пользователя можно обновить ключ к хранилищу данных AMS. Необходимо предоставить службам мультимедиа сведения о том, какой ключ использовать, выполнив следующие действия.

>[!NOTE]
> Если используется несколько учетных записей хранения, необходимо выполнить эту процедуру для каждой из них. Порядок, в котором следует чередовать ключи к хранилищу данных, не является фиксированным. Можно сначала сменить вторичный ключ, а затем первичный ключ, или наоборот.
>
> Перед выполнением шагов в рабочей учетной записи обязательно протестируйте их в предварительной учетной записи.
>

## <a name="steps-to-rotate-storage-keys"></a>Инструкции по чередованию ключей к хранилищу данных
 
 1. Измените первичный ключ учетной записи хранения с помощью командлета PowerShell или портала [Azure](https://portal.azure.com/) .
 2. Вызовите `Sync-AzMediaServiceStorageKeys` командлет с соответствующими параметрами, чтобы заставить учетная запись мультимедиа выбрать ключи учетной записи хранения.
 
    Следующий пример демонстрирует, как синхронизировать ключи с учетными записями хранения.
  
    `Sync-AzMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId`
  
 3. Подождите один час или около того. Проверьте работу потоковой передачи там, где она используется.
 4. Измените вторичный ключ учетной записи хранения с помощью командлета PowerShell или портал Azure.
 5. Вызовите `Sync-AzMediaServiceStorageKeys` PowerShell с соответствующими параметрами, чтобы указать учетной записи мультимедиа возможность выбора новых ключей учетной записи хранения.
 6. Подождите один час или около того. Проверьте работу потоковой передачи там, где она используется.
 
### <a name="a-powershell-cmdlet-example"></a>Пример командлета PowerShell

Ниже приведен пример, демонстрирующий, как получить учетную запись хранилища и синхронизировать ее с учетной записью AMS.

```console
$regionName = "West US"
$resourceGroupName = "SkyMedia-USWest-App"
$mediaAccountName = "sky"
$storageAccountName = "skystorage"
$storageAccountId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccountName"

Sync-AzMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId
```

## <a name="steps-to-add-storage-accounts-to-your-ams-account"></a>Инструкции по добавлению учетных записей хранения в учетную запись AMS

В следующей статье показано, как добавить учетные записи хранения в учетную запись AMS: [Управление активами служб мультимедиа в нескольких учетных записях хранения](media-services-managing-multiple-storage-accounts.md).
