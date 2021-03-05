---
title: Обновление средств Azure Dev Spaces
services: azure-dev-spaces
ms.date: 07/03/2018
ms.topic: conceptual
ms.custom: devx-track-azurecli
description: Узнайте, как обновить средства командной строки Azure Dev Spaces и расширения Visual Studio Code или Visual Studio.
keywords: Docker, Kubernetes, Azure, AKS, Azure Container Service, containers
ms.openlocfilehash: f17643e6130abbc9d5da8b484144c95b0e803f33
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102199243"
---
# <a name="how-to-upgrade-azure-dev-spaces-tools"></a>Обновление средств Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Если вы уже используете Azure Dev Spaces, при выходе нового выпуска вам следует обновить клиентские средства Azure Dev Spaces.

## <a name="update-the-azure-cli"></a>Обновление Azure CLI

При обновлении до последней версии Azure CLI вы автоматически получаете последнюю версию расширения интерфейса командной строки для Azure Dev Spaces.

Предыдущую версию удалять не нужно, просто найдите нужный файл для скачивания на странице [Azure CLI](/cli/azure/install-azure-cli).


## <a name="update-the-dev-spaces-cli-extension-and-command-line-tools"></a>Обновление расширения CLI для Azure Dev Spaces и средств командной строки

Выполните следующую команду:

```azurecli
az aks use-dev-spaces -n <your-aks-cluster> -g <your-aks-cluster-resource-group> --update
```

## <a name="update-the-vs-code-extension"></a>Обновление расширения VS Code

После установки расширение автоматически обновляется. Возможно, потребуется перезагрузить расширение, чтобы использовать новые возможности. В VS Code откройте панель **Расширения**, выберите расширения **Azure Dev Spaces** и щелкните **Перезагрузить**.

## <a name="update-visual-studio"></a>Обновление Visual Studio

Служба Azure Dev Spaces является частью рабочей нагрузки "Разработка для Azure" и присутствует во всех обновлениях Visual Studio.

## <a name="next-steps"></a>Дальнейшие действия

Протестируйте новые средства, создав новый кластер. Изучите руководства по [Azure Dev Spaces](../index.yml).
