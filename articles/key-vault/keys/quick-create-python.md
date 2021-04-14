---
title: Краткое руководство по использованию клиентской библиотеки Azure Key Vault для Python. Управление ключами
description: Узнайте, как создавать, извлекать и удалять ключи из Azure Key Vault с помощью клиентской библиотеки Python.
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: b1355acee5c115accb2515f797b402e66cd2ec0a
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107374732"
---
# <a name="quickstart-azure-key-vault-keys-client-library-for-python"></a>Краткое руководство. Использование клиентской библиотеки ключей Azure Key Vault для Python

Приступите к работе с клиентской библиотекой Azure Key Vault для Python. Чтобы установить пакет и испробовать пример кода для выполнения базовых задач, выполните описанные ниже шаги. Используя Key Vault для хранения криптографических ключей, вам не надо сохранять их в коде. Это повышает безопасность приложения.

[Справочная документация по API](/python/api/overview/azure/keyvault-keys-readme) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-keys) | [Пакет (Python Package Index)](https://pypi.org/project/azure-keyvault-keys/)

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
    pip install azure.identity
    ```


1. Установите клиентскую библиотеку ключей Azure Key Vault:

    ```terminal
    pip install azure-keyvault-keys
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

Windows
```cmd
set KEY_VAULT_NAME=<your-key-vault-name>
````
Windows PowerShell
```powershell
$Env:KEY_VAULT_NAME="<your-key-vault-name>"
```

macOS или Linux
```cmd
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="create-the-sample-code"></a>Создание примера кода

Клиентская библиотека ключей Azure Key Vault для Python позволяет управлять криптографическими ключами. В приведенном ниже примере кода показано, как создать клиент, а также как задать, извлечь и удалить ключ.

Создайте файл с именем *kv_keys.py*, который содержит этот код.

```python
import os
from azure.keyvault.keys import KeyClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = "https://" + keyVaultName + ".vault.azure.net"

credential = DefaultAzureCredential()
client = KeyClient(vault_url=KVUri, credential=credential)

keyName = input("Input a name for your key > ")

print(f"Creating a key in {keyVaultName} called '{keyName}' ...")

rsa_key = client.create_rsa_key(keyName, size=2048)

print(" done.")

print(f"Retrieving your key from {keyVaultName}.")

retrieved_key = client.get_key(keyName)

print(f"Key with name '{retrieved_key.name}' was found.")
print(f"Deleting your key from {keyVaultName} ...")

poller = client.begin_delete_key(keyName)
deleted_key = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Выполнение кода

Убедитесь, что код из предыдущего раздела находится в файле с именем *kv_keys.py*. Затем выполните следующую команду, чтобы запустить код:

```terminal
python kv_keys.py
```

- Если происходят ошибки разрешений, убедитесь, что вы [выполнили команду `az keyvault set-policy`](#grant-access-to-your-key-vault).
- Повторное выполнение кода с тем же именем ключа может вызвать ошибку (Conflict) Key <name> is currently in a deleted but recoverable state. ((Конфликт) Ключ <name> в настоящее время находится в удаленном состоянии, но допускающем восстановление). Используйте другое имя ключа.

## <a name="code-details"></a>Сведения о коде

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс ["DefaultAzureCredential()"](/python/api/azure-identity/azure.identity.defaultazurecredential), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения см. в статье [Проверка подлинности учетных данных Azure по умолчанию](https://docs.microsoft.com/python/api/overview/azure/identity-readme). 


```python
credential = DefaultAzureCredential()
client = KeyClient(vault_url=KVUri, credential=credential)
```

## <a name="save-a-key"></a>Сохранение ключа

После получения клиентского объекта для хранилища ключей можно сохранить ключ с помощью метода [create_rsa_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#create-rsa-key-name----kwargs-): 

```python
rsa_key = client.create_rsa_key(keyName, size=2048)
```

Кроме того, можно использовать метод [create_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#create-key-name--key-type----kwargs-) или [create_ec_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#create-ec-key-name----kwargs-).

Вызов метода `create` создает вызов REST API Azure для хранилища ключей.

При обработке запроса Azure проверяет подлинность удостоверения вызывающего объекта (субъекта-службы) с помощью объекта учетных данных, предоставленных клиенту.

## <a name="retrieve-a-key"></a>Получение ключа

Чтобы прочитать ключ из Key Vault, используйте метод [get_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#get-key-name--version-none----kwargs-):

```python
retrieved_key = client.get_key(keyName)
 ```

Команда Azure CLI [az keyvault key show](/cli/azure/keyvault/key?#az-keyvault-key-show) позволяет также убедиться, что ключ задан успешно.

### <a name="delete-a-key"></a>Удаление ключа

Чтобы удалить ключ, используйте метод [begin_delete_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?#begin-delete-key-name----kwargs-):

```python
poller = client.begin_delete_key(keyName)
deleted_key = poller.result()
```

Метод `begin_delete_key` является асинхронным и возвращает объект модуля опроса. При вызове `result` модуля опроса метод ожидает его завершения.

Команда Azure CLI [az keyvault key show](/cli/azure/keyvault/key?#az-keyvault-key-show) позволяет убедиться, что ключ удален.

После удаления ключ остается в удаленном, но восстанавливаемом состоянии в течение некоторого времени. При повторном выполнении кода используйте другое имя ключа.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите также поэкспериментировать с [сертификатами](../certificates/quick-create-python.md) и [секретами](../secrets/quick-create-python.md), можно повторно использовать Key Vault, созданный в этой статье.

В противном случае после завершения работы с созданными в этой статье ресурсами используйте следующую команду, чтобы удалить группу ресурсов и все содержащиеся в ней ресурсы:

```azurecli
az group delete --resource-group KeyVault-PythonQS-rg
```

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор хранилища ключей Azure](../general/overview.md)
- [Безопасный доступ к хранилищу ключей](../general/secure-your-key-vault.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- [Обзор системы безопасности Key Vault](../general/security-overview.md)
- [Проверка подлинности с помощью Key Vault](../general/authentication.md)
