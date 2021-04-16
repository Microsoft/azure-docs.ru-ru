---
title: включить файл
description: включить файл
services: azure-communication-services
author: tomaschladek
manager: nmurav
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: tchladek
ms.openlocfilehash: db6e4a9c6d2829c7980164d5b79bd33e4b3fb6eb
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113097"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Python](https://www.python.org/downloads/) 2.7, 3.6 или более поздней версии.
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../create-communication-resource.md)

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-python-application"></a>Создание приложения Python

1. Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.

   ```console
   mkdir access-tokens-quickstart && cd access-tokens-quickstart
   ```

1. Используйте текстовый редактор, чтобы создать файл с именем **issue-access-tokens.py** в корневом каталоге проекта и добавить структуру для программы, включая базовую обработку исключений. В следующих разделах показано, как добавить в этот файл весь исходный код для примера из этого краткого руководства.

   ```python
   import os
   from azure.communication.identity import CommunicationIdentityClient, CommunicationUserIdentifier

   try:
      print("Azure Communication Services - Access Tokens Quickstart")
      # Quickstart code goes here
   except Exception as ex:
      print("Exception:")
      print(ex)
   ```

### <a name="install-the-package"></a>Установка пакета

Оставаясь в каталоге приложения, установите пакет SDK для удостоверений Служб коммуникации для пакета Python с помощью команды `pip install`.

```console
pip install azure-communication-identity
```

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр `CommunicationIdentityClient` с использованием строки подключения. Приведенный ниже код извлекает строку подключения для ресурса из переменной среды с именем `COMMUNICATION_SERVICES_CONNECTION_STRING`. См. сведения о том, как [управлять строкой подключения ресурса](../create-communication-resource.md#store-your-connection-string).

Добавьте этот код в блок `try`.

```python
# This code demonstrates how to fetch your connection string
# from an environment variable.
connection_string = os.environ["COMMUNICATION_SERVICES_CONNECTION_STRING"]

# Instantiate the identity client
client = CommunicationIdentityClient.from_connection_string(connection_string)
```

Если же вы настроили управляемое удостоверение, то из статьи [Использование управляемых удостоверений](../managed-identity.md) можете узнать, как использовать его для аутентификации.
```python
endpoint = os.environ["COMMUNICATION_SERVICES_ENDPOINT"]
client = CommunicationIdentityClient(endpoint, DefaultAzureCredential())
```

## <a name="create-an-identity"></a>Создание удостоверения

Службы коммуникации Azure позволяют использовать упрощенный каталог удостоверений. Чтобы создать новую запись в каталоге с уникальным идентификатором (`Id`), используйте метод `create_user`. Сохраните полученное удостоверение с сопоставлением с пользователями вашего приложения. Например, сохраните их в базе данных сервера приложений. Удостоверение потребуется позже для выдачи маркеров доступа.

```python
identity = client.create_user()
print("\nCreated an identity with ID: " + identity.identifier)
```

## <a name="issue-access-tokens"></a>Выпуск маркеров доступа

Чтобы выдать маркер доступа для имеющегося удостоверения Служб коммуникации, используйте метод `get_token`. Параметр `scopes` определяет набор базовых функций, которые будут авторизовать этот маркер доступа. Ознакомьтесь со [списком поддерживаемых действий](../../concepts/authentication.md). Новый экземпляр параметра `CommunicationUserIdentifier` можно создать на основе строкового представления удостоверения Служб коммуникации Azure.

```python
# Issue an access token with the "voip" scope for an identity
token_result = client.get_token(identity, ["voip"])
expires_on = token_result.expires_on.strftime("%d/%m/%y %I:%M %S %p")
print("\nIssued an access token with 'voip' scope that expires at " + expires_on + ":")
print(token_result.token)
```

Маркеры доступа — это недолговечные учетные данные, которые необходимо выдавать повторно. Если этого не сделать, это может привести к нарушению работы пользователей приложения. Свойство ответа `expires_on` указывает время существования маркера доступа.

## <a name="create-an-identity-and-issue-an-access-token-within-the-same-request"></a>Создание удостоверения и выдача маркера доступа в рамках одного запроса

Чтобы создать удостоверение Служб коммуникации и выдать для него маркер доступа, используйте метод `create_user_and_token`. Параметр `scopes` определяет набор базовых функций, которые будут авторизовать этот маркер доступа. Ознакомьтесь со [списком поддерживаемых действий](../../concepts/authentication.md).

```python
# Issue an identity and an access token with the "voip" scope for the new identity
identity_token_result = client.create_user_and_token(["voip"])
identity = identity_token_result[0].identifier
token = identity_token_result[1].token
expires_on = identity_token_result[1].expires_on.strftime("%d/%m/%y %I:%M %S %p")
print("\nCreated an identity with ID: " + identity)
print("\nIssued an access token with 'voip' scope that expires at " + expires_on + ":")
print(token)
```

## <a name="refresh-access-tokens"></a>Обновление токенов доступа

Чтобы обновить маркер доступа, используйте объект `CommunicationUserIdentifier` для повторной выдачи:

```python
# Value existingIdentity represents identity of Azure Communication Services stored during identity creation
identity = CommunicationUserIdentifier(existingIdentity)
token_result = client.get_token( identity, ["voip"])
```

## <a name="revoke-access-tokens"></a>Отмена маркеров доступа

В некоторых случаях вы можете явно отменить маркеры доступа. Например, когда пользователь приложения меняет пароль, который он использует для проверки подлинности в службе. Метод `revoke_tokens` аннулирует все активные маркеры доступа, выданные удостоверению.

```python
client.revoke_tokens(identity)
print("\nSuccessfully revoked all access tokens for identity with ID: " + identity.identifier)
```

## <a name="delete-an-identity"></a>Удаление удостоверения

При удалении удостоверения отзываются все активные маркеры доступа и запрещается выдача последующих маркеров для удостоверения. Вместе с этим удаляется и все хранимое содержимое, связанное с удостоверением.

```python
client.delete_user(identity)
print("\nDeleted the identity with ID: " + identity.identifier)
```

## <a name="run-the-code"></a>Выполнение кода

В окне консоли перейдите в каталог, содержащий файл *issue-access-tokens.py*, а затем выполните команду `python`, чтобы запустить приложение.

```console
python ./issue-access-tokens.py
```
