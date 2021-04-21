---
title: Создание поставщика ресурсов
description: Сведения о создании поставщика ресурсов и развертывание его настраиваемых типов ресурсов.
author: MSEvanhi
ms.topic: tutorial
ms.date: 06/24/2020
ms.author: evanhi
ms.openlocfilehash: 721fe31f514cb948c93dc2a779a92ef3b2775ef9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107779243"
---
# <a name="quickstart-create-a-custom-provider-and-deploy-custom-resources"></a>Краткое руководство. Создание настраиваемого поставщика и развертывание настраиваемых ресурсов

С помощью этого краткого руководства вы создадите собственного поставщика ресурсов и развернете для него настраиваемые типы ресурсов. Дополнительные сведения о предварительной версии настраиваемых поставщиков Azure см. в [этой статье](overview.md).

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Для работы с этим кратким руководством требуется вызвать операции `REST`. Существуют [различные способы отправки запросов REST](/rest/api/azure/).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Подготовьте среду к работе с Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

В примерах Azure CLI используется `az rest` для запросов `REST`. Дополнительные сведения см. в документации по команде [az rest](/cli/azure/reference-index#az_rest).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Команды PowerShell выполняются в локальной среде с помощью PowerShell 7 или более поздней версии и модулей Azure PowerShell. Дополнительные сведения см. в статье [Установка Azure PowerShell](/powershell/azure/install-az-ps).
- Если у вас нет средства для операций `REST`, установите [ARMClient](https://github.com/projectkudu/ARMClient). Эта программа командной строки с открытым исходным кодом, которая упрощает вызов API Azure Resource Manager.
- После установки **ARMClient** вы сможете отобразить сведения об использовании из командной строки PowerShell, введя команду `armclient.exe`. Также можно открыть [вики-сайт ARMClient](https://github.com/projectkudu/ARMClient/wiki).

---

## <a name="deploy-custom-provider"></a>Развертывание настраиваемого поставщика

Чтобы настроить настраиваемый поставщик, разверните [пример шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/custom-providers/customprovider.json) в подписке Azure.

После развертывания шаблона в вашей подписке будут следующие ресурсы:

- Приложение-функция с операциями для ресурсов и действий.
- Учетная запись хранения для хранения пользователей, которые создаются с помощью настраиваемого поставщика.
- Настраиваемый поставщик, который определяет настраиваемые типы ресурсов и действия. Он использует конечную точку приложения-функции для отправки запросов.
- Настраиваемый ресурс из настраиваемого поставщика.

Чтобы развернуть настраиваемый поставщик, используйте Azure CLI, PowerShell или портал Azure.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

В этом примере вам будет предложено ввести имя группы ресурсов, расположение и имя приложения-функции, предоставленной поставщиком. Имена хранятся в переменных, которые используются в других командах. Команды [az group create](/cli/azure/group#az_group_create) и [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create) развертывают нужные ресурсы.

```azurecli-interactive
read -p "Enter a resource group name:" rgName &&
read -p "Enter the location (i.e. eastus):" location &&
read -p "Enter the provider's function app name:" funcName &&
templateUri="https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/custom-providers/customprovider.json" &&
az group create --name $rgName --location "$location" &&
az deployment group create --resource-group $rgName --template-uri $templateUri --parameters funcName=$funcName &&
echo "Press [ENTER] to continue ..." &&
read
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

В этом примере вам будет предложено ввести имя группы ресурсов, расположение и имя приложения-функции, предоставленной поставщиком. Имена хранятся в переменных, которые используются в других командах. Команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) и [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) развертывают ресурсы.

```powershell
$rgName = Read-Host -Prompt "Enter a resource group name"
$location = Read-Host -Prompt "Enter the location (i.e. eastus)"
$funcName = Read-Host -Prompt "Enter the provider's function app name"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/custom-providers/customprovider.json"
New-AzResourceGroup -Name $rgName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $rgName -TemplateUri $templateUri -funcName $funcName
Read-Host -Prompt "Press [ENTER] to continue ..."
```

---

Также можно развернуть решение на портале Azure. Нажмите кнопку **Развертывание в Azure**, чтобы открыть шаблон на портале Azure.

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-docs-json-samples%2Fmaster%2Fcustom-providers%2Fcustomprovider.json)

## <a name="view-custom-provider-and-resource"></a>Просмотр настраиваемого поставщика и ресурсов

На портале настраиваемый поставщик —это скрытый тип ресурсов. Чтобы убедиться, что поставщик ресурсов развернут, перейдите к группе ресурсов. Установите флажок **Показать скрытые типы**.

![Отображение скрытых типов ресурсов](./media/create-custom-provider/show-hidden.png)

Чтобы просмотреть развернутый тип настраиваемого ресурса, используйте операцию `GET` для типа ресурса.

```http
GET https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users?api-version=2018-09-01-preview
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
subID=$(az account show --query id --output tsv)
requestURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users?api-version=2018-09-01-preview"
az rest --method get --uri $requestURI
```

Вы получите такой ответ:

```json
{
  "value": [
    {
      "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/santa",
      "name": "santa",
      "properties": {
        "FullName": "Santa Claus",
        "Location": "NorthPole",
        "provisioningState": "Succeeded"
      },
      "resourceGroup": "<rg-name>",
      "type": "Microsoft.CustomProviders/resourceProviders/users"
    }
  ]
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$subID = (Get-AzContext).Subscription.Id
$requestURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users?api-version=2018-09-01-preview"

armclient GET $requestURI
```

Вы получите такой ответ:

```json
{
  "value": [
    {
      "properties": {
        "provisioningState": "Succeeded",
        "FullName": "Santa Claus",
        "Location": "NorthPole"
      },
      "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/santa",
      "name": "santa",
      "type": "Microsoft.CustomProviders/resourceProviders/users"
    }
  ]
}
```

---

## <a name="call-action"></a>Действие вызова

Настраиваемый поставщик также имеет действие с именем `ping`. Код, который обрабатывает запрос, реализуется в приложении-функции. Действие `ping` отправляет ответ с приветствием.

Чтобы отправить запрос `ping`, используйте операцию `POST` для настраиваемого поставщика.

```http
POST https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/ping?api-version=2018-09-01-preview
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
pingURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/ping?api-version=2018-09-01-preview"
az rest --method post --uri $pingURI
```

Вы получите такой ответ:

```json
{
  "message": "hello <function-name>.azurewebsites.net",
  "pingcontent": {
    "source": "<function-name>.azurewebsites.net"
  }
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$pingURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/ping?api-version=2018-09-01-preview"

armclient POST $pingURI
```

Вы получите такой ответ:

```json
{
  "pingcontent": {
    "source": "<function-name>.azurewebsites.net"
  },
  "message": "hello <function-name>.azurewebsites.net"
}
```

---

## <a name="create-a-resource-type"></a>Создание типа ресурса

Чтобы создать настраиваемый тип ресурса, можно развернуть ресурс в шаблоне. Этот подход показан в шаблоне, развернутом в этом кратком руководстве. Вы также можете отправлять запрос `PUT` для типа ресурса.

```http
PUT https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/<resource-name>?api-version=2018-09-01-preview

{"properties":{"FullName": "Test User", "Location": "Earth"}}
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
addURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users/testuser?api-version=2018-09-01-preview"
az rest --method put --uri $addURI --body "{'properties':{'FullName': 'Test User', 'Location': 'Earth'}}"
```

Вы получите такой ответ:

```json
{
  "id": "/subscriptions/<sub-ID>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/testuser",
  "name": "testuser",
  "properties": {
    "FullName": "Test User",
    "Location": "Earth",
    "provisioningState": "Succeeded"
  },
  "resourceGroup": "<rg-name>",
  "type": "Microsoft.CustomProviders/resourceProviders/users"
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$addURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users/testuser?api-version=2018-09-01-preview"
$requestBody = "{'properties':{'FullName': 'Test User', 'Location': 'Earth'}}"

armclient PUT $addURI $requestBody
```

Вы получите такой ответ:

```json
{
  "properties": {
    "provisioningState": "Succeeded",
    "FullName": "Test User",
    "Location": "Earth"
  },
  "id": "/subscriptions/<sub-ID>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/testuser",
  "name": "testuser",
  "type": "Microsoft.CustomProviders/resourceProviders/users"
}
```

---

## <a name="custom-resource-provider-commands"></a>Команды настраиваемого поставщика ресурсов

Для работы с настраиваемым поставщиком ресурсов используйте команды [custom-providers](/cli/azure/ext/custom-providers/custom-providers/resource-provider).

### <a name="list-custom-resource-providers"></a>Перечисление настраиваемых поставщиков ресурсов

Команда `list` отображает все пользовательские поставщики ресурсов в подписке. По умолчанию перечисляются пользовательские поставщики ресурсов для текущей подписки, но можно указать параметр `--subscription`. Чтобы получить список для группы ресурсов, используйте параметр `--resource-group`.

```azurecli-interactive
az custom-providers resource-provider list --subscription $subID
```

```json
[
  {
    "actions": [
      {
        "endpoint": "https://<provider-name>.azurewebsites.net/api/{requestPath}",
        "name": "ping",
        "routingType": "Proxy"
      }
    ],
    "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceproviders/<provider-name>",
    "location": "eastus",
    "name": "<provider-name>",
    "provisioningState": "Succeeded",
    "resourceGroup": "<rg-name>",
    "resourceTypes": [
      {
        "endpoint": "https://<provider-name>.azurewebsites.net/api/{requestPath}",
        "name": "users",
        "routingType": "Proxy, Cache"
      }
    ],
    "tags": {},
    "type": "Microsoft.CustomProviders/resourceproviders",
    "validations": null
  }
]
```

### <a name="show-the-properties"></a>Отображение свойств

Используйте команду `show`, чтобы отобразить свойства пользовательского поставщика ресурсов. Формат вывода напоминает вывод для `list`.

```azurecli-interactive
az custom-providers resource-provider show --resource-group $rgName --name $funcName
```

### <a name="create-a-new-resource"></a>Создать новый ресурс

Используйте команду `create` для создания или обновления настраиваемого поставщика ресурсов. В этом примере обновляются `actions` и `resourceTypes`.

```azurecli-interactive
az custom-providers resource-provider create --resource-group $rgName --name $funcName \
--action name=ping endpoint=https://myTestSite.azurewebsites.net/api/{requestPath} routing_type=Proxy \
--resource-type name=users endpoint=https://myTestSite.azurewebsites.net/api{requestPath} routing_type="Proxy, Cache"
```

```json
"actions": [
  {
    "endpoint": "https://myTestSite.azurewebsites.net/api/{requestPath}",
    "name": "ping",
    "routingType": "Proxy"
  }
],

"resourceTypes": [
  {
    "endpoint": "https://myTestSite.azurewebsites.net/api{requestPath}",
    "name": "users",
    "routingType": "Proxy, Cache"
  }
],
```

### <a name="update-the-providers-tags"></a>Обновление тегов поставщика

Команда `update` обновляет только теги для настраиваемого поставщика ресурсов. На портале Azure этот тег отображается службой приложений настраиваемого поставщика ресурсов.

```azurecli-interactive
az custom-providers resource-provider update --resource-group $rgName --name $funcName --tags new=tag
```

```json
"tags": {
  "new": "tag"
},
```

### <a name="delete-a-custom-resource-provider"></a>Удаление настраиваемого поставщика ресурсов

Команда `delete` запрашивает подтверждение на удаление и удаляет только настраиваемый поставщик ресурсов. Учетная запись хранения, служба приложений и план службы приложений не удаляются. После удаления поставщика вы вернетесь к командной строке.

```azurecli-interactive
az custom-providers resource-provider delete --resource-group $rgName --name $funcName
```

## <a name="next-steps"></a>Дальнейшие действия

Общие сведения о настраиваемых поставщиках см. в следующей статье:

> [!div class="nextstepaction"]
> [Обзор настраиваемых поставщиков Azure (предварительная версия)](overview.md)
