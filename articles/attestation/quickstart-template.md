---
title: Создание сертификата аттестации Azure с помощью шаблона Azure Resource Manager
description: Описание процесса создания сертификата аттестации Azure с помощью шаблона Azure Resource Manager.
services: azure-resource-manager
author: msmbaldwin
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mbaldwin
ms.date: 10/16/2020
ms.openlocfilehash: 8c56a37ebcc799b0170785666212eb4e3b00a5f0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92144976"
---
# <a name="quickstart-create-an-azure-attestation-provider-with-an-arm-template"></a>Краткое руководство. Создание поставщика аттестации Azure с помощью шаблона ARM

[Аттестация Microsoft Azure](overview.md) — это решение для аттестации доверенных сред выполнения (TEE). В этом кратком руководстве показано, как развернуть шаблон Azure Resource Manager (шаблон ARM), чтобы создать политику аттестации Microsoft Azure.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развернуть в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-attestation-provider-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-attestation-provider-create).

:::code language="json" source="~/quickstart-templates/101-attestation-provider-create/azuredeploy.json":::

В шаблоне определены следующие ресурсы Azure:

- Microsoft.Attestation/attestationProviders

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон.

    [![Развернуть в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-attestation-provider-create%2Fazuredeploy.json)

1. Введите или выберите следующие значения.

    Если поставщик аттестации не указан, при создании используйте значение по умолчанию.

    - **Имя поставщика аттестации.** Выберите имя поставщика аттестации Azure.
    - **Расположение.** Выберите расположение. Например, **центральная часть США**.
    - **Теги**. Выберите расположение. Например, **центральная часть США**.

1. Щелкните **Приобрести**. После успешного развертывания ресурса аттестации вы получите уведомление.

Для развертывания шаблона используется портал Azure. В дополнение к порталу Azure можно также использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Для проверки ресурса аттестации можно использовать портал Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов

Другая сборка аттестации Azure в этом кратком руководстве. Если вы планируете продолжить работу с последующими краткими руководствами и статьями, эти ресурсы можно не удалять.

Удалите группу ресурсов, которая удаляет ресурс аттестации, если она больше не нужна. Чтобы удалить группу ресурсов с помощью Azure CLI или Azure PowerShell, выполните следующие действия.

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

Из этого краткого руководства вы узнали, как создать ресурс аттестации с помощью шаблона ARM и проверить это развертывание. Дополнительные сведения о службе "Аттестация Azure" см. в статье [Аттестация Microsoft Azure (предварительная версия)](overview.md).
