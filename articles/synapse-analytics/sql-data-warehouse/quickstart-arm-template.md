---
title: Создание выделенного пула SQL (ранее — Хранилище данных SQL) с помощью шаблона Azure Resource Manager
description: Узнайте, как создать пул SQL Azure Synapse Analytics с помощью шаблона Azure Resource Manager.
services: azure-resource-manager
author: julieMSFT
ms.service: azure-resource-manager
ms.topic: quickstart
ms.author: jrasnick
ms.date: 06/09/2020
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 73f4baf1e48b5495d513fbabe66763538c8cca57
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107568477"
---
# <a name="quickstart-create-an-azure-synapse-analytics-dedicated-sql-pool-formerly-sql-dw-by-using-an-arm-template"></a>Краткое руководство. Создание выделенного пула SQL (ранее — Хранилище данных SQL) в Azure Synapse Analytics с помощью шаблона Resource Manager

Этот шаблон Azure Resource Manager (шаблон Resource Manager) создает выделенный пул SQL (ранее — Хранилище данных SQL) с включенным прозрачным шифрованием данных. Выделенный пул SQL (ранее — Хранилище данных SQL) относится к функциям хранения корпоративных данных, которые стали общедоступны в Azure Synapse.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-sql-data-warehouse-transparent-encryption-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/201-sql-data-warehouse-transparent-encryption-create/).

:::code language="json" source="~/quickstart-templates/201-sql-data-warehouse-transparent-encryption-create/azuredeploy.json":::

Шаблон определяет один ресурс:

- [Microsoft.Sql/servers](/azure/templates/microsoft.sql/servers)

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Этот шаблон создает выделенный пул SQL (ранее — Хранилище данных SQL).
   
   [![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-sql-data-warehouse-transparent-encryption-create%2Fazuredeploy.json)

1. Введите или измените следующие значения:

   * **Подписка**: Выберите подписку Azure.
   * **Группа ресурсов.** Выберите **Создать**, введите уникальное имя группы ресурсов и щелкните **ОК**. Использование новой группы ресурсов упростит очистку ресурсов.
   * **Регион**. Выберите регион.  Например, **центральная часть США**.
   * **Имя сервера SQL Server.** Примите имя по умолчанию или введите имя сервера SQL Server.
   * **Имя входа администратора SQL.** Введите имя пользователя администратора для сервера SQL Server.
   * **Пароль администратора SQL.** Введите пароль администратора для сервера SQL Server.
   * **Имя хранилища данных.** Введите имя выделенного пула SQL.
   * **Прозрачное шифрование данных.** Оставьте значение по умолчанию ("Включено"). 
   * **Целевой уровень обслуживания.** Примите значение по умолчанию (DW400c).
   * **Расположение.** Примите расположение группы ресурсов по умолчанию.
   * **Просмотр и создание.** Установите этот флажок.
   * **Создание**. Установите этот флажок.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Вы можете использовать портал Azure, чтобы проверить развернутые ресурсы, либо скрипт Azure CLI или Azure PowerShell, чтобы получить список развернутых ресурсов.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the resource group where your dedicated SQL pool (formerly SQL DW) exists:" &&
read resourcegroupName &&
az resource list --resource-group $resourcegroupName 
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the resource group name where your dedicated SQL pool (formerly SQL DW) account exists"
(Get-AzResource -ResourceType "Microsoft.Sql/servers/databases" -ResourceGroupName $resourceGroupName).Name
 Write-Host "Press [ENTER] to continue..."
```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Если группа ресурсов вам не нужна, удалите ее с помощью Azure CLI или Azure PowerShell.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Дальнейшие действия

Пройдя это краткое руководство, вы создали выделенный пул SQL (ранее — Хранилище данных SQL) с помощью шаблона Resource Manager и проверили развертывание. Дополнительные сведения об Azure Synapse Analytics и Azure Resource Manager см. в статьях ниже.

- См. статью [Сведения об Azure Synapse Analytics](sql-data-warehouse-overview-what-is.md)
- Сведения об [Azure Resource Manager](../../azure-resource-manager/management/overview.md)
- [Создание и развертывание первого шаблона ARM](../../azure-resource-manager/templates/template-tutorial-create-first-template.md)
