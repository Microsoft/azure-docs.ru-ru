---
title: Краткое руководство. Развертывание Azure API для FHIR с помощью Azure CLI
description: В этом кратком руководстве показано, как развернуть Azure API для FHIR в Azure с помощью Azure CLI.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 10/15/2019
ms.author: cavoeg
ms.custom: devx-track-azurecli
ms.openlocfilehash: 10bc0ac3a61ddba22dea46dfdd47a0305e2d9149
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103019706"
---
# <a name="quickstart-deploy-azure-api-for-fhir-using-azure-cli"></a>Краткое руководство. Развертывание Azure API для FHIR с помощью Azure CLI

В этом кратком руководстве показано, как развернуть Azure API для FHIR в Azure с помощью Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="add-healthcareapis-extension"></a>Добавление расширения HealthcareAPIs

```azurecli-interactive
az extension add --name healthcareapis
```

Получите список команд для HealthcareAPIs:

```azurecli-interactive
az healthcareapis --help
```

## <a name="create-azure-resource-group"></a>Создание группы ресурсов Azure

Выберите имя группы ресурсов, которая будет содержать Azure API для FHIR, и создайте ее:

```azurecli-interactive
az group create --name "myResourceGroup" --location westus2
```

## <a name="deploy-the-azure-api-for-fhir"></a>Развертывание Azure API для FHIR

```azurecli-interactive
az healthcareapis create --resource-group myResourceGroup --name nameoffhiraccount --kind fhir-r4 --location westus2 
```

## <a name="fetch-fhir-api-capability-statement"></a>Получение инструкции возможностей API для FHIR

Получите инструкцию возможностей из API FHIR:

```azurecli-interactive
curl --url "https://nameoffhiraccount.azurehealthcareapis.com/metadata"
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите группу ресурсов, выполнив следующие действия:

```azurecli-interactive
az group delete --name "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как развернуть Azure API для FHIR в своей подписке. Чтобы задать дополнительные параметры в Azure API для FHIR, см. соответствующее руководство. Если вы уже готовы к использованию API Azure для FHIR, ознакомьтесь с процессом регистрации приложений.

>[!div class="nextstepaction"]
>[Дополнительные параметры в Azure API для FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Общие сведения о регистрации приложений](fhir-app-registration.md)
