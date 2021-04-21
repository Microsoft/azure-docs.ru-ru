---
title: Краткое руководство. Создание фабрики данных в службе Azure Front Door Service с помощью шаблона Azure Resource Manager (ARM)
description: В этом руководстве объясняется, как создать Azure Front Door Service с использованием шаблона Azure Resource Manager (ARM).
services: front-door
documentationcenter: ''
author: duongau
ms.author: duau
editor: ''
ms.date: 09/14/2020
ms.topic: quickstart
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: de8a592f6eecbb43b58a044096e8ba2e0f9b5973
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107539000"
---
# <a name="quickstart-create-a-front-door-using-an-arm-template"></a>Краткое руководство. Создание Front Door с помощью шаблона Resource Manager

В этом руководстве объясняется, как создать Front Door, чтобы настроить высокий уровень доступности для веб-конечной точки с использованием шаблона Azure Resource Manager (ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-front-door-create-basic%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
* IP-адрес или полное доменное имя веб-сайта или веб-приложения.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-front-door-create-basic).

В этом кратком руководстве показано как, создать конфигурацию Front Door с одной серверной системой и одним путем по умолчанию, совпадающим с `/*`.

:::code language="json" source="~/quickstart-templates/101-front-door-create-basic/azuredeploy.json":::

В этом шаблоне определяется один ресурс Azure.

* [**Microsoft.Network/frontDoors**](/azure/templates/microsoft.network/frontDoors)

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите **Попробовать** в следующем блоке кода, чтобы открыть Azure Cloud Shell, и следуйте отображающимся инструкциям, чтобы войти в Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-front-door-create-basic/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Подождите, пока появится запрос из консоли.

1. Выберите **Копировать** из предыдущего блока кода, чтобы скопировать сценарий PowerShell.

1. Щелкните правой кнопкой в области консоли оболочки и выберите **Вставить**.

1. Введите значения.

    При развертывании шаблона создается служба Front Door с одной серверной системой. В этом примере в качестве **backendAddress** используется `microsoft.com`.

    Имя группы ресурсов — это имя проекта с добавлением **rg**.

    > [!NOTE]
    > Чтобы шаблон можно было успешно развернуть, имя **frontDoorName** должно быть глобально уникальным. Если развертывание завершается неудачно, начните с шага 1.

    Развертывание шаблона занимает несколько минут. По завершении выходные данные должны быть следующего вида:

    :::image type="content" source="./media/quickstart-create-front-door-template/front-door-template-deployment-powershell-output.png" alt-text="Результаты развертывания шаблона Resource Manager Front Door в PowerShell":::

Для развертывания шаблона используется Azure PowerShell. В дополнение к Azure PowerShell можно также использовать портал Azure, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Проверка развертывания

1. Войдите на [портал Azure](https://portal.azure.com).

1. В области слева выберите **Группы ресурсов**.

1. Выберите группу ресурсов, созданную при работе с предыдущим разделом. Имя группы ресурсов по умолчанию — это имя проекта с добавлением **rg**.

1. Выберите созданную ранее службу Front Door и щелкните ссылку **Интерфейсный узел**. Откроется веб-браузер, который перенаправляет вас к полному доменному имени сервера, определенному во время создания.

    :::image type="content" source="./media/quickstart-create-front-door-template/front-door-overview.png" alt-text="Обзор Front Door на портале":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если служба Front Door больше не нужна, удалите группу ресурсов. При этом будет удалена служба Front Door и все связанные с ней ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

Используя инструкции из этого краткого руководства, вы создали экземпляр Front Door.

Сведения о добавлении личного домена в Front Door см. в следующих учебниках.

> [!div class="nextstepaction"]
> [Учебники по работе со службой Front Door](front-door-custom-domain.md)
