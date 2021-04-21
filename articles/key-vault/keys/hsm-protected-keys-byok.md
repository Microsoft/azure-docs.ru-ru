---
title: Создание ключей, защищенных модулем HSM, и их передача — создание собственных ключей — Azure Key Vault
description: Данная статья поможет вам подготовить и создать ваши собственные ключи, защищенные модулем HSM, и передать их в Azure Key Vault. Этот подход также известен как создание собственных ключей (BYOK).
services: key-vault
author: amitbapat
manager: devtiw
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: tutorial
ms.date: 02/04/2021
ms.author: ambapat
ms.openlocfilehash: f1b5d6499594e9026e1615be5361c52c9ce2f4ef
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774815"
---
# <a name="import-hsm-protected-keys-to-key-vault-byok"></a>Импорт защищенных модулем HSM ключей в Key Vault (BYOK)

Чтобы обеспечить более высокий уровень защиты при работе с Azure Key Vault, можно импортировать ключи или создать их в аппаратных модулях безопасности (HSM). Ключи никогда не покидают их пределы. Этот сценарий обычно называется *созданием собственных ключей* (BYOK). Для защиты ключей Key Vault использует семейство модулей HSM nCipher nShield (проверенных по стандарту FIPS 140-2 уровня 2).

Информация в этой статье поможет вам подготовить и создать ваши собственные ключи, защищенные модулем HSM, и передать их в Azure Key Vault.

> [!NOTE]
> Эта функция недоступна в Azure для Китая (21Vianet). 
> 
> Этот метод импорта доступен только для [поддерживаемых модулей HSM](#supported-hsms). 

Дополнительные сведения и руководство по началу работы с Key Vault (в том числе по созданию хранилища ключей для ключей, защищенных модулем HSM) приведены в статье [Об Azure Key Vault](../general/overview.md).

## <a name="overview"></a>Обзор

Ниже представлен обзор этого процесса. Конкретные действия, которые необходимо для этого выполнить, описаны далее в этой статье.

* В Key Vault создайте ключ (который называется *ключом обмена ключами* (KEK)). KEK должен быть ключом RSA-HSM, с которым можно выполнять только операцию `import`. Ключи RSA-HSM поддерживает только Key Vault с номером SKU ценовой категории "Премиум".
* Скачайте открытый ключ KEK в виде PEM-файла.
* Перенесите открытый ключ KEK на автономный компьютер, подключенный к локальному модулю HSM.
* На автономном компьютере используйте средство BYOK, предоставленное поставщиком HSM, для создания файла BYOK. 
* Целевой ключ шифруется с помощью KEK и остается зашифрованным до тех пор, пока не будет передан в модуль HSM Key Vault. Локальный модуль HSM покидает только зашифрованная версия ключа.
* Ключ KEK, созданный в HSM Key Vault, экспортировать невозможно. HSM принудительно применяет правило, согласно которому за пределами HSM Key Vault не должно быть незашифрованной версии KEK.
* Ключ KEK должен находиться в том же хранилище ключей, в которое будет импортирован целевой ключ.
* Когда файл BYOK отправляется в Key Vault, HSM Key Vault использует закрытый ключ KEK для расшифровки материала целевого ключа и его импорта в качестве ключа HSM. Эта операция выполняется полностью в HSM Key Vault. Целевой ключ всегда остается под защитой HSM.

## <a name="prerequisites"></a>Предварительные требования

В следующей таблице перечислены предварительные требования для использования BYOK в Azure Key Vault.

| Требование | Дополнительные сведения |
| --- | --- |
| Подписка Azure |Для создания хранилища ключей в Azure Key Vault требуется подписка Azure. [Зарегистрируйтесь для получения бесплатной пробной версии](https://azure.microsoft.com/pricing/free-trial/). |
| Key Vault с номером SKU ценовой категории "Премиум" для импорта ключей, защищенных модулем HSM |Дополнительные сведения об уровнях служб и возможностях Azure Key Vault см. на странице [Цены на Key Vault](https://azure.microsoft.com/pricing/details/key-vault/). |
| HSM из списка поддерживаемых HSM, средство BYOK и инструкции, предоставленные поставщиком HSM | Необходимо иметь разрешения для модуля HSM и базовые знания о том, как им пользоваться. Просмотрите список [поддерживаемых модулей HSM](#supported-hsms). |
| Azure CLI версии 2.1.0 или более поздней | Подробнее см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).|

## <a name="supported-hsms"></a>Поддерживаемые модули HSM

|Имя поставщика|Тип поставщика|Поддерживаемые модели модуля HSM|Дополнительные сведения|
|---|---|---|---|
|nCipher|Производитель,<br/>HSM как услуга|<ul><li>Семейство модулей HSM nShield</li><li>nShield как услуга</ul>|[Новое средство BYOK и документация для nCipher](https://www.ncipher.com/products/key-management/cloud-microsoft-azure)|
|Thales|Изготовитель|<ul><li>Семейство HSM 7 Luna со встроенным ПО версии 7.3 или более поздней</li></ul>| [Новое средство BYOK и документация для Luna](https://supportportal.thalesgroup.com/csm?id=kb_article_view&sys_kb_id=3892db6ddb8fc45005c9143b0b961987&sysparm_article=KB0021016)|
|Fortanix|Производитель,<br/>HSM как услуга|<ul><li>Служба управления ключами для самостоятельной защиты (SDKMS)</li><li>Equinix SmartKey</li></ul>|[Экспорт ключей SDKMS в поставщики облачных служб для BYOK — Azure Key Vault](https://support.fortanix.com/hc/en-us/articles/360040071192-Exporting-SDKMS-keys-to-Cloud-Providers-for-BYOK-Azure-Key-Vault)|
|Marvell|Изготовитель|Все модули HSM LiquidSecurity со<ul><li>встроенным ПО версии 2.0.4 или более поздней</li><li>встроенным ПО версии 3.2 или более новой</li></ul>|[Новое средство BYOK и документация для Marvell](https://www.marvell.com/products/security-solutions/nitrox-hs-adapters/exporting-marvell-hsm-keys-to-cloud-azure-key-vault.html)|
|Cryptomathic|ISV (Enterprise Key Management System)|Несколько торговых марок и моделей HSM, в том числе<ul><li>nCipher</li><li>Thales</li><li>Utimaco</li></ul>Дополнительные сведения см. на [сайте Cryptomathic](https://www.cryptomathic.com/azurebyok).|[Новое средство BYOK и документация для Cryptomathic](https://www.cryptomathic.com/azurebyok)|
|Securosys SA|Производитель,<br/>HSM как услуга|Семейство Primus HSM, Securosys Clouds HSM|[Новое средство BYOK и документация для Primus](https://www.securosys.com/primus-azure-byok)|
|StorMagic|ISV (Enterprise Key Management System)|Несколько торговых марок и моделей HSM, в том числе<ul><li>Utimaco</li><li>Thales</li><li>nCipher</li></ul>Дополнительные сведения см. на [сайте StorMagic](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm).|[SvKMS и Azure Key Vault (BYOK)](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm)|
|IBM|Изготовитель|IBM 476x, CryptoExpress|[IBM Enterprise Key Management Foundation](https://www.ibm.com/security/key-management/ekmf-bring-your-own-key-azure)|
|Utimaco|Производитель,<br/>HSM как услуга|u.trust Anchor, CryptoServer|[Инструмент Utimaco BYOK и руководство по интеграции](https://support.hsm.utimaco.com/support/downloads/byok)|
||||


## <a name="supported-key-types"></a>Поддерживаемые типы ключей

|Имя раздела|Тип ключа|Размер ключа (кривая)|Исходный домен|Описание|
|---|---|---|---|---|
|Ключ обмена ключами (KEK)|RSA| 2048-разрядный<br />3072-разрядный<br />4096-разрядный|HSM Azure Key Vault|Защищенная HSM пара ключей RSA, созданная в Azure Key Vault|
|Целевой ключ|
||RSA|2048-разрядный<br />3072-разрядный<br />4096-разрядный|HSM от поставщика|Ключ для передачи в модуль HSM Azure Key Vault|
||EC|P-256<br />P-384<br />P-521|HSM от поставщика|Ключ для передачи в модуль HSM Azure Key Vault|
||||

## <a name="generate-and-transfer-your-key-to-the-key-vault-hsm"></a>Создание ключа и его передача в модуль HSM Key Vault

Чтобы создать ключ и передать его в модуль HSM Key Vault, выполните приведенные ниже действия.

* [Шаг 1. Создание KEK](#step-1-generate-a-kek)
* [Шаг 2. Скачивание открытого ключа KEK](#step-2-download-the-kek-public-key)
* [Шаг 3. Создание ключа и подготовка его к передаче](#step-3-generate-and-prepare-your-key-for-transfer)
* [Шаг 4. Передача ключа в Azure Key Vault](#step-4-transfer-your-key-to-azure-key-vault)

### <a name="step-1-generate-a-kek"></a>Шаг 1. Создание KEK

KEK — это ключ RSA, который создается в HSM Key Vault. KEK используется для шифрования ключа, который необходимо импортировать (*целевого* ключа).

Ключ KEK должен:
- быть ключом RSA-HSM (2048-, 3072- или 4096-разрядным);
- создаваться в том же хранилище ключей, в которое планируется импортировать целевой ключ;
- поддерживать только операцию `import`.

> [!NOTE]
> Единственной допустимой операцией с ключом KEK должна быть операция "import", которая является взаимоисключающей с любыми другими операциями с ключами.

С помощью команды [az keyvault key create](/cli/azure/keyvault/key#az_keyvault_key_create) создайте ключ KEK, для которого в качестве операции с ключами задана операция `import`. Запишите идентификатор ключа (`kid`), возвращаемый следующей командой. (Вы будете использовать значение `kid` на [шаге 3](#step-3-generate-and-prepare-your-key-for-transfer).)

```azurecli
az keyvault key create --kty RSA-HSM --size 4096 --name KEKforBYOK --ops import --vault-name ContosoKeyVaultHSM
```

### <a name="step-2-download-the-kek-public-key"></a>Шаг 2. Скачивание открытого ключа KEK

Чтобы скачать открытый ключ KEK в PEM-файл, используйте команду [az keyvault key download](/cli/azure/keyvault/key#az_keyvault_key_download). Импортируемый целевой ключ шифруется с помощью открытого ключа KEK.

```azurecli
az keyvault key download --name KEKforBYOK --vault-name ContosoKeyVaultHSM --file KEKforBYOK.publickey.pem
```

Перенесите файл KEKforBYOK.publickey.pem на автономный компьютер. Он потребуется для выполнения следующего шага.

### <a name="step-3-generate-and-prepare-your-key-for-transfer"></a>Шаг 3. Создание ключа и подготовка его к передаче

Сведения о том, как скачать и установить средство BYOK, см. в документации поставщика HSM. Следуйте инструкциям поставщика HSM, чтобы создать целевой ключ, а затем создайте пакет для перемещения ключа (файл BYOK). Средство BYOK будет использовать значение `kid`, полученное на [шаге 1](#step-1-generate-a-kek), и файл KEKforBYOK.publickey.pem, скачанный на [шаге 2](#step-2-download-the-kek-public-key), для создания зашифрованного целевого ключа в файле BYOK.

Перенесите файл BYOK на подключенный компьютер.

> [!NOTE] 
> Импорт 1024-разрядных ключей RSA не поддерживается. Импорт ключа с эллиптической кривой с P-256K не поддерживается.
> 
> **Известная проблема**. Импорт целевого 4096-разрядного ключа RSA из модулей HSM Luna поддерживается только при использовании встроенного ПО версии 7.4.0 или более поздней.

### <a name="step-4-transfer-your-key-to-azure-key-vault"></a>Шаг 4. Передача ключа в Azure Key Vault

Чтобы завершить импорт ключа, перенесите пакет обмена ключами (файл BYOK) с отключенного компьютера на компьютер, подключенный к Интернету. Чтобы передать файл BYOK в HSM Key Vault, используйте команду [az keyvault key import](/cli/azure/keyvault/key#az_keyvault_key_import).

Чтобы импортировать ключ RSA, используйте приведенную ниже команду. Параметр --kty является необязательным и по умолчанию имеет значение RSA-HSM.
```azurecli
az keyvault key import --vault-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file KeyTransferPackage-ContosoFirstHSMkey.byok
```

Чтобы импортировать ключ EC, нужно указать тип ключа и имя кривой.

```azurecli
az keyvault key import --vault-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file --kty EC-HSM --curve-name "P-256" KeyTransferPackage-ContosoFirstHSMkey.byok
```

Если отправка прошла успешно, в Azure CLI отобразятся свойства импортированного ключа.

## <a name="next-steps"></a>Дальнейшие действия

Теперь ключ, защищенный с помощью аппаратного модуля безопасности, можно использовать в хранилище ключей. Дополнительные сведения см. на [этой странице цен и сравнения характеристик](https://azure.microsoft.com/pricing/details/key-vault/).
