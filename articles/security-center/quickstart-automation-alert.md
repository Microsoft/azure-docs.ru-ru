---
title: Создание задачи автоматизации защиты при получении определенных оповещений системы безопасности с помощью шаблона Azure Resource Manager (ARM)
description: Узнайте, как с помощью шаблона Azure Resource Manager (ARM) создать задачу автоматизации в Центре безопасности Azure для активации приложения логики при получении определенных оповещений Центром безопасности.
services: azure-resource-manager
author: memildin
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: memildin
ms.date: 08/20/2020
ms.openlocfilehash: e53a5c4840cee0651090bec0b68cc53e13102299
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97705495"
---
# <a name="quickstart-create-an-automatic-response-to-a-specific-security-alert-using-an-arm-template"></a>Краткое руководство. Создание автоматического ответа на определенное оповещение системы безопасности с помощью шаблона ARM

В этом кратком руководстве описано, как использовать шаблон Azure Resource Manager (ARM) для создания службы автоматизации рабочего процесса, активирующего приложение логики при получении Центром безопасности Azure определенных оповещений системы безопасности.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-securitycenter-create-automation-for-alertnamecontains%2fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Список ролей и разрешений, необходимых для работы с функцией автоматизации рабочих процессов в Центре безопасности Azure, см. в статье [Автоматизация рабочих процессов](workflow-automation.md).

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-securitycenter-create-automation-for-alertnamecontains/).

:::code language="json" source="~/quickstart-templates/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json":::

### <a name="relevant-resources"></a>Материалы

- [**Microsoft.Security/automations**](/azure/templates/microsoft.security/automations). Служба автоматизации, активирующая приложение логики при получении Центром безопасности Azure оповещения, содержащего определенную строку.
- [**Microsoft.Logic/workflows**](/azure/templates/microsoft.logic/workflows). Пустое активируемое приложение логики.

Сведения о других шаблонах быстрого запуска Центра безопасности см. на странице [шаблонов, предоставленных сообществом](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Security&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

- **PowerShell**:

  ```azurepowershell-interactive
  New-AzResourceGroup -Name <resource-group-name> -Location <resource-group-location> #use this command when you need to create a new resource group for your deployment
  New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json
  ```

- **CLI**:

  ```azurecli-interactive
  az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new resource group for your deployment
  az deployment group create --resource-group <my-resource-group> --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json
  ```

- **Портал**.

  [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-securitycenter-create-automation-for-alertnamecontains%2fazuredeploy.json)

  Дополнительные сведения об этом варианте развертывания см. в статье [Использование кнопки развертывания для развертывания шаблонов из репозитория GitHub](../azure-resource-manager/templates/deploy-to-azure-button.md).

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

С помощью портала Azure проверьте, развернута ли автоматизация рабочего процесса.

1. На [портале Azure](https://portal.azure.com) откройте **Центр безопасности**.
1. В верхней строке меню выберите значок фильтра и конкретную подписку, в которой развернута новая служба автоматизации рабочего процесса.
1. На боковой панели Центра безопасности откройте раздел **автоматизации рабочего процесса** и проверьте наличие новой службы автоматизации.
    :::image type="content" source="./media/quickstart-automation-alert/validating-template-run.png" alt-text="Список настроенных автоматизаций" lightbox="./media/quickstart-automation-alert/validating-template-run.png":::
    >[!TIP]
    > Если в подписке имеется много служб автоматизации рабочих процессов, используйте параметр **фильтрации по имени**.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если служба автоматизации рабочего процесса больше не нужна, удалите ее с помощью портала Azure.

1. На [портале Azure](https://portal.azure.com) откройте **Центр безопасности**.
1. В верхней строке меню выберите значок фильтра и конкретную подписку, в которой развернута новая служба автоматизации рабочего процесса.
1. На боковой панели Центра безопасности откройте раздел **автоматизации рабочего процесса** и найдите службу автоматизации, которую необходимо удалить.
    :::image type="content" source="./media/quickstart-automation-alert/deleting-workflow-automation.png" alt-text="Действия по удалению автоматизации рабочих процессов" lightbox="./media/quickstart-automation-alert/deleting-workflow-automation.png":::
1. Установите флажок рядом с удаляемым элементом.
1. На панели инструментов выберите **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по созданию шаблона см. в следующей статье:

> [!div class="nextstepaction"]
> Создание и развертывание первого шаблона ARM[
