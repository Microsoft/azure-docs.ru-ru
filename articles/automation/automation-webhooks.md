---
title: Запуск runbook службы автоматизации Azure с помощью веб-перехватчика
description: В этой статье описывается, как применить веб-перехватчик для запуска runbook в службе автоматизации Azure с использованием HTTP-запросов.
services: automation
ms.subservice: process-automation
ms.date: 03/18/2021
ms.topic: conceptual
ms.openlocfilehash: c46a8753c87e981d9e3d6ecdd698bbbe6cba9894
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104775788"
---
# <a name="start-a-runbook-from-a-webhook"></a>Запуск модуля runbook с помощью веб-перехватчика

Веб-перехватчик позволяет внешним службам запускать определенную последовательность runbook в службе автоматизации Azure с помощью одного HTTP-запроса. Роль внешней службы могут выполнять Azure DevOps Services, GitHub, журналы Azure Monitor и пользовательские приложения. Эта служба будет вызывать веб-перехватчик для запуска runbook, и для этого ей не нужна полная реализация API службы автоматизации Azure. Сравнение веб-перехватчиков с другими методами запуска runbook см. в статье [Запуск модуля Runbook в службе автоматизации Azure](./start-runbooks.md).

> [!NOTE]
> Использование веб-перехватчиков для запуска runbook Python не поддерживается.

![Обзор веб-перехватчиков](media/automation-webhooks/webhook-overview-image.png)

Сведения о требованиях клиента к TLS 1,2 с веб-перехватчиками см. в статье [Принудительная поддержка tls 1,2 для службы автоматизации Azure](automation-managing-data.md#tls-12-enforcement-for-azure-automation).

## <a name="webhook-properties"></a>Свойства веб-перехватчика

В следующей таблице описаны свойства, которые необходимо настроить для объекта Webhook.

| Свойство | Описание |
|:--- |:--- |
| Имя |Имя веб-перехватчика. Вы можете указать любое имя, так как оно не предоставляется клиенту. Имя используется для идентификации модуля Runbook в службе автоматизации Azure. Рекомендуется задать веб-перехватчику имя, связанное с клиентом, который будет его использовать. |
| URL-адрес |URL-адрес веб-перехватчика. Это уникальный адрес, на который клиент отправляет HTTP-запрос POST для запуска последовательности runbook, связанной с этим веб-перехватчиком. URL-адрес создается автоматически при создании веб-перехватчика. Нельзя указать другой URL-адрес. <br> <br> URL-адрес содержит маркер безопасности, который позволяет сторонней системе вызывать runbook без дополнительной проверки подлинности. Это означает, что URL-адрес следует рассматривать как пароль. По соображениям безопасности просмотреть URL-адрес на портале Azure можно только при создании веб-перехватчика. Запишите URL-адрес в надежном месте, чтобы использовать его в дальнейшем. |
| Срок действия | Дата окончания срока действия веб-перехватчика, после которой он не может больше использоваться. Вы можете изменить срок действия после создания веб-перехватчика, если он еще не истек. |
| Активировано | Этот параметр обозначает, будет ли веб-перехватчик автоматически включен после создания. Если задать для этого свойства значение Disabled (Отключено), клиенты не смогут использовать веб-перехватчик. Это свойство можно задать при создании веб-перехватчика или изменить в любое время позднее. |

## <a name="parameters-used-when-the-webhook-starts-a-runbook"></a>Параметры, используемые при запуске runbook веб-перехватчиком

Веб-перехватчик может определять значения для параметров runbook, которые используются при запуске runbook. Веб-перехватчик должен содержать значения всех обязательных параметров runbook и может включать необязательные параметры. Значение параметра, настроенного для веб-перехватчика, можно изменить даже после создания веб-перехватчика. Несколько веб-перехватчиков, привязанных к одной последовательности runbook, могут использовать разные значения параметров. Когда клиент запускает модуль Runbook с помощью веб-перехватчика, клиент не может переопределить значения параметров, определяемые веб-перехватчиком.

Чтобы получать данные от клиента, runbook поддерживает один параметр с именем `WebhookData`. Этот параметр определяет объект с данными, которые клиент добавляет в запрос POST.

![Свойства WebhookData](media/automation-webhooks/webhook-data-properties.png)

Параметр `WebhookData` содержит следующие свойства:

| Свойство | Описание |
|:--- |:--- |
| `WebhookName` | Имя веб-перехватчика. |
| `RequestHeader` | Хэш-таблица, содержащая заголовки входящего запроса POST. |
| `RequestBody` | Текст входящего запроса POST. В нем сохраняется любое форматирование (строка, JSON, XML, кодировка данных в форме и т. п.). Модуль Runbook должен быть написан для работы с форматом данных, который ожидается. |

Веб-перехватчик не нужно настраивать для поддержки параметра `WebhookData`, и последовательность runbook не обязана его принимать. Если runbook не определяет параметр, любые сведения в запросе клиента игнорируются.

> [!NOTE]
> При вызове веб-перехватчика клиент всегда должен сохранять значения параметров на случай сбоя при этом вызове. Если случится сбой сети или проблемы с подключением, приложение не сможет получить вызовы веб-перехватчика, завершившиеся сбоем.

Если вы укажете значение `WebhookData` при создании веб-перехватчика, при запуске runbook оно переопределится данными из запроса POST от клиента. Это происходит даже в том случае, если приложение не передает данные в тексте запроса. 

При запуске последовательности runbook, определяющей `WebhookData`, любым другим способом, кроме веб-перехватчика, вы можете указать распознаваемое значение для `WebhookData`. Это значение должно быть объектом с теми же [свойствами](#webhook-properties), что у параметра `WebhookData`, чтобы последовательность runbook могла работать с ним точно так же, как с обычными объектами `WebhookData`, полученными от веб-перехватчика.

Например, если вы запускаете на портале Azure следующую последовательность runbook и хотите передать примеры данных от веб-перехватчика для тестирования, то эти данные нужно передавать в формате JSON из пользовательского интерфейса.

![Параметр WebhookData из пользовательского интерфейса](media/automation-webhooks/WebhookData-parameter-from-UI.png)

Для следующего примера runbook мы определим следующие свойства `WebhookData`.

* **WebhookName**: MyWebhook.
* **RequestBody**: `*[{'ResourceGroup': 'myResourceGroup','Name': 'vm01'},{'ResourceGroup': 'myResourceGroup','Name': 'vm02'}]*`.

Эти данные мы передаем в виде следующего объекта JSON через параметр `WebhookData` в пользовательском интерфейсе. Этот пример с символами возврата каретки и новой строки соответствует формату, который передается из веб-перехватчика.

```json
{"WebhookName":"mywebhook","RequestBody":"[\r\n {\r\n \"ResourceGroup\": \"vm01\",\r\n \"Name\": \"vm01\"\r\n },\r\n {\r\n \"ResourceGroup\": \"vm02\",\r\n \"Name\": \"vm02\"\r\n }\r\n]"}
```

![Запуск параметра WebhookData из пользовательского интерфейса](media/automation-webhooks/Start-WebhookData-parameter-from-UI.png)

> [!NOTE]
> Служба автоматизации Azure сохраняет в журнал значения всех входных параметров вместе с заданием runbook. Таким образом, любые предоставленные клиентом входные данные из запроса веб-перехватчика будут записаны в журнал и станут доступны любому пользователю с доступом к заданию автоматизации. По этой причине необходимо соблюдать осторожность при указании критических данных в вызовах Webhook.

## <a name="webhook-security"></a>Безопасность веб-перехватчика

Безопасность веб-перехватчика зависит от защищенности URL-адреса, который содержит маркер безопасности, позволяющий вызывать этот веб-перехватчик. Служба автоматизации Azure не выполняет проверку подлинности для запроса, если он выполняется с правильным URL-адресом. По этой причине клиентам не следует применять веб-перехватчики для runbook, которые выполняют важные функции, без использования дополнительной проверки запроса.

Рассмотрим следующие стратегии.

* Вы можете включить в последовадовательность runbook дополнительную логику для проверки, вызывается ли она веб-перехватчиком. Убедитесь, что runbook проверяет свойство `WebhookName` из параметра `WebhookData`. Для дополнительной проверки runbook может искать наличие определенной информации в свойствах `RequestHeader`или `RequestBody`.

* При получении запроса веб-перехватчика модуль Runbook выполняет некоторую проверку внешнего условия. Для примера рассмотрим runbook, вызываемую из GitHub каждый раз, когда в репозитории появляется новая фиксация. Такая последовательность runbook может подключаться к GitHub, чтобы проверить наличие фиксации, прежде чем продолжать работу.

* Служба автоматизации Azure поддерживает теги службы виртуальной сети Azure, в частности [гуестандхибридманажемент](../virtual-network/service-tags-overview.md). Теги службы можно использовать для определения элементов управления доступом к сети для [групп безопасности сети](../virtual-network/network-security-groups-overview.md#security-rules) или [брандмауэра Azure](../firewall/service-tags.md) , а также для активации веб-перехватчиков в виртуальной сети. Теги службы можно использовать вместо конкретных IP-адресов при создании правил безопасности. Указав имя тега службы **гуестандхибридманажемент**  в соответствующем поле источника или назначения правила, можно разрешить или запретить трафик для службы автоматизации. Этот тег службы не поддерживает более детализированный контроль за счет ограниченного диапазона IP-адресов в определенном регионе.

## <a name="create-a-webhook"></a>Создание веб-перехватчика

Чтобы создать новый веб-перехватчик, связанный с модулем Runbook, на портале Azure, воспользуйтесь следующей процедурой.

1. На странице "Модули Runbook" на портале Azure щелкните запускаемую веб-перехватчиком последовательность runbook, чтобы открыть страницу с подробной информацией о ней. Убедитесь, что для поля **Состояние** runbook задано значение **Опубликовано**.
2. Щелкните **Веб-перехватчик** в верхней части страницы, чтобы открыть страницу "Добавить веб-перехватчик".
3. Щелкните **Создать новый веб-перехватчик**, чтобы открыть страницу "Создать веб-перехватчик".
4. Заполните для веб-перехватчика поля **Имя** и **Срок действия** и укажите, следует ли его сразу включить. Дополнительные сведения о свойствах см. в разделе [Свойства веб-перехватчика](#webhook-properties).
5. Щелкните значок копирования и нажмите Ctrl+C, чтобы скопировать URL-адрес объекта Webhook. Сохраните URL-адрес в безопасном месте. 

    > [!IMPORTANT]
    > После создания веб-перехватчика URL-адрес нельзя получить повторно. Убедитесь, что вы скопировали и записываете его, как показано выше.

   ![URL-адрес Webhook](media/automation-webhooks/copy-webhook-url.png)

1. Щелкните **Параметры**, чтобы указать значения для параметров модуля Runbook. Если у runbook есть обязательные параметры, вы не сможете создать веб-перехватчик без значений для этих параметров.

2. Щелкните **Создать**, чтобы создать объект Webhook.

## <a name="use-a-webhook"></a>Использование веб-перехватчика

Чтобы использовать созданный веб-перехватчик, клиент должен создать HTTP-запрос `POST` на URL-адрес этого веб-перехватчика. Синтаксис:

```http
http://<Webhook Server>/token?=<Token Value>
```

В ответ на запрос `POST` клиент получит один из следующих кодов возврата.

| Код | текст | Описание |
|:--- |:--- |:--- |
| 202 |Принято |Запрос был принят, и модуль Runbook успешно поставлен в очередь. |
| 400 |Ошибка запроса |Запрос не был принят по одной из следующих причин: <ul> <li>Срок действия объекта webhook истек.</li> <li>Объект webhook отключен.</li> <li>Недопустимый токен в URL-адресе.</li>  </ul> |
| 404 |Не найдено |Запрос не был принят по одной из следующих причин: <ul> <li>Объект webhook не найден.</li> <li>Модуль Runbook не найден.</li> <li>Учетная запись не найдена.</li>  </ul> |
| 500 |Внутренняя ошибка сервера |URL-адрес допустимый, но произошла ошибка. Отправьте запрос повторно. |

Если запрос выполняется успешно, то ответ веб-перехватчика будет содержать идентификатор задания в формате JSON, как показано далее. Это будет только один идентификатор задания, но формат JSON позволяет в будущем расширить эти возможности.

```json
{"JobIds":["<JobId>"]}
```

Клиент не может определить момент завершения задания Runbook и состояние его завершения из веб-перехватчика. Он может находить эти сведения на основе идентификатора задания, используя другой механизм, например [Windows PowerShell](/powershell/module/servicemanagement/azure.service/get-azureautomationjob) или [API службы автоматизации Azure](/rest/api/automation/job).

### <a name="use-a-webhook-from-an-arm-template"></a>Использование веб-перехватчика из шаблона ARM

Веб-перехватчики службы автоматизации также можно вызывать с помощью [шаблонов Azure Resource Manager (ARM)](/azure/azure-resource-manager/templates/overview). Шаблон ARM выдает `POST` запрос и получает код возврата, как и любой другой клиент. См. раздел [Использование веб-перехватчика](#use-a-webhook).

   > [!NOTE]
   > По соображениям безопасности URI возвращается только при первом развертывании шаблона.

Этот пример шаблона создает тестовую среду и возвращает универсальный код ресурса (URI) для создаваемого веб-перехватчика.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "automationAccountName": {
            "type": "String",
            "metadata": {
                "description": "Automation account name"
            }
        },
        "webhookName": {
            "type": "String",
            "metadata": {
                "description": "Webhook Name"
            }
        },
        "runbookName": {
            "type": "String",
            "metadata": {
                "description": "Runbook Name for which webhook will be created"
            }
        },
        "WebhookExpiryTime": {
            "type": "String",
            "metadata": {
                "description": "Webhook Expiry time"
            }
        },
        "_artifactsLocation": {
            "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-automation/",
            "type": "String",
            "metadata": {
                "description": "URI to artifacts location"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Automation/automationAccounts",
            "apiVersion": "2020-01-13-preview",
            "name": "[parameters('automationAccountName')]",
            "location": "[resourceGroup().location]",
            "properties": {
                "sku": {
                    "name": "Free"
                }
            },
            "resources": [
                {
                    "type": "runbooks",
                    "apiVersion": "2018-06-30",
                    "name": "[parameters('runbookName')]",
                    "location": "[resourceGroup().location]",
                    "dependsOn": [
                        "[parameters('automationAccountName')]"
                    ],
                    "properties": {
                        "runbookType": "Python2",
                        "logProgress": "false",
                        "logVerbose": "false",
                        "description": "Sample Runbook",
                        "publishContentLink": {
                            "uri": "[uri(parameters('_artifactsLocation'), 'scripts/AzureAutomationTutorialPython2.py')]",
                            "version": "1.0.0.0"
                        }
                    }
                },
                {
                    "type": "webhooks",
                    "apiVersion": "2018-06-30",
                    "name": "[parameters('webhookName')]",
                    "dependsOn": [
                        "[parameters('automationAccountName')]",
                        "[parameters('runbookName')]"
                    ],
                    "properties": {
                        "isEnabled": true,
                        "expiryTime": "[parameters('WebhookExpiryTime')]",
                        "runbook": {
                            "name": "[parameters('runbookName')]"
                        }
                    }
                }
            ]
        }
    ],
    "outputs": {
        "webhookUri": {
            "type": "String",
            "value": "[reference(parameters('webhookName')).uri]"
        }
    }
}
```

## <a name="renew-a-webhook"></a>Обновление веб-перехватчика

При создании веб-перехватчика устанавливается срок действия 10 лет, по истечении которого он автоматически прекращает работу. Веб-перехватчик с истекшим сроком действия невозможно активировать повторно. Можно лишь удалить его и создать заново. 

Но вы можете продлить срок действия веб-перехватчика, пока этот срок еще не истек. Чтобы продлить срок действия веб-перехватчика, сделайте следующее:

1. Перейдите к последовательности runbook, которая содержит веб-перехватчик. 
2. Выберите **Веб-перехватчики** в разделе **Ресурсы**. 
3. Щелкните веб-перехватчик, для которого нужно продлить срок действия. 
4. На странице сведений о веб-перехватчике введите новую дату окончания срока действия и щелкните **Сохранить**.

## <a name="sample-runbook"></a>Пример Runbook

Следующий пример модуля Runbook принимает данные веб-перехватчика и запускает виртуальные машины, указанные в тексте запроса. Для тестирования этой последовательности runbook в учетной записи службы автоматизации в разделе **Модули Runbook** нажмите кнопку **Создать Runbook**. Если вы не знаете, как создать модуль Runbook, ознакомьтесь с [этой статьей](automation-quickstart-create-runbook.md).

> [!NOTE]
> Для неграфических runbook PowerShell `Add-AzAccount` и `Add-AzureRMAccount` являются псевдонимами для [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Вы можете использовать эти командлеты или [обновить модули](automation-update-azure-modules.md) в учетной записи службы автоматизации до последних версий. Обновление модулей может потребоваться, даже если учетная запись службы автоматизации только что создана.

```powershell
param
(
    [Parameter (Mandatory = $false)]
    [object] $WebhookData
)

# If runbook was called from Webhook, WebhookData will not be null.
if ($WebhookData) {

    # Check header for message to validate request
    if ($WebhookData.RequestHeader.message -eq 'StartedbyContoso')
    {
        Write-Output "Header has required information"}
    else
    {
        Write-Output "Header missing required information";
        exit;
    }

    # Retrieve VMs from Webhook request body
    $vms = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)

    # Authenticate to Azure by using the service principal and certificate. Then, set the subscription.

    Write-Output "Authenticating to Azure with service principal and certificate"
    $ConnectionAssetName = "AzureRunAsConnection"
    Write-Output "Get connection asset: $ConnectionAssetName"

    $Conn = Get-AutomationConnection -Name $ConnectionAssetName
            if ($Conn -eq $null)
            {
                throw "Could not retrieve connection asset: $ConnectionAssetName. Check that this asset exists in the Automation account."
            }
            Write-Output "Authenticating to Azure with service principal."
            Add-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint | Write-Output

        # Start each virtual machine
        foreach ($vm in $vms)
        {
            $vmName = $vm.Name
            Write-Output "Starting $vmName"
            Start-AzVM -Name $vm.Name -ResourceGroup $vm.ResourceGroup
        }
}
else {
    # Error
    write-Error "This runbook is meant to be started from an Azure alert webhook only."
}
```

## <a name="test-the-sample"></a>Тестирование примера

В следующем примере модуль Runbook запускается с помощью Webhook из Windows PowerShell. Веб-перехватчик можно использовать на любом языке, который позволяет создать HTTP-запрос. В нашем примере используется Windows PowerShell.

Модуль Runbook ожидает список виртуальных машин, отформатированный в JSON, в теле запроса. Runbook также проверяет наличие в заголовках определенного сообщения для подтверждения допустимости вызывающего веб-перехватчика.

```azurepowershell-interactive
$uri = "<webHook Uri>"

$vms  = @(
            @{ Name="vm01";ResourceGroup="vm01"},
            @{ Name="vm02";ResourceGroup="vm02"}
        )
$body = ConvertTo-Json -InputObject $vms
$header = @{ message="StartedbyContoso"}
$response = Invoke-WebRequest -Method Post -Uri $uri -Body $body -Headers $header
$jobid = (ConvertFrom-Json ($response.Content)).jobids[0]
```

В следующем примере показан текст запроса, который runbook получает в свойстве `RequestBody` объекта `WebhookData` Это значение имеет формат JSON для совместимости с форматом, включенным в текст запроса.

```json
[
    {
        "Name":  "vm01",
        "ResourceGroup":  "myResourceGroup"
    },
    {
        "Name":  "vm02",
        "ResourceGroup":  "myResourceGroup"
    }
]
```

На следующем рисунке показан запрос, отправляемый из Windows PowerShell, а также получаемый в результате ответ. Идентификатор задания извлекается из ответа и преобразуется в строку.

![Кнопка Webhook](media/automation-webhooks/webhook-request-response.png)

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы вызвать runbook с помощью предупреждения, см. статью [Использование оповещения для активации runbook службы автоматизации Azure](automation-create-alert-triggered-runbook.md).