---
title: Сведения об управлении учетными записями базы данных в Azure Cosmos DB
description: Узнайте, как управлять ресурсами Azure Cosmos DB с помощью шаблонов портал Azure, PowerShell, CLI и Azure Resource Manager
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 01/06/2021
ms.author: mjbrown
ms.openlocfilehash: d542e2b4e5db86fd3354514790e718f0694a09a5
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102489758"
---
# <a name="manage-an-azure-cosmos-account"></a>Управление учетной записью Azure Cosmos
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описывается, как управлять различными задачами в учетной записи Azure Cosmos с помощью портала Azure, Azure PowerShell, Azure CLI и шаблонов Azure Resource Manager.

## <a name="create-an-account"></a>Создание учетной записи

### <a name="azure-portal"></a><a id="create-database-account-via-portal"></a>Портал Azure

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

### <a name="azure-cli"></a><a id="create-database-account-via-cli"></a>Интерфейс командной строки Azure

См. статью [Создание учетной записи Azure Cosmos DB с помощью Azure CLI](manage-with-cli.md#create-an-azure-cosmos-db-account)

### <a name="azure-powershell"></a><a id="create-database-account-via-ps"></a>Azure PowerShell

См. статью [Создание учетной записи Azure Cosmos DB с помощью PowerShell](manage-with-powershell.md#create-account) .

### <a name="azure-resource-manager-template"></a><a id="create-database-account-via-arm-template"></a>Шаблон Azure Resource Manager

См. раздел [Создание учетной записи Azure Cosmos DB с помощью шаблонов Azure Resource Manager](./manage-with-templates.md)

## <a name="addremove-regions-from-your-database-account"></a>Добавление и удаление регионов из учетной записи базы данных

### <a name="azure-portal"></a><a id="add-remove-regions-via-portal"></a>Портал Azure

1. Войдите на [портал Azure](https://portal.azure.com).

1. Перейдите к учетной записи Azure Cosmos и откройте меню **Глобальная репликация данных**.

1. Чтобы добавить регионы, выберите шестиугольники на карте с **+** меткой, которая соответствует нужным регионам. Кроме того, вы можете добавить регион, щелкнув параметр **+ Добавить регион** и выбрав регион из раскрывающегося меню.

1. Чтобы удалить регионы, очистите один или несколько регионов на карте, выбрав синие шестиугольники с флажками. Или выберите "мусорной" значок (🗑) рядом с регионом с правой стороны.

1. Нажмите кнопку **ОК**, чтобы сохранить изменения.

   :::image type="content" source="./media/how-to-manage-database-account/add-region.png" alt-text="Меню добавления и удаления регионов":::

В режиме записи в одном регионе вы не можете удалить регион записи. Перед удалением текущего региона записи нужно выполнить отработку отказа в другой регион.

В режиме записи в нескольких регионах вы можете добавлять и удалять любой регион при условии, что останется по крайней мере один регион.

### <a name="azure-cli"></a><a id="add-remove-regions-via-cli"></a>Интерфейс командной строки Azure

См. раздел [Добавление и удаление регионов с Azure CLI](manage-with-cli.md#add-or-remove-regions)

### <a name="azure-powershell"></a><a id="add-remove-regions-via-ps"></a>Azure PowerShell

См. раздел [Добавление и удаление регионов с помощью PowerShell](manage-with-powershell.md#update-account) .

## <a name="configure-multiple-write-regions"></a><a id="configure-multiple-write-regions"></a>Настройка нескольких регионов записи

### <a name="azure-portal"></a><a id="configure-multiple-write-regions-portal"></a>Портал Azure

Откройте вкладку **Глобальная репликация данных** и выберите **Включить**, чтобы включить операции записи в нескольких регионах. После включения операций записи в нескольких регионах все регионы чтения, которые в текущий момент есть в вашей учетной записи, станут регионами как для чтения, так и для записи.

:::image type="content" source="./media/how-to-manage-database-account/single-to-multi-master.png" alt-text="Снимок экрана с учетной записью Cosmos для нескольких регионов":::

### <a name="azure-cli"></a><a id="configure-multiple-write-regions-cli"></a>Интерфейс командной строки Azure

См. раздел [Включение нескольких регионов записи с помощью Azure CLI](manage-with-cli.md#enable-multiple-write-regions)

### <a name="azure-powershell"></a><a id="configure-multiple-write-regions-ps"></a>Azure PowerShell

См. раздел [Включение нескольких регионов записи с помощью PowerShell](manage-with-powershell.md#multi-region-writes) .

### <a name="resource-manager-template"></a><a id="configure-multiple-write-regions-arm"></a>Шаблон Resource Manager

Учетную запись можно перенести из одного региона записи в несколько регионов записи, развернув шаблон диспетчер ресурсов, используемый для создания учетной записи и настройки `enableMultipleWriteLocations: true` . Следующий шаблон Azure Resource Manager — это самый простой шаблон, который позволяет развернуть учетную запись Azure Cosmos для API SQL с двумя регионами и несколькими включенными расположениями записи.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "String"
        },
        "location": {
            "type": "String",
            "defaultValue": "[resourceGroup().location]"
        },
        "primaryRegion":{
            "type":"string",
            "metadata": {
                "description": "The primary replica region for the Cosmos DB account."
            }
        },
        "secondaryRegion":{
            "type":"string",
            "metadata": {
              "description": "The secondary replica region for the Cosmos DB account."
          }
        }
    },
    "resources": [
        {
            "type": "Microsoft.DocumentDb/databaseAccounts",
            "kind": "GlobalDocumentDB",
            "name": "[parameters('name')]",
            "apiVersion": "2019-08-01",
            "location": "[parameters('location')]",
            "tags": {},
            "properties": {
                "databaseAccountOfferType": "Standard",
                "consistencyPolicy": { "defaultConsistencyLevel": "Session" },
                "locations":
                [
                    {
                        "locationName": "[parameters('primaryRegion')]",
                        "failoverPriority": 0,
                        "isZoneRedundant": false
                    },
                    {
                        "locationName": "[parameters('secondaryRegion')]",
                        "failoverPriority": 1,
                        "isZoneRedundant": false
                    }
                ],
                "enableMultipleWriteLocations": true
            }
        }
    ]
}
```

## <a name="enable-automatic-failover-for-your-azure-cosmos-account"></a><a id="automatic-failover"></a>Включение автоматического перехода на другой ресурс для учетной записи Azure Cosmos

Если какой-либо регион становится недоступным, возможность автоматического перехода на другой ресурс позволяет Azure Cosmos DB выполнять отработку отказа в регион с наибольшим приоритетом отработки отказа без каких-либо действий со стороны пользователя. При включенном автоматическом переходе на другой ресурс приоритет региона можно изменить. В учетной записи должно быть два или более регионов, чтобы включить автоматический переход на другой ресурс.

### <a name="azure-portal"></a><a id="enable-automatic-failover-via-portal"></a>Портал Azure

1. В учетной записи Azure Cosmos откройте панель **Глобальная репликация данных**.

2. В верхней части панели выберите **Автоматический переход на другой ресурс**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Меню глобальной репликации данных":::

3. На панели **Автоматический переход на другой ресурс** убедитесь, что для параметра **Включить автоматическую отработку отказа** установлено значение **ВКЛ**. 

4. Щелкните **Сохранить**.

   :::image type="content" source="./media/how-to-manage-database-account/automatic-failover.png" alt-text="Меню автоматического перехода на другой ресурс на портале":::

### <a name="azure-cli"></a><a id="enable-automatic-failover-via-cli"></a>Интерфейс командной строки Azure

См. раздел [Включение автоматической отработки отказа с помощью Azure CLI](manage-with-cli.md#enable-automatic-failover)

### <a name="azure-powershell"></a><a id="enable-automatic-failover-via-ps"></a>Azure PowerShell

См. раздел [Включение автоматической отработки отказа с помощью PowerShell](manage-with-powershell.md#enable-automatic-failover) .

## <a name="set-failover-priorities-for-your-azure-cosmos-account"></a>Настройка приоритетов при отработке отказа для учетной записи Azure Cosmos

После настройки автоматического перехода на другой ресурс в учетной записи Cosmos можно изменить приоритет при отработке отказа для других регионов.

> [!IMPORTANT]
> Вы не можете изменить регион записи (приоритет отработки отказа равен нулю) после настройки учетной записи для автоматического перехода на другой ресурс. Чтобы изменить регион записи, необходимо отключить автоматический переход на другой ресурс и выполнить переход на другой ресурс вручную.

### <a name="azure-portal"></a><a id="set-failover-priorities-via-portal"></a>Портал Azure

1. В учетной записи Azure Cosmos откройте панель **Глобальная репликация данных**.

2. В верхней части панели выберите **Автоматический переход на другой ресурс**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Меню глобальной репликации данных":::

3. На панели **Автоматический переход на другой ресурс** убедитесь, что для параметра **Включить автоматическую отработку отказа** установлено значение **ВКЛ**.

4. Чтобы изменить приоритет при отработке отказа, щелкните и перетащите регионы чтения, щелкнув три точки в левой части строки, которые появляются при наведении указателя мыши на строку.

5. Щелкните **Сохранить**.

   :::image type="content" source="./media/how-to-manage-database-account/automatic-failover.png" alt-text="Меню автоматического перехода на другой ресурс на портале":::

### <a name="azure-cli"></a><a id="set-failover-priorities-via-cli"></a>Интерфейс командной строки Azure

См. раздел [Установка приоритета отработки отказа с помощью Azure CLI](manage-with-cli.md#set-failover-priority)

### <a name="azure-powershell"></a><a id="set-failover-priorities-via-ps"></a>Azure PowerShell

См. раздел [Установка приоритета отработки отказа с помощью PowerShell](manage-with-powershell.md#modify-failover-priority)

## <a name="perform-manual-failover-on-an-azure-cosmos-account"></a><a id="manual-failover"></a>Выполнение перехода на другой ресурс вручную для учетной записи Azure Cosmos

> [!IMPORTANT]
> Для успеха этой операции учетная запись Azure Cosmos должна быть настроена на переход на другой ресурс вручную.

Процесс выполнения перехода на другой ресурс вручную включает изменение региона записи учетной записи (приоритет отработки отказа = 0) на другой регион, настроенный для учетной записи.

> [!NOTE]
> Для учетных записей с несколькими регионами записи невозможно выполнить отработку отказа вручную. Для приложений, использующих пакет SDK для Azure Cosmos, пакет SDK определит, когда регион станет недоступным, а затем автоматически перенаправит его в ближайший регион.

### <a name="azure-portal"></a><a id="enable-manual-failover-via-portal"></a>Портал Azure

1. Перейдите к учетной записи Azure Cosmos и откройте меню **Глобальная репликация данных**.

2. В верхней части меню выберите **Переход на другой ресурс вручную**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Меню глобальной репликации данных":::

3. В меню **Переход на другой ресурс вручную** выберите новый регион записи. Установите флажок, чтобы указать, что вы понимаете, что этот параметр изменяет ваш регион записи.

4. Нажмите кнопку **ОК**, чтобы активировать отработку отказа.

   :::image type="content" source="./media/how-to-manage-database-account/manual-failover.png" alt-text="Меню перехода на другой ресурс вручную на портале":::

### <a name="azure-cli"></a><a id="enable-manual-failover-via-cli"></a>Интерфейс командной строки Azure

См. статью [Активация перехода на другой ресурс вручную с помощью Azure CLI](manage-with-cli.md#trigger-manual-failover)

### <a name="azure-powershell"></a><a id="enable-manual-failover-via-ps"></a>Azure PowerShell

См. раздел [Активация перехода на другой ресурс вручную с помощью PowerShell](manage-with-powershell.md#trigger-manual-failover) .

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения и примеры по управлению учетной записью Azure Cosmos, а также базами данных и контейнерами см. в следующих статьях:

* [Manage Azure Cosmos DB SQL API resources using PowerShell](manage-with-powershell.md) (Управление ресурсами API SQL Azure Cosmos DB с помощью PowerShell)
* [Manage Azure Cosmos resources using Azure CLI](manage-with-cli.md) (Управление ресурсами Azure Cosmos с помощью Azure CLI)