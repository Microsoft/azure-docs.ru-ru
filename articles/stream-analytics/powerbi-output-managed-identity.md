---
title: Использование управляемого удостоверения для проверки подлинности задания Azure Stream Analytics для Power BI выходных данных
description: В этой статье описывается, как использовать управляемые удостоверения для проверки подлинности задания Azure Stream Analytics для Power BI выходных данных.
ms.service: stream-analytics
author: enkrumah
ms.author: ebnkruma
ms.topic: how-to
ms.date: 3/10/2020
ms.openlocfilehash: b468df97f168562f4f5ce9e0e6d6e9a70835648b
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102436113"
---
# <a name="use-managed-identity-to-authenticate-your-azure-stream-analytics-job-to-power-bi-preview"></a>Использование управляемого удостоверения для проверки подлинности задания Azure Stream Analytics в Power BI (Предварительная версия)

[Проверка подлинности с помощью управляемого удостоверения](../active-directory/managed-identities-azure-resources/overview.md) для выходных данных Power BI предоставляет Stream Analytics заданиям прямой доступ к рабочей области в учетной записи Power BI. Эта функция позволяет развертывать задания Stream Analytics для полной автоматизации, поскольку больше не требуется пользователю интерактивно входить в систему для Power BI с помощью портал Azure. Кроме того, длительные задания, выполняющие запись в Power BI, теперь лучше поддерживаются, так как вам не потребуется периодически повторно авторизовать задание.

В этой статье показано, как включить управляемое удостоверение для Power BI выходных данных Stream Analytics задания с помощью портал Azure и Azure Resource Manager развертывания.

## <a name="prerequisites"></a>Предварительные требования

Для использования этой функции необходимо следующее:

- Учетная запись Power BI с [лицензией Pro](/power-bi/service-admin-purchasing-power-bi-pro).

- Обновленная рабочая область в учетной записи Power BI. Дополнительные сведения см. в описании [Power BI](https://powerbi.microsoft.com/blog/announcing-new-workspace-experience-general-availability-ga/) этой функции.

## <a name="create-a-stream-analytics-job-using-the-azure-portal"></a>Создание задания Stream Analytics с помощью портал Azure

1. Создайте новое Stream Analytics задание или откройте существующее задание в портал Azure. В строке меню, расположенной в левой части экрана, выберите **управляемое удостоверение** , расположенное в разделе **Настройка**. Убедитесь, что выбран параметр "использовать управляемое системой удостоверение", а затем нажмите кнопку **сохранить** в нижней части экрана.

   ![Настройка управляемого удостоверения Stream Analytics](./media/common/stream-analytics-enable-managed-identity.png)

2. Перед настройкой выходных данных предоставьте Stream Analyticsному заданию доступ к рабочей области Power BI, следуя указаниям в разделе [предоставление Stream Analytics заданий доступ к рабочей области Power BI](#give-the-stream-analytics-job-access-to-your-power-bi-workspace) этой статьи.

3. Перейдите к разделу **выходные данные** задания аналитики Stream, выберите **+ Добавить**, а затем выберите **Power BI**. Затем нажмите кнопку **авторизовать** и войдите в систему, используя учетную запись Power BI.

   ![Авторизация с учетной записью Power BI](./media/stream-analytics-powerbi-output-managed-identity/stream-analytics-authorize-powerbi.png)

4. После авторизации раскрывающийся список будет заполнен всеми рабочими областями, к которым у вас есть доступ. Выберите рабочую область, которая была проверена на предыдущем шаге. Затем выберите **управляемое удостоверение** в качестве режима проверки подлинности. Наконец, нажмите кнопку **сохранить** .

   ![Настройка вывода Power BI с помощью управляемого удостоверения](./media/stream-analytics-powerbi-output-managed-identity/stream-analytics-configure-powerbi-with-managed-id.png)

## <a name="azure-resource-manager-deployment"></a>Развертывание Azure Resource Manager

Azure Resource Manager позволяет полностью автоматизировать развертывание задания Stream Analytics. Шаблоны диспетчер ресурсов можно развернуть с помощью Azure PowerShell или [Azure CLI](/cli/azure/). В приведенных ниже примерах используется Azure CLI.


1. Вы можете создать ресурс **Microsoft. StreamAnalytics/стреамингжобс** с управляемым удостоверением, включив следующее свойство в раздел ресурсов шаблона диспетчер ресурсов:

    ```json
    "identity": {
        "type": "SystemAssigned",
    }
    ```

   Это свойство указывает Azure Resource Manager создать удостоверение для задания Stream Analytics и управлять им. Ниже приведен пример диспетчер ресурсов шаблона, который развертывает задание Stream Analytics с включенным управляемым удостоверением и приемником Power BI вывода, который использует управляемое удостоверение:

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
            {
                "apiVersion": "2017-04-01-preview",
                "name": "pbi_managed_id",
                "location": "[resourceGroup().location]",
                "type": "Microsoft.StreamAnalytics/StreamingJobs",
                "identity": {
                    "type": "systemAssigned"
                },
                "properties": {
                    "sku": {
                        "name": "standard"
                    },
                    "outputs":[
                        {
                            "name":"output",
                            "properties":{
                                "datasource":{
                                    "type":"PowerBI",
                                    "properties":{
                                        "dataset": "dataset_name",
                                        "table": "table_name",
                                        "groupId": "01234567-89ab-cdef-0123-456789abcdef",
                                        "authenticationMode": "Msi"
                                    }
                                }
                            }
                        }
                    ]
                }
            }
        ]
    }
    ```

    Разверните указанное выше задание в группе ресурсов **ексамплеграуп** с помощью следующей Azure CLI команды:

    ```azurecli
    az deployment group create --resource-group ExampleGroup -template-file StreamingJob.json
    ```

2. После создания задания используйте Azure Resource Manager, чтобы получить полное определение задания.

    ```azurecli
    az resource show --ids /subscriptions/<subsription-id>/resourceGroups/<resource-group>/providers/Microsoft.StreamAnalytics/StreamingJobs/<resource-name>
    ```

    Приведенная выше команда возвратит ответ, подобный приведенному ниже.

    ```json
    {
        "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.StreamAnalytics/streamingjobs/<resource-name>",
        "identity": {
            "principalId": "<principal-id>",
            "tenantId": "<tenant-id>",
            "type": "SystemAssigned",
            "userAssignedIdentities": null
        },
        "kind": null,
        "location": "West US",
        "managedBy": null,
        "name": "<resource-name>",
        "plan": null,
        "properties": {
            "compatibilityLevel": "1.0",
            "createdDate": "2019-07-12T03:11:30.39Z",
            "dataLocale": "en-US",
            "eventsLateArrivalMaxDelayInSeconds": 5,
            "jobId": "<job-id>",
            "jobState": "Created",
            "jobStorageAccount": null,
            "jobType": "Cloud",
            "outputErrorPolicy": "Stop",
            "package": null,
            "provisioningState": "Succeeded",
            "sku": {
                "name": "Standard"
            }
        },
        "resourceGroup": "<resource-group>",
        "sku": null,
        "tags": null,
        "type": "Microsoft.StreamAnalytics/streamingjobs"
    }
    ```

    Если вы планируете использовать REST API Power BI, чтобы добавить задание Stream Analytics в рабочую область Power BI, запишите возвращаемое значение "principalId".

3. Теперь, когда задание создано, перейдите к разделу [предоставление Stream Analyticsного задания доступ к рабочей области Power BI](#give-the-stream-analytics-job-access-to-your-power-bi-workspace) этой статьи.


## <a name="give-the-stream-analytics-job-access-to-your-power-bi-workspace"></a>Предоставление Stream Analytics заданий доступа к рабочей области Power BI

После создания задания Stream Analytics ему может быть предоставлен доступ к рабочей области Power BI. Получив доступ к задачам, подождите несколько минут, пока удостоверение не будет распространено.

### <a name="use-the-power-bi-ui"></a>Использование пользовательского интерфейса Power BI

   > [!Note]
   > Чтобы добавить Stream Analytics задание в рабочую область Power BI с помощью пользовательского интерфейса, необходимо также включить доступ субъекта-службы в **параметрах разработчика** на портале администрирования Power BI. Дополнительные сведения см. [в статье Приступая к работе с субъектом-службой](/power-bi/developer/embed-service-principal) .

1. Перейдите к параметрам доступа к рабочей области. Дополнительные сведения см. в этой статье: [Предоставьте доступ к рабочей области](/power-bi/service-create-the-new-workspaces#give-access-to-your-workspace).

2. Введите имя задания Stream Analytics в текстовом поле и выберите **участник** в качестве уровня доступа.

3. Выберите **Добавить** и закройте панель.

   ![Добавление Stream Analytics задания в рабочую область Power BI](./media/stream-analytics-powerbi-output-managed-identity/stream-analytics-add-job-to-powerbi-workspace.png)

### <a name="use-the-power-bi-powershell-cmdlets"></a>Использование командлетов PowerShell Power BI

1. Установите `MicrosoftPowerBIMgmt` командлеты PowerShell Power BI.

   > [!Important]
   > Убедитесь, что вы используете командлеты версии 1.0.821 или более поздней.

```powershell
Install-Module -Name MicrosoftPowerBIMgmt
```

2. Войдите в Power BI.

```powershell
Login-PowerBI
```

3. Добавьте Stream Analytics задание в качестве участника в рабочую область.

```powershell
Add-PowerBIWorkspaceUser -WorkspaceId <group-id> -PrincipalId <principal-id> -PrincipalType App -AccessRight Contributor
```

### <a name="use-the-power-bi-rest-api"></a>Использование REST API Power BI

Stream Analytics задание можно также добавить в качестве участника в рабочую область, воспользовавшись REST API напрямую с помощью команды "добавить пользователя группы". Полную документацию по этому API можно найти здесь: [группы — добавить пользователя группы](/rest/api/power-bi/groups/addgroupuser).

**Пример запроса**
```http
POST https://api.powerbi.com/v1.0/myorg/groups/{groupId}/users
```
Текст запроса
```json
{
    "groupUserAccessRight": "Contributor",
    "identifier": "<principal-id>",
    "principalType": "App"
}
```

## <a name="remove-managed-identity"></a>Удалить управляемое удостоверение

Управляемое удостоверение, созданное для задания Stream Analytics, удаляется только при удалении задания. Невозможно удалить управляемое удостоверение, не удаляя задание. Если вы больше не хотите использовать управляемое удостоверение, можно изменить метод проверки подлинности для выходных данных. Управляемое удостоверение будет продолжать существовать до тех пор, пока задание не будет удалено, и будет использоваться при повторном использовании проверки подлинности с управляемым удостоверением.

## <a name="limitations"></a>Ограничения
Ниже перечислены ограничения этой функции.

- Классические рабочие области Power BI не поддерживаются.

- Учетные записи Azure без Azure Active Directory.

- Многоклиентский доступ не поддерживается. Субъект-служба, созданная для данного задания Stream Analytics, должна находиться в том же Azure Active Directoryном клиенте, в котором было создано задание, и не может использоваться с ресурсом, который находится в другом клиенте Azure Active Directory.

- [Пользователь, которому назначено удостоверение](../active-directory/managed-identities-azure-resources/overview.md) , не поддерживается. Это означает, что вы не можете ввести собственный субъект-службу, который будет использоваться их Stream Analyticsным заданием. Субъект-служба необходимо создавать с помощью Azure Stream Analytics.

## <a name="next-steps"></a>Дальнейшие действия

* [Интеграция панели мониторинга Power BI с Azure Stream Analytics](./stream-analytics-power-bi-dashboard.md)
* [Описание выходных данных из Azure Stream Analytics](./stream-analytics-define-outputs.md)