---
title: Приведите собственную спецификацию ключа — Azure Key Vault | Документация Майкрософт
description: В этом документе описывается, как взять собственную спецификацию ключа.
services: key-vault
author: amitbapat
manager: devtiw
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: conceptual
ms.date: 02/04/2021
ms.author: ambapat
ms.openlocfilehash: 141abea0c0946c98b6dfe627f32f01682a18be44
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99581029"
---
# <a name="bring-your-own-key-specification"></a>Спецификация BYOK

В этом документе описываются спецификации для импорта ключей, защищенных с АППАРАТным модулем безопасности, из локальной HSM клиентов в Key Vault.

## <a name="scenario"></a>Сценарий

Key Vault клиент хочет безопасно передавать ключ из своего локального HSM-модуля, находящегося за пределами Azure, в резервную Azure Key Vault HSM. Процесс импорта ключа, созданного за пределами Key Vault, обычно называется создание собственных ключей (BYOK).

Ниже приведены требования.
* Ключ, который необходимо передать, никогда не существует вне HSM в виде обычного текста.
* За пределами HSM ключ, который необходимо передать, всегда защищен ключом, который хранится в Azure Key Vault HSM.

## <a name="terminology"></a>Терминология

|Имя ключа|Тип ключа|Исходный домен|Описание|
|---|---|---|---|
|Ключ обмена ключами (KEK)|RSA|HSM Azure Key Vault|Пара ключей RSA с защитой HSM, созданная в Azure Key Vault
Перенос ключа|AES|HSM от поставщика|Ключ AES [эфемерного], созданный HSM в локальной системе
Целевой ключ|RSA, EC, AES (только управляемый HSM)|HSM от поставщика|Ключ для передачи в модуль HSM Azure Key Vault

Ключ **обмена ключами**: ключ, поддерживающий аппаратный модуль HSM, созданный клиентом в хранилище ключей, в котором будет ИМПОРТИРОВАН ключ BYOK. Этот KEK должен иметь следующие свойства:

* Это ключ RSA-HSM (4096-разрядная или 3072-разрядная или 2048-bit).
* Он будет иметь фиксированный key_ops (только "import"), который позволит использовать его только во время BYOK
* Должен находиться в том же хранилище, в котором будет импортирован целевой ключ

## <a name="user-steps"></a>Шаги пользователя

Чтобы выполнить перенос ключа, пользователь выполняет следующие действия:

1. Создание KEK.
2. Получите открытый ключ KEK.
3. С помощью средства BYOK, предоставленного поставщиком HSM, импортируйте KEK в целевой модуль HSM и экспортируете целевой ключ, защищенный KEK.
4. Импортируйте защищенный целевой ключ в Azure Key Vault.

Клиенты используют средство BYOK и документацию, предоставляемые поставщиком HSM, для выполнения шагов 3. Он создает большой двоичный объект для обмена ключами (файл byok).


## <a name="hsm-constraints"></a>Ограничения HSM

Существующий HSM может применять ограничения к управляемому им ключу, в том числе:
* Модуль HSM может потребоваться настроить для разрешения экспорта на основе шифрования ключей
* Возможно, необходимо пометить целевой ключ CKA_EXTRACTABLE для HSM, чтобы разрешить контролируемый экспорт
* В некоторых случаях KEK и ключ переноса могут помечаться как CKA_TRUSTED. Это позволяет использовать его для переноса ключей в HSM.

Конфигурация исходного АППАРАТного модуля безопасности обычно выходит за рамки данной спецификации. Корпорация Майкрософт предполагает, что поставщик HSM создает документацию, сопровождающую их средство BYOK, для включения любых таких шагов настройки.

> [!NOTE]
> Шаги 1, 2 и 4, описанные ниже, можно выполнить с помощью других интерфейсов, таких как Azure PowerShell и портале Azure. Их также можно выполнять программно с помощью эквивалентных функций в Key Vault SDK.

### <a name="step-1-generate-kek"></a>Шаг 1. Создание KEK

Используйте команду **AZ keyvault Key Create** , чтобы создать KEK с операциями Key, для которых задано значение Import. Запишите идентификатор ключа "ребенок", возвращенный из команды ниже.

```azurecli
az keyvault key create --kty RSA-HSM --size 4096 --name KEKforBYOK --ops import --vault-name ContosoKeyVaultHSM
```

### <a name="step-2-retrieve-the-public-key-of-the-kek"></a>Шаг 2. Получение открытого ключа KEK

Скачайте открытый ключ KEK и сохраните его в PEM-файле.

```azurecli
az keyvault key download --name KEKforBYOK --vault-name ContosoKeyVaultHSM --file KEKforBYOK.publickey.pem
```

### <a name="steps-3-generate-key-transfer-blob-using-hsm-vendor-provided-byok-tool"></a>Шаги 3. Создание большого двоичного объекта для обмена ключами с помощью средства BYOK, предоставленного поставщиком HSM

Клиент будет использовать средство BYOK, предоставляемое поставщиком HSM, для создания большого двоичного объекта для обмена ключами (хранящегося в файле ". BYOK"). KEK открытый ключ (в виде PEM-файла) будет одним из входных данных этого средства.

#### <a name="key-transfer-blob"></a>Большой двоичный объект для обмена ключами
В долгосрочной перспективе Майкрософт хочет использовать механизм CKM_RSA_AES_KEY_WRAP PKCS # 11 для перемещения целевого ключа в Azure Key Vault поскольку этот механизм создает один большой двоичный объект и, что более важно, промежуточный ключ AES обрабатывается двумя HSM и гарантированно является временным. Этот механизм сейчас недоступен в некоторых HSM, но сочетание защиты целевого ключа с CKM_AES_KEY_WRAP_PAD помощью ключа AES и защиты ключа AES с CKM_RSA_PKCS_OAEP создает эквивалентный большой двоичный объект.

Открытый текст целевого ключа зависит от типа ключа: 
* Для ключа RSA закрытый ключ ASN. 1 DER Encoding [RFC3447] упакован в PKCS # 8 [RFC5208] 
* Для ключа EC закрытый ключ ASN. 1 DER Encoding [RFC5915] упакован в PKCS # 8 [RFC5208]
* Для ключа октета необработанные байты ключа

Байты для ключа в виде открытого текста преобразуются с помощью механизма CKM_RSA_AES_KEY_WRAP:
* Создается временный ключ AES, который шифруется с помощью инкапсуляции ключа RSA с использованием RSA-OAEP с SHA1.
* Зашифрованный открытый ключ шифруется с помощью ключа AES с использованием ключа AES с заполнением.
* Зашифрованный ключ AES и зашифрованный открытый ключ объединяются для создания последнего зашифрованного BLOB-объекта.

Формат большого двоичного объекта передачи данных использует сериализацию JSON Web Encryption Compact (RFC7516) в основном как транспортное средство для доставки необходимых метаданных в службу для правильной расшифровки.

Если используется CKM_RSA_AES_KEY_WRAP_PAD, сериализация JSON BLOB-объекта перемещения будет выглядеть так:

```json
{
  "schema_version": "1.0.0",
  "header":
  {
    "kid": "<key identifier of the KEK>",
    "alg": "dir",
    "enc": "CKM_RSA_AES_KEY_WRAP"
  },
  "ciphertext":"BASE64URL(<ciphertext contents>)",
  "generator": "BYOK tool name and version; source HSM name and firmware version"
}

```

* Детская = идентификатор ключа KEK. Для Key Vaultных ключей он выглядит следующим образом: https://ContosoKeyVaultHSM.vault.azure.net/keys/mykek/eba63d27e4e34e028839b53fac905621
* ALG = алгоритм. 
* dir = прямой режим, т. е. для непосредственной защиты зашифрованного текста, который является точным представлением CKM_RSA_AES_KEY_WRAP
* Generator — информационное поле, определяющее имя и версию средства BYOK, а также изготовителя и модель исходного модуля HSM. Эти сведения предназначены для использования при устранении неполадок и поддержке.

Большой двоичный объект JSON хранится в файле с расширением byok, чтобы клиенты Azure PowerShell/CLI правильно обрабатывают его при использовании команд "Add-Азкэйваулткэй" (КОМАНДНОМ PSH) или "az keyvault Key import" (CLI).

### <a name="step-4-upload-key-transfer-blob-to-import-hsm-key"></a>Шаг 4. Отправка большого двоичного объекта передачи ключа для импорта HSM-ключа

Клиент перейдет по ключу (файл с расширением byok) на рабочую станцию, а затем выполните команду **AZ keyvault Key Import** , чтобы импортировать этот большой двоичный объект в качестве нового ключа, поддерживающего HSM, в Key Vault. 

Чтобы импортировать ключ RSA, используйте следующую команду:
```azurecli
az keyvault key import --vault-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file KeyTransferPackage-ContosoFirstHSMkey.byok --ops encrypt decrypt
```
Чтобы импортировать ключ EC, нужно указать тип ключа и имя кривой.

```azurecli
az keyvault key import --vault-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file --kty EC-HSM --curve-name "P-256" KeyTransferPackage-ContosoFirstHSMkey.byok --ops sign verify
```


При выполнении приведенной выше команды она приводит к отправке запроса REST API следующим образом:

```
PUT https://contosokeyvaulthsm.vault.azure.net/keys/ContosoFirstHSMKey?api-version=7.0
```

Текст запроса при импорте ключа RSA:
```json
{
  "key": {
    "kty": "RSA-HSM",
    "key_ops": [
      "decrypt",
      "encrypt"
    ],
    "key_hsm": "<Base64 encoded BYOK_BLOB>"
  },
  "attributes": {
    "enabled": true
  }
}
```

Текст запроса при импорте ключа EC:
```json
{
  "key": {
    "kty": "EC-HSM",
    "crv": "P-256",
    "key_ops": [
      "sign",
      "verify"
    ],
    "key_hsm": "<Base64 encoded BYOK_BLOB>"
  },
  "attributes": {
    "enabled": true
  }
}
```

значение "key_hsm" — это все содержимое KeyTransferPackage-ContosoFirstHSMkey. byok, закодированное в формате Base64.

## <a name="references"></a>Ссылки
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)

## <a name="next-steps"></a>Дальнейшие действия
* Пошаговые инструкции BYOK: [Импорт ключей, защищенных с помощью HSM, в Key Vault (BYOK)](hsm-protected-keys-byok.md)

