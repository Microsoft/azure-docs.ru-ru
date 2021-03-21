---
title: Проверка подлинности выходных данных BLOB-объекта с помощью управляемого Azure Stream Analytics
description: В этой статье описывается, как использовать управляемые удостоверения для проверки подлинности задания Azure Stream Analytics в выходных данных хранилища BLOB-объектов Azure.
author: kim-ale
ms.author: kimal
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/15/2020
ms.openlocfilehash: 369348133f7395f5db5b5923bd438cec8e4ad733
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98954384"
---
# <a name="use-managed-identity-preview-to-authenticate-your-azure-stream-analytics-job-to-azure-blob-storage"></a>Использование управляемого удостоверения (Предварительная версия) для проверки подлинности задания Azure Stream Analytics в хранилище BLOB-объектов Azure

[Проверка подлинности управляемого удостоверения](../active-directory/managed-identities-azure-resources/overview.md) (Предварительная версия) для вывода в хранилище BLOB-объектов Azure предоставляет Stream Analytics заданиям прямой доступ к учетной записи хранения вместо использования строки подключения. Помимо улучшенной безопасности, эта функция также позволяет записывать данные в учетную запись хранения в виртуальной сети (VNET) в Azure.

В этой статье показано, как включить управляемое удостоверение для выходных данных большого двоичного объекта Stream Analytics задания с помощью портал Azure и развертывания Azure Resource Manager.

## <a name="create-the-stream-analytics-job-using-the-azure-portal"></a>Создание задания Stream Analytics с помощью портал Azure

1. Создайте новое Stream Analytics задание или откройте существующее задание в портал Azure. В строке меню, расположенной в левой части экрана, выберите **управляемое удостоверение** , расположенное в разделе **Настройка**. Убедитесь, что выбран параметр "использовать управляемое системой удостоверение", а затем нажмите кнопку **сохранить** в нижней части экрана.

   ![Настройка управляемого удостоверения Stream Analytics](./media/common/stream-analytics-enable-managed-identity.png)

2. В окне Выходные свойства приемника выходных данных хранилища BLOB-объектов Azure выберите раскрывающийся список режим проверки подлинности и щелкните **управляемое удостоверение**. Дополнительные сведения о других выходных свойствах см. в разделе сведения о [выходных данных из Azure Stream Analytics](./stream-analytics-define-outputs.md). Закончив, нажмите кнопку **Сохранить**.

   ![Настройка выходных данных хранилища BLOB-объектов Azure](./media/stream-analytics-managed-identities-blob-output-preview/stream-analytics-blob-output-blade.png)

3. Теперь, когда задание создано, см. раздел [предоставление Stream Analyticsного задания доступ к учетной записи хранения](#give-the-stream-analytics-job-access-to-your-storage-account) этой статьи.

## <a name="azure-resource-manager-deployment"></a>Развертывание Azure Resource Manager

Использование Azure Resource Manager позволяет полностью автоматизировать развертывание задания Stream Analytics. Шаблоны диспетчер ресурсов можно развернуть с помощью Azure PowerShell или [Azure CLI](/cli/azure/). В приведенных ниже примерах используется Azure CLI.


1. Вы можете создать ресурс **Microsoft. StreamAnalytics/стреамингжобс** с управляемым удостоверением, включив следующее свойство в раздел ресурсов шаблона диспетчер ресурсов:

    ```json
    "Identity": {
      "Type": "SystemAssigned",
    },
    ```

   Это свойство указывает Azure Resource Manager создать удостоверение для задания Stream Analytics и управлять им. Ниже приведен пример диспетчер ресурсов шаблона, который развертывает задание Stream Analytics с включенным управляемым удостоверением и приемником выходных данных большого двоичного объекта, который использует управляемое удостоверение:

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
            {
                "apiVersion": "2017-04-01-preview",
                "name": "MyStreamingJob",
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
                                "serialization": {
                                    "type": "JSON",
                                    "properties": {
                                        "encoding": "UTF8"
                                    }
                                },
                                "datasource":{
                                    "type":"Microsoft.Storage/Blob",
                                    "properties":{
                                        "storageAccounts": [
                                            { "accountName": "MyStorageAccount" }
                                        ],
                                        "container": "test",
                                        "pathPattern": "segment1/{date}/segment2/{time}",
                                        "dateFormat": "yyyy/MM/dd",
                                        "timeFormat": "HH",
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

    Приведенное выше задание можно развернуть в группе ресурсов **ексамплеграуп** с помощью следующей Azure CLI команды:

    ```azurecli
    az deployment group create --resource-group ExampleGroup -template-file StreamingJob.json
    ```

2. После создания задания можно использовать Azure Resource Manager для получения полного определения задания.

    ```azurecli
    az resource show --ids /subscriptions/{SUBSCRIPTION_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.StreamAnalytics/StreamingJobs/{RESOURCE_NAME}
    ```

    Приведенная выше команда возвратит ответ, подобный приведенному ниже.

    ```json
    {
        "id": "/subscriptions/{SUBSCRIPTION_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.StreamAnalytics/streamingjobs/{RESOURCE_NAME}",
        "identity": {
            "principalId": "{PRINCIPAL_ID}",
            "tenantId": "{TENANT_ID}",
            "type": "SystemAssigned",
            "userAssignedIdentities": null
        },
        "kind": null,
        "location": "West US",
        "managedBy": null,
        "name": "{RESOURCE_NAME}",
        "plan": null,
        "properties": {
            "compatibilityLevel": "1.0",
            "createdDate": "2019-07-12T03:11:30.39Z",
            "dataLocale": "en-US",
            "eventsLateArrivalMaxDelayInSeconds": 5,
            "jobId": "{JOB_ID}",
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
        "resourceGroup": "{RESOURCE_GROUP}",
        "sku": null,
        "tags": null,
        "type": "Microsoft.StreamAnalytics/streamingjobs"
    }
    ```

   Обратите внимание на **principalId** из определения задания, которое определяет управляемое удостоверение задания в Azure Active Directory и будет использоваться на следующем шаге, чтобы предоставить Stream Analyticsному заданию доступ к учетной записи хранения.

3. Теперь, когда задание создано, см. раздел [предоставление Stream Analyticsного задания доступ к учетной записи хранения](#give-the-stream-analytics-job-access-to-your-storage-account) этой статьи.


## <a name="give-the-stream-analytics-job-access-to-your-storage-account"></a>Предоставление Stream Analytics заданий доступа к учетной записи хранения

Существует два уровня доступа, которые можно использовать для задания Stream Analytics.

1. **Доступ на уровне контейнера** . Этот параметр предоставляет заданию доступ к конкретному существующему контейнеру.
2. **Доступ на уровне учетной записи.** этот параметр предоставляет заданию общий доступ к учетной записи хранения, включая возможность создания новых контейнеров.

Если вам не требуется задание для создания контейнеров от вашего имени, следует выбрать **доступ на уровне контейнера** , так как этот параметр предоставит заданию минимальный уровень доступа. Оба варианта описаны ниже для портал Azure и командной строки.

### <a name="grant-access-via-the-azure-portal"></a>Предоставление доступа через портал Azure

#### <a name="container-level-access"></a>Доступ на уровне контейнера

1. Перейдите в область конфигурации контейнера в своей учетной записи хранения.

2. Выберите **Управление доступом (IAM)** в левой части.

3. В разделе "Добавление назначения ролей" нажмите кнопку **Добавить**.

4. В области назначения ролей выполните следующие действия.

    1. Задать **роль** "участник данных BLOB-объектов хранилища"
    2. Убедитесь, что для параметра **назначить доступ в** раскрывающийся список выбрано значение "пользователь Azure AD, группа или субъект-служба".
    3. В поле поиска введите имя задания Stream Analytics.
    4. Выберите Stream Analytics задание и нажмите кнопку **сохранить**.

   ![Предоставление доступа к контейнеру](./media/stream-analytics-managed-identities-blob-output-preview/stream-analytics-container-access-portal.png)

#### <a name="account-level-access"></a>Доступ на уровне учетной записи

1. Войдите в свою учетную запись хранения.

2. Выберите **Управление доступом (IAM)** в левой части.

3. В разделе "Добавление назначения ролей" нажмите кнопку **Добавить**.

4. В области назначения ролей выполните следующие действия.

    1. Задать **роль** "участник данных BLOB-объектов хранилища"
    2. Убедитесь, что для параметра **назначить доступ в** раскрывающийся список выбрано значение "пользователь Azure AD, группа или субъект-служба".
    3. В поле поиска введите имя задания Stream Analytics.
    4. Выберите Stream Analytics задание и нажмите кнопку **сохранить**.

   ![Предоставление доступа к учетной записи](./media/stream-analytics-managed-identities-blob-output-preview/stream-analytics-account-access-portal.png)

### <a name="grant-access-via-the-command-line"></a>Предоставление доступа через командную строку

#### <a name="container-level-access"></a>Доступ на уровне контейнера

Чтобы предоставить доступ к определенному контейнеру, выполните следующую команду, используя Azure CLI:

   ```azurecli
   az role assignment create --role "Storage Blob Data Contributor" --assignee <principal-id> --scope /subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container-name>
   ```

#### <a name="account-level-access"></a>Доступ на уровне учетной записи

Чтобы предоставить доступ ко всей учетной записи, выполните следующую команду, используя Azure CLI:

   ```azurecli
   az role assignment create --role "Storage Blob Data Contributor" --assignee <principal-id> --scope /subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
   ```

## <a name="enable-vnet-access"></a>Включить доступ к виртуальной сети

При настройке **брандмауэров и виртуальных сетей** учетной записи хранения можно дополнительно разрешить в сетевом трафике от других доверенных служб Майкрософт. Когда Stream Analytics проходит проверку подлинности с помощью управляемого удостоверения, она предоставляет доказательства о том, что запрос исходит от доверенной службы. Ниже приведены инструкции по включению этого исключения доступа к виртуальной сети.

1.    Перейдите в область "брандмауэры и виртуальные сети" в области конфигурации учетной записи хранения.
2.    Убедитесь, что включен параметр "разрешить доступ к доверенным службам Майкрософт этой учетной записи хранения".
3.    Если вы включили его, нажмите кнопку **сохранить**.

   ![Включить доступ к виртуальной сети](./media/stream-analytics-managed-identities-blob-output-preview/stream-analytics-vnet-exception.png)

## <a name="remove-managed-identity"></a>Удалить управляемое удостоверение

Управляемое удостоверение, созданное для задания Stream Analytics, удаляется только при удалении задания. Невозможно удалить управляемое удостоверение, не удаляя задание. Если вы больше не хотите использовать управляемое удостоверение, можно изменить метод проверки подлинности для выходных данных. Управляемое удостоверение будет продолжать существовать до тех пор, пока задание не будет удалено, и будет использоваться при повторном использовании проверки подлинности с управляемым удостоверением.

## <a name="limitations"></a>Ограничения
Ниже приведены текущие ограничения этой функции.

1. Классические учетные записи хранения Azure.

2. Учетные записи Azure без Azure Active Directory.

3. Многоклиентский доступ не поддерживается. Субъект-служба, созданная для данного задания Stream Analytics, должна находиться в том же Azure Active Directoryном клиенте, в котором было создано задание, и не может использоваться с ресурсом, который находится в другом клиенте Azure Active Directory.

4. [Пользователь, которому назначено удостоверение](../active-directory/managed-identities-azure-resources/overview.md) , не поддерживается. Это означает, что пользователь не может ввести собственный субъект-службу, который будет использоваться их Stream Analyticsным заданием. Субъект-служба необходимо создавать с помощью Azure Stream Analytics.

## <a name="next-steps"></a>Дальнейшие действия

* [Описание выходных данных из Azure Stream Analytics](./stream-analytics-define-outputs.md)
* [Секционирование выходных данных пользовательского большого двоичного объекта Azure Stream Analytics](./stream-analytics-custom-path-patterns-blob-storage-output.md)
