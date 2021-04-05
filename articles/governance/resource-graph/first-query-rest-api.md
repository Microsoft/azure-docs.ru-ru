---
title: Краткое руководство. Ваш первый запрос REST API
description: В этом кратком руководстве показано, как вызвать конечную точку Resource Graph для REST API и выполнить первый запрос.
ms.date: 01/27/2021
ms.topic: quickstart
ms.openlocfilehash: 670ceba95d937be278c44c34704cb844eead480d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98920077"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-rest-api"></a>Краткое руководство. Выполнение первого запроса к Resource Graph с помощью REST API

Чтобы начать использовать Azure Resource Graph с REST API, нужно сначала проверить, есть ли у вас средство для вызова интерфейсов REST API. Затем по инструкциям из этого краткого руководства вы выполните запрос и получите результаты, вызвав конечную точку REST API Azure Resource Graph.

По завершении у вас будут средства для вызова конечных точек REST API и результат выполнения первого запроса к Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="getting-started-with-rest-api"></a>Начало работы с REST API

Если у вас нет опыта работы с интерфейсом REST API, сначала ознакомьтесь со [справочником по REST API Azure](/rest/api/azure/), чтобы получить общее представление о нем, в частности об URI запроса и тексте запроса. Это нужно для работы с Azure Resource Graph, поэтому предполагается, что у вас есть соответствующие практические навыки. Начинающим рекомендуется применять такие средства, как [ARMClient](https://github.com/projectkudu/ARMClient), обеспечивающие автоматическую авторизацию.

Спецификации Azure Resource Graph см. в статье [REST API Azure Resource Graph](/rest/api/azure-resourcegraph/).

### <a name="rest-api-and-powershell"></a>REST API и PowerShell

Если у вас еще нет средства для выполнения вызовов REST API, рекомендуем использовать среду PowerShell. Следующий пример кода возвращает заголовок для аутентификации в Azure. Создайте заголовок проверки подлинности, иногда называемый **токеном носителя**, и укажите универсальный код ресурса (URI) REST API, к которому нужно подключиться, а также параметры или **текст запроса**.

```azurepowershell-interactive
# Log in first with Connect-AzAccount if not using Cloud Shell

$azContext = Get-AzContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Invoke the REST API
$restUri = 'https://management.azure.com/subscriptions/{subscriptionId}?api-version=2020-01-01'
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader
```

Замените `{subscriptionId}` в переменной `$restUri`, чтобы получить сведения о своей подписке.
Переменная `$response` содержит результат выполнения командлета `Invoke-RestMethod`, который может быть проанализирован с помощью таких командлетов, как [ConvertFrom-Json](/powershell/module/microsoft.powershell.utility/convertfrom-json). Если конечная точка службы REST API требует **текста запроса**, укажите переменную в формате JSON в качестве значения параметра `-Body` командлета `Invoke-RestMethod`.

## <a name="run-your-first-resource-graph-query"></a>Выполните первый запрос график ресурсов

Теперь, когда средства REST API добавлены в вашу среду, попробуем выполнить простой запрос к Resource Graph. Запрос возвращает первые пять ресурсов Azure с указанными значениями **Имя** и **Тип ресурса** для каждого ресурса.

В тексте запроса каждого вызова REST API используется переменная, которую необходимо заменить собственным значением:

- `{subscriptionID}` — замените это значение идентификатором своей подписки.

1. Выполните первый запрос к Azure Resource Graph с помощью REST API и конечной точки `resources`:

   - Универсальный код ресурса (URI) REST API

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Текст запроса

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | limit 5"
     }
     ```

   > [!NOTE]
   > Так как этот пример запроса не содержит модификатор сортировки, такой как `order by`, повторное выполнение запроса может каждый раз возвращать разные наборы ресурсов.

1. Обновите запрос к конечной точке `resouces` и измените **query** для упорядочивания (`order by`) по свойству **Name**:

   - Универсальный код ресурса (URI) REST API

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Текст запроса

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | limit 5 | order by name asc"
     }
     ```

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Обновите вызов к конечной точке`resources`, чтобы изменить **query** для упорядочивания (`order by`) по свойству **Name** и ограничения (`limit`) вывода пятью первыми результатами:

   - Универсальный код ресурса (URI) REST API

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Текст запроса

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | order by name asc | limit 5"
     }
     ```

Повторное выполнение последней версии запроса, если ничего не изменяется в самой среде, возвращает стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

Дополнительные примеры вызовов REST API к Azure Resource Graph см. в разделе [Примеры](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources#examples).

## <a name="clean-up-resources"></a>Очистка ресурсов

REST API не содержит библиотеки или модули для удаления. Если вы установили средство для вызовов, например _ARMClient_ или _Postman_, и оно вам больше не нужно, можно удалить это средство прямо сейчас.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как вызвать конечную точку REST API Resource Graph и выполнить первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).
