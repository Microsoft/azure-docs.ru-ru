---
title: Создание учетной записи Azure Maps с помощью шаблона ARM | Карты Microsoft Azure
description: Узнайте, как создать учетную запись Azure Maps с помощью шаблона Azure Resource Manager (ARM).
author: philmea
ms.author: philmea
ms.date: 10/20/2020
ms.topic: how-to
ms.service: azure-maps
ms.openlocfilehash: c715c0639e962f76f669515c1d2c826c8cf6cc9e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92525187"
---
# <a name="create-your-azure-maps-account-using-an-arm-template"></a>Создание учетной записи Azure Maps с помощью шаблона ARM

Учетную запись Azure Maps можно создать с помощью шаблона Azure Resource Manager (ARM). Если у вас есть учетная запись, вы можете реализовать API-интерфейсы на своем веб-сайте или в мобильном приложении.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-maps-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей необходимо сделать следующее:

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-maps-create/).

:::code language="json" source="~/quickstart-templates/101-maps-create/azuredeploy.json":::

Ресурс учетной записи Azure Maps определен в этом шаблоне:

* [**Microsoft. Maps/Accounts**](/azure/templates/microsoft.maps/accounts): создание учетной записи Azure Maps.

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон создает учетную запись Azure Maps.

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-maps-create%2Fazuredeploy.json)

2. Введите или выберите следующие значения.

    ![Портал развертывания шаблона ARM](./media/how-to-create-template/create-account-using-template-portal.png)

    Если он не указан, используйте значение по умолчанию, чтобы создать учетную запись Azure Maps.

    * **Подписка**. Выберите нужную подписку Azure.
    * **Группа ресурсов**. Щелкните **Создать**, введите уникальное имя новой группы ресурсов и нажмите кнопку **ОК**.
    * **Расположение**. Выберите расположение. Например, **Западная часть США 2**.
    * **Имя учетной записи**. Введите имя учетной записи Azure Maps, которое должно быть глобально уникальным.
    * **Ценовая** Категория. Выберите соответствующую ценовую категорию, значение по умолчанию для шаблона — S0.

3. Выберите **Review + create** (Просмотреть и создать). 
4. Подтвердите параметры на странице "Проверка" и нажмите кнопку **создать**. После успешного развертывания Azure Maps вы получите уведомление:

    ![Уведомление на портале развертывания шаблона ARM](./media/how-to-create-template/resource-manager-template-portal-deployment-notification.png)

Портал Azure используется для развертывания шаблона. Вы можете также использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Вы можете использовать портал Azure для проверки учетной записи Azure Maps и просмотра ключей. Для перечисления ключей учетной записи также можно использовать следующий сценарий Azure CLI.

```azurecli-interactive
az maps account keys list --name MyMapsAccount --resource-group MyResourceGroup
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите группу ресурсов, которая также удаляет учетную запись Azure Maps, если она больше не нужна. Чтобы удалить группу ресурсов с помощью Azure CLI, выполните следующие действия.

```azurecli-interactive
az group delete --name MyResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать больше о Azure Maps и Azure Resource Manager, перейдите к статьям, приведенным ниже.

- Создание [демонстрационного приложения](quick-demo-map-app.md) Azure Maps
- Дополнительные сведения о [шаблонах ARM](../azure-resource-manager/templates/overview.md)