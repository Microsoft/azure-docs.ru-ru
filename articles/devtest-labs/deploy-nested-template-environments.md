---
title: Развертывание вложенных сред шаблонов в Azure DevTest Labs
description: Узнайте, как развертывать вложенные шаблоны Azure Resource Manager для предоставления сред с Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 39002e286fafd4f813333a14ed86256a517897e9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85481346"
---
# <a name="deploy-nested-azure-resource-manager-templates-for-testing-environments"></a>Развертывание вложенных шаблонов Azure Resource Manager для тестовых сред
Вложенное развертывание позволяет выполнять другие шаблоны Azure Resource Manager в основном шаблоне диспетчер ресурсов. Она позволяет разбивать развертывание на набор целевых шаблонов и определенных целей. Он предоставляет преимущества в отношении тестирования, повторного использования и удобочитаемости. В статье [Использование связанных шаблонов при развертывании ресурсов Azure](../azure-resource-manager/templates/linked-templates.md) представлено хорошее описание решения с несколькими примерами кода. В этой статье приведен пример, относящийся к Azure DevTest Labs. 

## <a name="key-parameters"></a>Параметры ключа
Хотя вы можете создать собственный шаблон диспетчер ресурсов с нуля, мы рекомендуем использовать [проект группы ресурсов Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) в Visual Studio, что упрощает разработку и отладку шаблонов. При добавлении вложенного ресурса развертывания в azuredeploy.jsв Visual Studio добавляет несколько элементов, чтобы сделать шаблон более гибким. Эти элементы включают вложенную папку с дополнительным шаблоном и файлом параметров, имена переменных в основном файле шаблона и два параметра для расположения хранилища для новых файлов. **_ArtifactsLocation** и **_artifactsLocationSasToken** являются ключевыми параметрами, которые использует DevTest Labs. 

Если вы не знакомы с тем, как DevTest Labs работает с средами, см. статью [создание сред с несколькими ВМ и ресурсов PaaS с помощью шаблонов Azure Resource Manager](devtest-lab-create-environment-from-arm.md). Шаблоны хранятся в репозитории, связанном с лабораторией в DevTest Labs. При создании новой среды с этими шаблонами файлы перемещаются в контейнер службы хранилища Azure в лаборатории. Чтобы иметь возможность идентифицировать и копировать вложенные файлы, DevTest Labs определяет параметры _artifactsLocation и _artifactsLocationSasToken, а затем копирует вложенные папки в контейнер хранилища. Затем он автоматически вставляет расположение и маркер подписанного URL-адрес (SaS) в параметры. 

## <a name="nested-deployment-example"></a>Пример вложенного развертывания
Ниже приведен простой пример вложенного развертывания:

```json

"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
"contentVersion": "1.0.0.0",
"parameters": {
    "_artifactsLocation": {
        "type": "string"
    },
    "_artifactsLocationSasToken": {
        "type": "securestring"
    }},
"variables": {
    "NestOneTemplateFolder": "nestedtemplates",
    "NestOneTemplateFileName": "NestOne.json",
    "NestOneTemplateParametersFileName": "NestOne.parameters.json"},
    "resources": [
    {
        "name": "NestOne",
        "type": "Microsoft.Resources/deployments",
        "apiVersion": "2016-09-01",
        "dependsOn": [ ],
        "properties": {
            "mode": "Incremental",
            "templateLink": {
                "uri": "[concat(parameters('_artifactsLocation'), '/', variables('NestOneTemplateFolder'), '/', variables('NestOneTemplateFileName'), parameters('_artifactsLocationSasToken'))]",
                "contentVersion": "1.0.0.0"
            },
            "parametersLink": {
                "uri": "[concat(parameters('_artifactsLocation'), '/', variables('NestOneTemplateFolder'), '/', variables('NestOneTemplateParametersFileName'), parameters('_artifactsLocationSasToken'))]",
                "contentVersion": "1.0.0.0"
            }
        }    
    }],
"outputs": {}
```

Папка в репозитории, содержащей этот шаблон, имеет вложенную папку `nestedtemplates` с файлами, **NestOne.js** и **NestOne.parameters.js**. В **azuredeploy.json** универсальный код ресурса (URI) для шаблона строится с использованием расположения артефактов, вложенной папки шаблона, имени файла вложенного шаблона. Аналогичным образом URI для параметров создается с помощью расположения артефактов, вложенной папки шаблонов и файла параметров для вложенного шаблона. 

Ниже приведен образ той же структуры проекта в Visual Studio: 

![Структура проекта в Visual Studio](./media/deploy-nested-template-environments/visual-studio-project-structure.png)

Можно добавить дополнительные папки в основную папку, но не на более чем на одном уровне. 

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о средах см. в следующих статьях: 

- [Создание сред со множеством виртуальных машин и ресурсов PaaS с помощью шаблонов Azure Resource Manager](devtest-lab-create-environment-from-arm.md)
- [Настройка и использование общедоступных сред в Azure DevTest Labs](devtest-lab-configure-use-public-environments.md)
- [Подключите среду к виртуальной сети лаборатории в Azure DevTest Labs](connect-environment-lab-virtual-network.md)