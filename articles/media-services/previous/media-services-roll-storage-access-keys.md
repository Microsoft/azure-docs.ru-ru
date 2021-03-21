---
title: Обновление служб мультимедиа после оборота ключей доступа к хранилищу | Документация Майкрософт
description: В этой статье содержится руководство по обновлению служб мультимедиа после оборота ключей доступа к хранилищу.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: a892ebb0-0ea0-4fc8-b715-60347cc5c95b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: inhenkel
ms.reviewer: milanga;cenkdin
ms.openlocfilehash: 12732171f774e6ce010f722cde4a27bb298275b9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103007946"
---
# <a name="update-media-services-after-rolling-storage-access-keys"></a>Обновление Служб мультимедиа после смены ключей доступа к хранилищу

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

При создании новой учетной записи служб мультимедиа Azure (AMS) пользователю предлагается выбрать учетную запись хранения Azure, которая используется для хранения мультимедийного содержимого. К учетной записи служб мультимедиа можно добавить несколько учетных записей хранения. В этой статье показано, как чередовать ключи к хранилищу данных. В нем также показано, как добавлять учетные записи хранения в учетную запись служб мультимедиа. 

Чтобы выполнить действия, описанные в этой статье, необходимо использовать [API Azure Resource Manager](/rest/api/media/operations/azure-media-services-rest-api-reference) и [PowerShell](/powershell/module/az.media).  Дополнительные сведения см. [в статье Управление ресурсами Azure с помощью PowerShell и диспетчер ресурсов](../../azure-resource-manager/management/manage-resource-groups-powershell.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Обзор

При создании новой учетной записи хранения служба Azure создает два 512-разрядных ключа доступа к хранилищу, используемые для проверки подлинности во время доступа к вашей учетной записи хранения. Для безопасности подключений к хранилищу рекомендуется периодически повторно создавать и менять ключи доступа к хранилищу. Два ключа доступа (основной и дополнительный) дают возможность поддерживать подключение к учетной записи хранения с помощью одного ключа доступа в то время, как выполняется повторное создание второго ключа доступа. Этот процесс называется "оборот ключей доступа".

Службы мультимедиа зависят от предоставленного ключа к хранилищу данных. В частности, от ключа доступа к хранилищу зависят указатели, используемые для потоковой передачи или скачивания ресурсов-контейнеров. При создании учетная запись AMS по умолчанию зависит от ключа доступа к основному хранилищу, но пользователь может обновить ключ хранилища в AMS. Необходимо указать службам мультимедиа, какой ключ использовать, выполнив шаги, описанные в данной статье.  

>[!NOTE]
> Если используется несколько учетных записей хранения, необходимо выполнить эту процедуру для каждой из них. Порядок, в котором следует чередовать ключи к хранилищу данных, не является фиксированным. Можно сначала сменить вторичный ключ, а затем первичный ключ, или наоборот.
>
> Перед выполнением описанных в этой статье действий с рабочей учетной записью попробуйте выполнить их на подготовительной учетной записи.
>

## <a name="steps-to-rotate-storage-keys"></a>Инструкции по чередованию ключей к хранилищу данных 
 
 1. Измените первичный ключ учетной записи хранения с помощью командлета PowerShell или портала [Azure](https://portal.azure.com/).
 2. Вызовите командлет Sync-AzMediaServiceStorageKeys с соответствующими параметрами, чтобы принудительно использовать ключи учетной записи хранения для учетной записи мультимедиа.
 
    Следующий пример демонстрирует, как синхронизировать ключи с учетными записями хранения.
  
    `Sync-AzMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId`
  
 3. Подождите один час или около того. Проверьте работу потоковой передачи там, где она используется.
 4. Измените вторичный ключ учетной записи хранения с помощью командлета PowerShell или портала Azure.
 5. Вызовите Sync-AzMediaServiceStorageKeys PowerShell с соответствующими параметрами, чтобы принудительно выбрать новые ключи учетной записи хранения для учетной записи мультимедиа. 
 6. Подождите один час или около того. Проверьте работу потоковой передачи там, где она используется.
 
### <a name="a-powershell-cmdlet-example"></a>Примеры использования командлетов PowerShell 

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

В следующей статье показано, как добавить учетные записи хранения в учетную запись AMS: [Управление активами служб мультимедиа в нескольких учетных записях хранения](./media-services-managing-multiple-storage-accounts.md).

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

### <a name="acknowledgments"></a>Благодарности
Мы выражаем признательность тем, кто помог нам в составлении этого документа, — это Сенк Динджилоглу (Cenk Dingiloglu), Милан Гада (Milan Gada) и Сева Титов.
