---
title: Создание ресурса Azure Cognitive Services с помощью шаблонов ARM | Документация Майкрософт
description: Создание ресурса Azure Cognitive Services с помощью шаблона ARM.
keywords: cognitive services, когнитивные решения, когнитивная аналитика, когнитивные решения ии
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: quickstart
ms.date: 3/22/2021
ms.author: aahi
ms.custom: subject-armqs, devx-track-azurecli
ms.openlocfilehash: 161c5779926acad8814ec057f24e36f371738483
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864369"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-an-arm-template"></a>Краткое руководство. Создание ресурса Cognitive Services с помощью шаблона ARM

В этом руководстве показано, как создать экземпляр Cognitive Services с помощью шаблона Azure Resource Manager (ARM).

Azure Cognitive Services — это облачные службы с REST API и пакетами SDK клиентских библиотек, которые помогают разработчикам без опыта работы со средствами искусственного интеллекта (ИИ) и обработки и анализа данных создавать когнитивные интеллектуальные приложения. С помощью Azure Cognitive Services разработчики могут без усилий добавлять в свои приложения когнитивные функции, создавая когнитивные решения, которые могут видеть, слышать, говорить, понимать и даже в некоторой степени размышлять.

Создайте ресурс с помощью шаблона Azure Resource Manager (ARM). Этот ресурс для нескольких служб дает вам следующие возможности:

* доступ к нескольким службам Azure Cognitive Services с помощью одного ключа и конечной точки;
* объединение счетов из используемых служб.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание экземпляра Cognitive Services в Azure](../media/template-deployments/deploy-to-azure.svg "Развертывание экземпляра Cognitive Services в Azure")](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cognitive-services-universalkey%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/cognitive-services).

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-cognitive-services-universalkey/).

:::code language="json" source="~/quickstart-templates/101-cognitive-services-universalkey/azuredeploy.json":::

В этом шаблоне определяется один ресурс Azure.
* [Microsoft.CognitiveServices/accounts](/azure/templates/microsoft.cognitiveservices/accounts): создает ресурс Cognitive Services.

## <a name="deploy-the-template"></a>Развертывание шаблона

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

1. Нажмите кнопку **Deploy to Azure** (Развернуть в Azure).

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cognitive-services-universalkey%2Fazuredeploy.json)

2. Введите указанные ниже значения.

    |Значение  |Описание  |
    |---------|---------|
    | **Подписка** | Выберите подписку Azure. |
    | **Группа ресурсов** | Щелкните **Создать**, введите уникальное имя группы ресурсов и нажмите кнопку **ОК**. |
    | **Регион** | Выберите регион.  Например, **Восточная часть США**. |
    | **Имя Cognitive Service** | Замените уникальным именем для ресурса. В следующем разделе при проверке развертывания необходимо имя. |
    | **Расположение** | Замените на регион, используемый выше. |
    | **SKU** | [Ценовая категория](https://azure.microsoft.com/pricing/details/cognitive-services/) ресурса. |

    :::image type="content" source="media/arm-template/universal-key-portal-template.png" alt-text="Снимок экрана создания ресурса":::

3. Выберите **Проверить и создать**, а затем — **Создать**. После успешного развертывания ресурса будет выделена кнопка **Перейти к ресурсу**.

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

> [!NOTE]
> Для создания `az deployment group` требуется Azure CLI 2.6 или более поздней версии. Для отображения версии введите `az --version`. Дополнительные сведения см. в этой [документации](/cli/azure/deployment/group).

Выполните следующий сценарий с помощью интерфейса командной строки Azure (CLI) [на локальном компьютере](/cli/azure/install-azure-cli) или в браузере с помощью кнопки **Попробовать**. Введите имя и расположение (например, `centralus`) для новой группы ресурсов, а для развертывания в ней ресурса Cognitive Services будет использоваться шаблон ARM. Запомните или запишите имя, которое вы используете. Вы будете использовать его позже для проверки развертывания.


```azurecli-interactive
read -p "Enter a name for your new resource group:" resourceGroupName &&
read -p "Enter the location (i.e. centralus):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-cognitive-services-universalkey/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

---

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]


## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

# <a name="portal"></a>[Портал](#tab/portal)

По завершении развертывания можно будет нажать кнопку **Перейти к ресурсу**, чтобы увидеть новый ресурс. Группу ресурсов можно также найти, выбрав:

1. параметр **Группы ресурсов** в меню навигации слева;
2. имя группы ресурсов.

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

С помощью Azure CLI выполните следующий сценарий и введите имя группы ресурсов, которую вы создали ранее.

```azurecli-interactive
echo "Enter the resource group where the Cognitive Services resource exists:" &&
read resourceGroupName &&
az cognitiveservices account list -g $resourceGroupName
```

---


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При удалении группы ресурсов удаляются также все другие ресурсы, содержащиеся в этой группе.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

1. На портале Azure разверните меню слева, чтобы открыть меню служб, и выберите **Resource Groups** (Группы ресурсов), чтобы просмотреть список групп ресурсов.
2. Найдите группу ресурсов, содержащую ресурс, который необходимо удалить.
3. Щелкните правой кнопкой мыши список групп ресурсов. Выберите **Удалить группу ресурсов** и подтвердите выбор.

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

С помощью Azure CLI выполните следующий сценарий и введите имя группы ресурсов, которую вы создали ранее.

```azurecli-interactive
echo "Enter the resource group name, for deletion:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

---

## <a name="see-also"></a>См. также статью

* Сведения о безопасной работе с Cognitive Services см. в статье **[Проверка подлинности запросов к Azure Cognitive Services](authentication.md)** .
* См. статью **[Общие сведения об Azure Cognitive Services](./what-are-cognitive-services.md)** , в которой можно увидеть список различных категорий в Cognitive Services.
* Список естественных языков, поддерживаемых Cognitive Services, см. в статье **[Поддержка естественного языка](language-support.md)** .
* Сведения об использовании Cognitive Services в локальной среде см. в статье **[Использование Cognitive Services в качестве контейнеров](cognitive-services-container-support.md)** .
* См. статью **[Планирование затрат и управление ими в Cognitive Services](plan-manage-costs.md)** , чтобы получить сведения о том, как оценивать затраты на использование Cognitive Services.
