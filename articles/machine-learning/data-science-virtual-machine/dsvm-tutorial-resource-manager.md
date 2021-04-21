---
title: Краткое руководство. Создание виртуальной машины обработки и анализа данных с помощью шаблона Resource Manager
titleSuffix: Azure Data Science Virtual Machine
description: В этом кратком руководстве вы используете шаблон Azure Resource Manager для быстрого развертывания виртуальной машины для обработки и анализа данных
services: machine-learning
author: lobrien
ms.author: laobri
ms.date: 06/10/2020
ms.topic: quickstart
ms.service: data-science-vm
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 0683634223a63281ce2b42ebb02f87f9211a589e
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530656"
---
# <a name="quickstart-create-an-ubuntu-data-science-virtual-machine-using-an-arm-template"></a>Краткое руководство. Создание виртуальной машины для анализа данных на платформе Ubuntu с помощью шаблона ARM

В этом кратком руководстве показано, как создать виртуальную машину Ubuntu 18.04 для обработки и анализа данных с помощью шаблона Azure Resource Manager (ARM). Виртуальные машины для обработки и анализа данных — это облачные виртуальные машины, предварительно загруженные с помощью набора платформ и средств для обработки и анализа данных и машинного обучения. При развертывании на основе ресурсов вычислений на платформе GPU все средства и библиотеки настраиваются на использование GPU.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-ubuntu-DSVM-GPU-or-CPU%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/services/machine-learning/), прежде чем начинать работу.

* Чтобы выполнять приведенные в этом документе команды CLI в **локальной среде**, вам потребуется [Azure CLI](/cli/azure/install-azure-cli).

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-vm-ubuntu-DSVM-GPU-or-CPU/).

:::code language="json" source="~/quickstart-templates/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json":::

В шаблоне определены следующие ресурсы:

* [Microsoft.Network/networkInterfaces](/azure/templates/microsoft.network/networkinterfaces)
* [Microsoft.Network/networkSecurityGroups](/azure/templates/microsoft.network/networksecuritygroups)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines): Создайте виртуальную машину на основе облака. В этом шаблоне виртуальная машина настроена как виртуальная машина для обработки и анализа данных под управлением Ubuntu 18.04.

## <a name="deploy-the-template"></a>Развертывание шаблона

Чтобы воспользоваться шаблоном в Azure CLI, войдите в систему и выберите свою подписку (см. статью [Вход с помощью Azure CLI](/cli/azure/authenticate-azure-cli)). Далее выполните:

```azurecli-interactive
read -p "Enter the name of the resource group to create:" resourceGroupName &&
read -p "Enter the Azure location (e.g., centralus):" location &&
read -p "Enter the authentication type (must be 'password' or 'sshPublicKey') :" authenticationType &&
read -p "Enter the login name for the administrator account (may not be 'admin'):" adminUsername &&
read -p "Enter administrator account secure string (value of password or ssh public key):" adminPasswordOrKey &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri $templateUri --parameters adminUsername=$adminUsername authenticationType=$authenticationType adminPasswordOrKey=$adminPasswordOrKey &&
echo "Press [ENTER] to continue ..." &&
read
```

Перед выполнением приведенной выше команды укажите следующее:

1. Имя группы ресурсов, которую вы хотите создать для хранения DSVM и связанных ресурсов.
1. Расположение Azure, в котором вы хотите выполнить развертывание.
1. Тип аутентификации, который вы хотите использовать (введите строку `password` или `sshPublicKey`).
1. Имя учетной записи администратора (это значение не может быть `admin`).
1. Значение пароля или открытого ключа SSH для учетной записи.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Чтобы просмотреть виртуальную машину для обработки и анализа данных:

1. Перейдите на [портал Azure](https://portal.azure.com).
1. Войдите.
1. Выберите группу ресурсов, созданную ранее.

Вы увидите сведения о группе ресурсов:

:::image type="content" source="media/dsvm-tutorial-resource-manager/resource-group-home.png" alt-text="Снимок экрана: базовая группа ресурсов, содержащая DSVM":::

Щелкните ресурс виртуальной машины, чтобы открыть его информационную страницу. Здесь можно найти сведения о виртуальной машине, в том числе сведения о подключении.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если эта виртуальная машина вам не нужна, удалите ее. Так как она связана с другими ресурсами, например с учетной записью хранения, вы, вероятно, захотите удалить всю созданную группу ресурсов. Группу ресурсов можно удалить с помощью портала. Для этого нажмите кнопку **Удалить** и подтвердите удаление. Группу ресурсов можно удалить также с помощью CLI.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали виртуальную машину для обработки и анализа данных из шаблона ARM.

> [!div class="nextstepaction"]
> [Примеры программ и машинного обучения](dsvm-samples-and-walkthroughs.md)
