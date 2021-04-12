---
title: Краткое руководство по использованию клиентской библиотеки Azure Key Vault для Python. Управление сертификатами
description: Узнайте, как создавать, извлекать и удалять сертификаты из Azure Key Vault с помощью клиентской библиотеки Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: 2026b1721dbe05af1ecc52324d1dd0ebf2ec3d08
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104802664"
---
# <a name="quickstart-azure-key-vault-certificate-client-library-for-python"></a>Краткое руководство. Использование клиентской библиотеки сертификатов Azure Key Vault для Python

Приступите к работе с клиентской библиотекой сертификатов Azure Key Vault для Python. Чтобы установить пакет и испробовать пример кода для выполнения базовых задач, выполните описанные ниже шаги. Используя Key Vault для хранения сертификатов, вам не надо сохранять их в коде. Это повышает безопасность приложения.

[Справочная документация по API](/python/api/overview/azure/keyvault-certificates-readme) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-certificates) | [Пакет (Python Package Index)](https://pypi.org/project/azure-keyvault-certificates)

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


1. Установите клиентскую библиотеку сертификатов Key Vault:

    ```terminal
    pip install azure-keyvault-certificates
    ```

### <a name="create-a-resource-group-and-key-vault"></a>Создание группы ресурсов и хранилища ключей

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-python-qs-rg-kv-creation.md)]

### <a name="grant-access-to-your-key-vault"></a>Предоставление доступа к хранилищу ключей

Создайте для хранилища ключей политику доступа, которая предоставляет учетной записи пользователя разрешения на использование сертификата.

```console
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --certificate-permissions delete get list create
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

Клиентская библиотека сертификатов Azure Key Vault для Python позволяет управлять сертификатами. В приведенном ниже примере кода показано, как создать клиент, а также как указать, получить и удалить сертификат.

Создайте файл с именем *kv_certificates.py*, который содержит этот код.

```python
import os
from azure.keyvault.certificates import CertificateClient, CertificatePolicy,CertificateContentType, WellKnownIssuerNames 
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = "https://" + keyVaultName + ".vault.azure.net"

credential = DefaultAzureCredential()
client = CertificateClient(vault_url=KVUri, credential=credential)

certificateName = input("Input a name for your certificate > ")

print(f"Creating a certificate in {keyVaultName} called '{certificateName}' ...")

policy = CertificatePolicy.get_default()
poller = client.begin_create_certificate(certificate_name=certificateName, policy=policy)
certificate = poller.result()

print(" done.")

print(f"Retrieving your certificate from {keyVaultName}.")

retrieved_certificate = client.get_certificate(certificateName)

print(f"Certificate with name '{retrieved_certificate.name}' was found'.")
print(f"Deleting your certificate from {keyVaultName} ...")

poller = client.begin_delete_certificate(certificateName)
deleted_certificate = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Выполнение кода

Убедитесь, что код из предыдущего раздела находится в файле с именем *kv_certificates.py*. Затем выполните следующую команду, чтобы запустить код:

```terminal
python kv_certificates.py
```

- Если происходят ошибки разрешений, убедитесь, что вы [выполнили команду `az keyvault set-policy`](#grant-access-to-your-key-vault).
- Повторное выполнение кода с тем же именем ключа может вызвать ошибку (Conflict) Certificate <name> is currently in a deleted but recoverable state. ((Конфликт) Сертификат <name> в настоящее время находится в удаленном состоянии, но допускающем восстановление). Используйте другое имя ключа.

## <a name="code-details"></a>Сведения о коде

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс ["DefaultAzureCredential()"](/python/api/azure-identity/azure.identity.defaultazurecredential), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения см. в статье [Проверка подлинности учетных данных Azure по умолчанию](https://docs.microsoft.com/python/api/overview/azure/identity-readme). 

```python
credential = DefaultAzureCredential()
client = CertificateClient(vault_url=KVUri, credential=credential)
```

### <a name="save-a-certificate"></a>Сохранение сертификата

После получения клиентского объекта для хранилища ключей можно создать сертификат с помощью метода [begin_create_certificate](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificateclient?#begin-create-certificate-certificate-name--policy----kwargs-): 

```python
policy = CertificatePolicy.get_default()
poller = client.begin_create_certificate(certificate_name=certificateName, policy=policy)
certificate = poller.result()
```

В этом случае для сертификата требуется политика, которая получена с помощью метода [CertificatePolicy.get_default](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificatepolicy?#get-default--).

Вызов метода `begin_create_certificate` создает асинхронный вызов REST API Azure для хранилища ключей. Асинхронный вызов возвращает объект модуля опроса. Чтобы дождаться результата операции, вызовите метод `result` модуля опроса.

При обработке запроса Azure проверяет подлинность удостоверения вызывающего объекта (субъекта-службы) с помощью объекта учетных данных, предоставленных клиенту.


### <a name="retrieve-a-certificate"></a>Получение сертификата

Чтобы прочитать сертификат из Key Vault, используйте метод [get_certificate](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificateclient?#get-certificate-certificate-name----kwargs-):

```python
retrieved_certificate = client.get_certificate(certificateName)
 ```

Команда Azure CLI [az keyvault certificate show](/cli/azure/keyvault/certificate?#az-keyvault-certificate-show) позволяет также убедиться, что сертификат задан успешно.

### <a name="delete-a-certificate"></a>Удаление сертификата

Чтобы удалить сертификат, используйте метод [begin_delete_certificate](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificateclient?#begin-delete-certificate-certificate-name----kwargs-):

```python
poller = client.begin_delete_certificate(certificateName)
deleted_certificate = poller.result()
```

Метод `begin_delete_certificate` является асинхронным и возвращает объект модуля опроса. При вызове `result` модуля опроса метод ожидает его завершения.

Команда Azure CLI [az keyvault certificate show](/cli/azure/keyvault/certificate?#az-keyvault-certificate-show) позволяет убедиться, что сертификат удален.

После удаления сертификат остается в удаленном, но восстанавливаемом состоянии в течение некоторого времени. При повторном выполнении кода используйте другое имя сертификата.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите также поэкспериментировать с [секретами](../secrets/quick-create-python.md) и [ключами](../keys/quick-create-python.md), можно повторно использовать Key Vault, созданный в этой статье.

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
