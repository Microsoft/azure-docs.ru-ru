---
title: Создание и получение атрибутов ключа в Azure Key Vault с помощью Azure CLI
description: Краткое руководство по настройке и получению ключа из Azure Key Vault с помощью Azure CLI
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.date: 01/27/2021
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4141e60370b397e799664b7d42384bbeb096bd05
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99071194"
---
# <a name="quickstart-set-and-retrieve-a-key-from-azure-key-vault-using-azure-cli"></a>Краткое руководство. Настройка и получение ключа из Azure Key Vault с помощью Azure CLI

Из этого краткого руководства вы узнаете, как создать хранилище ключей в Azure Key Vault с помощью Azure CLI. Azure Key Vault — это облачная служба, которая работает как защищенное хранилище секретов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Дополнительные сведения о хранилище ключей см. в статье [обзора](../general/overview.md). Azure CLI используется для создания ресурсов Azure и управления ими с помощью скриптов и команд. Создав ресурс, вы сможете сохранить в нем ключ.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим кратким руководством требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create a resource group](../../../includes/key-vault-cli-rg-creation.md)]

## <a name="create-a-key-vault"></a>Создание хранилища ключей

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-kv-creation.md)]

## <a name="add-a-key-to-key-vault"></a>Добавление ключа в Key Vault

Чтобы добавить ключ в хранилище, вам нужно выполнить несколько дополнительных действий. Этот ключ может использоваться приложением. 

Выполните приведенные ниже команды, чтобы создать ключ **ExampleKey**:

```azurecli
az keyvault key create --vault-name "<your-unique-keyvault-name>" -n ExampleKey --protection software
```

Теперь ключ, добавленный в Azure Key Vault, можно вызвать, используя его URI. Чтобы получить текущую версию, используйте **https://<your-unique-keyvault-name>.vault.azure.net/keys/ExampleKey**. 

Чтобы просмотреть ранее сохраненный ключ, выполните следующие действия:

```azurecli

az keyvault key show --name "ExampleKey" --vault-name "<your-unique-keyvault-name>"
```

Вы создали Key Vault, сохранили в нем ключ и извлекли его.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-delete-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы создали Key Vault и сохранили в нем ключ. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- Справочник по [командам az keyvault для Azure CLI](/cli/azure/keyvault)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)
