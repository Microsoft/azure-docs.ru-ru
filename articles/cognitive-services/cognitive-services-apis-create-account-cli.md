---
title: Создание ресурса Cognitive Services с помощью интерфейса командной строки Azure
titleSuffix: Azure Cognitive Services
description: Начните работу с Azure Cognitive Services, создав ресурс и оформив подписку на него с помощью интерфейса командной строки Azure.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
keywords: Cognitive Services, когнитивня аналитика, когнитивные решения, службы ИИ
ms.topic: quickstart
ms.date: 3/22/2021
ms.author: aahi
ms.openlocfilehash: 26e3b264b7268f7a9ffdb592beef7d76844646f5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789147"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-the-azure-command-line-interfacecli"></a>Краткое руководство. Создание ресурса Cognitive Services с помощью интерфейса командной строки Azure (Azure CLI)

Это краткое руководство позволит вам начать работу с Azure Cognitive Services с помощью [интерфейса командной строки Azure (Azure CLI)](/cli/azure/install-azure-cli).

Azure Cognitive Services — это облачные службы с REST API и пакетами SDK клиентских библиотек, которые помогают разработчикам без опыта работы со средствами искусственного интеллекта (ИИ) и обработки и анализа данных создавать когнитивные интеллектуальные приложения. С помощью Azure Cognitive Services разработчики могут без усилий добавлять в свои приложения когнитивные функции, создавая когнитивные решения, которые могут видеть, слышать, говорить, понимать и даже в некоторой степени размышлять.

Cognitive Services представлены [ресурсами](../azure-resource-manager/management/manage-resources-portal.md) Azure, которые вы создаете в подписке Azure. После создания ресурса вы будете применять созданные для вас ключи и конечную точку для аутентификации приложений.

Из этого краткого руководства вы узнаете, как зарегистрироваться в Azure Cognitive Services и создать учетную запись с подпиской на одну или несколько служб, используя [интерфейс командной строки Azure (Azure CLI)](/cli/azure/install-azure-cli). Эти службы представлены [ресурсами](../azure-resource-manager/management/manage-resources-portal.md) Azure, которые позволяют подключиться к одному или нескольким API-интерфейсам Azure Cognitive Services.

[!INCLUDE [cognitive-services-subscription-types](../../includes/cognitive-services-subscription-types.md)]

## <a name="prerequisites"></a>Предварительные требования

* Действующая подписка Azure ([создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services)).
* [Интерфейс командной строки (CLI) Azure](/cli/azure/install-azure-cli).

## <a name="install-the-azure-cli-and-sign-in"></a>Установка Azure CLI и вход

Установка [Azure CLI](/cli/azure/install-azure-cli). Чтобы войти в локальную установку CLI, выполните команду [az login](/cli/azure/reference-index#az_login):

```azurecli-interactive
az login
```

Для выполнения этих команд можно также использовать зеленую кнопку **Попробовать** в браузере.

## <a name="create-a-new-azure-cognitive-services-resource-group"></a>Создание группы ресурсов Cognitive Services

Чтобы создать ресурс Cognitive Services, в учетной записи должна присутствовать группа ресурсов Azure для размещения этого ресурса. При создании ресурса вы можете создать новую или использовать имеющуюся группу ресурсов. В этой статье показано, как создать группу ресурсов.

### <a name="choose-your-resource-group-location"></a>Выбор расположения группы ресурсов

Чтобы создать ресурс, для подписки должно быть доступно одно из расположений Azure. Список доступных расположений можно получить с помощью команды [az account list-locations](/cli/azure/account#az_account_list_locations). Доступ к большинству служб Cognitive Services можно получить из нескольких расположений. Выберите ближайший вариант из тех расположений, которые доступны для службы.

> [!IMPORTANT]
> * Запомните это расположение Azure, так как оно понадобится при вызове Azure Cognitive Services.
> * Доступность некоторых служб Cognitive Services может варьироваться в зависимости от региона. Дополнительные сведения см. на странице [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services).

```azurecli-interactive
az account list-locations \
    --query "[].{Region:name}" \
    --out table
```

Создав расположение Azure, создайте новую группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create) в Azure CLI.

В приведенном ниже примере замените расположение Azure `westus2` одним из расположений Azure, доступных для вашей подписки.

```azurecli-interactive
az group create \
    --name cognitive-services-resource-group \
    --location westus2
```

## <a name="create-a-cognitive-services-resource"></a>Создание ресурса Cognitive Services

### <a name="choose-a-cognitive-service-and-pricing-tier"></a>Выбор службы Cognitive Services и ценовой категории

При создании ресурса необходимо знать, какой вид службы вы хотите использовать, а также требуемую [ценовую категорию](https://azure.microsoft.com/pricing/details/cognitive-services/) (номер SKU). Эту и другую информацию вы укажете в качестве параметров при создании ресурса.

### <a name="multi-service"></a>Несколько служб

| Служба                    | Вид                      |
|----------------------------|---------------------------|
| Несколько служб. Дополнительные сведения см. на [странице с ценами](https://azure.microsoft.com/pricing/details/cognitive-services/).            | `CognitiveServices`     |


> [!NOTE]
> Многие из перечисленных ниже служб Cognitive Services имеют бесплатный уровень, который можно использовать для пробного использования службы. Чтобы использовать бесплатный уровень, укажите для ресурса номер SKU `F0`.

### <a name="vision"></a>Компьютерное зрение

| Служба                    | Вид                      |
|----------------------------|---------------------------|
| Компьютерное зрение            | `ComputerVision`          |
| Ресурс прогнозирования службы "Пользовательское визуальное распознавание" | `CustomVision.Prediction` |
| Ресурс обучения службы "Пользовательское визуальное распознавание"   | `CustomVision.Training`   |
| Распознавание лиц                       | `Face`                    |
| Распознаватель документов            | `FormRecognizer`          |
| Распознаватель рукописного текста             | `InkRecognizer`           |

### <a name="speech"></a>Речь

| Служба            | Вид                 |
|--------------------|----------------------|
| Службы "Речь"    | `SpeechServices`     |
| Распознавание речи | `SpeakerRecognition` |

### <a name="language"></a>Язык

| Служба            | Вид                |
|--------------------|---------------------|
| Анализ форм | `FormUnderstanding` |
| LUIS               | `LUIS`              |
| QnA Maker          | `QnAMaker`          |
| Анализ текста     | `TextAnalytics`     |
| Преобразование текста   | `TextTranslation`   |

### <a name="decision"></a>Решение

| Служба           | Вид               |
|-------------------|--------------------|
| Детектор аномалий  | `AnomalyDetector`  |
| Content Moderator | `ContentModerator` |
| Персонализатор      | `Personalizer`     |

Список доступных видов Cognitive Services можно получить с помощью команды [az cognitiveservices account list-kinds](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_list_kinds):

```azurecli-interactive
az cognitiveservices account list-kinds
```

### <a name="add-a-new-resource-to-your-resource-group"></a>Добавление нового ресурса в группу ресурсов

Чтобы создать ресурс Cognitive Services и подписку на него, выполните команду [az cognitiveservices account create](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_create). Эта команда добавляет новый платный ресурс в созданную ранее группу ресурсов. При создании ресурса нужно знать, какой вид службы вы хотите использовать, а также ценовую категорию (номер SKU) и расположение Azure.

Следующая команда позволяет создать ресурс ценовой категории F0 ("Бесплатный") для Детектора аномалий с именем `anomaly-detector-resource`.

```azurecli-interactive
az cognitiveservices account create \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group \
    --kind AnomalyDetector \
    --sku F0 \
    --location westus2 \
    --yes
```

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]

## <a name="get-the-keys-for-your-resource"></a>Получение ключей для ресурса

Чтобы войти в локальную установку интерфейса командной строки (CLI), выполните команду [az login](/cli/azure/reference-index#az_login).

```azurecli-interactive
az login
```

Команда [az cognitiveservices account keys list](/cli/azure/cognitiveservices/account/keys#az_cognitiveservices_account_keys_list) позволяет получить ключи для ресурса службы Cognitive Services.

```azurecli-interactive
    az cognitiveservices account keys list \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group
```

[!INCLUDE [cognitive-services-environment-variables](../../includes/cognitive-services-environment-variables.md)]

## <a name="pricing-tiers-and-billing"></a>Ценовые категории и выставление счетов

Ценовые категории (и сумма в выставленных счетах) основаны на количестве отправленных вами транзакций с использованием данных аутентификации. Каждая ценовая категория определяет:
* максимальное количество разрешенных транзакций, обрабатываемых в секунду (TPS);
* функции службы, включенные в ценовой категории;
* стоимость предопределенного количества транзакций. При превышении этого ограничения будет взиматься дополнительная плата, как указано в [сведениях о ценах](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/) для вашей службы.

## <a name="get-current-quota-usage-for-your-resource"></a>Получение сведений об использовании квоты для ресурса

Команда [az cognitiveservices account list-usage](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_list_usage) позволяет получить сведения о потреблении для ресурса Cognitive Services.

```azurecli-interactive
az cognitiveservices account list-usage \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group \
    --subscription subscription-name
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить ресурс Cognitive Services, вы можете удалить его отдельно или вместе с группой ресурсов. При удалении группы ресурсов удаляются также все другие ресурсы, содержащиеся в этой группе.

Команда az group delete позволяет удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name cognitive-services-resource-group
```

## <a name="see-also"></a>См. также раздел

* Сведения о безопасной работе с Cognitive Services см. в статье **[Проверка подлинности запросов к Azure Cognitive Services](authentication.md)** .
* См. статью **[Общие сведения об Azure Cognitive Services](./what-are-cognitive-services.md)** , в которой можно увидеть список различных категорий в Cognitive Services.
* Список естественных языков, поддерживаемых Cognitive Services, см. в статье **[Поддержка естественного языка](language-support.md)** .
* Сведения об использовании Cognitive Services в локальной среде см. в статье **[Использование Cognitive Services в качестве контейнеров](cognitive-services-container-support.md)** .
* См. статью **[Планирование затрат и управление ими в Cognitive Services](plan-manage-costs.md)** , чтобы получить сведения о том, как оценивать затраты на использование Cognitive Services.
