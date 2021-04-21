---
title: Создание записи и зоны Azure DNS с помощью шаблона Azure Resource Manager (ARM)
titleSuffix: Azure DNS
description: Узнайте, как создать зону и запись DNS в службе DNS Azure. В этом пошаговом руководстве описано, как создать зону и запись DNS, а также как управлять ими с помощью шаблона Azure Resource Manager (ARM).
services: dns
author: duongau
ms.author: duau
ms.date: 09/8/2020
ms.topic: quickstart
ms.service: dns
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 34e810fa8244cf3ff00e57f193624a2c1fa4d755
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107539012"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-an-arm-template"></a>Создание зоны и записи Azure DNS с помощью шаблона ARM

В этом руководстве объясняется, как создать зону DNS, содержащую запись `A`, с использованием шаблона Azure Resource Manager.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azure-dns-new-zone%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-azure-dns-new-zone).

В рамках этого руководства вы создадите уникальную зону DNS с суффиксом `azurequickstart.org`. В зоне также будет размещена запись `A`, указывающая на два IP-адреса.

:::code language="json" source="~/quickstart-templates/101-azure-dns-new-zone/azuredeploy.json":::

В шаблоне определено два ресурса Azure:

- [**Microsoft.Network/dnsZones**](/azure/templates/microsoft.network/dnsZones)
- [**Microsoft.Network/dnsZones/A**](/azure/templates/microsoft.network/dnsZones/A): Используется для создания записи `A` в зоне.

Чтобы найти дополнительные связанные шаблоны Диспетчера трафика Azure, перейдите на страницу [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите **Попробовать** в следующем блоке кода, чтобы открыть Azure Cloud Shell, и следуйте отображающимся инструкциям, чтобы войти в Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-azure-dns-new-zone/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Подождите, пока появится запрос из консоли.

1. Выберите **Копировать** из предыдущего блока кода, чтобы скопировать сценарий PowerShell.

1. Щелкните правой кнопкой в области консоли оболочки и выберите **Вставить**.

1. Введите значения.

    При развертывании шаблона создается зона с одной записью `A`, указывающей на два IP-адреса. Имя группы ресурсов — это имя проекта с добавлением **rg**.

    Развертывание шаблона занимает пару секунд. По завершении выходные данные должны быть следующего вида:

    :::image type="content" source="./media/dns-getstarted-template/create-dns-zone-powershell-output.png" alt-text="Результаты развертывания шаблона Resource Manager зоны Azure DNS в PowerShell":::

Для развертывания шаблона используется Azure PowerShell. В дополнение к Azure PowerShell можно также использовать портал Azure, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Проверка развертывания

1. Войдите на [портал Azure](https://portal.azure.com).

1. В области слева выберите **Группы ресурсов**.

1. Выберите группу ресурсов, созданную при работе с предыдущим разделом. Имя группы ресурсов по умолчанию — это имя проекта с добавлением **rg**.

1. Группа ресурсов должна содержать следующие ресурсы:

    :::image type="content" source="./media/dns-getstarted-template/resource-group-dns-zone.png" alt-text="Группа ресурсов развертывания зоны DNS":::

1. Выберите зону DNS с суффиксом `azurequickstart.org`, чтобы убедиться, что зона создана правильно с записью `A`, ссылающейся на значения `1.2.3.4` и `1.2.3.5`.

    :::image type="content" source="./media/dns-getstarted-template/dns-zone-overview.png" alt-text="Развертывание зоны DNS":::

1. Скопируйте имя одного из серверов из данных, полученных на предыдущем этапе.

1. Откройте командную строку и выполните приведенную ниже команду.

   ```cmd
   nslookup www.<dns zone name> <name server name>
   ```

   Пример:

   ```cmd
   nslookup www.2lwynbseszpam.azurequickstart.org ns1-09.azure-dns.com.
   ```

   Результат будет подобен приведенному на следующем снимке экрана.

    :::image type="content" source="./media/dns-getstarted-template/dns-zone-validation.png" alt-text="nslookup зоны DNS":::

Имя узла `www.2lwynbseszpam.azurequickstart.org` разрешается как `1.2.3.4` и `1.2.3.5`, как и было настроено. Такой результат подтверждает, что разрешение имен работает правильно.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с использованием зоны DNS, удалите группу ресурсов. При этом будет удалена зона DNS и все связанные с ней ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать:

- Зона DNS
- Запись `A`

Теперь, когда вы создали свою первую зону и запись DNS с помощью шаблона Resource Manager, можно создать записи для веб-приложения в личном домене.

> [!div class="nextstepaction"]
> [Создание записей DNS для веб-приложения в пользовательском домене](./dns-web-sites-custom-domain.md)
