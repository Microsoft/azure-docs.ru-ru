---
title: Как переместить ресурс службы в регионы
titleSuffix: Azure Cognitive Search
description: В этой статье показано, как переместить ресурсы Azure Когнитивный поиск из одного региона в другой в облаке Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 09/10/2020
ms.openlocfilehash: a5050958f01743ff3c6fdcdecfee3067b1d2073e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98934267"
---
# <a name="move-your-azure-cognitive-search-service-to-another-azure-region"></a>Перемещение службы Когнитивный поиск Azure в другой регион Azure

Иногда клиенты запрашивают о перемещении службы поиска в другой регион. В настоящее время нет встроенного механизма или инструментария для помощи в этой задаче, но эта статья поможет вам понять, как выполнить действия, выполняемые вручную для достижения того же результата.

> [!NOTE]
> В портал Azure все службы имеют команду **Экспорт шаблона** . В случае Когнитивный поиск Azure эта команда создает базовое определение службы (имя, расположение, уровень, реплику и число секций), но не распознает содержимое службы, а также не выполняет перенос ключей, ролей или журналов. Хотя эта команда существует, мы не рекомендуем использовать ее для перемещения службы поиска.

## <a name="prerequisites"></a>Предварительные условия

+ Убедитесь, что службы и функции, используемые вашей учетной записью, поддерживаются в целевом регионе.

+ Для функций предварительной версии убедитесь, что подписка утверждена для целевого региона.

## <a name="prepare-and-move"></a>Подготовка и перемещение

1. Выявление зависимостей и связанных служб для понимания полного влияния перемещения службы на тот случай, если необходимо переместить не только Azure Когнитивный поиск.

   Служба хранилища Azure используется для ведения журнала, создания хранилища знаний и является широко используемым внешним источником данных для обогащения и индексирования искусственного интеллекта. Cognitive Services является зависимостью в обогащении искусственного интеллекта. Как Cognitive Services, так и служба поиска должны находиться в одном регионе, если используется обогащение искусственного интеллекта.

1. Создайте инвентаризацию всех объектов в службе, чтобы вы могли узнать, что перемещать: индексы, карты синонимов, индексаторы, источники данных, навыков. Если вы включили ведение журнала, создайте и заархивируйте все отчеты, которые могут потребоваться для записи журнала.

1. Проверьте цены и доступность в новом регионе, чтобы обеспечить доступность Azure Когнитивный поиск и всех связанных служб в новом регионе. Большинство функций доступны во всех регионах, но доступность некоторых функций предварительной версии ограничена.

1. Создание службы в новом регионе и повторная публикация из исходного кода всех существующих индексов, сопоставлений синонимов, индексаторов, источников данных и навыков. Помните, что имена служб должны быть уникальными, поэтому нельзя повторно использовать существующее имя. Проверьте каждый набор навыков, чтобы узнать, являются ли подключения к Cognitive Services по-прежнему допустимыми в условиях того же требования к региону. Кроме того, если созданы хранилища знаний, проверьте строки подключения для службы хранилища Azure, если вы используете другую службу.

1. Перезагрузить индексы и хранилища знаний, если это применимо. Вы будете использовать код приложения для отправки данных JSON в индекс или повторного запуска индексаторов для извлечения документов из внешних источников. 

1. Включите ведение журнала и, если вы используете их, повторно создайте роли безопасности.

1. Обновите клиентские приложения и наборы тестов, чтобы использовать новое имя службы и ключи API, а также протестируйте все приложения.

## <a name="discard-or-clean-up"></a>Отмена или очистка

Удалите старую службу после того, как новая служба будет полностью протестирована и работоспособна. При удалении службы автоматически удаляются все содержимое, связанное со службой.

## <a name="next-steps"></a>Дальнейшие действия

Следующие ссылки помогут вам получить дополнительные сведения при выполнении описанных выше действий.

+ [Цены и регионы Azure Когнитивный поиск](https://azure.microsoft.com/pricing/details/search/)
+ [Выбор уровня](search-sku-tier.md)
+ [Создание службы поиска](search-create-service-portal.md)
+ [Загрузить документы поиска](search-what-is-data-import.md)
+ [Включение ведения журналов](search-monitor-logs.md)


<!-- To move your Azure Cognitive Service account from one region to another, you will create an export template to move your subscription(s). After moving your subscription, you will need to move your data and recreate your service.

In this article, you'll learn how to:

> [!div class="checklist"]
> * Export a template.
> * Modify the template: adding the target region, search and storage account names.
> * Deploy the template to create the new search and storage accounts.
> * Verify your service status in the new region
> * Clean up resources in the source region.

## Prerequisites

- Ensure that the services and features that your account uses are supported in the target region.

- For preview features, ensure that your subscription is allowlisted for the target region. For more information about preview features, see [knowledge stores](./knowledge-store-concept-intro.md), [incremental enrichment](./cognitive-search-incremental-indexing-conceptual.md), and [private endpoint](./service-create-private-endpoint.md).

## Assessment and planning

When you move your search service to the new region, you will need to [move your data to the new storage service](../storage/common/storage-account-move.md?tabs=azure-portal#configure-the-new-storage-account) and then rebuild your indexes, skillsets and knowledge stores. You should record current settings and copy json files to make the rebuilding of your service easier and faster.

## Moving your search service's resources

To start you will export and then modify a Resource Manager template.

### Export a template

1. Sign in to the [Azure portal](https://portal.azure.com).

2. Go to your Resource Group page.

> [!div class="mx-imgBorder"]
> ![Resource Group page example](./media/search-move-resource/export-template-sample.png)

3. Select **All resources**.

3. In the left hand navigation menu select **Export template**.

4. Choose **Download** in the **Export template** page.

5. Locate the .zip file that you downloaded from the portal, and unzip that file to a folder of your choice.

The zip file contains the .json files that comprise the template and scripts to deploy the template.

### Modify the template

You will modify the template by changing the search and storage account names and regions. The names must follow the rules for each service and region naming conventions. 

To obtain region location codes, see [Azure Locations](https://azure.microsoft.com/global-infrastructure/locations/).  The code for a region is the region name with no spaces, **Central US** = **centralus**.

1. In the Azure portal, select **Create a resource**.

2. In **Search the Marketplace**, type **template deployment**, and then press **ENTER**.

3. Select **Template deployment**.

4. Select **Create**.

5. Select **Build your own template in the editor**.

6. Select **Load file**, and then follow the instructions to load the **template.json** file that you downloaded and unzipped in the previous section.

7. In the **template.json** file, name the target search and storage accounts by setting the default value of the search and storage account names. 

8. Edit the **location** property in the **template.json** file to the target region for both your search and storage services. This example sets the target region to `centralus`.

```json
},
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Search/searchServices",
            "apiVersion": "2020-03-13",
            "name": "[parameters('searchServices_target_region_search_name')]",
            "location": "centralus",
            "sku": {
                "name": "standard"
            },
            "properties": {
                "replicaCount": 1,
                "partitionCount": 1,
                "hostingMode": "Default"
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "[parameters('storageAccounts_tagetstorageregion_name')]",
            "location": "centralus",
            "sku": {
                "name": "Standard_RAGRS",
                "tier": "Standard"
            },
```

### Deploy the template

1. Save the **template.json** file.

2. Enter or select the property values:

- **Subscription**: Select an Azure subscription.

- **Resource group**: Select **Create new** and give the resource group a name.

- **Location**: Select an Azure location.

3. Click the **I agree to the terms and conditions stated above** checkbox, and then click the **Select Purchase** button.

## Verifying your services' status in new region

To verify the move, open the new resource group and your services will be listed with the new region.

To move your data from your source region to the target region, please see this article's guidelines for [moving your data to the new storage account](../storage/common/storage-account-move.md?tabs=azure-portal#move-data-to-the-new-storage-account).

## Clean up resources in your original region

To commit the changes and complete the move of your service account, delete the source service account.

## Next steps

[Create an index](./search-get-started-portal.md)

[Create a skillset](./cognitive-search-quickstart-blob.md)

[Create a knowledge store](./knowledge-store-create-portal.md) -->