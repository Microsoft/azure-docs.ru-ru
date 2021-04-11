---
title: Краткое руководство. Создание назначения политики с помощью Terraform
description: В этом кратком руководстве с помощью Terraform и синтаксиса HCL создается назначение политики для обнаружения ресурсов, не соответствующих требованиям.
ms.date: 03/31/2021
ms.topic: quickstart
ms.openlocfilehash: 6c1a43010c48f71872d935a8fb52f608b479da8a
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106092353"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-terraform"></a>Краткое руководство. Создание назначения политики для выявления ресурсов, не соответствующих требованиям, с использованием Terraform

Чтобы понять, соответствуют ли ресурсы требованиям в Azure, прежде всего нужно определить их состояние.
В этом кратком руководстве описано, как создать назначение политики для определения виртуальных машин, которые не используют управляемые диски.

Завершив работу, вы узнаете, какие виртуальные машины не используют управляемые диски, так как _не соответствуют_ назначению политики.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.
- [Средство Terraform](https://www.terraform.io/) версии 0.12.0 или выше, настроенное в вашей среде.
  Инструкции см. в кратком руководстве [Настройка Terraform с помощью Azure Cloud Shell](/azure/developer/terraform/get-started-cloud-shell).
- Для целей этого краткого руководства требуется запустить Azure CLI 2.13.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="create-the-terraform-configuration-variable-and-output-file"></a>Создание конфигурации, переменной и выходного файла Terraform

С помощью этого краткого руководства вы создадите назначение политики и назначите определение **Аудит виртуальных машин, которые не используют управляемые диски** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Это определение политики идентифицирует ресурсы, которые не соответствуют заданным в нем условиям.

Сначала настройте конфигурацию Terraform, переменную и выходные файлы. Ресурсы Terraform для Политики Azure используют [Поставщик Azure](https://www.terraform.io/docs/providers/azurerm/index.html).

1. Создайте новую папку с именем `policy-assignment` и измените в ней каталоги.

1. Создайте `main.tf`, используя следующий код:

   ```hcl
   provider "azurerm" {
       version = "~>2.0"
       features {}
   }
   
   resource "azurerm_policy_assignment" "auditvms" {
       name = "audit-vm-manageddisks"
       scope = var.cust_scope
       policy_definition_id = "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d"
       description = "Shows all virtual machines not using managed disks"
       display_name = "Audit VMs without managed disks Assignment"
   }
   ```
1. Создайте `variables.tf`, используя следующий код:

   ```hcl
   variable "cust_scope" {
       default = "{scope}"
   }
   ```

   Она определяет, к каким ресурсам или группе ресурсов принудительно применяется назначение политики. Она может варьироваться от группы управления до отдельного ресурса. Обязательно замените `{scope}` на один из следующих шаблонов:

   - Подписка: `/subscriptions/{subscriptionId}`
   - группу ресурсов `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`;
   - Ресурс: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`.

1. Создайте `output.tf`, используя следующий код:

   ```hcl
   output "assignment_id" {
       value = azurerm_policy_assignment.auditvms.id
   }
   ```

## <a name="initialize-terraform-and-create-plan"></a>Инициализация Terraform и создание плана

Затем инициализируйте Terraform, чтобы скачать необходимые поставщики, после чего создайте план.

1. Выполните команду [terraform init](https://www.terraform.io/docs/commands/init.html). В результате этой команды будут скачаны модули Azure, необходимые для создания ресурсов Azure в конфигурации Terraform.

   ```bash
   terraform init
   ```

   :::image type="content" source="./media/assign-policy-terraform/terraform-initialize.png" alt-text="Снимок экрана: выполнение команды terraform init, загрузка модуля azurerm и сообщение об успешном выполнении.":::

1. Выполните проверку подлинности с помощью [Azure CLI](/cli/azure/) для Terraform. Дополнительные сведения см. в документации [Поставщик Azure. Проверка подлинности с помощью Azure CLI](https://www.terraform.io/docs/providers/azurerm/guides/azure_cli.html).

   ```bash
   az login
   ```

1. Создайте план выполнения с помощью команды [terraform plan](https://www.terraform.io/docs/commands/plan.html) и параметра **out**.

   ```bash
   terraform plan -out assignment.tfplan
   ```

   :::image type="content" source="./media/assign-policy-terraform/terraform-plan-out.png" alt-text="Снимок экрана: выполнение команды terraform plan и параметра out для отображения ресурса Azure, который будет создан.":::

   > [!NOTE]
   > Сведения о сохранении планов выполнения и безопасности см. в разделе [План Terraform. Предупреждение системы безопасности](https://www.terraform.io/docs/commands/plan.html#security-warning).

## <a name="apply-the-terraform-execution-plan"></a>Применение плана выполнения Terraform

Наконец, примените план выполнения.

Выполните команду [terraform apply](https://www.terraform.io/docs/commands/apply.html) и укажите уже созданный `assignment.tfplan`.

```bash
terraform apply assignment.tfplan
```

:::image type="content" source="./media/assign-policy-terraform/terraform-apply.png" alt-text="Снимок экрана: выполнение команды terraform apply и создание результирующего ресурса.":::

Если вы получили сообщение "Применено! Ресурсы: 1 добавлен, 0 изменено, 0 удалено", назначение политики создано. Поскольку мы определили файл `outputs.tf`, также возвращается значение _assignment\_id_.

## <a name="identify-non-compliant-resources"></a>Выявление несоответствующих ресурсов

Чтобы просмотреть ресурсы, которые не соответствуют новому назначению, используйте значение _assignment\_id_, возвращенное командой `terraform apply`. Теперь выполните следующую команду, чтобы вывести идентификаторы несоответствующих ресурсов в JSON-файле:

```console
armclient post "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$filter=IsCompliant eq false and PolicyAssignmentId eq '<policyAssignmentID>'&$apply=groupby((ResourceId))" > <json file to direct the output with the resource IDs into>
```

Результаты должны выглядеть примерно так:

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 3,
    "value": [{
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachineId>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine2Id>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3ID>"
        }

    ]
}
```

Результаты похожи на содержимое списка **несовместимых ресурсов** в представлении портала Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить созданное назначение, используйте Azure CLI или отмените план выполнения Terraform с помощью команды `terraform destroy`.

- Azure CLI

  ```azurecli-interactive
  az policy assignment delete --name 'audit-vm-manageddisks' --scope '/subscriptions/<subscriptionID>/<resourceGroupName>'
  ```

- Terraform

  ```bash
  terraform destroy assignment.tfplan
  ```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы назначили определение политики для идентификации ресурсов, не соответствующих требованиям, в среде Azure.

Следующее руководство серии содержит сведения о назначении политик для проверки новых ресурсов на соответствие требованиям:

> [!div class="nextstepaction"]
> [Создание политик и управление ими](./tutorials/create-and-manage.md)
