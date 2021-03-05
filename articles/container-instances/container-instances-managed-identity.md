---
title: Включение управляемого удостоверения в группе контейнеров
description: Узнайте, как включить управляемое удостоверение в службе "экземпляры контейнеров Azure", которое может проходить проверку подлинности в других службах Azure
ms.topic: article
ms.date: 07/02/2020
ms.openlocfilehash: a0d029e39122ca7bb858103f4d7f88e2536850d5
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102198325"
---
# <a name="how-to-use-managed-identities-with-azure-container-instances"></a>Использование управляемых удостоверений для службы "Экземпляры контейнеров Azure"

Используйте [управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md), чтобы выполнять в службе "Экземпляры контейнеров Azure" код, который взаимодействует с другими службами Azure, без обработки секретов или учетных данных в коде. Эта функция позволяет создать развертывание службы "Экземпляры контейнеров Azure" с автоматическим управлением удостоверениями через Azure Active Directory.

Из этой статьи вы узнаете, как использовать управляемые удостоверения в службе "Экземпляры контейнеров Azure" и выполнять следующие действия:

> [!div class="checklist"]
> * применение к группе контейнеров удостоверения, назначаемого пользователем или системой;
> * Предоставление удостоверениям доступа к хранилищу ключей Azure
> * Использование управляемого удостоверения для доступа к хранилищу ключей из работающего контейнера

Вы можете адаптировать эти примеры, чтобы применить собственные удостоверения в службе "Экземпляры контейнеров Azure" для доступа к другим службам Azure. Это интерактивные примеры. Но на практике для доступа к службам Azure в образах контейнеров будет выполняться код.
 
> [!IMPORTANT]
> Эта функция в настоящее время находится на стадии предварительной версии. Предварительные версии предоставляются при условии, что вы принимаете [дополнительные условия использования](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии. В настоящее время управляемые удостоверения в службе "экземпляры контейнеров Azure" поддерживаются только в контейнерах Linux, но не в контейнерах Windows.

## <a name="why-use-a-managed-identity"></a>Для чего нужны управляемые удостоверения?

Управляемое удостоверение в запущенном контейнере можно использовать для проверки подлинности в любой [службе, которая поддерживает проверку подлинности Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication), без необходимости управлять учетными данными в коде. Для служб, которые не поддерживают проверку подлинности Active Directory, вы можете хранить секреты в хранилище ключей Azure и использовать управляемое удостоверение для доступа к хранилищу ключей для получения учетных данных. Дополнительные сведения об использовании управляемых удостоверений см. в статье [Что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md)

### <a name="enable-a-managed-identity"></a>Включение управляемого удостоверения

 При создании группы контейнеров вы можете включить одно или несколько управляемых удостоверений, задав свойство [ContainerGroupIdentity](/rest/api/container-instances/containergroups/createorupdate#containergroupidentity). Вы также можете включить или обновить управляемые удостоверения после того, как группа контейнеров выполняет любое действие, приводит к перезапуску группы контейнеров. Чтобы задать удостоверения для новой или существующей группы контейнеров, используйте Azure CLI, шаблон диспетчер ресурсов, файл YAML или другое средство Azure. 

Служба "Экземпляры контейнеров Azure" поддерживает оба типа управляемых удостоверений Azure: назначаемые пользователем и назначаемые системой. Для группы контейнеров вы можете указать назначаемое системой удостоверение, одно или несколько назначаемых пользователей удостоверений или даже удостоверения обоих типов. Если вы не знакомы с управляемыми удостоверениями для ресурсов Azure, см. [Обзор](../active-directory/managed-identities-azure-resources/overview.md).

### <a name="use-a-managed-identity"></a>Использование управляемого удостоверения

Чтобы использовать управляемое удостоверение, удостоверению необходимо предоставить доступ к одному или нескольким ресурсам службы Azure (например, к веб-приложению, хранилищу ключей или учетной записи хранения) в подписке. Использование управляемого удостоверения в работающем контейнере аналогично использованию удостоверения на виртуальной машине Azure. Ознакомьтесь с документацией по виртуальным машинам, где описано использование [маркера](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md), [Azure PowerShell или Azure CLI](../active-directory/managed-identities-azure-resources/how-to-use-vm-sign-in.md) либо [пакетов SDK для Azure](../active-directory/managed-identities-azure-resources/how-to-use-vm-sdk.md).

### <a name="limitations"></a>Ограничения

* В настоящее время управляемое удостоверение нельзя использовать в группе контейнеров, развернутой в виртуальной сети.
* Вы не можете использовать управляемое удостоверение для извлечения образа из реестра контейнеров Azure при создании группы контейнеров. Удостоверение доступно только в работающем контейнере.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется версия 2.0.49 или более поздняя Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-an-azure-key-vault"></a>Создание хранилища ключей Azure

В примерах, приведенных в этой статье, для доступа к секрету хранилища ключей Azure используется управляемое удостоверение в службе "экземпляры контейнеров Azure". 

Для начала создайте группу ресурсов с именем *myResourceGroup* в регионе *eastus* с помощью следующей команды [az group create](/cli/azure/group#az-group-create):

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Чтобы создать хранилище ключей, используйте команду [AZ keyvault Create](/cli/azure/keyvault#az-keyvault-create) . Не забудьте указать уникальное имя хранилища ключей. 

```azurecli-interactive
az keyvault create \
  --name mykeyvault \
  --resource-group myResourceGroup \ 
  --location eastus
```

Сохраните пример секрета в хранилище ключей с помощью команды [AZ keyvault Secret Set](/cli/azure/keyvault/secret#az-keyvault-secret-set) :

```azurecli-interactive
az keyvault secret set \
  --name SampleSecret \
  --value "Hello Container Instances" \
  --description ACIsecret --vault-name mykeyvault
```

Перейдите к следующим примерам, чтобы получить доступ к хранилищу ключей с помощью назначенного пользователем или управляемого системой удостоверения в службе "экземпляры контейнеров Azure".

## <a name="example-1-use-a-user-assigned-identity-to-access-azure-key-vault"></a>Пример 1. Использование назначенного пользователем удостоверения для доступа к хранилищу ключей Azure

### <a name="create-an-identity"></a>Создание удостоверения

Первым делом создайте в подписке удостоверение, используя команду [az identity create](/cli/azure/identity#az-identity-create). Вы можете использовать ту же группу ресурсов, которая использовалась для создания хранилища ключей, или использовать другое.

```azurecli-interactive
az identity create \
  --resource-group myResourceGroup \
  --name myACIId
```

Чтобы применить это удостоверение на следующих шагах, выполните команду [az identity show](/cli/azure/identity#az-identity-show) для сохранения идентификатора субъекта-службы и идентификатора ресурса в переменных.

```azurecli-interactive
# Get service principal ID of the user-assigned identity
spID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACIId \
  --query principalId --output tsv)

# Get resource ID of the user-assigned identity
resourceID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACIId \
  --query id --output tsv)
```

### <a name="grant-user-assigned-identity-access-to-the-key-vault"></a>Предоставление пользователю доступа к хранилищу ключей

Выполните следующую команду [AZ keyvault Set-Policy](/cli/azure/keyvault) , чтобы задать политику доступа для хранилища ключей. В следующем примере можно назначить пользователю удостоверение для получения секретов из хранилища ключей:

```azurecli-interactive
 az keyvault set-policy \
    --name mykeyvault \
    --resource-group myResourceGroup \
    --object-id $spID \
    --secret-permissions get
```

### <a name="enable-user-assigned-identity-on-a-container-group"></a>Включение назначенного пользователем удостоверения в группе контейнеров

Выполните следующую команду [AZ Container Create](/cli/azure/container#az-container-create) , чтобы создать экземпляр контейнера на основе `azure-cli` образа Майкрософт. В этом примере представлена группа из одного контейнера, которую можно использовать в интерактивном режиме для запуска Azure CLI для доступа к другим службам Azure. В этом разделе используется только базовая операционная система. Пример использования Azure CLI в контейнере см. в разделе [Включение назначенного системой удостоверения для группы контейнеров](#enable-system-assigned-identity-on-a-container-group). 

Параметр `--assign-identity` передает группе назначаемое пользователем управляемое удостоверение. Длительно выполняющаяся команда поддерживает выполнение контейнера. В этом примере используется та же группа ресурсов, которая использовалась для создания хранилища ключей, но можно указать другой.

```azurecli-interactive
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azure-cli \
  --assign-identity $resourceID \
  --command-line "tail -f /dev/null"
```

Через несколько секунд вы должны получить ответ из интерфейса командной строки Azure, указывающий, что развертывание завершено. Проверьте его состояние с помощью команды [AZ Container показывать](/cli/azure/container#az-container-show) .

```azurecli-interactive
az container show \
  --resource-group myResourceGroup \
  --name mycontainer
```

Раздел `identity` в выходных данных выглядит примерно так, как показано ниже, и подтверждает, что удостоверение настроено в группе контейнеров. Параметр `principalID` под `userAssignedIdentities` обозначает субъект-службу удостоверения, которое вы создали в Azure Active Directory:

```console
[...]
"identity": {
    "principalId": "null",
    "tenantId": "xxxxxxxx-f292-4e60-9122-xxxxxxxxxxxx",
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "/subscriptions/xxxxxxxx-0903-4b79-a55a-xxxxxxxxxxxx/resourcegroups/danlep1018/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myACIId": {
        "clientId": "xxxxxxxx-5523-45fc-9f49-xxxxxxxxxxxx",
        "principalId": "xxxxxxxx-f25b-4895-b828-xxxxxxxxxxxx"
      }
    }
  },
[...]
```

### <a name="use-user-assigned-identity-to-get-secret-from-key-vault"></a>Использовать назначенное пользователем удостоверение для получения секрета из хранилища ключей

Теперь вы можете использовать управляемое удостоверение в работающем экземпляре контейнера для доступа к хранилищу ключей. Сначала запустите оболочку Bash в контейнере:

```azurecli-interactive
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/bash"
```

Выполните приведенные ниже команды в этой оболочке bash в контейнере. Чтобы получить маркер доступа для использования Azure Active Directory для проверки подлинности в хранилище ключей, выполните следующую команду:

```bash
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true -s
```

Выходные данные:

```bash
{"access_token":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSIsImtpZCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSJ9......xxxxxxxxxxxxxxxxx","refresh_token":"","expires_in":"28799","expires_on":"1539927532","not_before":"1539898432","resource":"https://vault.azure.net/","token_type":"Bearer"}
```

Чтобы сохранить маркер доступа в переменной для проверки подлинности в последующих командах, выполните такую команду:

```bash
token=$(curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true | jq -r '.access_token')

```

Теперь используйте маркер доступа для проверки подлинности в хранилище ключей и чтения секрета. Обязательно замените имя хранилища ключей URL-адресом (*https: \/ /mykeyvault.Vault.Azure.NET/...*):

```bash
curl https://mykeyvault.vault.azure.net/secrets/SampleSecret/?api-version=2016-10-01 -H "Authorization: Bearer $token"
```

Ответ с полученным значением секрета выглядит примерно так: Для получения секрета в коде необходимо выполнить синтаксический анализ этих выходных данных. Далее этот секрет применяется в последующих операциях для доступа к другому ресурсу Azure.

```bash
{"value":"Hello Container Instances","contentType":"ACIsecret","id":"https://mykeyvault.vault.azure.net/secrets/SampleSecret/xxxxxxxxxxxxxxxxxxxx","attributes":{"enabled":true,"created":1539965967,"updated":1539965967,"recoveryLevel":"Purgeable"},"tags":{"file-encoding":"utf-8"}}
```

## <a name="example-2-use-a-system-assigned-identity-to-access-azure-key-vault"></a>Пример 2. Использование назначенного системой удостоверения для доступа к хранилищу ключей Azure

### <a name="enable-system-assigned-identity-on-a-container-group"></a>Включение назначенного системой удостоверения для группы контейнеров

Выполните следующую команду [AZ Container Create](/cli/azure/container#az-container-create) , чтобы создать экземпляр контейнера на основе `azure-cli` образа Майкрософт. В этом примере представлена группа из одного контейнера, которую можно использовать в интерактивном режиме для запуска Azure CLI для доступа к другим службам Azure. 

Параметр `--assign-identity` без дополнительных значений включает для группы назначаемое системой управляемое удостоверение. Удостоверение ограничивается группой ресурсов группы контейнеров. Длительно выполняющаяся команда поддерживает выполнение контейнера. В этом примере используется та же группа ресурсов, которая используется для создания хранилища ключей, которое находится в области удостоверения.

```azurecli-interactive
# Get the resource ID of the resource group
rgID=$(az group show --name myResourceGroup --query id --output tsv)

# Create container group with system-managed identity
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azure-cli \
  --assign-identity --scope $rgID \
  --command-line "tail -f /dev/null"
```

Через несколько секунд вы должны получить ответ из интерфейса командной строки Azure, указывающий, что развертывание завершено. Проверьте его состояние с помощью команды [AZ Container показывать](/cli/azure/container#az-container-show) .

```azurecli-interactive
az container show \
  --resource-group myResourceGroup \
  --name mycontainer
```

Раздел `identity` в выходных данных выглядит примерно так, как показано ниже, и подтверждает создание назначаемого системой удостоверения в Azure Active Directory:

```console
[...]
"identity": {
    "principalId": "xxxxxxxx-528d-7083-b74c-xxxxxxxxxxxx",
    "tenantId": "xxxxxxxx-f292-4e60-9122-xxxxxxxxxxxx",
    "type": "SystemAssigned",
    "userAssignedIdentities": null
},
[...]
```

Присвойте переменной значение `principalId` (идентификатор субъекта-службы), чтобы использовать его в последующих шагах.

```azurecli-interactive
spID=$(az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query identity.principalId --out tsv)
```

### <a name="grant-container-group-access-to-the-key-vault"></a>Предоставление группе контейнеров доступа к хранилищу ключей

Выполните следующую команду [AZ keyvault Set-Policy](/cli/azure/keyvault) , чтобы задать политику доступа для хранилища ключей. В следующем примере управляемое системой удостоверение позволяет получить секреты из хранилища ключей:

```azurecli-interactive
 az keyvault set-policy \
   --name mykeyvault \
   --resource-group myResourceGroup \
   --object-id $spID \
   --secret-permissions get
```

### <a name="use-container-group-identity-to-get-secret-from-key-vault"></a>Использовать удостоверение группы контейнеров для получения секрета из хранилища ключей

Теперь можно использовать управляемое удостоверение для доступа к хранилищу ключей в работающем экземпляре контейнера. Сначала запустите оболочку Bash в контейнере:

```azurecli-interactive
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/bash"
```

Выполните приведенные ниже команды в этой оболочке bash в контейнере. Сначала войдите в Azure CLI с помощью управляемого удостоверения:

```bash
az login --identity
```

Из работающего контейнера извлеките секрет из хранилища ключей:

```bash
az keyvault secret show \
  --name SampleSecret \
  --vault-name mykeyvault --query value
```

Значение секрета извлекается:

```bash
"Hello Container Instances"
```

## <a name="enable-managed-identity-using-resource-manager-template"></a>Включение управляемого удостоверения с помощью шаблона Resource Manager

Чтобы включить управляемое удостоверение для группы контейнеров с помощью [шаблона Resource Manager](container-instances-multi-container-group.md), присвойте свойству `identity` объекта `Microsoft.ContainerInstance/containerGroups` значение объекта `ContainerGroupIdentity`. В следующих фрагментах кода показана настройка свойства `identity` для разных сценариев. Подробные сведения см. в [справочнике по шаблонам Resource Manager](/azure/templates/microsoft.containerinstance/containergroups). Укажите минимум `apiVersion` `2018-10-01` .

### <a name="user-assigned-identity"></a>Назначаемое пользователем удостоверение

Назначаемое пользователем удостоверение — это идентификатор ресурса в следующем формате:

```
"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}"
``` 

Вы можете указать одно или несколько назначаемых пользователем удостоверений.

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "myResourceID1": {
            }
        }
    }
```

### <a name="system-assigned-identity"></a>Назначаемое системой удостоверение

```json
"identity": {
    "type": "SystemAssigned"
    }
```

### <a name="system--and-user-assigned-identities"></a>Удостоверения, назначаемые системой и пользователями

Вы можете одновременно включить для группы контейнеров назначаемое системой удостоверение и одно или несколько назначаемых пользователей удостоверений.

```json
"identity": {
    "type": "System Assigned, UserAssigned",
    "userAssignedIdentities": {
        "myResourceID1": {
            }
        }
    }
...
```

## <a name="enable-managed-identity-using-yaml-file"></a>Включение управляемого удостоверения с помощью YAML-файла

Чтобы включить управляемое удостоверение в группе контейнеров, развернутой с помощью [YAML-файла](container-instances-multi-container-yaml.md), добавьте следующий YAML-файл.
Укажите минимум `apiVersion` `2018-10-01` .

### <a name="user-assigned-identity"></a>Назначаемое пользователем удостоверение

Назначаемое пользователем удостоверение — это идентификатор ресурса в следующем формате: 

```
'/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}'
```

Вы можете указать одно или несколько назначаемых пользователем удостоверений.

```YAML
identity:
  type: UserAssigned
  userAssignedIdentities:
    {'myResourceID1':{}}
```

### <a name="system-assigned-identity"></a>Назначаемое системой удостоверение

```YAML
identity:
  type: SystemAssigned
```

### <a name="system--and-user-assigned-identities"></a>Удостоверения, назначаемые системой и пользователями

Вы можете одновременно включить для группы контейнеров назначаемое системой удостоверение и одно или несколько назначаемых пользователей удостоверений.

```YAML
identity:
  type: SystemAssigned, UserAssigned
  userAssignedIdentities:
   {'myResourceID1':{}}
```

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как использовать управляемые удостоверения в службе "Экземпляры контейнеров Azure" и выполнять следующие действия:

> [!div class="checklist"]
> * применение к группе контейнеров удостоверения, назначаемого пользователем или системой;
> * Предоставление удостоверениям доступа к хранилищу ключей Azure
> * Использование управляемого удостоверения для доступа к хранилищу ключей из работающего контейнера

* Дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](../active-directory/managed-identities-azure-resources/index.yml).

* См. Пример использования управляемого удостоверения для доступа к хранилищу ключей из службы "экземпляры контейнеров Azure [" в пакете SDK для Azure Go](https://medium.com/@samkreter/c98911206328) .
