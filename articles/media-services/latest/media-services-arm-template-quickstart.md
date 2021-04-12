---
заголовок: Шаблон ARM для учетной записи Служб мультимедиа Службы мультимедиа Azure description: В этой статье описано, как использовать шаблон Resource Manager для создания учетной записи Служб мультимедиа.
services: media-services documentationcenter: '' author: IngridAtMicrosoft manager: femila editor: ''

ms.service: media-services ms.workload: ms.topic: quickstart ms.date: 03/23/2021 ms.author: inhenkel ms.custom: subject-armqs

---

# <a name="quickstart-media-services-account-arm-template"></a>Краткое руководство. Шаблон Resource Manager учетной записи Служб мультимедиа

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье описано, как использовать шаблон Azure Resource Manager (шаблон ARM) для создания учетной записи Служб мультимедиа.

## <a name="introduction"></a>Введение

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Читатели, имеющие опыт работы с шаблонами Resource Manager, могут перейти к [разделу о развертывании](#deploy-the-template).

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-media-services-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Если вы еще не развертывали шаблон Resource Manager, рекомендуется ознакомиться со статьей о [шаблонах Azure ARM](../../azure-resource-manager/templates/index.yml) и [этим учебником](../../azure-resource-manager/templates/template-tutorial-create-first-template.md?tabs=azure-powershell).

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-media-services-create/).

Синтаксис для границы кода JSON:

:::code language="json" source="~/quickstart-templates/101-media-services-create/azuredeploy.json":::

В шаблоне определены три типа ресурса Azure:

- С помощью [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts) создается учетная запись хранения.
- С помощью [Microsoft.Media/mediaservices](/azure/templates/microsoft.media/mediaservices) создается учетная запись Служб мультимедиа.

## <a name="set-the-account"></a>Настройка учетной записи

```azurecli-interactive

az account set --subscription {your subscription name or id}

```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

```azurecli-interactive

az group create --name {the name you want to give your resource group} --location "{pick a location}"

```

## <a name="assign-a-variable-to-your-deployment-file"></a>Назначение переменной для файла развертывания

Для удобства создайте переменную, в которой хранится путь к файлу шаблона. Эта переменная упрощает запуск команд развертывания, так как вам не нужно будет повторно вводить путь при каждом развертывании.

```azurecli-interactive

templateFile="{provide the path to the template file}"

```

## <a name="deploy-the-template"></a>Развертывание шаблона

Вам будет предложено ввести имя учетной записи Служб мультимедиа.

```azurecli-interactive

az deployment group create --name {the name you want to give to your deployment} --resource-group {the name of resource group you created} --template-file $templateFile

```

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Должен отобразиться ответ JSON следующего вида:

```json
{
  "id": "/subscriptions/{subscriptionid}/resourceGroups/amsarmquickstartrg/providers/Microsoft.Resources/deployments/amsarmquickstartdeploy",
  "location": null,
  "name": "amsarmquickstartdeploy",
  "properties": {
    "correlationId": "{correlationid}",
    "debugSetting": null,
    "dependencies": [
      {
        "dependsOn": [
          {
            "id": "/subscriptions/{subscriptionid}/resourceGroups/amsarmquickstartrg/providers/Microsoft.Storage/storageAccounts/storagey44cfdmliwatk",
            "resourceGroup": "amsarmquickstartrg",
            "resourceName": "storagey44cfdmliwatk",
            "resourceType": "Microsoft.Storage/storageAccounts"
          }
        ],
        "id": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/amsarmquickstartrg/providers/Microsoft.Media/mediaServices/{accountname}",
        "resourceGroup": "amsarmquickstartrg",
        "resourceName": "{accountname}",
        "resourceType": "Microsoft.Media/mediaServices"
      }
    ],
    "duration": "PT1M10.8615001S",
    "error": null,
    "mode": "Incremental",
    "onErrorDeployment": null,
    "outputResources": [
      {
        "id": "/subscriptions/{subscriptionid}/resourceGroups/amsarmquickstartrg/providers/Microsoft.Media/mediaServices/{accountname}",
        "resourceGroup": "amsarmquickstartrg"
      },
      {
        "id": "/subscriptions/{subscriptionid}/resourceGroups/amsarmquickstartrg/providers/Microsoft.Storage/storageAccounts/storagey44cfdmliwatk",
        "resourceGroup": "amsarmquickstartrg"
      }
    ],
    "outputs": null,
    "parameters": {
      "mediaServiceName": {
        "type": "String",
        "value": "{accountname}"
      }
    },
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.Media",
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiVersions": null,
            "capabilities": null,
            "locations": [
              "eastus"
            ],
            "properties": null,
            "resourceType": "mediaServices"
          }
        ]
      },
      {
        "id": null,
        "namespace": "Microsoft.Storage",
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiVersions": null,
            "capabilities": null,
            "locations": [
              "eastus"
            ],
            "properties": null,
            "resourceType": "storageAccounts"
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "templateHash": "{templatehash}",
    "templateLink": null,
    "timestamp": "2020-11-24T23:25:52.598184+00:00",
    "validatedResources": null
  },
  "resourceGroup": "amsarmquickstartrg",
  "tags": null,
  "type": "Microsoft.Resources/deployments"
}

```

На портале Azure убедитесь, что ресурсы созданы.

![Созданные ресурсы для работы с кратким руководством](./media/media-services-arm-template-quickstart/quickstart-arm-template-resources.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не планируете использовать только что созданные ресурсы, группу ресурсов можно удалить.

```azurecli-interactive

az group delete --name {name of the resource group}

```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании шаблона Resource Manager, о процессе создания шаблона с параметрами, переменными и т. д. см. в следующей статье:

> [!div class="nextstepaction"]
> [Создание и развертывание первого шаблона ARM](../../azure-resource-manager/templates/template-tutorial-create-first-template.md)