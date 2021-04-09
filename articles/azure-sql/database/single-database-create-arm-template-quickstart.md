---
title: 'Azure Resource Manager: Создание отдельной базы данных'
description: Создайте отдельную базу данных в Базе данных SQL Azure с помощью шаблона Azure Resource Manager.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: subject-armqs sqldbrb=1
ms.devlang: ''
ms.topic: quickstart
author: mumian
ms.author: jgao
ms.reviewer: sstein
ms.date: 06/24/2020
ms.openlocfilehash: b7883144509760c9670decd50062d2595b8dc495
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96187091"
---
# <a name="quickstart-create-a-single-database-in-azure-sql-database-using-an-arm-template"></a>Краткое руководство. Создание отдельной базы данных в Базе данных SQL Azure с помощью шаблона ARM

Создание [отдельной базы данных](single-database-overview.md) — это самый быстрый и простой вариант для создания базы данных в Базе данных SQL Azure. В этом кратком руководстве описано, как создать отдельную базу данных с помощью шаблона Azure Resource Manager (шаблона ARM).

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-sql-database%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).

## <a name="review-the-template"></a>Изучение шаблона

Отдельная база данных имеет заданный набор ресурсов вычислений, памяти, операций ввода-вывода и хранения и доступна в [двух моделях приобретения](purchasing-models.md). При создании отдельной базы данных можете также определить [сервер](logical-servers.md), чтобы управлять им и разместить его в [группе ресурсов Azure](../../active-directory-b2c/overview.md) в указанном регионе.

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-sql-database/).

:::code language="json" source="~/quickstart-templates/101-sql-database/azuredeploy.json":::

В шаблоне определены следующие ресурсы:

- [**Microsoft.Sql/servers**](/azure/templates/microsoft.sql/servers)
- [**Microsoft.Sql/servers/databases**](/azure/templates/microsoft.sql/servers/databases)

Дополнительные примеры шаблонов Базы данных Azure SQL можно найти в [шаблонах быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sql&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

Выберите **Попробовать** из следующего блока кода PowerShell, чтобы открыть Azure Cloud Shell.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
$location = Read-Host -Prompt "Enter an Azure location (i.e. centralus)"
$adminUser = Read-Host -Prompt "Enter the SQL server administrator username"
$adminPassword = Read-Host -Prompt "Enter the SQl server administrator password" -AsSecureString

$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-sql-database/azuredeploy.json" -administratorLogin $adminUser -administratorLoginPassword $adminPassword

Read-Host -Prompt "Press [ENTER] to continue ..."
```

## <a name="validate-the-deployment"></a>Проверка развертывания

См. дополнительные сведения о том, как [создать запрос к базе данных](single-database-create-quickstart.md#query-the-database).

## <a name="clean-up-resources"></a>Очистка ресурсов

Сохраните эту группу ресурсов, сервер и отдельную базу данных, чтобы перейти к [дальнейшим действиям](#next-steps). Далее показано, как установить подключение и выполнить запрос к базе данных с помощью разных методов.

Чтобы удалить группу ресурсов, сделайте следующее:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

- Создайте правило брандмауэра на уровне сервера, чтобы подключиться к отдельной базе данных с помощью локальных или удаленных средств. Подробные сведения см. в разделе [Создание правила брандмауэра на уровне сервера](firewall-create-server-level-portal-quickstart.md).
- Создав правило брандмауэра на уровне сервера, вы сможете [устанавливать подключение и выполнять запросы к базе данных](connect-query-content-reference-guide.md) с помощью разных средств и языков.
  - [подключиться и создать запрос с помощью SQL Server Management Studio](connect-query-ssms.md);
  - [подключиться и создать запрос с помощью Azure Data Studio](/sql/azure-data-studio/quickstart-sql-database?toc=%2fazure%2fsql-database%2ftoc.json).
- Чтобы создать отдельную базу данных с помощью Azure CLI, ознакомьтесь с [этими примерами Azure CLI](az-cli-script-samples-content-guide.md).
- Чтобы создать отдельную базу данных с помощью Azure PowerShell, ознакомьтесь со статьей [Примеры Azure PowerShell для базы данных SQL Azure](powershell-script-content-guide.md).
- Сведения о создании шаблонов ARM см. в статье [Руководство. Создание и развертывание шаблона ARM](../../azure-resource-manager/templates/template-tutorial-create-first-template.md).