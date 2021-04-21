---
title: Краткое руководство. Создание учетной записи Azure Purview с помощью Azure PowerShell и Azure CLI (предварительная версия)
description: В этом кратком руководстве описано создание учетной записи Azure Purview с помощью Azure PowerShell и Azure CLI.
author: hophanms
ms.author: hophan
ms.date: 11/23/2020
ms.topic: quickstart
ms.service: purview
ms.subservice: purview-data-catalog
ms.custom:
- mode-api
ms.openlocfilehash: 6266aedaec8f171a1a6ff3e0d15abdad0263767a
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530882"
---
# <a name="quickstart-create-an-azure-purview-account-using-azure-powershellazure-cli"></a>Краткое руководство. Создание учетной записи Azure Purview с помощью Azure PowerShell и Azure CLI

> [!IMPORTANT]
> Сейчас предоставляется предварительная версия Azure Purview. [Дополнительные условия использования Azure для предварительных версий в Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) содержат дополнительные юридические условия, применимые к функциям Azure, которые предоставляются в бета-версии, предварительной версии или еще не выпущены в общедоступной версии по другим причинам.

Из этого краткого руководства вы узнаете, как создать учетную запись Azure Purview с помощью Azure PowerShell и Azure CLI.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

* Учетная запись пользователя, используемая для входа в Azure, должна быть назначена на роль участника, владельца либо администратора подписки Azure.

* Собственный [клиент Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

* Для развертывания шаблона установите на клиентском компьютере Azure PowerShell или Azure CLI. [Развертывание из командной строки](../azure-resource-manager/templates/template-tutorial-create-first-template.md?tabs=azure-cli#command-line-deployment)

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com) с помощью своей учетной записи Azure.

## <a name="configure-your-subscription"></a>Настройка подписки

При необходимости выполните следующие действия, чтобы настроить для подписки возможность запуска Azure Purview.

   1. На портале Azure найдите и выберите **Подписки**.

   1. Из списка подписок выберите ту, которую вы намерены использовать. Требуется административное разрешение на доступ к подписке.

      :::image type="content" source="./media/create-catalog-portal/select-subscription.png" alt-text="Снимок экрана: выбор подписки на портале Azure":::

   1. Для своей подписки выберите **Поставщики ресурсов**. На панели **Поставщики ресурсов** найдите и зарегистрируйте все три поставщика ресурсов: 
       1. **Microsoft.Purview**
       1. **Microsoft.Storage**
       1. **Microsoft.EventHub** 
      
      Если они не зарегистрированы, зарегистрируйте их, выбрав действие **Зарегистрировать**.

      :::image type="content" source="./media/create-catalog-portal/register-purview-resource-provider.png" alt-text="Снимок экрана: регистрация поставщика ресурсов Microsoft.Purview на портале Azure":::

## <a name="create-an-azure-purview-account-instance"></a>Создание экземпляра учетной записи Azure Purview

1. Вход с использованием учетных данных Azure

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    Connect-AzAccount
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    ```azurecli
    az login
    ```
    
    ---

1. Если у вас несколько подписок Azure, выберите ту, которую хотите использовать.

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    Set-AzContext [SubscriptionID/SubscriptionName]
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    ```azurecli
    az account set --subscription [SubscriptionID/SubscriptionName]
    ```
    
    ---

1. Создайте группу ресурсов для учетной записи Purview. Этот шаг можно пропустить, если она у вас уже есть.

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzResourceGroup `
      -Name myResourceGroup `
      -Location "East US"
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    ```azurecli
    az group create \
      --name myResourceGroup \
      --location "East US"
    ```
    
    ---

1. Создайте файл шаблона Purview, например `purviewtemplate.json`. Вы можете обновить `name`, `location` и `capacity` (`4` или `16`).

    ```json
    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": [
        {
          "name": "<yourPurviewAccountName>",
          "type": "Microsoft.Purview/accounts",
          "apiVersion": "2020-12-01-preview",
          "location": "EastUs",
          "identity": {
            "type": "SystemAssigned"
          },
          "properties": {
            "networkAcls": {
              "defaultAction": "Allow"
            }
          },
          "dependsOn": [],
          "sku": {
            "name": "Standard",
            "capacity": "4"
          },
          "tags": {}
        }
      ],
      "outputs": {}
    }
    ```

1. Развертывание шаблона Purview

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzResourceGroupDeployment -ResourceGroupName "<myResourceGroup>" -TemplateFile "<PATH TO purviewtemplate.json>"
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    Чтобы выполнить эту команду развертывания, необходимо иметь [последнюю версию](/cli/azure/install-azure-cli) Azure CLI.
    
    ```azurecli
    az deployment group create --resource-group "<myResourceGroup>" --template-file "<PATH TO purviewtemplate.json>"
    ```
    
    ---

1. Результаты выполнения команды развертывания выглядят так: Найдите `ProvisioningState`, чтобы определить успешность развертывания.
    
## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать учетную запись Azure Purview.

Перейдите к следующей статье, чтобы узнать, как разрешить пользователям доступ к учетной записи Azure Purview. 

> [!div class="nextstepaction"]
> [Добавление пользователей в учетную запись Azure Purview](catalog-permissions.md)
