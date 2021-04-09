---
title: Устранение распространенных ошибок
description: Узнайте, как устранять неполадки в работе различных пакетов SDK при запросе ресурсов Azure с помощью Azure Resource Graph.
ms.date: 01/27/2021
ms.topic: troubleshooting
ms.openlocfilehash: 0d783b9cb4fa30e3c3e0ff82536b4878c29c7a0d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98916708"
---
# <a name="troubleshoot-errors-using-azure-resource-graph"></a>Устранение неполадок с помощью службы Azure Resource Graph

При запросе ресурсов Azure с помощью Azure Resource Graph могут возникать ошибки. В этой статье описываются различные ошибки, которые могут возникнуть, и пути их решения.

## <a name="finding-error-details"></a>Поиск сведений об ошибке

Большинство ошибок возникают в результате неполадок при выполнении запроса с использованием Azure Resource Graph. При сбое запроса пакет SDK выводит сведения о сбое в обработке запроса. Эти сведения указывают на проблемы, которые нужно исправить, чтобы следующий запрос был выполнен успешно.

## <a name="general-errors"></a>Общие ошибки

### <a name="scenario-throttled-requests"></a><a name="throttled"></a>Сценарий. Регулируемые запросы

#### <a name="issue"></a>Проблема

Для заказчиков, отправляющих большие запросы ресурсов или отправляющих их часто, запросы регулируются.

#### <a name="cause"></a>Причина

Azure Resource Graph выделяет количественную квоту для каждого пользователя с учетом временного окна. Например, пользователь не может отправить более 15 запросов в течение 5 секунд, иначе применяется регулирование. Значение квоты определяется множеством факторов и может быть изменено. Дополнительные сведения см. в разделе [Регулирование в Azure Resource Graph](../overview.md#throttling).

#### <a name="resolution"></a>Решение

Существует несколько способов работы с регулируемыми запросами.

- [Группирование запросов](../concepts/guidance-for-throttled-requests.md#grouping-queries)
- [Поочередная отправка запросов](../concepts/guidance-for-throttled-requests.md#staggering-queries)
- [Параллельная отправка запросов](../concepts/guidance-for-throttled-requests.md#query-in-parallel)
- [Разбиение на страницы](../concepts/guidance-for-throttled-requests.md#pagination)

### <a name="scenario-too-many-subscriptions"></a><a name="toomanysubscription"></a>Сценарий. Слишком большое количество подписок

#### <a name="issue"></a>Проблема

Заказчики, у которых есть доступ более чем к 1000 подпискам, включая подписки для нескольких арендаторов в [Azure Lighthouse](../../../lighthouse/overview.md), не могут получать данные из всех подписок в рамках одного вызова Azure Resource Graph.

#### <a name="cause"></a>Причина

Azure CLI и PowerShell переадресуют только первую 1000 подписок в Azure Resource Graph. REST API для Azure Resource Graph принимает максимальное число подписок, для которых выполняется запрос.

#### <a name="resolution"></a>Решение

Число запросов с набором подписок, добавляемых в пакет, не должно превышать 1000 подписок. Решение использует параметр **Subscription** в PowerShell.

```azurepowershell-interactive
# Replace this query with your own
$query = 'Resources | project type'

# Fetch the full array of subscription IDs
$subscriptions = Get-AzSubscription
$subscriptionIds = $subscriptions.Id

# Create a counter, set the batch size, and prepare a variable for the results
$counter = [PSCustomObject] @{ Value = 0 }
$batchSize = 1000
$response = @()

# Group the subscriptions into batches
$subscriptionsBatch = $subscriptionIds | Group -Property { [math]::Floor($counter.Value++ / $batchSize) }

# Run the query for each batch
foreach ($batch in $subscriptionsBatch){ $response += Search-AzGraph -Query $query -Subscription $batch.Group }

# View the completed results of the query on all subscriptions
$response
```

### <a name="scenario-unsupported-content-type-rest-header"></a><a name="rest-contenttype"></a>Сценарий. Неподдерживаемый заголовок Content-Type REST

#### <a name="issue"></a>Проблема

Заказчики, запрашивающие REST API Azure Resource Graph, получают ответ _500_ (Внутренняя ошибка сервера).

#### <a name="cause"></a>Причина

REST API Azure Resource Graph поддерживает только `Content-Type` из **application/json**. Для отдельных инструментов или агентов REST значение по умолчанию равно **text/plain**. Это значение не поддерживается в REST API.

#### <a name="resolution"></a>Решение

Убедитесь в том, что инструмент или агент, который используется для отправки запроса в Azure Resource Graph, содержит заголовок REST API `Content-Type`, настроенный для **application/json**.

### <a name="scenario-no-read-permission-to-all-subscriptions-in-list"></a><a name="rest-403"></a>Сценарий. Отсутствует разрешение на чтение для всех подписок в списке

#### <a name="issue"></a>Проблема

Заказчики, которые явным образом передают список подписок с запросом Azure Resource Graph, получают ответ _403_ (Запрещено).

#### <a name="cause"></a>Причина

Если у клиента нет разрешения на чтение для всех предоставленных подписок, запрос отклоняется из-за недостаточных прав доступа.

#### <a name="resolution"></a>Решение

Добавьте хотя бы одну подписку в список подписок, чтобы предоставить заказчику, отправляющему запрос, хотя бы доступ для чтения. Дополнительные сведения см. в разделе [Разрешения в Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph).

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

- Получите консультацию специалистов по Azure на [форумах Azure](https://azure.microsoft.com/support/forums/).
- Подключитесь к [@AzureSupport](https://twitter.com/azuresupport) — официальной учетной записи Microsoft Azure. Она помогает оптимизировать работу пользователей благодаря возможности доступа к ресурсам сообщества Azure (ответы на вопросы, поддержка и консультации специалистов).
- Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните **Получить поддержку**.