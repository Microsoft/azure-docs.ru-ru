---
title: Использование шаблона Azure Resource Manager для развертывания частной конечной точки для веб-приложения
description: Узнайте, как с помощью шаблона ARM развернуть частную конечную точку для веб-приложения.
author: ericgre
ms.assetid: 49e460d0-7759-4ceb-b5a4-f1357e4fde56
ms.topic: sample
ms.date: 07/08/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: acf7198477ce0c55f9321fd3f8e57294a60c0961
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89652013"
---
# <a name="create-an-app-service-app-and-deploy-a-private-endpoint-by-using-an-azure-resource-manager-template"></a>Создание приложения Службы приложений и развертывание частной конечной точки с помощью шаблона Azure Resource Manager

В рамках этого краткого руководства для создания веб-приложения и предоставления доступа к нему с помощью частной конечной точки используется шаблон Azure Resource Manager (ARM).

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisite"></a>Предварительные требования

Вам потребуется учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

## <a name="create-a-private-endpoint"></a>Создание частной конечной точки

Этот шаблон создает частную конечную точку для веб-приложения Azure.

### <a name="review-the-template"></a>Изучение шаблона

:::code language="json" source="~/quickstart-templates/101-private-endpoint-webapp/azuredeploy.json" :::

### <a name="deploy-the-template"></a>Развертывание шаблона

Ниже показано, как развернуть шаблон Azure Resource Manager в Azure.

1. Чтобы войти в Azure и открыть шаблон, щелкните эту ссылку:  [Развертывание в Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-webapp%2Fazuredeploy.json). Этот шаблон создает виртуальную сеть, веб-приложение, частную конечную точку и частную зону DNS.
2. Выберите или создайте группу ресурсов.
3. Укажите имя веб-приложения, план службы приложений и частную конечную точку.
5. Ознакомьтесь с заявлением об условиях использования. Если вы согласны с ними, выберите **Я принимаю указанные выше условия** > **Покупка**. Развертывание может занять несколько минут.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные вместе с частной конечной точкой, удалите эту группу ресурсов. При этом будут удалены частная конечная точка и все связанные с ней ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные шаблоны Azure Resource Manager для веб-приложений Службы приложений Azure можно найти в статье [Шаблоны Azure Resource Manager для Службы приложений](../samples-resource-manager-templates.md).
