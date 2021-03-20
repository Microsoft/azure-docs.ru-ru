---
title: Шифрование данных развертывания
description: Узнайте о шифровании данных, сохраненных для ресурсов экземпляра контейнера, и о том, как шифровать данные с помощью ключа, управляемого клиентом.
ms.topic: article
ms.date: 01/17/2020
author: macolso
ms.author: macolso
ms.openlocfilehash: 1b73ce5c994231a1c7b2f26ad702f2ad5880ba44
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94686282"
---
# <a name="encrypt-deployment-data"></a>Шифрование данных развертывания

При выполнении ресурсов службы "экземпляры контейнеров Azure" (ACI) в облаке служба ACI собирает и сохраняет данные, связанные с контейнерами. ACI автоматически шифрует эти данные при сохранении в облаке. Это шифрование защищает данные для удовлетворения обязательств по обеспечению безопасности и соответствия требованиям вашей организации. ACI также позволяет шифровать эти данные с помощью собственного ключа, обеспечивая больший контроль над данными, связанными с развертываниями ACI.

## <a name="about-aci-data-encryption"></a>О шифровании данных ACI 

Данные в ACI шифруются и расшифровываются с помощью 256-разрядного шифрования AES. Он включен для всех развертываний ACI, и вам не нужно изменять развертывание или контейнеры, чтобы воспользоваться преимуществами этого шифрования. Сюда входят метаданные о развертывании, переменные среды, ключи, передаваемые в контейнеры, и журналы, сохраненные после остановки контейнеров, чтобы их можно было увидеть. Шифрование не влияет на производительность группы контейнеров, а дополнительные затраты на шифрование не взимается.

## <a name="encryption-key-management"></a>Управление ключами шифрования

Вы можете использовать ключи, управляемые корпорацией Майкрософт, для шифрования данных контейнера или управлять шифрованием с помощью собственных ключей. Эти параметры сравниваются в следующей таблице. 

|    |    Ключи, управляемые Майкрософт     |     Ключи, управляемые клиентом     |
|----|----|----|
|    **Операции шифрования и расшифровки**    |    Azure    |    Azure    |
|    **Хранилище ключей**    |    Хранилище ключей (Майкрософт)    |    Azure Key Vault    |
|    **Ответственность за смену ключей**    |    Microsoft    |    Клиент    |
|    **Доступ к ключам**    |    Только Майкрософт    |    Майкрософт, клиент    |

Оставшаяся часть документа описывает шаги, необходимые для шифрования данных развертывания ACI с помощью ключа (управляемого клиентом ключа). 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="encrypt-data-with-a-customer-managed-key"></a>Шифрование данных с помощью управляемого клиентом ключа

### <a name="create-service-principal-for-aci"></a>Создание субъекта-службы для ACI

Первым делом необходимо убедиться, что у [клиента Azure](../active-directory/develop/quickstart-create-new-tenant.md) есть субъект-служба, назначенный для предоставления разрешений службе "экземпляры контейнеров Azure". 

> [!IMPORTANT]
> Чтобы выполнить следующую команду и успешно создать субъект-службу, убедитесь, что у вас есть разрешения на создание субъектов-служб в клиенте.
>

Следующая команда CLI настроит ACI SP в среде Azure:

```azurecli-interactive
az ad sp create --id 6bb8e274-af5d-4df2-98a3-4fd78b4cafd9
```

В выходных данных выполнения этой команды должен отображаться субъект-служба, настроенный с "displayName": "служба экземпляра контейнера Azure".

Если вы не можете успешно создать субъект-службу:
* Убедитесь, что у вас есть разрешения на это в клиенте.
* Убедитесь, что субъект-служба уже существует в вашем клиенте для развертывания в ACI. Это можно сделать, запустив `az ad sp show --id 6bb8e274-af5d-4df2-98a3-4fd78b4cafd9` и используя этот субъект-службу.

### <a name="create-a-key-vault-resource"></a>Создание ресурса хранилища ключей

Создайте хранилище ключей Azure с помощью [портала Azure](../key-vault/general/quick-create-portal.md), [Azure CLI](../key-vault/general/quick-create-cli.md) или [Azure PowerShell](../key-vault/general/quick-create-powershell.md).

Для свойств хранилища ключей используйте следующие рекомендации. 
* Имя: требуется уникальное имя. 
* Подписка: Выберите подписку.
* В разделе Группа ресурсов выберите существующую группу ресурсов либо создайте новую и введите имя группы ресурсов.
* Выберите расположение в раскрывающемся меню "Расположение".
* Можно оставить другие параметры по умолчанию или выбрать их в зависимости от дополнительных требований.

> [!IMPORTANT]
> При использовании управляемых клиентом ключей для шифрования шаблона развертывания ACI рекомендуется установить следующие два свойства в хранилище ключей, обратимое удаление и не очищать. Эти свойства не включены по умолчанию, но их можно включить с помощью PowerShell или Azure CLI в новом или существующем хранилище ключей.

### <a name="generate-a-new-key"></a>Создать новый ключ 

После создания хранилища ключей перейдите к ресурсу в портал Azure. В меню навигации слева в колонке ресурсов в разделе Параметры щелкните **ключи**. В представлении "ключи" нажмите кнопку "создать/импортировать", чтобы создать новый ключ. Используйте любое уникальное имя для этого ключа и другие параметры в соответствии с вашими требованиями. 

![Создать новый ключ](./media/container-instances-encrypt-data/generate-key.png)

### <a name="set-access-policy"></a>Настройка политики доступа

Создайте новую политику доступа, чтобы разрешить службе ACI доступ к ключу.

* После создания ключа вернитесь в колонку ресурсов хранилища ключей, в разделе Параметры щелкните **политики доступа**.
* На странице "политики доступа" для хранилища ключей щелкните **Добавить политику доступа**.
* Задайте *разрешения для ключа* , включающие разрешения на **Получение** ключа Key Set и **разобертывания** . ![](./media/container-instances-encrypt-data/set-key-permissions.png)
* Для *выбора субъекта* выберите **служба экземпляра контейнера Azure** .
* Нажмите кнопку **Добавить** внизу. 

Политика доступа должна отобразиться в политиках доступа в хранилище ключей.

![Новая политика доступа](./media/container-instances-encrypt-data/access-policy.png)

### <a name="modify-your-json-deployment-template"></a>Изменение шаблона развертывания JSON

> [!IMPORTANT]
> Шифрование данных развертывания с помощью управляемого клиентом ключа доступно в последней версии API (2019-12-01), которая в настоящее время истекает. Укажите версию API в шаблоне развертывания. Если у вас возникли проблемы, обратитесь в службу поддержки Azure.

После настройки ключа и политики доступа к хранилищу ключей добавьте следующие свойства в шаблон развертывания ACI. Дополнительные сведения о развертывании ресурсов ACI с помощью шаблона см. в [руководстве по развертыванию группы с несколькими контейнерами с использованием шаблона диспетчер ресурсов](./container-instances-multi-container-group.md). 
* В разделе `resources` задайте `apiVersion` для значение `2019-12-01` .
* В разделе Свойства группы контейнеров шаблона развертывания добавьте объект `encryptionProperties` , который содержит следующие значения:
  * `vaultBaseUrl`: DNS-имя хранилища ключей можно найти в колонке "Обзор" ресурса хранилища ключей на портале.
  * `keyName`: имя созданного ранее ключа.
  * `keyVersion`: Текущая версия ключа. Это можно найти, щелкнув сам ключ (в разделе "ключи" в разделе "Параметры" вашего ресурса хранилища ключей).
* В свойствах группы контейнеров добавьте `sku` свойство со значением `Standard` . `sku`Свойство является обязательным в API версии 2019-12-01.

В следующем фрагменте шаблона показаны эти дополнительные свойства для шифрования данных развертывания.

```json
[...]
"resources": [
    {
        "name": "[parameters('containerGroupName')]",
        "type": "Microsoft.ContainerInstance/containerGroups",
        "apiVersion": "2019-12-01",
        "location": "[resourceGroup().location]",    
        "properties": {
            "encryptionProperties": {
                "vaultBaseUrl": "https://example.vault.azure.net",
                "keyName": "acikey",
                "keyVersion": "xxxxxxxxxxxxxxxx"
            },
            "sku": "Standard",
            "containers": {
                [...]
            }
        }
    }
]
```

Ниже приведен полный шаблон, адаптированный из шаблона в [учебнике развертывание многоконтейнерной группы с помощью шаблона диспетчер ресурсов](./container-instances-multi-container-group.md). 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "containerGroupName": {
      "type": "string",
      "defaultValue": "myContainerGroup",
      "metadata": {
        "description": "Container Group name."
      }
    }
  },
  "variables": {
    "container1name": "aci-tutorial-app",
    "container1image": "mcr.microsoft.com/azuredocs/aci-helloworld:latest",
    "container2name": "aci-tutorial-sidecar",
    "container2image": "mcr.microsoft.com/azuredocs/aci-tutorial-sidecar"
  },
  "resources": [
    {
      "name": "[parameters('containerGroupName')]",
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "encryptionProperties": {
            "vaultBaseUrl": "https://example.vault.azure.net",
            "keyName": "acikey",
            "keyVersion": "xxxxxxxxxxxxxxxx"
        },
        "sku": "Standard",  
        "containers": [
          {
            "name": "[variables('container1name')]",
            "properties": {
              "image": "[variables('container1image')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "port": 80
                },
                {
                  "port": 8080
                }
              ]
            }
          },
          {
            "name": "[variables('container2name')]",
            "properties": {
              "image": "[variables('container2image')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              }
            }
          }
        ],
        "osType": "Linux",
        "ipAddress": {
          "type": "Public",
          "ports": [
            {
              "protocol": "tcp",
              "port": "80"
            },
            {
                "protocol": "tcp",
                "port": "8080"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "containerIPv4Address": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.ContainerInstance/containerGroups/', parameters('containerGroupName'))).ipAddress.ip]"
    }
  }
}
```

### <a name="deploy-your-resources"></a>Развертывание ресурсов

Если файл шаблона был создан и изменен на рабочем столе, его можно передать в каталог Cloud Shell, перетащив в него файл. 

Создайте группу ресурсов с помощью команды [az group create][az-group-create].

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Разверните шаблон с помощью команды [AZ Deployment Group Create][az-deployment-group-create] .

```azurecli-interactive
az deployment group create --resource-group myResourceGroup --template-file deployment-template.json
```

В течение нескольких секунд вы должны получить исходный ответ Azure. После завершения развертывания все данные, связанные с ним, сохраненные службой ACI, будут зашифрованы с помощью предоставленного ключа.

<!-- LINKS - Internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-deployment-group-create]: /cli/azure/deployment/group/#az-deployment-group-create
