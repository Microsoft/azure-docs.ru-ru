---
title: Создание Центра уведомлений Azure с помощью шаблона Azure Resource Manager
description: Узнайте, как создать концентратор уведомлений Azure с помощью шаблона Azure Resource Manager (шаблона ARM).
services: notification-hubs
author: sethmanheim
ms.author: sethm
ms.reviewer: thsomasu
ms.date: 08/04/2020
ms.lastreviewed: 05/15/2020
ms.topic: quickstart
ms.service: notification-hubs
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: a328d6b8942c3209e13dc91a2fb892e98e3016f6
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533470"
---
# <a name="quickstart-create-a-notification-hub-using-an-arm-template"></a>Краткое руководство. Создание концентратора уведомлений с помощью шаблона Resource Manager

Центры уведомлений Azure обеспечивают простой в использовании и масштабируемый механизм отправки push-уведомлений, который позволяет отправлять уведомления на любую платформу (iOS, Android, Windows, Kindle и т. д.) c любой серверной части (облачной или локальной). Дополнительные сведения о службе см. в статье [Что такое Центры уведомлений Azure?](notification-hubs-push-notification-overview.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

В этом кратком руководстве используется шаблон Azure Resource Manager (шаблон ARM) для создания пространства имен Центров уведомлений Azure и концентратора уведомлений **MyHub** в этом пространстве имен.

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-notification-hub%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-notification-hub/).

:::code language="json" source="~/quickstart-templates/101-notification-hub/azuredeploy.json":::

* [Справочник по шаблонам пространства имен Microsoft.NotificationHubs](/azure/templates/microsoft.notificationhubs/namespaces).
* [Справочник по шаблонам Microsoft.NotificationHubs namespaces/notificationHubs](/azure/templates/microsoft.notificationhubs/namespaces/notificationhubs).

## <a name="deploy-the-template"></a>Развертывание шаблона

Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон принимает имя пространства имен Центров уведомлений в качестве параметра. Затем он создает пространство имен с этим именем и Центр уведомлений **MyHub** в этом пространстве имен.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-notification-hub%2Fazuredeploy.json)

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Вы можете использовать портал Azure для проверки развернутых ресурсов или использовать скрипт Azure CLI или Azure PowerShell, чтобы получить список развернутых пространств имен Центров уведомлений и концентраторов:

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzNotificationHub -Namespace "nhtestns123" -ResourceGroup "ContosoNotificationsGroup"
Get-AzNotificationHubsNamespace -Namespace "nhtestns123"
```

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
az notification-hub show --resource-group ContosoNotificationsGroup --namespace-name nhtestns123 --name MyHub
az notification-hub namespace show --resource-group ContosoNotificationsGroup --name nhtestns123
```

---
The output looks similar to: null
':::image type="content" source="media/create-notification-hub-template/verify-deploy.png" alt-text="Verify deployment"::': null
---

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите ненужную группу ресурсов. Ресурсы в ней будут также удалены.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по созданию шаблона см. в следующей статье:

> [!div class="nextstepaction"]
> Создание и развертывание первого шаблона ARM[
