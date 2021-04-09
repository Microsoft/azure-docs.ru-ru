---
title: Регистрация поставщика ресурсов для Решения Azure VMware
description: Инструкции по регистрации поставщика ресурсов для Решения Azure VMware.
ms.topic: include
ms.date: 02/17/2021
ms.openlocfilehash: d2363ca2672f7f668d8f9b3816447f316d8b7347
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103555884"
---
<!-- Used in deploy-azure-vmware-solution.md and tutorial-create-private-cloud.md -->

Чтобы использовать Решение Azure VMware, необходимо сначала зарегистрировать поставщик ресурсов в подписке. Дополнительные сведения о поставщиках ресурсов см. в статье [Поставщики и типы ресурсов Azure](../../azure-resource-manager/management/resource-providers-and-types.md).


### <a name="portal"></a>[Портал](#tab/azure-portal)
 
1. Войдите на [портал Azure](https://portal.azure.com).

1. В меню портала Azure выберите **Все службы**.

1. В поле **Все службы** введите **подписка**, а затем выберите **Подписки**.

1. Выберите подписку из списка подписок для просмотра.

1. Выберите **Поставщики ресурсов** и введите **Microsoft.AVS** в поле поиска. 
 
1. Если поставщик ресурсов не зарегистрирован, выберите **Зарегистрировать**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы начать работу с Azure CLI:

[!INCLUDE [azure-cli-prepare-your-environment-no-header](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

Войдите в подписку Azure, используемую вами для развертывания Решения Azure VMware через Azure CLI. Зарегистрируйте поставщик ресурсов `Microsoft.AVS` с помощью команды [az provider register](/cli/azure/provider#az_provider_register):

```azurecli-interactive
az provider register -n Microsoft.AVS --subscription <your subscription ID>
```

Для просмотра всех доступных поставщиков можно использовать команду [az provider list](/cli/azure/provider#az_provider_list).

---


 
