---
title: Краткое руководство. Направление веб-трафика с использованием шаблона Resource Manager
titleSuffix: Azure Application Gateway
description: В этом кратком руководстве показано, как с помощью шаблона Resource Manager создать Шлюз приложений Azure, который направляет веб-трафик к виртуальным машинам в серверном пуле.
services: application-gateway
author: vhorne
ms.author: victorh
ms.date: 01/20/2021
ms.topic: quickstart
ms.service: application-gateway
ms.custom:
- mvc
- subject-armqs
- mode-arm
ms.openlocfilehash: dd100361ba5d4ff175e340ced782999e52c720c4
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538438"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway---arm-template"></a>Краткое руководство. Направление веб-трафика с помощью Шлюза приложений Azure и шаблона ARM

В этом кратком руководстве показано, как с помощью шаблона Azure Resource Manager (шаблона ARM) создать Шлюз приложений Azure. Затем вы проверите шлюз приложений, чтобы убедиться, что он правильно работает.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Инструкции в этом кратком руководстве можно также выполнить с помощью [портала Azure](quick-create-portal.md), [Azure PowerShell](quick-create-powershell.md) или [Azure CLI](quick-create-cli.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-qs%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

## <a name="review-the-template"></a>Изучение шаблона

Чтобы упростить работу, в этом шаблоне используется несложная настройка с открытым интерфейсным IP-адресом, базовый прослушиватель для размещения одного сайта в шлюзе приложений, базовое правило маршрутизации запросов и две виртуальные машины в серверном пуле.

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/ag-docs-qs/).

:::code language="json" source="~/quickstart-templates/ag-docs-qs/azuredeploy.json":::

В шаблоне определено несколько ресурсов Azure:

- [**Microsoft.Network/applicationgateways**](/azure/templates/microsoft.network/applicationgateways);
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses): один для шлюза приложений и два для виртуальных машин;
- [**Microsoft.Network/networkSecurityGroups;**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks);
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines): две виртуальные машины;
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces): два для виртуальных машин;
- [**Microsoft.Compute/virtualMachine/extensions**](/azure/templates/microsoft.compute/virtualmachines/extensions): для настройки IIS и веб-страниц.

## <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон ARM в Azure:

1. Выберите элемент **Развертывание в Azure**, чтобы войти в Azure и открыть шаблон. С помощью шаблона будут созданы шлюз приложений, сетевая инфраструктура и две виртуальные машины в серверном пуле, где запущены службы IIS.

   [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-qs%2Fazuredeploy.json)

2. Выберите или создайте группу ресурсов, затем введите имя пользователя и пароль администратора виртуальной машины.
3. Выберите **Review + Create** (Просмотр и создание), а затем выберите **Создать**.

   Для развертывания может потребоваться 20 минут или более.

## <a name="validate-the-deployment"></a>Проверка развертывания

Для создания шлюза приложений не нужно устанавливать службы IIS. Но они устанавливаются, чтобы проверить, создан ли он. Используйте IIS для тестирования шлюза приложений.

1. На странице **Обзор** найдите общедоступный IP-адрес для шлюза приложений. ![Запишите общедоступный IP-адрес шлюза приложений](./media/application-gateway-create-gateway-portal/application-gateway-record-ag-address.png). В качестве альтернативы вы можете выбрать **Все ресурсы** и ввести в поисковое поле *myAGPublicIPAddress*, а затем выбрать этот адрес из результатов поиска. Общедоступный IP-адрес отобразится в Azure на странице **Обзор**.
2. Чтобы перейти по общедоступному IP-адресу, скопируйте и вставьте его в адресную строку браузера.
3. Проверьте ответ. Допустимый ответ подтверждает, что шлюз приложений создан и может успешно подключиться к серверу.

   ![Тестирование шлюза приложений](./media/application-gateway-create-gateway-portal/application-gateway-iistest.png)

   Обновите браузер несколько раз, чтобы отобразились сведения о подключениях к myVM1 и myVM2.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью шлюза приложений, удалите группу ресурсов. При этом будут удалены шлюз приложений и все связанные с ним ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство по управлению веб-трафиком с помощью шлюза приложений и Azure CLI](./tutorial-manage-web-traffic-cli.md)
