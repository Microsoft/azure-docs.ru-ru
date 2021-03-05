---
title: Создание и перенос ключей, защищенных АППАРАТным модулем безопасности, для Azure Key Vault управляемого HSM-Azure Key Vault | Документация Майкрософт
description: В этой статье содержатся сведения о планировании, создании и переносе собственных ключей, защищенных АППАРАТным модулем безопасности, для использования с управляемым HSM. Этот подход также известен как создание собственных ключей (BYOK).
services: key-vault
author: amitbapat
tags: azure-resource-manager
ms.service: key-vault
ms.topic: conceptual
ms.date: 02/04/2021
ms.author: ambapat
ms.openlocfilehash: dd5b38a858ceba12f5d48f1782da5b85228c4b06
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102212116"
---
# <a name="import-hsm-protected-keys-to-managed-hsm-byok"></a>Импорт ключей, защищенных с АППАРАТным модулем безопасности, в управляемый HSM (BYOK)

 Azure Key Vault управляемый модуль HSM поддерживает импорт ключей, созданных в локальном аппаратном модуле безопасности (HSM); ключи никогда не оставляют границы защиты HSM. Этот сценарий обычно называется *созданием собственных ключей* (BYOK). Управляемый модуль HSM использует адаптеры HSM Ликуидсекурити (FIPS 140-2 с проверкой уровня 3) для защиты ключей.

Используйте сведения в этой статье для планирования, создания и перемещения собственных ключей, защищенных АППАРАТным модулем безопасности, для использования с управляемым HSM.

> [!NOTE]
> Эта функция недоступна в Azure для Китая (21Vianet). Этот метод импорта доступен только для [поддерживаемых модулей HSM](#supported-hsms). 

Дополнительные сведения и руководство по началу работы с управляемым модулем HSM см. в статье [что такое управляемый HSM?](overview.md).

## <a name="overview"></a>Обзор

Ниже представлен обзор этого процесса. Конкретные действия, которые необходимо для этого выполнить, описаны далее в этой статье.

* В управляемом модуле HSM создайте ключ (который называется *ключом обмена ключами* (KEK)). KEK должен быть ключом RSA-HSM, с которым можно выполнять только операцию `import`. 
* Скачайте открытый ключ KEK в виде PEM-файла.
* Перенесите открытый ключ KEK на автономный компьютер, подключенный к локальному модулю HSM.
* На автономном компьютере используйте средство BYOK, предоставленное поставщиком HSM, для создания файла BYOK. 
* Целевой ключ шифруется с помощью KEK, который остается зашифрованным до тех пор, пока он не будет передан в управляемый HSM-модуль. Локальный модуль HSM покидает только зашифрованная версия ключа.
* KEK, созданный внутри управляемого модуля HSM, не может быть экспортирован. HSM принудительно применяет правило, не имеющее четкой версии KEK, за пределами управляемого HSM.
* KEK должен находиться в том же управляемом модуле HSM, где будет импортирован целевой ключ.
* Когда файл BYOK отправляется в управляемый модуль HSM, управляемый модуль HSM использует закрытый ключ KEK для расшифровки материала целевого ключа и его импорта в качестве ключа HSM. Эта операция выполняется полностью внутри HSM. Целевой ключ всегда остается под защитой HSM.


## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать команды Azure CLI из этой строки, вам необходимо следующее:

* подписка на Microsoft Azure. Если у вас ее нет, зарегистрируйтесь, чтобы воспользоваться [бесплатной пробной версией](https://azure.microsoft.com/pricing/free-trial).
* Azure CLI 2.12.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).
* Управляемый модуль HSM — [поддерживаемый список HSM](#supported-hsms) в вашей подписке. См. [Краткое руководство. Подготовка и активация управляемого устройства HSM с помощью Azure CLI](quick-create-cli.md) для выполнения соответствующих действий.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Чтобы войти в Azure с помощью CLI, введите следующее:

```azurecli
az login
```

Дополнительные сведения о параметрах входа с помощью интерфейса командной строки см. в статье [Вход с помощью Azure CLI](/cli/azure/authenticate-azure-cli).

## <a name="supported-hsms"></a>Поддерживаемые модули HSM

|Имя поставщика|Тип поставщика|Поддерживаемые модели модуля HSM|Дополнительные сведения|
|---|---|---|---|
|nCipher|Производитель,<br/>HSM как услуга|<ul><li>Семейство модулей HSM nShield</li><li>nShield как услуга</ul>|[Новое средство BYOK и документация для nCipher](https://www.ncipher.com/products/key-management/cloud-microsoft-azure)|
|Thales|Изготовитель|<ul><li>Семейство HSM 7 Luna со встроенным ПО версии 7.3 или более поздней</li></ul>| [Новое средство BYOK и документация для Luna](https://supportportal.thalesgroup.com/csm?id=kb_article_view&sys_kb_id=3892db6ddb8fc45005c9143b0b961987&sysparm_article=KB0021016)|
|Fortanix|Производитель,<br/>HSM как услуга|<ul><li>Служба управления ключами для самостоятельной защиты (SDKMS)</li><li>Equinix SmartKey</li></ul>|[Экспорт ключей SDKMS в поставщики облачных служб для BYOK — Azure Key Vault](https://support.fortanix.com/hc/en-us/articles/360040071192-Exporting-SDKMS-keys-to-Cloud-Providers-for-BYOK-Azure-Key-Vault)|
|Marvell|Изготовитель|Все модули HSM LiquidSecurity со<ul><li>встроенным ПО версии 2.0.4 или более поздней</li><li>встроенным ПО версии 3.2 или более новой</li></ul>|[Новое средство BYOK и документация для Marvell](https://www.marvell.com/products/security-solutions/nitrox-hs-adapters/exporting-marvell-hsm-keys-to-cloud-azure-key-vault.html)|
|Cryptomathic|ISV (Enterprise Key Management System)|Несколько торговых марок и моделей HSM, в том числе<ul><li>nCipher</li><li>Thales</li><li>Utimaco</li></ul>Дополнительные сведения см. на [сайте Cryptomathic](https://www.cryptomathic.com/azurebyok).|[Новое средство BYOK и документация для Cryptomathic](https://www.cryptomathic.com/azurebyok)|
|Securosys SA|Производитель, HSM как услуга|Семейство Primus HSM, Securosys Clouds HSM|[Новое средство BYOK и документация для Primus](https://www.securosys.com/primus-azure-byok)|
|StorMagic|ISV (Enterprise Key Management System)|Несколько торговых марок и моделей HSM, в том числе<ul><li>Utimaco</li><li>Thales</li><li>nCipher</li></ul>Дополнительные сведения см. на [сайте StorMagic](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm).|[SvKMS и Azure Key Vault (BYOK)](https://stormagic.com/doc/svkms/Content/Integrations/Azure_KeyVault_BYOK.htm)|
|IBM|Изготовитель|IBM 476x, CryptoExpress|[IBM Enterprise Key Management Foundation](https://www.ibm.com/security/key-management/ekmf-bring-your-own-key-azure)|
|Utimaco|Производитель,<br/>HSM как услуга|u.trust Anchor, CryptoServer|[Инструмент Utimaco BYOK и руководство по интеграции](https://support.hsm.utimaco.com/support/downloads/byok)|
||||


## <a name="supported-key-types"></a>Поддерживаемые типы ключей

|Имя раздела|Тип ключа|Размер ключа (кривая)|Исходный домен|Описание|
|---|---|---|---|---|
|Ключ обмена ключами (KEK)|RSA| 2048-разрядный<br />3072-разрядный<br />4096-разрядный|Управляемое устройство HSM|Пара ключей RSA с АППАРАТным модулем безопасности, созданная в управляемом HSM|
|Целевой ключ|
||RSA|2048-разрядный<br />3072-разрядный<br />4096-разрядный|HSM от поставщика|Ключ для передачи в управляемый модуль HSM|
||EC|P-256<br />P-384<br />P-521|HSM от поставщика|Ключ для передачи в управляемый модуль HSM|
||Симметричный ключ (Oct-HSM)|128-разрядное<br />192-разрядный<br />256-разрядный|HSM от поставщика|Ключ для передачи в управляемый модуль HSM|
||||
## <a name="generate-and-transfer-your-key-to-the-managed-hsm"></a>Создание и перенос ключа в управляемый модуль HSM

Чтобы создать и переместить ключ в управляемый модуль HSM, выполните следующие действия.

* [Шаг 1. Создание KEK](#step-1-generate-a-kek)
* [Шаг 2. Скачивание открытого ключа KEK](#step-2-download-the-kek-public-key)
* [Шаг 3. Создание ключа и подготовка его к передаче](#step-3-generate-and-prepare-your-key-for-transfer)
* [Шаг 4. Перенос ключа в управляемый модуль HSM](#step-4-transfer-your-key-to-managed-hsm)

### <a name="step-1-generate-a-kek"></a>Шаг 1. Создание KEK

KEK — это ключ RSA, который создается в управляемом HSM. KEK используется для шифрования ключа, который необходимо импортировать (*целевого* ключа).

Ключ KEK должен:
- быть ключом RSA-HSM (2048-, 3072- или 4096-разрядным);
- Создано в том же управляемом модуле HSM, где планируется импортировать целевой ключ
- поддерживать только операцию `import`.

> [!NOTE]
> Единственной допустимой операцией с ключом KEK должна быть операция "import", которая является взаимоисключающей с любыми другими операциями с ключами.

С помощью команды [az keyvault key create](/cli/azure/keyvault/key#az-keyvault-key-create) создайте ключ KEK, для которого в качестве операции с ключами задана операция `import`. Запишите идентификатор ключа (`kid`), возвращаемый следующей командой. (Вы будете использовать значение `kid` на [шаге 3](#step-3-generate-and-prepare-your-key-for-transfer).)

```azurecli-interactive
az keyvault key create --kty RSA-HSM --size 4096 --name KEKforBYOK --ops import --hsm-name ContosoKeyVaultHSM
```
---


### <a name="step-2-download-the-kek-public-key"></a>Шаг 2. Скачивание открытого ключа KEK

Чтобы скачать открытый ключ KEK в PEM-файл, используйте команду [az keyvault key download](/cli/azure/keyvault/key#az-keyvault-key-download). Импортируемый целевой ключ шифруется с помощью открытого ключа KEK.

```azurecli-interactive
az keyvault key download --name KEKforBYOK --hsm-name ContosoKeyVaultHSM --file KEKforBYOK.publickey.pem
```
---

Перенесите файл KEKforBYOK.publickey.pem на автономный компьютер. Он потребуется для выполнения следующего шага.

### <a name="step-3-generate-and-prepare-your-key-for-transfer"></a>Шаг 3. Создание ключа и подготовка его к передаче

Сведения о том, как скачать и установить средство BYOK, см. в документации поставщика HSM. Следуйте инструкциям поставщика HSM, чтобы создать целевой ключ, а затем создайте пакет для перемещения ключа (файл BYOK). Средство BYOK будет использовать значение `kid`, полученное на [шаге 1](#step-1-generate-a-kek), и файл KEKforBYOK.publickey.pem, скачанный на [шаге 2](#step-2-download-the-kek-public-key), для создания зашифрованного целевого ключа в файле BYOK.

Перенесите файл BYOK на подключенный компьютер.

> [!NOTE] 
> Импорт 1024-разрядных ключей RSA не поддерживается. В настоящее время импорт ключа эклиптической кривой (EC) не поддерживается.
>
> **Известная проблема**. Импорт целевого 4096-разрядного ключа RSA из модулей HSM Luna поддерживается только при использовании встроенного ПО версии 7.4.0 или более поздней.

### <a name="step-4-transfer-your-key-to-managed-hsm"></a>Шаг 4. Перенос ключа в управляемый модуль HSM

Чтобы завершить импорт ключа, перенесите пакет обмена ключами (файл BYOK) с отключенного компьютера на компьютер, подключенный к Интернету. Чтобы передать файл BYOK в управляемый модуль HSM, используйте команду [AZ keyvault Key Import](/cli/azure/keyvault/key#az-keyvault-key-import) .

```azurecli-interactive
az keyvault key import --hsm-name ContosoKeyVaultHSM --name ContosoFirstHSMkey --byok-file KeyTransferPackage-ContosoFirstHSMkey.byok
```

Если отправка прошла успешно, в Azure CLI отобразятся свойства импортированного ключа.

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы можете использовать этот ключ, защищенный АППАРАТным модулем безопасности, в управляемом HSM. Дополнительные сведения см. на [этой странице цен и сравнения характеристик](https://azure.microsoft.com/pricing/details/key-vault/).



