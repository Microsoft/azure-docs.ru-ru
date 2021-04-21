---
title: Мониторинг и администрирование процесса создания сертификатов
description: Сценарии, демонстрирующие ряд возможностей для создания, мониторинга и взаимодействия с процессом создания сертификата с помощью Key Vault.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: mbaldwin
ms.openlocfilehash: 8e4acb5195497dd31f466829b1cde301ba9696b3
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107751100"
---
# <a name="monitor-and-manage-certificate-creation"></a>Мониторинг и администрирование процесса создания сертификатов
Область применения: Azure

следующие сценарии и операции:

- Запрос сертификата Key Vault с помощью поддерживаемого издателя
- Получение ожидающего запроса (состояние запроса "Выполняется").
- Получение ожидающего запроса (состояние запроса "Готово").
- Получение ожидающих запросов (состояние ожидающего запроса "Отменено" или "Сбой").
- Получение ожидающих запросов (состояние ожидающего запроса "Удалено" или "Перезаписано").
- Создание (или импорт) при имеющемся ожидающем запросе (состояние "Выполняется").
- Слияние при создании ожидающего выполнения запроса с помощью издателя (например, DigiCert).
- Запрос отмены при состоянии ожидающего запроса "Выполняется".
- Удаление ожидающего объекта запроса.
- Создание сертификата Key Vault вручную.
- Слияние при создании ожидающего запроса (создание сертификата вручную).

## <a name="request-a-kv-certificate-with-a-supported-issuer"></a>Запрос сертификата Key Vault с помощью поддерживаемого издателя 

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

Для следующего примера необходимо, чтобы объект с именем mydigicert уже был доступен в хранилище ключей с поставщиком издателя DigiCert. Издатель сертификата — это сущность, представленная в Azure Key Vault (KV) как ресурс CertificateIssuer. Она используется для предоставления сведений об источнике сертификата KV, имени издателя, поставщика, учетных данных и других административных сведений.

### <a name="request"></a>Запрос

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "mydigicert",
      "cty": "OV-SSL",
    }
  }
}
```

### <a name="response"></a>Ответ

```
StatusCode: 202, ReasonPhrase: 'Accepted'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "mydigicert"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "InProgress",
  "status_details": "Pending certificate created. Certificate request is in progress. This may take some time based on the issuer provider. Please check again later",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="get-pending-request---request-status-is-inprogress"></a>Получение ожидающего запроса (состояние запроса "Выполняется").

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Запрос
GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

> [!NOTE]
> Если *request_id* указан в запросе, он действует как фильтр. Если *request_id* в запросе и ожидающем объекте различается, возвращается код состояния HTTP 404.

### <a name="response"></a>Ответ

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "inProgress",
  "status_details": "…",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="get-pending-request---request-status-is-complete"></a>Получение ожидающего запроса (состояние запроса "Готово").

### <a name="request"></a>Запрос

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>Ответ

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "completed",
  "request_id": "a76827a18b63421c917da80f28e9913d",
  "target": “https://mykeyvault.vault.azure.net/certificates/mycert1?api-version={api-version}"
}

```

## <a name="get-pending-request---pending-request-status-is-canceled-or-failed"></a>Получение ожидающих запросов (состояние ожидающего запроса "Отменено" или "Сбой").

### <a name="request"></a>Запрос

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>Ответ

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "failed",
  "status_details": "",
  "request_id": "a76827a18b63421c917da80f28e9913d",
  "error": {
    "code": "<errorcode>",
    "message": "<message>"
  }
}

```

> [!NOTE]
> Значение *errorcode* может быть следующим: Certificate issuer error (Ошибка, связанная с издателем сертификата) или "Запрос отклонен". Это зависит от ошибки издателя или пользователя соответственно.

## <a name="get-pending-request---pending-request-status-is-deleted-or-overwritten"></a>Получение ожидающих запросов (состояние ожидающего запроса "Удалено" или "Перезаписано").
Если ожидающий объект не находится в состоянии "Выполняется", он может быть удален или перезаписан при выполнении операции создания или импорта.

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Запрос
GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>Ответ

```
StatusCode: 404, ReasonPhrase: 'Not Found'
{
  "error": {
    "code": "PendingCertificateNotFound",
    "message": "…"
  }
}

```

## <a name="create-or-import-when-pending-request-exists---status-is-inprogress"></a>Создание (или импорт) при имеющемся ожидающем запросе (состояние "Выполняется").
Ожидающий объект может находиться в одном из четырех возможных состояний: "Выполняется", "Отменено", "Сбой" или "Готово".

Если состояние ожидающего запроса — "Выполняется", операции создания и импорта завершаются ошибкой с кодом состояния HTTP 409 (конфликт).

Чтобы устранить конфликт, сделайте следующее.

- Если сертификат создается вручную, можно завершить создание, выполнив слияние или удалив ожидающий объект.

- Если сертификат создается с помощью издателя, можно подождать, пока сертификат создастся или же произойдет сбой либо отмена операции. Кроме того, можно удалить ожидающий объект.

> [!NOTE]
> Удаление ожидающего объекта может отменить запрос сертификата x509 с помощью поставщика.

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

### <a name="request"></a>Запрос

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "mydigicert"
    }
  }
}
```

### <a name="response"></a>Ответ

```
StatusCode: 409, ReasonPhrase: 'Conflict'
{
  "error": {
    "code": "Forbidden",
    "message": "A new key vault certificate can not be created or imported while a pending key vault certificate's status is inProgress."
  }
}

```

## <a name="merge-when-pending-request-is-created-with-an-issuer"></a>Слияние при создании ожидающего выполнения запроса с помощью издателя
Слияние запрещено, если ожидающий объект создан с помощью издателя, но разрешено, если он находится в состоянии "Выполняется". 

Если запрос на создание сертификата x509 завершается сбоем или отменяется по некоторым причинам и если сертификат x509 можно получить другим способом, для завершения создания сертификата Key Vault можно выполнить операцию слияния.

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending/merge?api-version={api-version}`|

### <a name="request"></a>Запрос

```json
{
  "x5c": [ "MIICxTCCAbi………………………trimmed for brevitiy……………………………………………EPAQj8=" ]
}

```

### <a name="response"></a>Ответ

```json
StatusCode: 403, ReasonPhrase: 'Forbidden'
{
  "error": {
    "code": "Forbidden",
    "message": "Merge is forbidden on pending object created with issuer : <issuer-name> while it is in progess."
  }
}

```

## <a name="request-a-cancellation-while-the-pending-request-status-is-inprogress"></a>Запрос отмены при состоянии ожидающего запроса "Выполняется".
Отмену можно только запросить. При необходимости запрос можно отменить. Если запрос не выполняется, возвращается код состояния HTTP 400 (недопустимый запрос).

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|PATCH|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Запрос
PATCH `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

PATCH `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

```json
{
  "cancellation_requested": true
}

```

### <a name="response"></a>Ответ

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": true,
  "status": "inProgress",
  "status_details": "…",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}
```

## <a name="delete-a-pending-request-object"></a>Удаление ожидающего объекта запроса.

> [!NOTE]
> Удаление ожидающего объекта может отменить запрос сертификата x509 с помощью поставщика.

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|DELETE|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Запрос
DELETE `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

DELETE `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>Ответ

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "inProgress",
  "request_id": "a76827a18b63421c917da80f28e9913d",
}
```

## <a name="create-a-kv-certificate-manually"></a>Создание сертификата Key Vault вручную.
Вы можете вручную создать сертификат, выданный в центре сертификации. В качестве имени издателя укажите Unknown либо оставьте поле издателя пустым.

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

### <a name="request"></a>Запрос

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "Unknown"
    }
  }
}

```

### <a name="response"></a>Ответ

```
StatusCode: 202, ReasonPhrase: 'Accepted'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "Unknown"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "status": "inProgress",
  "status_details": "Pending certificate created. Please Perform Merge to complete the request.",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="merge-when-a-pending-request-is-created---manual-certificate-creation"></a>Слияние при создании ожидающего запроса (создание сертификата вручную).

|Метод|Универсальный код ресурса (URI) запроса|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending/merge?api-version={api-version}`|

### <a name="request"></a>Запрос

```json
{
  "x5c": [ "MIICxTCCAbi………………………trimmed for brevitiy……………………………………………EPAQj8=" ]
}

```

|Имя элемента|Обязательно|Тип|Версия|Описание|
|------------------|--------------|----------|-------------|-----------------|
|x5c|Да|массиве|\<introducing version>|Цепочка сертификатов X509 как массив строк в кодировке Base 64.|

### <a name="response"></a>Ответ

```
StatusCode: 201, ReasonPhrase: 'Created'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1?api-version={api-version}"
{
    "id": "https mykeyvault.vault.azure.net/certificates/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "kid": "https:// mykeyvault.vault.azure.net/keys/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "sid": " mykeyvault.vault.azure.net/secrets/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "cer": "……de34534……",
    "x5t": "n14q2wbvyXr71Pcb58NivuiwJKk",
    "attributes": {
        "enabled": true,
        "exp": 1530394215,
        "nbf": 1435699215,
        "created": 1435699919,
        "updated": 1435699919
    },
    "pending": {
        "id": "https:// mykeyvault.vault.azure.net/certificates/mycert1/pending"
    },
    "policy": {
        "id": "https:// mykeyvault.vault.azure.net/certificates/mycert1/policy",
        "key_props": {
            "exportable": false,
            "kty": "RSA",
            "key_size": 2048,
            "reuse_key": false
        },
        "secret_props": {
            "contentType": "application/x-pkcs12"
        },
        "x509_props": {
            "subject": "CN=Mycert1",
            "ekus": ["1.3.6.1.5.5.7.3.1", "1.3.6.1.5.5.7.3.2"],
            "validity_months": 12
        },
        "lifetime_actions": [{
            "trigger": {
                "lifetime_percentage": 80
            },
            "action": {
                "action_type": "EmailContacts"
            }
        }],
        "issuer": {
            "name": "Unknown"
        },
        "attributes": {
            "enabled": true,
            "created": 1435699811,
            "updated": 1435699811
        }
    }
}

```
