---
title: Краткое руководство по использованию клиентской библиотеки Azure Key Vault для Python. Управление секретами
description: Узнайте, как создавать, извлекать и удалять секреты из Azure Key Vault с помощью клиентской библиотеки Python.
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: e06881d078b4e881174c3e931f7898cb622ad7f9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766359"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-python"></a>Краткое руководство. Использование клиентской библиотеки секретов Azure Key Vault для Python

Начало работы с клиентской библиотекой секретов Azure Key Vault для Python. Чтобы установить пакет и испробовать пример кода для выполнения базовых задач, выполните описанные ниже шаги. Используя Key Vault для хранения секретов, вам не надо сохранять их в коде. Это повышает безопасность приложения.

[Справочная документация по API](/python/api/overview/azure/keyvault-secrets-readme) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-secrets) | [Пакет (Python Package Index)](https://pypi.org/project/azure-keyvault-secrets/)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python версии 2.7 и выше или 3.6 и выше.](/azure/developer/python/configure-local-development-environment)
- [Azure CLI](/cli/azure/install-azure-cli)

В этом кратком руководстве предполагается, что вы используете [Azure CLI](/cli/azure/install-azure-cli) в окне терминала Linux.


## <a name="set-up-your-local-environment"></a>Настройка локальной среды
В этом кратком руководстве используется библиотека удостоверений Azure и Azure CLI для проверки подлинности пользователя в службах Azure. Разработчики также могут использовать Visual Studio или Visual Studio Code для проверки подлинности своих вызовов. Дополнительные сведения см. в статье [Проверка подлинности клиента с помощью клиентской библиотеки удостоверений Azure](/python/api/overview/azure/identity-readme).

### <a name="sign-in-to-azure"></a>Вход в Azure

1. Выполните команду `login`.

    ```azurecli-interactive
    az login
    ```

    Если в CLI можно запустить браузер по умолчанию, откроется браузер со страницей входа.

    Если нет, самостоятельно откройте в браузере страницу [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и введите код авторизации, отображаемый в терминале.

2. Выполните вход в браузере с помощью учетных данных.

### <a name="install-the-packages"></a>Установка пакетов

1. В терминале или в командной строке создайте подходящую папку проекта, а затем создайте и активируйте виртуальную среду Python, как описано в разделе [Использование виртуальных окружений Python](/azure/developer/python/configure-local-development-environment?tabs=cmd#use-python-virtual-environments).

1. Установите библиотеку удостоверений Azure Active Directory:

    ```terminal
    pip install azure-identity
    ```


1. Установите библиотеку секретов Azure Key Vault:

    ```terminal
    pip install azure-keyvault-secrets
    ```

### <a name="create-a-resource-group-and-key-vault"></a>Создание группы ресурсов и хранилища ключей

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-python-qs-rg-kv-creation.md)]

### <a name="grant-access-to-your-key-vault"></a>Предоставление доступа к хранилищу ключей

Создайте для хранилища ключей политику доступа, которая предоставляет учетной записи пользователя разрешения на использование секрета.

```console
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --secret-permissions delete get list set
```

#### <a name="set-environment-variables"></a>Настройка переменных среды

Это приложение использует имя хранилища ключей в качестве переменной среды с именем `KEY_VAULT_NAME`.

```bash
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="create-the-sample-code"></a>Создание примера кода

Клиентская библиотека секретов Azure Key Vault для Python позволяет управлять секретами. В приведенном примере кода показано, как создать клиент, а также как задать, извлечь и удалить секрет.

Создайте файл с именем *kv_secrets.py*, который содержит этот код.

```python
import os
import cmd
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = f"https://{keyVaultName}.vault.azure.net"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)

secretName = input("Input a name for your secret > ")
secretValue = input("Input a value for your secret > ")

print(f"Creating a secret in {keyVaultName} called '{secretName}' with the value '{secretValue}' ...")

client.set_secret(secretName, secretValue)

print(" done.")

print(f"Retrieving your secret from {keyVaultName}.")

retrieved_secret = client.get_secret(secretName)

print(f"Your secret is '{retrieved_secret.value}'.")
print(f"Deleting your secret from {keyVaultName} ...")

poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Выполнение кода

Убедитесь, что код из предыдущего раздела находится в файле с именем *kv_secrets.py*. Затем выполните следующую команду, чтобы запустить код:

```terminal
python kv_secrets.py
```

- Если происходят ошибки разрешений, убедитесь, что вы [выполнили команду `az keyvault set-policy`](#grant-access-to-your-key-vault).
- Повторное выполнение кода с тем же именем секрета может вызвать ошибку (Conflict) Secret <name> is currently in a deleted but recoverable state. ((Конфликт) Секрет <name> в настоящее время находится в удаленном состоянии, но допускающем восстановление). Используйте другое имя секрета.

## <a name="code-details"></a>Сведения о коде

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс ["DefaultAzureCredential()"](https://docs.microsoft.com/python/api/azure-identity/azure.identity.defaultazurecredential), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения см. в статье [Проверка подлинности учетных данных Azure по умолчанию](https://docs.microsoft.com/python/api/overview/azure/identity-readme). 

```python
credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)
```

### <a name="save-a-secret"></a>Сохранение секрета

После получения клиентского объекта для хранилища ключей можно сохранить секрет с помощью метода [set_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#set-secret-name--value----kwargs-): 

```python
client.set_secret(secretName, secretValue)
```

Вызов `set_secret` создает вызов REST API Azure для хранилища ключей.

При обработке запроса Azure проверяет подлинность удостоверения вызывающего объекта (субъекта-службы) с помощью объекта учетных данных, предоставленных клиенту.

### <a name="retrieve-a-secret"></a>Получение секрета

Чтобы считать секрет из Key Vault, используйте метод [get_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#get-secret-name--version-none----kwargs-):

```python
retrieved_secret = client.get_secret(secretName)
 ```

Значение секрета содержится в `retrieved_secret.value`.

Команда Azure CLI [az keyvault secret show](/cli/azure/keyvault/secret?#az_keyvault_secret_show) также позволяет получить секрет.

### <a name="delete-a-secret"></a>Удаление секрета.

Чтобы удалить секрет, используйте метод [begin_delete_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#begin-delete-secret-name----kwargs-):

```python
poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()
```

Метод `begin_delete_secret` является асинхронным и возвращает объект модуля опроса. При вызове `result` модуля опроса метод ожидает его завершения.

Команда Azure CLI [az keyvault secret show](/cli/azure/keyvault/secret?#az_keyvault_secret_show) позволяет убедиться, что секрет удален.

После удаления секрет остается в удаленном, но восстанавливаемом состоянии в течение некоторого времени. При повторном выполнении кода используйте другое имя секрета.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите также поэкспериментировать с [сертификатами](../certificates/quick-create-python.md) и [ключами](../keys/quick-create-python.md), можно повторно использовать Key Vault, созданный в этой статье.

В противном случае после завершения работы с созданными в этой статье ресурсами используйте следующую команду, чтобы удалить группу ресурсов и все содержащиеся в ней ресурсы:

```azurecli
az group delete --resource-group KeyVault-PythonQS-rg
```

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор хранилища ключей Azure](../general/overview.md)
- [Безопасный доступ к хранилищу ключей](../general/security-overview.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- [Обзор системы безопасности Key Vault](../general/security-overview.md)
- [Проверка подлинности с помощью Key Vault](../general/authentication.md)
