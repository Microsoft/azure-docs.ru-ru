---
title: Краткое руководство. Создание назначения политики на языке Python
description: В этом кратком руководстве вы используете Python, чтобы создать назначение в службе "Политика Azure", позволяющее определить ресурсы, которые не соответствуют требованиям.
ms.date: 03/02/2021
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: 2c9a0d6bb00d82748505304264aeaefa409c4b06
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107379406"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-python"></a>Краткое руководство. Создание назначения политики для выявления ресурсов, не соответствующих требованиям, с использованием Python

Чтобы понять, соответствуют ли ресурсы требованиям в Azure, прежде всего нужно определить их состояние. В рамках этого краткого руководства вы создадите назначение политики для выявления виртуальных машин, которые не используют управляемые диски. После выполнения задач в руководстве вы сможете выявлять _несоответствующие_ виртуальные машины.

Библиотека Python используется для управления ресурсами Azure из командной строки или с помощью сценариев. В этом руководстве объясняется, как создавать назначение политики с помощью библиотеки Python.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-policy-library"></a>Добавление библиотеки Политики

Чтобы обеспечить функционирование Python с Политикой Azure, нужно добавить библиотеку. Эта библиотека работает во всех средах, где можно использовать Python, в том числе в [оболочке Bash для Windows 10](/windows/wsl/install-win10). Также ее можно установить локально.

1. Убедитесь, что установлена последняя версия Python (не ниже **3.8**). Если он еще не установлен, скачайте его на сайте [Python.org](https://www.python.org/downloads/).

1. Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.5.1**). Если Azure CLI еще не установлен, см. [эту статью](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Azure CLI требуется для того, чтобы с помощью Python можно было использовать **проверки подлинности на основе CLI** в следующих примерах. Дополнительные сведения о других параметрах см. в статье о [проверке подлинности с помощью библиотек управления Azure для Python](/azure/developer/python/azure-sdk-authenticate).

1. Проверка подлинности с помощью Azure CLI.

   ```azurecli
   az login
   ```

1. В выбранной вами среде Python установите необходимые библиотеки для Политики Azure:

   ```bash
   # Add the Python library for Python
   pip install azure-mgmt-policyinsights

   # Add the Resources library for Python
   pip install azure-mgmt-resource

   # Add the CLI Core library for Python for authentication (development only!)
   pip install azure-cli-core
   ```

   > [!NOTE]
   > Если Python установлен для всех пользователей, эти команды необходимо запустить из консоли с повышенными привилегиями.

1. Убедитесь, что библиотеки установлены. `azure-mgmt-policyinsights` должна быть версии **0.5.0** или более поздней, `azure-mgmt-resource` ⁠— **9.0.0** или более поздней, а `azure-cli-core` ⁠— **2.5.0** или более поздней.

   ```bash
   # Check each installed library
   pip show azure-mgmt-policyinsights azure-mgmt-resource azure-cli-core
   ```

## <a name="create-a-policy-assignment"></a>Создание назначения политики

С помощью этого краткого руководства вы создадите назначение политики и назначите определение **Аудит виртуальных машин, которые не используют управляемые диски** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Это определение политики идентифицирует ресурсы, которые не соответствуют заданным в нем условиям.

Чтобы создать назначение политики, запустите следующий код:

```python
# Import specific methods and models from other libraries
from azure.common.credentials import get_azure_cli_credentials
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.resource.policy import PolicyClient
from azure.mgmt.resource.policy.models import PolicyAssignment

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyClient = get_client_from_cli_profile(PolicyClient)

# Create details for the assignment
policyAssignmentDetails = PolicyAssignment(display_name="Audit VMs without managed disks Assignment", policy_definition_id="/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d", scope="{scope}", description="Shows all virtual machines not using managed disks")

# Create new policy assignment
policyAssignment = policyClient.policy_assignments.create("{scope}", "audit-vm-manageddisks", policyAssignmentDetails)

# Show results
print(policyAssignment)
```

В указанных выше командах используются следующие сведения:

Подробные сведения о назначении:
- **display_name** — отображаемое имя назначения политики. В этом случае используется определение _аудита виртуальных машин без назначения управляемых дисков_.
- **policy_definition_id** — путь определения политики, на основе которого вы создаете назначение. В нашем случае это идентификатор определения политики _аудита виртуальных машин, которые не используют управляемые диски_. В этом примере определение политики является встроенным, а путь не содержит информацию о группе управления или подписке.
- **scope** — область, которая определяет, к каким ресурсам или группе ресурсов принудительно применяется назначение политики. Она может варьироваться от группы управления до отдельного ресурса. Обязательно замените `{scope}` на один из следующих шаблонов:
  - Группа управления: `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Подписка: `/subscriptions/{subscriptionId}`
  - группу ресурсов `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`;
  - Ресурс: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`.
- **description** — более подробное объяснение того, что делает политика или почему она относится к этой области.

Создание назначения:

- Scope — область, которая определяет, где сохраняется назначение политики. В этой области должна существовать область, заданная в сведениях назначения.
- Name — фактическое имя назначения. В этом примере использовалось имя _audit-vm-manageddisks_.
- Policy assignment — объект Python **PolicyAssignment**, созданный на предыдущем шаге.

Теперь все готово к обнаружению ресурсов, которые не соответствуют требованиям, что позволит оценить состояние соответствия в среде.

## <a name="identify-non-compliant-resources"></a>Выявление несоответствующих ресурсов

Используйте приведенные ниже сведения для идентификации ресурсов, которые не соответствуют созданному вами назначению политики. Выполните следующий код:

```python
# Import specific methods and models from other libraries
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.policyinsights._policy_insights_client import PolicyInsightsClient
from azure.mgmt.policyinsights.models import QueryOptions

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyInsightsClient = get_client_from_cli_profile(PolicyInsightsClient)

# Set the query options
queryOptions = QueryOptions(filter="IsCompliant eq false and PolicyAssignmentId eq 'audit-vm-manageddisks'",apply="groupby((ResourceId))")

# Fetch 'latest' results for the subscription
results = policyInsightsClient.policy_states.list_query_results_for_subscription(policy_states_resource="latest", subscription_id="{subscriptionId}", query_options=queryOptions)

# Show results
print(results)
```

Замените `{subscriptionId}` подпиской, в которой вы хотите видеть результаты соответствия для этого назначения политики. Список других областей и способов обобщения данных см. в разделе [Методы](/python/api/azure-mgmt-policyinsights/azure.mgmt.policyinsights.operations.policystatesoperations#methods).

Результаты должны выглядеть примерно так:

```output
{
    'additional_properties': {
        '@odata.nextLink': None
    },
    'odatacontext': 'https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest',
    'odatacount': 12,
    'value': [{data}]
}
```

Результаты соответствуют тому, что вы видите на вкладке **Соответствие ресурсов** назначения политики в представлении портала Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить созданное назначение, выполните следующую команду:

```python
# Import specific methods and models from other libraries
from azure.common.client_factory import get_client_from_cli_profile
from azure.mgmt.resource.policy import PolicyClient

# Get your credentials from Azure CLI (development only!) and get your subscription list
policyClient = get_client_from_cli_profile(PolicyClient)

# Delete the policy assignment
policyAssignment = policyClient.policy_assignments.delete("{scope}", "audit-vm-manageddisks")

# Show results
print(policyAssignment)
```

Замените `{scope}` той же областью, которую вы использовали для создания назначения политики.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы назначили определение политики для идентификации ресурсов, не соответствующих требованиям, в среде Azure.

Чтобы узнать больше об определениях политики назначения для проверки новых ресурсов на соответствие требованиям, см. следующее руководство:

> [!div class="nextstepaction"]
> [Создание политик и управление ими](./tutorials/create-and-manage.md)
