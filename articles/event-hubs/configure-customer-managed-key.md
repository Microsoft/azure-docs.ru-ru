---
title: Настройка собственного ключа для шифрования неактивных данных концентраторов событий Azure
description: В этой статье содержатся сведения о том, как настроить собственный ключ для шифрования оставшейся работы данных концентраторов событий Azure.
ms.topic: conceptual
ms.date: 02/01/2021
ms.openlocfilehash: c9d1ac1c3a3387600fed80939598baafe658054b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "100595986"
---
# <a name="configure-customer-managed-keys-for-encrypting-azure-event-hubs-data-at-rest-by-using-the-azure-portal"></a>Настройка управляемых клиентом ключей для шифрования неактивных данных концентраторов событий Azure с помощью портал Azure
Концентраторы событий Azure обеспечивают шифрование неактивных данных с помощью Azure Шифрование службы хранилища (Azure SSE). Служба концентраторов событий использует службу хранилища Azure для хранения данных. Все данные, хранящиеся в службе хранилища Azure, шифруются с помощью ключей, управляемых корпорацией Майкрософт. Если вы используете собственный ключ (также называемый создание собственных ключей (BYOK) или ключом, управляемым клиентом), данные по-прежнему шифруются с помощью ключа, управляемого корпорацией Майкрософт. Кроме того, ключ, управляемый Майкрософт, будет зашифрован с помощью ключа, управляемого клиентом. Эта функция позволяет создавать, поворачивать, отключать и отменять доступ к управляемым клиентом ключам, которые используются для шифрования ключей, управляемых корпорацией Майкрософт. Включение функции BYOK — это процесс установки в пространстве имен.

> [!NOTE]
> - Возможность BYOK поддерживается [выделенными в концентраторами событий](event-hubs-dedicated-overview.md) кластерами с одним клиентом. Его нельзя включить для стандартных пространств имен концентраторов событий.
> - Шифрование может быть включено только для новых или пустых пространств имен. Если пространство имен содержит концентраторы событий, операция шифрования завершится ошибкой.

Вы можете использовать Azure Key Vault для управления ключами и аудита использования ключа. Можно либо создать собственные ключи и хранить их в хранилище ключей, либо использовать для их генерации API-интерфейсы Azure Key Vault. Дополнительные сведения о хранилище ключей Azure см. в статье [Что такое хранилище ключей Azure?](../key-vault/general/overview.md)

В этой статье показано, как настроить хранилище ключей с ключами, управляемыми клиентом, с помощью портал Azure. Сведения о создании хранилища ключей с помощью портал Azure см. в разделе [Краткое руководство. создание Azure Key Vault с помощью портал Azure](../key-vault/general/quick-create-portal.md).

> [!IMPORTANT]
> Для использования ключей, управляемых клиентом, с концентраторами событий Azure необходимо, чтобы в хранилище ключей было настроено два обязательных свойства. Это:  **обратимое удаление** и **не очищается**. Эти свойства включены по умолчанию при создании нового хранилища ключей в портал Azure. Однако если необходимо включить эти свойства в существующем хранилище ключей, необходимо использовать PowerShell или Azure CLI.

## <a name="enable-customer-managed-keys"></a>Активация управляемых клиентом ключей
Чтобы включить управляемый клиентом ключ на портале Azure, выполните следующие действия.

1. Перейдите к кластеру Центры событий (цен. категория "Выделенный").
1. Выберите пространство имен, в котором необходимо включить BYOK.
1. На странице **Параметры** пространства имен концентраторов событий выберите **Шифрование**. 
1. Выберите **управляемое клиентом шифрование ключа неактивных** , как показано на следующем рисунке. 

    ![Включение управляемого клиентом ключа](./media/configure-customer-managed-key/enable-customer-managed-key.png)

## <a name="set-up-a-key-vault-with-keys"></a>Настройка хранилища ключей с ключами
После включения ключей, управляемых клиентом, необходимо связать ключ, управляемый клиентом, с пространством имен концентраторов событий Azure. Концентраторы событий поддерживают только Azure Key Vault. Если включить **Шифрование с помощью ключа, управляемого клиентом** , в предыдущем разделе, необходимо импортировать ключ в Azure Key Vault. Кроме того, ключи должны иметь **обратимое удаление** и **не следует очищать** ключ. Эти параметры можно настроить с помощью [PowerShell](../key-vault/general/key-vault-recovery.md) или [CLI](../key-vault/general/key-vault-recovery.md).

1. Чтобы создать новое хранилище ключей, следуйте инструкциям в [кратком руководстве](../key-vault/general/overview.md)по Azure Key Vault. Дополнительные сведения об импорте существующих ключей см. в разделе [о ключах, секретах и сертификатах](../key-vault/general/about-keys-secrets-certificates.md).
1. Чтобы включить защиту с обратимым удалением и очисткой при создании хранилища, используйте команду [AZ keyvault Create](/cli/azure/keyvault#az-keyvault-create) .

    ```azurecli-interactive
    az keyvault create --name ContosoVault --resource-group ContosoRG --location westus --enable-soft-delete true --enable-purge-protection true
    ```    
1. Чтобы добавить защиту от очистки в существующем хранилище (для которого уже включено обратимое удаление), используйте команду [AZ keyvault Update](/cli/azure/keyvault#az-keyvault-update) .

    ```azurecli-interactive
    az keyvault update --name ContosoVault --resource-group ContosoRG --enable-purge-protection true
    ```
1. Создайте ключи, выполнив следующие действия.
    1. Чтобы создать новый ключ, выберите **Создать или импортировать** из меню **Ключи** в разделе **Параметры**.
        
        ![Кнопка "создать/импортировать"](./media/configure-customer-managed-key/select-generate-import.png)
    1. Задайте **Параметры**, чтобы **Создать** и присвоить имя для ключа.

        ![Создание ключа](./media/configure-customer-managed-key/create-key.png) 
    1. Теперь можно выбрать этот ключ, чтобы связать с пространством имен концентраторов событий для шифрования из раскрывающегося списка. 

        ![Выбор ключа из хранилища ключей](./media/configure-customer-managed-key/select-key-from-key-vault.png)
    1. Введите сведения о ключе и нажмите кнопку **выбрать**. Это позволит включить шифрование ключа, управляемого корпорацией Майкрософт, с помощью ключа (управляемого клиентом ключа). 


## <a name="rotate-your-encryption-keys"></a>Смена ключей шифрования
Вы можете повернуть ключ в хранилище ключей, используя механизм смены хранилища ключей Azure. Для автоматизации смены ключей можно также задать даты активации и истечения срока действия. Служба концентраторов событий обнаружит новые версии ключей и начнет использовать их автоматически.

## <a name="revoke-access-to-keys"></a>Отозвать доступ к ключам
Отмена доступа к ключам шифрования не приводит к очистке данных из концентраторов событий. Однако доступ к данным из пространства имен концентраторов событий невозможен. Вы можете отозвать ключ шифрования с помощью политики доступа или путем удаления ключа. Узнайте больше о политиках доступа и защите хранилища ключей от [безопасного доступа к хранилищу ключей](../key-vault/general/secure-your-key-vault.md).

После отзыва ключа шифрования служба концентраторов событий в зашифрованном пространстве имен станет неработоспособной. Если доступ к ключу включен или восстановлен ключ удаления, служба концентраторов событий выберет ключ, чтобы получить доступ к данным из зашифрованного пространства имен концентраторов событий.

## <a name="set-up-diagnostic-logs"></a>Настройка журналов диагностики 
Настройка журналов диагностики для пространств имен с включенным BYOK предоставляет необходимые сведения об операциях. Эти журналы можно включить и позже передать в концентратор событий или проанализировать с помощью log Analytics или передать в хранилище для выполнения настраиваемой аналитики. Дополнительные сведения о журналах диагностики см. в статье [Обзор журналов диагностики Azure](../azure-monitor/essentials/platform-logs-overview.md).

## <a name="enable-user-logs"></a>Включить журналы пользователей
Выполните следующие действия, чтобы включить журналы для ключей, управляемых клиентом.

1. В портал Azure перейдите к пространству имен с включенным BYOK.
1. Выберите **параметры диагностики** в разделе **мониторинг**.

    ![Выбор параметров диагностики](./media/configure-customer-managed-key/select-diagnostic-settings.png)
1. Выберите **+ Добавить параметр диагностики**. 

    ![Выберите Добавить параметр диагностики.](./media/configure-customer-managed-key/select-add-diagnostic-setting.png)
1. Укажите **имя** и выберите, куда следует выполнять потоковую передачу журналов.
1. Выберите **кустомерманажедкэйусерлогс** и **Сохраните**. Это действие активирует журналы для BYOK в пространстве имен.

    ![Выбор параметра "пользовательские журналы управляемого ключа для клиентов"](./media/configure-customer-managed-key/select-customer-managed-key-user-logs.png)

## <a name="log-schema"></a>Схема журнала 
Все журналы хранятся в формате JSON (нотация объектов JavaScript). Каждая запись содержит строковые поля, в которых используется формат, описанный в следующей таблице. 

| Имя | Описание |
| ---- | ----------- | 
| TaskName | Описание задачи, завершившейся сбоем. |
| ActivityId | Внутренний идентификатор, используемый для отслеживания. |
| категория | Определяет классификацию задачи. Например, если ключ из хранилища ключей отключен, то это будет категория сведений или если ключ не может быть распакован, он может попадать в ошибку. |
| resourceId | Идентификатор ресурса Azure Resource Manager |
| keyVault | Полное имя хранилища ключей. |
| ключ | Имя ключа, используемое для шифрования пространства имен концентраторов событий. |
| version | Версия используемого ключа. |
| Операция | Операция, выполняемая над ключом в хранилище ключей. Например, отключение/включение ключа, перенос или распаковка |
| code | Код, связанный с операцией. Пример: код ошибки, 404 означает, что ключ не найден. |
| message | Любое сообщение об ошибке, связанное с операцией |

Ниже приведен пример журнала для ключа, управляемого клиентом:

```json
{
   "TaskName": "CustomerManagedKeyUserLog",
   "ActivityId": "11111111-1111-1111-1111-111111111111",
   "category": "error"
   "resourceId": "/SUBSCRIPTIONS/11111111-1111-1111-1111-11111111111/RESOURCEGROUPS/DEFAULT-EVENTHUB-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/FBETTATI-OPERA-EVENTHUB",
   "keyVault": "https://mykeyvault.vault-int.azure-int.net",
   "key": "mykey",
   "version": "1111111111111111111111111111111",
   "operation": "wrapKey",
   "code": "404",
   "message": "Key not found: ehbyok0/111111111111111111111111111111",
}



{
   "TaskName": "CustomerManagedKeyUserLog",
   "ActivityId": "11111111111111-1111-1111-1111111111111",
   "category": "info"
   "resourceId": "/SUBSCRIPTIONS/111111111-1111-1111-1111-11111111111/RESOURCEGROUPS/DEFAULT-EVENTHUB-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/FBETTATI-OPERA-EVENTHUB",
   "keyVault": "https://mykeyvault.vault-int.azure-int.net",
   "key": "mykey",
   "version": "111111111111111111111111111111",
   "operation": "disable" | "restore",
   "code": "",
   "message": "",
}
```

## <a name="use-resource-manager-template-to-enable-encryption"></a>Использование шаблона диспетчер ресурсов для включения шифрования
В этом разделе показано, как выполнять следующие задачи с помощью **шаблонов Azure Resource Manager**. 

1. Создайте **пространство имен концентраторов событий** с управляемым удостоверением службы.
2. Создайте **хранилище ключей** и предоставьте удостоверению службы доступ к хранилищу ключей. 
3. Обновите пространство имен концентраторов событий, указав сведения о хранилище ключей (ключ/значение). 


### <a name="create-an-event-hubs-cluster-and-namespace-with-managed-service-identity"></a>Создание кластера концентраторов событий и пространства имен с управляемым удостоверением службы
В этом разделе показано, как создать пространство имен концентраторов событий Azure с управляемым удостоверением службы с помощью шаблона Azure Resource Manager и PowerShell. 

1. Создайте шаблон Azure Resource Manager, чтобы создать пространство имен концентраторов событий с управляемым удостоверением службы. Присвойте файлу имя: **CreateEventHubClusterAndNamespace.js**: 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Event Hub cluster."
             }
          },
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace to be created in cluster."
             }
          },
          "location":{
             "type":"string",
             "defaultValue":"[resourceGroup().location]",
             "metadata":{
                "description":"Specifies the Azure location for all resources."
             }
          }
       },
       "resources":[
          {
             "type":"Microsoft.EventHub/clusters",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('clusterName')]",
             "location":"[parameters('location')]",
             "sku":{
                "name":"Dedicated",
                "capacity":1
             }
          },
          {
             "type":"Microsoft.EventHub/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Standard",
                "tier":"Standard",
                "capacity":1
             },
             "properties":{
                "isAutoInflateEnabled":false,
                "maximumThroughputUnits":0,
                "clusterArmId":"[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]"
             },
             "dependsOn":[
                "[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]"
             ]
          }
       ],
       "outputs":{
          "EventHubNamespaceId":{
             "type":"string",
             "value":"[resourceId('Microsoft.EventHub/namespaces',parameters('namespaceName'))]"
          }
       }
    }
    ```
2. Создайте файл параметров шаблона с именем: **CreateEventHubClusterAndNamespaceParams.jsв**. 

    > [!NOTE]
    > Измените следующие значения: 
    > - `<EventHubsClusterName>` — Имя кластера концентраторов событий;    
    > - `<EventHubsNamespaceName>` — Имя пространства имен концентраторов событий.
    > - `<Location>` — Расположение пространства имен концентраторов событий

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "value":"<EventHubsClusterName>"
          },
          "namespaceName":{
             "value":"<EventHubsNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          }
       }
    }
    
    ```
3. Выполните следующую команду PowerShell, чтобы развернуть шаблон для создания пространства имен концентраторов событий. Затем получите идентификатор пространства имен концентраторов событий, чтобы использовать его позже. Замените `{MyRG}` именем группы ресурсов перед выполнением команды.  

    ```powershell
    $outputs = New-AzResourceGroupDeployment -Name CreateEventHubClusterAndNamespace -ResourceGroupName {MyRG} -TemplateFile ./CreateEventHubClusterAndNamespace.json -TemplateParameterFile ./CreateEventHubClusterAndNamespaceParams.json

    $EventHubNamespaceId = $outputs.Outputs["eventHubNamespaceId"].value
    ```
 
### <a name="grant-event-hubs-namespace-identity-access-to-key-vault"></a>Предоставление удостоверению пространства имен концентраторов событий доступа к хранилищу ключей

1. Выполните следующую команду, чтобы создать хранилище ключей с включенной **защитой очистки** и **обратимого удаления** . 

    ```powershell
    New-AzureRmKeyVault -Name {keyVaultName} -ResourceGroupName {RGName}  -Location {location} -EnableSoftDelete -EnablePurgeProtection    
    ```     
    
    (ИЛИ)    
    
    Выполните следующую команду, чтобы обновить **существующее хранилище ключей**. Перед выполнением команды укажите значения для имен группы ресурсов и хранилища ключей. 
    
    ```powershell
    ($updatedKeyVault = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -ResourceGroupName {RGName} -VaultName {keyVaultName}).ResourceId).Properties| Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"-Force | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true" -Force
    ``` 
2. Задайте политику доступа к хранилищу ключей, чтобы управляемая идентификация пространства имен концентраторов событий могла получить доступ к значению ключа в хранилище ключей. Используйте идентификатор пространства имен концентраторов событий из предыдущего раздела. 

    ```powershell
    $identity = (Get-AzureRmResource -ResourceId $EventHubNamespaceId -ExpandProperties).Identity
    
    Set-AzureRmKeyVaultAccessPolicy -VaultName {keyVaultName} -ResourceGroupName {RGName} -ObjectId $identity.PrincipalId -PermissionsToKeys get,wrapKey,unwrapKey,list
    ```

### <a name="encrypt-data-in-event-hubs-namespace-with-customer-managed-key-from-key-vault"></a>Шифрование данных в пространстве имен концентраторов событий с помощью ключа, управляемого клиентом, из хранилища ключей
На данный момент вы выполнили следующие действия: 

1. Создано пространство имен Premium с управляемым удостоверением.
2. Создание хранилища ключей и предоставление управляемому удостоверению доступа к хранилищу ключей. 

На этом шаге вы обновите пространство имен концентраторов событий, используя сведения о хранилище ключей. 

1. Создайте файл JSON с именем **CreateEventHubClusterAndNamespace.jsдля** со следующим содержимым: 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Event Hub cluster."
             }
          },
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace to be created in cluster."
             }
          },
          "location":{
             "type":"string",
             "defaultValue":"[resourceGroup().location]",
             "metadata":{
                "description":"Specifies the Azure location for all resources."
             }
          },
          "keyVaultUri":{
             "type":"string",
             "metadata":{
                "description":"URI of the KeyVault."
             }
          },
          "keyName":{
             "type":"string",
             "metadata":{
                "description":"KeyName."
             }
          }
       },
       "resources":[
          {
             "type":"Microsoft.EventHub/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Standard",
                "tier":"Standard",
                "capacity":1
             },
             "properties":{
                "isAutoInflateEnabled":false,
                "maximumThroughputUnits":0,
                "clusterArmId":"[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]",
                "encryption":{
                   "keySource":"Microsoft.KeyVault",
                   "keyVaultProperties":[
                      {
                         "keyName":"[parameters('keyName')]",
                         "keyVaultUri":"[parameters('keyVaultUri')]"
                      }
                   ]
                }
             }
          }
       ]
    }
    ``` 

2. Создайте файл параметров шаблона: **UpdateEventHubClusterAndNamespaceParams.jsв**. 

    > [!NOTE]
    > Измените следующие значения: 
    > - `<EventHubsClusterName>` — Имя кластера концентраторов событий.        
    > - `<EventHubsNamespaceName>` — Имя пространства имен концентраторов событий.
    > - `<Location>` — Расположение пространства имен концентраторов событий
    > - `<KeyVaultName>` — Имя хранилища ключей.
    > - `<KeyName>` — Имя ключа в хранилище ключей.

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "value":"<EventHubsClusterName>"
          },
          "namespaceName":{
             "value":"<EventHubsNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          },
          "keyName":{
             "value":"<KeyName>"
          },
          "keyVaultUri":{
             "value":"https://<KeyVaultName>.vault.azure.net"
          }
       }
    }
    ```             
3. Выполните следующую команду PowerShell, чтобы развернуть шаблон диспетчер ресурсов. Замените `{MyRG}` именем группы ресурсов перед выполнением команды. 

    ```powershell
    New-AzResourceGroupDeployment -Name UpdateEventHubNamespaceWithEncryption -ResourceGroupName {MyRG} -TemplateFile ./UpdateEventHubClusterAndNamespace.json -TemplateParameterFile ./UpdateEventHubClusterAndNamespaceParams.json 
    ```

## <a name="troubleshoot"></a>Диагностика
Рекомендуется всегда включать журналы, как показано в предыдущем разделе. Он помогает отслеживать действия при включенном шифровании BYOK. Он также помогает в области видимости проблем.

Ниже приведены распространенные коды ошибок для поиска при включенном шифровании BYOK.

| Действие | Код ошибки | Итоговое состояние данных |
| ------ | ---------- | ----------------------- | 
| Удаление разрешения на перенос и распаковки из хранилища ключей | 403 |    Нет доступа |
| Удаление членства в роли AAD из субъекта AAD, которому предоставлено разрешение на отправку или перенос | 403 |  Нет доступа |
| Удаление ключа шифрования из хранилища ключей | 404 | Нет доступа |
| Удаление хранилища ключей | 404 | Недоступно (предполагается, что обратимое удаление является обязательным параметром.) |
| Изменение срока действия ключа шифрования так, чтобы его срок действия был уже истек | 403 |   Нет доступа  |
| Изменение NBF (не ранее), если ключ шифрования ключа не активен | 403 | Нет доступа  |
| Выбор параметра **Разрешить службы MSFT** для брандмауэра хранилища ключей или блокировка сетевого доступа к хранилищу ключей, имеющему ключ шифрования; | 403 | Нет доступа |
| Перемещение хранилища ключей в другой клиент | 404 | Нет доступа |  
| Неисправность сети или сбой DNS/AAD/MSI |  | Доступ с помощью кэшированного ключа шифрования данных |

> [!IMPORTANT]
> Чтобы включить геоаварийное восстановление в пространстве имен, использующем шифрование BYOK, вторичное пространство имен для связывания должно быть в выделенном кластере и на нем должно быть включено управляемое удостоверение, назначенное системой. Дополнительные сведения см. в статье [управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи:
- [Общие сведения о Центрах событий](event-hubs-about.md)
- [Обзор Key Vault](../key-vault/general/overview.md)