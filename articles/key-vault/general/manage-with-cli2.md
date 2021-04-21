---
title: Управление Azure Key Vault с помощью интерфейса командной строки — Azure Key Vault | Документация Майкрософт
description: Из этой статьи вы узнаете об автоматизации основных задач в Key Vault с использованием Azure CLI.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 08/12/2019
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: d4037e66e653bc6e958020a5ef8722f2febb53d0
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772186"
---
# <a name="manage-key-vault-using-the-azure-cli"></a>Управление Key Vault с помощью интерфейса командной строки Azure 

В этой статье описывается, как начать работу с Azure Key Vault с помощью Azure CLI.  Вы можете просмотреть следующие данные:

- как создать контейнер (хранилище) с усиленной защитой в Azure;
- Добавление ключа, секрета или сертификата в хранилище ключей
- Регистрация приложения в Azure Active Directory
- Авторизация приложения для использования ключа или секрета
- Установка политик расширенного доступа к хранилищу ключей
- Работа с аппаратными модулями безопасности (HSM)
- Удаление хранилища ключей, а также связанных ключей и секретов
- Прочие команды в межплатформенном интерфейсе командной строки Azure


Хранилище ключей Azure доступно в большинстве регионов. Дополнительные сведения см. на странице [цен на хранилище ключей](https://azure.microsoft.com/pricing/details/key-vault/).

> [!NOTE]
> В этой статье нет указаний по написанию приложения Azure, что предусматривает один из шагов, но показано, как авторизовать приложение для использования ключей или секретов в хранилище ключей.
>

Общие сведения о Key Vault см. в [этой статье](overview.md). Если у вас нет подписки Azure, сначала создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать команды Azure CLI из этой строки, вам необходимо следующее:

* подписка на Microsoft Azure. Если у вас ее нет, зарегистрируйтесь, чтобы воспользоваться [бесплатной пробной версией](https://azure.microsoft.com/pricing/free-trial).
* Интерфейс командной строки Azure версии 2.0 или более поздней версии. Чтобы установить последнюю версию, обратитесь к статье [Установка Azure CLI](/cli/azure/install-azure-cli).
* Приложение, для которого вы будете настраивать использование ключа или пароля, созданного по этой статье. Пример приложения доступен в [Центре загрузки Майкрософт](https://www.microsoft.com/download/details.aspx?id=45343). Инструкции см. в приложенном файле README.

### <a name="getting-help-with-azure-cross-platform-command-line-interface"></a>Справка по межплатформенному интерфейсу командной строки Azure

В этой статье предполагается, что вы знакомы с интерфейсом командной строки (Bash, терминал, командная строка).

Параметр --help или -h может использоваться для просмотра справки по определенным командам. Также можно использовать формат Azure help [команда] [параметры]. Если вы сомневаетесь в том, какие параметры необходимы команде, обратитесь к справке. Например, все следующие команды возвращают одинаковые сведения:

```azurecli
az account set --help
az account set -h
```

Чтобы узнать, как работать с Azure Resource Manager в кроссплатформенном интерфейсе командной строки Azure, также прочитайте следующие статьи:

* [Установка Azure CLI](/cli/azure/install-azure-cli)
* [Приступая к работе с Azure CLI](/cli/azure/get-started-with-azure-cli)

## <a name="how-to-create-a-hardened-container-a-vault-in-azure"></a>как создать контейнер (хранилище) с усиленной защитой в Azure;

Хранилища являются защищенными контейнерами, которые поддерживают аппаратные модули безопасности. Хранилища позволяют уменьшить вероятность случайной потери информации о безопасности путем централизации хранения секретов приложений. Хранилища Key Vault также контролируют и регистрируют доступ к чему-либо, что хранится в них. Azure Key Vault может обрабатывать запросы и обновлять сертификаты протокола TLS, предоставляя функции, необходимые для надежного решения по управлению жизненным циклом сертификата. На следующих шагах создается хранилище.

### <a name="connect-to-your-subscriptions"></a>Подключение к своим подпискам

Чтобы войти в интерактивный режим, используйте следующую команду:

```azurecli
az login
```
Чтобы войти с помощью учетной записи организации, можно передать имя пользователя и пароль.

```azurecli
az login -u username@domain.com -p password
```

Если у вас несколько подписок и нужно указать, какая из них будет использоваться, введите приведенную ниже команду, чтобы увидеть подписки своей учетной записи:

```azurecli
az account list
```

Укажите подписку с помощью параметра подписки.

```azurecli
az account set --subscription <subscription name or ID>
```

Дополнительные сведения о настройке кроссплатформенного интерфейса командной строки Azure см. в разделе [Установка Azure CLI 1.0](/cli/azure/install-azure-cli).

### <a name="create-a-new-resource-group"></a>Создание новой группы ресурсов

При использовании диспетчера ресурсов Azure все связанные ресурсы создаются внутри группы ресурсов. Хранилище ключей можно создать в существующей группе ресурсов. Чтобы использовать новую группу ресурсов, создайте ее.

```azurecli
az group create -n "ContosoResourceGroup" -l "East Asia"
```

Первый параметр — имя группы ресурсов, а второй параметр — его расположение. Чтобы получить список доступных расположений, введите следующую команду:

```azurecli
az account list-locations
``` 

### <a name="register-the-key-vault-resource-provider"></a>Регистрация поставщика ресурсов хранилища ключей

 При попытке создать хранилище ключей может возникнуть ошибка "Подписка не зарегистрирована для использования пространства имен Microsoft.KeyVault". Если возникла эта ошибка, убедитесь, что поставщик ресурсов хранилища ключей зарегистрирован в вашей подписке. Эта операция выполняется один раз для каждой подписки.

```azurecli
az provider register -n Microsoft.KeyVault
```

### <a name="create-a-key-vault"></a>Создайте хранилище ключей.

Для создания хранилища ключей используйте команду `az keyvault create` . Этот сценарий предусматривает три обязательных параметра: имя группы ресурсов, имя хранилища ключей и географическое расположение.

Чтобы создать новое хранилище с именем **ContosoKeyVault**, в группе ресурсов **ContosoResourceGroup**, находящейся в местоположении **Восточная Азия**, введите: 

```azurecli
az keyvault create --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --location "East Asia"
```

В выходных данных команды будут показаны свойства созданного хранилища ключей. Среди всех свойств есть два самых важных:

* **name**. В этом примере — ContosoKeyVault. Вы будете использовать это имя для выполнения других команд хранилища ключей.
* **vaultUri**. В данном примере универсальный код ресурса (URI) — это https://contosokeyvault.vault.azure.net. Необходимо, чтобы приложения, использующие ваше хранилище через REST API, использовали этот URI.

Теперь ваша учетная запись Azure авторизована, и вы можете выполнять любые операции в этом хранилище ключей. Но пока никто другой не авторизован.

## <a name="adding-a-key-secret-or-certificate-to-the-key-vault"></a>Добавление ключа, секрета или сертификата в хранилище ключей

Если вам необходимо создать для себя ключ с программной защитой с помощью Azure Key Vault, используйте команду `az key create`.

```azurecli
az keyvault key create --vault-name "ContosoKeyVault" --name "ContosoFirstKey" --protection software
```

Если у вас уже есть ключ в PEM-файле, его можно загрузить в Azure Key Vault. Вы можете защитить ключ с помощью программного обеспечения или HSM. В этом примере ключ импортируется из PEM-файла и защищается с помощью программного обеспечения и пароля hVFkk965BuUv:

```azurecli
az keyvault key import --vault-name "ContosoKeyVault" --name "ContosoFirstKey" --pem-file "./softkey.pem" --pem-password "hVFkk965BuUv" --protection software
```

Теперь для доступа к ключу, созданному или загруженному в хранилище ключей Azure, вы сможете использовать его URI. `https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey` всегда предоставляет текущую версию. https://[keyvault-name].vault.azure.net/keys/[keyname]/[key-unique-id] предоставляет эту конкретную версию. Например, `https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey/cgacf4f763ar42ffb0a1gca546aygd87`. 

Добавьте в Azure Key Vault секрет, который представляет собой пароль с именем SQLPassword и значением Pa$$hVFkk965BuUv. 

```azurecli
az keyvault secret set --vault-name "ContosoKeyVault" --name "SQLPassword" --value "hVFkk965BuUv "
```

Ссылайтесь на этот пароль с помощью URI. **https://ContosoVault.vault.azure.net/secrets/SQLPassword** всегда предоставляет актуальную версию, а https://[keyvault-name].vault.azure.net/secret/[secret-name]/[secret-unique-id] предоставляет эту конкретную версию. (например, **https://ContosoVault.vault.azure.net/secrets/SQLPassword/90018dbb96a84117a0d2847ef8e7189d** ).

Импортируйте сертификат в хранилище с помощью PEM- или PFX-файла.

```azurecli
az keyvault certificate import --vault-name "ContosoKeyVault" --file "c:\cert\cert.pfx" --name "ContosoCert" --password "hVFkk965BuUv"
```

Давайте посмотрим на созданный вами ключ, секрет или сертификат:

* Чтобы просмотреть свои ключи, введите: 

```azurecli
az keyvault key list --vault-name "ContosoKeyVault"
```

* Чтобы просмотреть свои секреты, введите: 

```azurecli
az keyvault secret list --vault-name "ContosoKeyVault"
```

* Чтобы просмотреть сертификаты, введите: 

```azurecli
az keyvault certificate list --vault-name "ContosoKeyVault"
```

## <a name="registering-an-application-with-azure-active-directory"></a>Регистрация приложения в Azure Active Directory

Обычно этот шаг выполняет разработчик на отдельном компьютере. Он не относится к Azure Key Vault, но включен в эту статью для информации. Чтобы завершить регистрацию приложения, поместите вашу учетную запись, хранилище и приложение в один каталог Azure.

Приложения, которые используют хранилища ключей, должны пройти аутентификацию с использованием токена Azure Active Directory.  Владелец приложения должен сначала зарегистрировать его в Azure Active Directory. В конце регистрации владелец приложения получает следующие значения.

- **Идентификатор приложения** (также известный как идентификатор клиента AAD или appID)
- **Ключ проверки подлинности** (также известный как общий секрет). 

Для получения маркера приложение должно предоставить Azure Active Directory оба этих значения. Настройка приложения для получения маркера будет зависеть от приложения. В [примере приложения, использующего Key Vault](https://www.microsoft.com/download/details.aspx?id=45343), владелец приложения задает эти значения в файле app.config.

Пошаговые инструкции по регистрации приложения с помощью Azure Active Directory см. в статьях [Краткое руководство. Регистрация приложения в конечной точке Azure Active Directory версии 1.0](../../active-directory/develop/quickstart-register-app.md), [Создание приложения Azure Active Directory и субъекта-службы с доступом к ресурсам с помощью портала](../../active-directory/develop/howto-create-service-principal-portal.md) и [Создание субъекта-службы Azure с помощью Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli).

Чтобы зарегистрировать приложение в Azure Active Directory:

```azurecli
az ad sp create-for-rbac -n "MyApp" --password "hVFkk965BuUv" --skip-assignment
# If you don't specify a password, one will be created for you.
```

## <a name="authorizing-an-application-to-use-a-key-or-secret"></a>Авторизация приложения для использования ключа или секрета

Чтобы авторизовать приложение для получения доступа к ключу или секрету в хранилище, используйте команду `az keyvault set-policy` .

Например, если имя хранилища — ContosoKeyVault, идентификатор приложения — 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed, и вам нужно авторизовать это приложение для расшифровки и подписи с использованием ключей в вашем хранилище, выполните следующую команду:

```azurecli
az keyvault set-policy --name "ContosoKeyVault" --spn 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed --key-permissions decrypt sign
```

Чтобы разрешить этому же приложению читать секреты в хранилище, выполните следующую команду:

```azurecli
az keyvault set-policy --name "ContosoKeyVault" --spn 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed --secret-permissions get
```

## <a name="setting-key-vault-advanced-access-policies"></a><a name="bkmk_KVperCLI"></a> Установка политик расширенного доступа к хранилищу ключей

Включите политики расширенного доступа к хранилищу ключей с помощью команды [az keyvault update](/cli/azure/keyvault#az_keyvault_update).

 Включение хранилища ключей для развертывания: позволяет виртуальным машинам получать сертификаты, хранимые в виде секретов, из хранилища.

 ```azurecli
 az keyvault update --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --enabled-for-deployment "true"
 ```

Включение хранилища ключей для шифрования дисков: требуется при использовании хранилища для шифрования дисков Azure.

 ```azurecli
 az keyvault update --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --enabled-for-disk-encryption "true"
 ```  

Включение хранилища ключей для развертывания шаблона: позволяет Resource Manager получать секреты из хранилища.

```azurecli
az keyvault update --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --enabled-for-template-deployment "true"
```

## <a name="working-with-hardware-security-modules-hsms"></a>Работа с аппаратными модулями безопасности (HSM)

Чтобы обеспечить более высокий уровень защиты, можно импортировать ключи или создать их в аппаратных модулях безопасности (HSM) — ключи никогда не покидают их пределы. В качестве аппаратных модулей безопасности используются модули FIPS 140-2 с проверкой уровня 2. Если это требование вас не касается, пропустите этот подраздел и перейдите к разделу "Удаление хранилища ключей, а также связанных ключей и секретов".

Чтобы создать ключи, защищенные аппаратным модулем безопасности, у вас должна быть подписка на хранилище, которая поддерживает ключи, защищенные аппаратным модулем безопасности.

При создании хранилища добавьте параметр sku:

```azurecli
az keyvault create --name "ContosoKeyVaultHSM" --resource-group "ContosoResourceGroup" --location "East Asia" --sku "Premium"
```

В это хранилище можно добавлять ключи с программной защитой (как показано выше) и ключи, защищенные аппаратным модулем безопасности. Чтобы создать ключ, защищенный аппаратным модулем безопасности, задайте значение HSM для параметра Destination:

```azurecli
az keyvault key create --vault-name "ContosoKeyVaultHSM" --name "ContosoFirstHSMKey" --protection "hsm"
```

Вы можете использовать следующую команду для импорта ключа из PEM-файла на своем компьютере. Эта команда импортирует ключ в аппаратные модули безопасности в службе хранилища ключей:

```azurecli
az keyvault key import --vault-name "ContosoKeyVaultHSM" --name "ContosoFirstHSMKey" --pem-file "/.softkey.pem" --protection "hsm" --pem-password "PaSSWORD"
```

Следующая команда импортирует пакет собственного ключа клиента (BYOK). Это дает возможность создать ключ в своем локальном аппаратном модуле безопасности и передать его в аппаратные модули безопасности в службе хранилища ключей так, чтобы ключ не покидал их пределы:

```azurecli
az keyvault key import --vault-name "ContosoKeyVaultHSM" --name "ContosoFirstHSMKey" --byok-file "./ITByok.byok" --protection "hsm"
```

Более подробные инструкции по созданию пакета BYOK см. в статье [Создание ключей, защищенных аппаратным модулем безопасности, и их передача в хранилище ключей Azure](../keys/hsm-protected-keys.md).

## <a name="deleting-the-key-vault-and-associated-keys-and-secrets"></a>Удаление хранилища ключей, а также связанных ключей и секретов

Если хранилище ключей и его ключи и секреты больше не нужны, хранилище можно удалить, используя команду `az keyvault delete`:

```azurecli
az keyvault delete --name "ContosoKeyVault"
```

Как вариант, можно удалить всю группу ресурсов Azure, которая включает в себя хранилище ключей и другие ресурсы, которые вы добавили в эту группу:

```azurecli
az group delete --name "ContosoResourceGroup"
```

## <a name="miscellaneous-azure-cross-platform-command-line-interface-commands"></a>Прочие команды в межплатформенном интерфейсе командной строки Azure

Другие команды, которые могут быть полезны при управлении Azure Key Vault.

Эта команда перечисляет все ключи и выбранные свойства в виде таблицы:

```azurecli
az keyvault key list --vault-name "ContosoKeyVault"
```

Эта команда отображает полный список свойств для указанного ключа:

```azurecli
az keyvault key show --vault-name "ContosoKeyVault" --name "ContosoFirstKey"
```

Эта команда перечисляет все имена секретов и выбранные свойства в виде таблицы:

```azurecli
az keyvault secret list --vault-name "ContosoKeyVault"
```

Пример удаления конкретного ключа:

```azurecli
az keyvault key delete --vault-name "ContosoKeyVault" --name "ContosoFirstKey"
```

Пример удаления конкретного секрета:

```azurecli
az keyvault secret delete --vault-name "ContosoKeyVault" --name "SQLPassword"
```

## <a name="next-steps"></a>Дальнейшие действия

- Полное описание команд Azure CLI для хранилища ключей доступно в [справочнике по интерфейсу командной строки Key Vault](/cli/azure/keyvault).

- Справочные материалы по программированию см. в статье [Руководство разработчика хранилища ключей Azure](developers-guide.md).

- Сведения об Azure Key Vault и HSM см. в статье [Создание ключей, защищенных аппаратным модулем безопасности, и их передача в хранилище ключей Azure](../keys/hsm-protected-keys.md).
