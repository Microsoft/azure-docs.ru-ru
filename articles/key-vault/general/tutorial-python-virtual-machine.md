---
title: Учебник по использованию Azure Key Vault с виртуальной машиной в Python | Документация Майкрософт
description: В этом руководстве показано, как, используя Python, настроить виртуальную машину для чтения секрета из хранилища ключей.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/20/2020
ms.author: mbaldwin
ms.custom: mvc, devx-track-python, devx-track-azurecli
ms.openlocfilehash: 25182105db831724565c6bf3dbbbb79832b677f7
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772066"
---
# <a name="tutorial-use-azure-key-vault-with-a-virtual-machine-in-python"></a>Руководство по Использование Azure Key Vault с виртуальной машиной в Python

Azure Key Vault помогает защищать ключи, секреты и сертификаты, например ключи API и строки подключения к базам данных.

В этом учебнике описано, как настроить приложение Python, чтобы оно считывало данные из Azure Key Vault с помощью управляемых удостоверений для ресурсов Azure. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создайте хранилище ключей.
> * хранение секрета в Key Vault;
> * создание виртуальной машины Linux в Azure;
> * включение [управляемого удостоверения](../../active-directory/managed-identities-azure-resources/overview.md) для виртуальной машины;
> * предоставление разрешений, необходимых консольному приложению для чтения данных из Key Vault;
> * получение секрета из Key Vault.

Перед началом работы ознакомьтесь c [основными понятиями службы Key Vault](basic-concepts.md). 

Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="prerequisites"></a>Предварительные требования

Для Windows, Mac и Linux:
  * [Git](https://git-scm.com/downloads);
  * Для этого руководства требуется запустить Azure CLI локально. Необходимо установить Azure CLI версии 2.0.4 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо установить или обновить CLI, ознакомьтесь со статьей [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="log-in-to-azure"></a>Вход в Azure

Чтобы войти в Azure с помощью Azure CLI, введите следующую команду:

```azurecli
az login
```

## <a name="create-a-resource-group-and-key-vault"></a>Создание группы ресурсов и хранилища ключей

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

## <a name="populate-your-key-vault-with-a-secret"></a>Запись секрета в хранилище ключей

[!INCLUDE [Create a secret](../../../includes/key-vault-create-secret.md)]

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с именем **myVM** одним из следующих способов:

| Linux | Windows |
|--|--|
| [Azure CLI](../../virtual-machines/linux/quick-create-cli.md) | [Azure CLI](../../virtual-machines/windows/quick-create-cli.md) |
| [PowerShell](../../virtual-machines/linux/quick-create-powershell.md) | [PowerShell](../../virtual-machines/windows/quick-create-powershell.md) |
| [Портал Azure](../../virtual-machines/linux/quick-create-portal.md) | [Портал Azure](../../virtual-machines/windows/quick-create-portal.md) |

Чтобы создать виртуальную машину Linux с помощью Azure CLI, используйте команду [az vm create](/cli/azure/vm).  В примере ниже добавляется учетная запись пользователя с именем *azureuser*. Параметр `--generate-ssh-keys` автоматически создает ключ SSH и сохраняет его в стандартное расположение ( *~/.ssh*). 

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

Запишите значение `publicIpAddress` в выходных данных.

## <a name="assign-an-identity-to-the-vm"></a>Назначение удостоверения виртуальной машине

Создайте назначаемое системой удостоверение для виртуальной машины с помощью команды Azure CLI [az vm identity assign](/cli/azure/vm/identity#az_vm_identity_assign):

```azurecli
az vm identity assign --name "myVM" --resource-group "myResourceGroup"
```

Обратите внимание на присвоенный системой идентификатор, который отображается в следующем коде. Выходные данные приведенной выше команды выглядят следующим образом. 

```output
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## <a name="assign-permissions-to-the-vm-identity"></a>Назначение разрешения для идентификатора виртуальной машины

Теперь вы можете предоставить созданному выше идентификатору разрешение для хранилища ключей, выполнив следующую команду.

```azurecli
az keyvault set-policy --name "<your-unique-keyvault-name>" --object-id "<systemAssignedIdentity>" --secret-permissions get list
```

## <a name="log-in-to-the-vm"></a>Вход на виртуальную машину

Чтобы войти в виртуальную машину, следуйте инструкциям по подключению и входу в виртуальную машину Azure на платформе [Windows](../../virtual-machines/windows/connect-logon.md) или [Linux](../../virtual-machines/linux/login-using-aad.md).


Чтобы войти на виртуальную машину Linux, можно использовать команду SSH с "<publicIpAddress>" из шага по [созданию виртуальной машины](#create-a-virtual-machine):

```terminal
ssh azureuser@<PublicIpAddress>
```

## <a name="install-python-libraries-on-the-vm"></a>Установка библиотек Python на виртуальной машине

На виртуальной машине установите две библиотеки Python, которые будут использоваться в нашем скрипте Python: `azure-keyvault-secrets` и `azure.identity`.  

Например, на виртуальной машине Linux их можно установить с помощью `pip3`:

```bash
pip3 install azure-keyvault-secrets

pip3 install azure.identity
```

## <a name="create-and-edit-the-sample-python-script"></a>Создание и изменение примера скрипта Python

На виртуальной машине создайте файл Python с именем **sample.py**. Измените файл, чтобы он содержал следующий код, заменив "<your-unique-keyvault-name>" именем своего хранилища ключей:

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = "<your-unique-keyvault-name>"
KVUri = f"https://{keyVaultName}.vault.azure.net"
secretName = "mySecret"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)
retrieved_secret = client.get_secret(secretName)

print(f"The value of secret '{secretName}' in '{keyVaultName}' is: '{retrieved_secret.value}'")
```

## <a name="run-the-sample-python-app"></a>Запуск примера приложения Python

Теперь выполните **sample.py**. Если все прошло хорошо, должно вернуться значение секрета:

```bash
python3 sample.py

The value of secret 'mySecret' in '<your-unique-keyvault-name>' is: 'Success!'
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите виртуальную машину и хранилище ключей, если они больше не нужны.  Это можно сделать быстро, просто удалив группу ресурсов, к которой они относятся:

```azurecli
az group delete -g myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

[Справочник по REST API для Azure Key Vault](/rest/api/keyvault/)