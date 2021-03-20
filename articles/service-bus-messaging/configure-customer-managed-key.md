---
title: Настройка собственного ключа для шифрования неактивных данных служебной шины Azure
description: В этой статье содержатся сведения о том, как настроить собственный ключ для шифрования оставшейся работы данных служебной шины Azure.
ms.topic: conceptual
ms.date: 02/10/2021
ms.openlocfilehash: 5d14c8953819575d1c2688520838135efc7121e5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100378321"
---
# <a name="configure-customer-managed-keys-for-encrypting-azure-service-bus-data-at-rest-by-using-the-azure-portal"></a>Настройка управляемых клиентом ключей для шифрования неактивных данных служебной шины Azure с помощью портал Azure
Служебная шина Azure Premium обеспечивает шифрование неактивных данных с помощью Azure Шифрование службы хранилища (Azure SSE). Служебная шина уровня "Премиум" использует хранилище Azure для хранения данных. Все данные, хранящиеся в службе хранилища Azure, шифруются с помощью ключей, управляемых корпорацией Майкрософт. Если вы используете собственный ключ (также называемый создание собственных ключей (BYOK) или ключом, управляемым клиентом), данные по-прежнему шифруются с помощью ключа, управляемого корпорацией Майкрософт. Кроме того, ключ, управляемый Майкрософт, будет зашифрован с помощью ключа, управляемого клиентом. Эта функция позволяет создавать, поворачивать, отключать и отменять доступ к управляемым клиентом ключам, которые используются для шифрования ключей, управляемых корпорацией Майкрософт. Включение функции BYOK — это процесс установки в пространстве имен.

Существует ряд предостережений, которыми управляет ключ клиента для шифрования на стороне службы. 
- Эта функция поддерживается уровнем [Premium служебной шины Azure](service-bus-premium-messaging.md) . Его нельзя включить для пространств имен служебной шины уровня "Стандартный".
- Шифрование можно включить только для новых или пустых пространств имен. Если пространство имен содержит любые очереди или разделы, операция шифрования завершится ошибкой.

Вы можете использовать Azure Key Vault для управления ключами и аудита использования ключа. Можно либо создать собственные ключи и хранить их в хранилище ключей, либо использовать для их генерации API-интерфейсы Azure Key Vault. Дополнительные сведения о хранилище ключей Azure см. в статье [Что такое хранилище ключей Azure?](../key-vault/general/overview.md)

В этой статье показано, как настроить хранилище ключей с ключами, управляемыми клиентом, с помощью портал Azure. Сведения о создании хранилища ключей с помощью портал Azure см. в разделе [Краткое руководство. создание Azure Key Vault с помощью портал Azure](../key-vault/general/quick-create-portal.md).

> [!IMPORTANT]
> Для использования управляемых клиентом ключей с помощью служебной шины Azure необходимо, чтобы в хранилище ключей было настроено два обязательных свойства. Это:  **обратимое удаление** и **не очищается**. Эти свойства включены по умолчанию при создании нового хранилища ключей в портал Azure. Однако если необходимо включить эти свойства в существующем хранилище ключей, необходимо использовать PowerShell или Azure CLI.

## <a name="enable-customer-managed-keys"></a>Активация управляемых клиентом ключей
Чтобы включить управляемый клиентом ключ на портале Azure, выполните следующие действия.

1. Перейдите к пространству имен уровня "Премиум" служебной шины.
2. На странице **Параметры** пространства имен служебной шины выберите **Шифрование**.
3. Выберите **управляемое клиентом шифрование ключа неактивных** , как показано на следующем рисунке.

    ![Включение управляемого клиентом ключа](./media/configure-customer-managed-key/enable-customer-managed-key.png)


## <a name="set-up-a-key-vault-with-keys"></a>Настройка хранилища ключей с ключами

После включения ключей, управляемых клиентом, необходимо связать ключ, управляемый клиентом, с пространством имен служебной шины Azure. Служебная шина поддерживает только Azure Key Vault. Если включить **Шифрование с помощью ключа, управляемого клиентом** , в предыдущем разделе, необходимо импортировать ключ в Azure Key Vault. Кроме того, ключи должны иметь **обратимое удаление** и **не следует очищать** ключ. Эти параметры можно настроить с помощью [PowerShell](../key-vault/general/key-vault-recovery.md) или [CLI](../key-vault/general/key-vault-recovery.md).

1. Чтобы создать новое хранилище ключей, следуйте инструкциям в [кратком руководстве](../key-vault/general/overview.md)по Azure Key Vault. Дополнительные сведения об импорте существующих ключей см. в разделе [о ключах, секретах и сертификатах](../key-vault/general/about-keys-secrets-certificates.md).
1. Чтобы включить защиту с обратимым удалением и очисткой при создании хранилища, используйте команду [AZ keyvault Create](/cli/azure/keyvault#az-keyvault-create) .

    ```azurecli-interactive
    az keyvault create --name contoso-SB-BYOK-keyvault --resource-group ContosoRG --location westus --enable-soft-delete true --enable-purge-protection true
    ```    
1. Чтобы добавить защиту от очистки в существующем хранилище (для которого уже включено обратимое удаление), используйте команду [AZ keyvault Update](/cli/azure/keyvault#az-keyvault-update) .

    ```azurecli-interactive
    az keyvault update --name contoso-SB-BYOK-keyvault --resource-group ContosoRG --enable-purge-protection true
    ```
1. Создайте ключи, выполнив следующие действия.
    1. Чтобы создать новый ключ, выберите **Создать или импортировать** из меню **Ключи** в разделе **Параметры**.
        
        ![Кнопка "создать/импортировать"](./media/configure-customer-managed-key/select-generate-import.png)

    1. Задайте **Параметры**, чтобы **Создать** и присвоить имя для ключа.

        ![Создание ключа](./media/configure-customer-managed-key/create-key.png) 

    1. Теперь можно выбрать этот ключ, чтобы связать с пространством имен служебной шины для шифрования из раскрывающегося списка. 

        ![Выбор ключа из хранилища ключей](./media/configure-customer-managed-key/select-key-from-key-vault.png)
        > [!NOTE]
        > Для обеспечения избыточности можно добавить до 3 ключей. В случае, если срок действия одного из ключей истек или недоступен, для шифрования будут использоваться другие ключи.
        
    1. Введите сведения о ключе и нажмите кнопку **выбрать**. Это позволит включить шифрование ключа, управляемого корпорацией Майкрософт, с помощью ключа (управляемого клиентом ключа). 


    > [!IMPORTANT]
    > Если вы хотите использовать управляемый клиентом ключ вместе с географическим аварийным восстановлением, ознакомьтесь с этим разделом. 
    >
    > Чтобы включить шифрование ключа, управляемого корпорацией Майкрософт, с помощью управляемого клиентом ключа, [Политика доступа](../key-vault/general/secure-your-key-vault.md) настраивается для управляемого удостоверения служебной шины на указанном KeyVault Azure. Это обеспечит контролируемый доступ к KeyVault Azure из пространства имен служебной шины Azure.
    >
    > Из-за этого:
    > 
    >   * Если [географическое аварийное восстановление](service-bus-geo-dr.md) уже включено для пространства имен служебной шины и вы хотите включить управляемый клиентом ключ, то 
    >     * Разорвать связывание
    >     * [Настройте политику доступа](../key-vault/general/assign-access-policy-portal.md) для управляемого удостоверения как для основного, так и для дополнительного пространств имен в хранилище ключей.
    >     * Настройте шифрование для основного пространства имен.
    >     * Повторно свяжите первичные и вторичные пространства имен.
    > 
    >   * Если вы хотите включить геоаварийное восстановление в пространстве имен служебной шины, где уже настроен управляемый ключ клиента, то-
    >     * [Настройте политику доступа](../key-vault/general/assign-access-policy-portal.md) для управляемого удостоверения для дополнительного пространства имен в хранилище ключей.
    >     * Свяжите первичные и вторичные пространства имен.


## <a name="rotate-your-encryption-keys"></a>Смена ключей шифрования

Вы можете повернуть ключ в хранилище ключей, используя механизм смены хранилища ключей Azure. Для автоматизации смены ключей можно также задать даты активации и истечения срока действия. Служба служебной шины обнаружит новые ключевые версии и начнет использовать их автоматически.

## <a name="revoke-access-to-keys"></a>Отозвать доступ к ключам

Отмена доступа к ключам шифрования не приводит к очистке данных из служебной шины. Однако доступ к данным из пространства имен служебной шины невозможен. Вы можете отозвать ключ шифрования с помощью политики доступа или путем удаления ключа. Узнайте больше о политиках доступа и защите хранилища ключей от [безопасного доступа к хранилищу ключей](../key-vault/general/secure-your-key-vault.md).

После отзыва ключа шифрования служба служебной шины в зашифрованном пространстве имен станет неработоспособной. Если доступ к ключу включен или восстановлен удаленный ключ, служба служебной шины выберет ключ, чтобы получить доступ к данным из зашифрованного пространства имен служебной шины.

## <a name="caching-of-keys"></a>Кэширование ключей
Экземпляр служебной шины опрашивает список ключей шифрования каждые 5 минут. Он кэширует и использует их до следующего опроса, который находится в течение 5 минут. Если доступен по крайней мере один ключ, очереди и разделы доступны. Если все перечисленные ключи недоступны во время опроса, все очереди и разделы станут недоступными. 

Ниже приведены дополнительные сведения. 

- Каждые 5 минут служба служебной шины опрашивает все управляемые клиентом ключи, перечисленные в записи пространства имен:
    - Если ключ был повернут, запись обновляется новым ключом.
    - Если ключ был отозван, ключ удаляется из записи.
    - Если все ключи были отозваны, состояние шифрования для пространства имен будет равно " **отозвано**". Доступ к данным из пространства имен служебной шины невозможен. 
    

## <a name="use-resource-manager-template-to-enable-encryption"></a>Использование шаблона диспетчер ресурсов для включения шифрования
В этом разделе показано, как выполнять следующие задачи с помощью **шаблонов Azure Resource Manager**. 

1. Создайте пространство имен служебной шины **Premium** с помощью **управляемого удостоверения службы**.
2. Создайте **хранилище ключей** и предоставьте удостоверению службы доступ к хранилищу ключей. 
3. Обновите пространство имен служебной шины, указав сведения о хранилище ключей (ключ/значение). 


### <a name="create-a-premium-service-bus-namespace-with-managed-service-identity"></a>Создание пространства имен служебной шины Premium с удостоверением управляемой службы
В этом разделе показано, как создать пространство имен служебной шины Azure с удостоверением управляемой службы с помощью шаблона Azure Resource Manager и PowerShell. 

1. Создайте шаблон Azure Resource Manager, чтобы создать пространство имен уровня Premium служебной шины с управляемым удостоверением службы. Присвойте файлу имя: **CreateServiceBusPremiumNamespace.js**: 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace."
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
             "type":"Microsoft.ServiceBus/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Premium",
                "tier":"Premium",
                "capacity":1
             },
             "properties":{
    
             }
          }
       ],
       "outputs":{
          "ServiceBusNamespaceId":{
             "type":"string",
             "value":"[resourceId('Microsoft.ServiceBus/namespaces',parameters('namespaceName'))]"
          }
       }
    }
    ```
2. Создайте файл параметров шаблона с именем: **CreateServiceBusPremiumNamespaceParams.jsв**. 

    > [!NOTE]
    > Измените следующие значения: 
    > - `<ServiceBusNamespaceName>` — Имя пространства имен служебной шины.
    > - `<Location>` — Расположение пространства имен служебной шины

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "value":"<ServiceBusNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          }
       }
    }
    ```
3. Выполните следующую команду PowerShell, чтобы развернуть шаблон для создания пространства имен служебной шины Premium. Затем получите идентификатор пространства имен служебной шины, чтобы использовать его позже. Замените `{MyRG}` именем группы ресурсов перед выполнением команды.  

    ```powershell
    $outputs = New-AzResourceGroupDeployment -Name CreateServiceBusPremiumNamespace -ResourceGroupName {MyRG} -TemplateFile ./CreateServiceBusPremiumNamespace.json -TemplateParameterFile ./CreateServiceBusPremiumNamespaceParams.json
    
    $ServiceBusNamespaceId = $outputs.Outputs["serviceBusNamespaceId"].value
    ```
 
### <a name="grant-service-bus-namespace-identity-access-to-key-vault"></a>Предоставление удостоверению пространства имен служебной шины доступ к хранилищу ключей

1. Выполните следующую команду, чтобы создать хранилище ключей с включенной **защитой очистки** и **обратимого удаления** . 

    ```powershell
    New-AzureRmKeyVault -Name "{keyVaultName}" -ResourceGroupName {RGName}  -Location "{location}" -EnableSoftDelete -EnablePurgeProtection    
    ```
    
    (ИЛИ)
    
    Выполните следующую команду, чтобы обновить **существующее хранилище ключей**. Перед выполнением команды укажите значения для имен группы ресурсов и хранилища ключей. 
    
    ```powershell
    ($updatedKeyVault = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -ResourceGroupName {RGName} -VaultName {keyVaultName}).ResourceId).Properties| Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"-Force | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true" -Force
    ``` 
2. Настройте политику доступа к хранилищу ключей таким образом, чтобы управляемое удостоверение пространства имен служебной шины может получить доступ к значению ключа в хранилище ключей. Используйте идентификатор пространства имен служебной шины из предыдущего раздела. 

    ```powershell
    $identity = (Get-AzureRmResource -ResourceId $ServiceBusNamespaceId -ExpandProperties).Identity
    
    Set-AzureRmKeyVaultAccessPolicy -VaultName {keyVaultName} -ResourceGroupName {RGName} -ObjectId $identity.PrincipalId -PermissionsToKeys get,wrapKey,unwrapKey,list
    ```

### <a name="encrypt-data-in-service-bus-namespace-with-customer-managed-key-from-key-vault"></a>Шифрование данных в пространстве имен служебной шины с помощью ключа, управляемого клиентом, из хранилища ключей
На данный момент вы выполнили следующие действия: 

1. Создано пространство имен Premium с управляемым удостоверением.
2. Создание хранилища ключей и предоставление управляемому удостоверению доступа к хранилищу ключей. 

На этом шаге вы обновите пространство имен служебной шины с помощью сведений о хранилище ключей. 

1. Создайте файл JSON с именем **UpdateServiceBusNamespaceWithEncryption.jsдля** со следующим содержимым: 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
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
             "type":"Microsoft.ServiceBus/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Premium",
                "tier":"Premium",
                "capacity":1
             },
             "properties":{
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

2. Создайте файл параметров шаблона: **UpdateServiceBusNamespaceWithEncryptionParams.jsв**.

    > [!NOTE]
    > Измените следующие значения: 
    > - `<ServiceBusNamespaceName>` — Имя пространства имен служебной шины.
    > - `<Location>` — Расположение пространства имен служебной шины
    > - `<KeyVaultName>` — Имя хранилища ключей.
    > - `<KeyName>` — Имя ключа в хранилище ключей.  

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "value":"<ServiceBusNamespaceName>"
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
    New-AzResourceGroupDeployment -Name UpdateServiceBusNamespaceWithEncryption -ResourceGroupName {MyRG} -TemplateFile ./UpdateServiceBusNamespaceWithEncryption.json -TemplateParameterFile ./UpdateServiceBusNamespaceWithEncryptionParams.json
    ```
    

## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи:
- [Обзор служебной шины](service-bus-messaging-overview.md)
- [Обзор Key Vault](../key-vault/general/overview.md)