---
title: Шифрование неактивных с помощью управляемых клиентом ключей
titleSuffix: Azure Cognitive Search
description: Дополнительное шифрование на стороне сервера по индексам и картам синонимов в Azure Когнитивный поиск с помощью ключей, которые вы создаете и управляете в Azure Key Vault.
manager: nitinme
author: NatiNimni
ms.author: natinimn
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/02/2020
ms.custom: references_regions
ms.openlocfilehash: 6b1079797f1a753fa8362d6e920f3394087d7e9f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98119294"
---
# <a name="configure-customer-managed-keys-for-data-encryption-in-azure-cognitive-search"></a>Настройка ключей, управляемых клиентом, для шифрования данных в Azure Когнитивный поиск

Azure Когнитивный поиск автоматически шифрует индексированное содержимое при хранении с помощью [ключей, управляемых службой](../security/fundamentals/encryption-atrest.md#azure-encryption-at-rest-components). Если требуется дополнительная защита, можно дополнить шифрование по умолчанию дополнительным уровнем шифрования с помощью ключей, которые вы создаете и управляете в Azure Key Vault. В этой статье описано, как настроить шифрование ключа, управляемого клиентом.

Шифрование ключа, управляемого клиентом, зависит от [Azure Key Vault](../key-vault/general/overview.md). Можно создавать собственные ключи шифрования и хранить их в хранилище ключей либо использовать API-интерфейсы Azure Key Vault для генерации этих ключей. С помощью Azure Key Vault можно также выполнить аудит использования ключа, если [включено ведение журнала](../key-vault/general/logging.md).  

Шифрование с помощью ключей, управляемых клиентом, применяется к отдельным индексам или сопоставлениям синонимов при создании этих объектов и не указывается на самом уровне службы поиска. Шифровать можно только новые объекты. Вы не можете зашифровать уже существующее содержимое.

Ключи не должны находиться в одном хранилище ключей. Одна служба поиска может размещать несколько зашифрованных индексов или сопоставлений синонимов, каждый из которых шифруется с помощью собственных ключей шифрования, управляемых клиентом, которые хранятся в разных хранилищах ключей. Индексы и карты синонимов можно также использовать в той же службе, которая не зашифрована с помощью управляемых клиентом ключей.

>[!Important]
> При реализации ключей, управляемых клиентом, следует соблюдать требования к долгосрочным процедурам во время процедуры смены ключей хранилища ключей, а также Active Directory секретов и регистрации приложений. Всегда обновляйте все зашифрованное содержимое для использования новых секретов и ключей перед удалением старых. Если пропустить этот шаг, содержимое не будет расшифровано.

## <a name="double-encryption"></a>Двойное шифрование

Для служб, созданных после 1 августа 2020 и в конкретных регионах, область шифрования ключа, управляемого клиентом, включает временные диски, что обеспечивает [полное двойное шифрование](search-security-overview.md#double-encryption), которое в настоящее время доступно в этих регионах: 

+ Западная часть США 2
+ Восточная часть США
+ Центрально-южная часть США
+ US Gov (Вирджиния)
+ US Gov (Аризона)

Если вы используете другой регион или служба, созданная до 1 августа, шифрование управляемого ключа ограничено только диском данных, исключая временные диски, используемые службой.

## <a name="prerequisites"></a>Предварительные условия

В этом сценарии используются следующие средства и службы.

+ [Azure когнитивный Поиск](search-create-service-portal.md) на [уровне "оплачиваемый](search-sku-tier.md#tier-descriptions) " (базовый или выше в любом регионе).
+ [Azure Key Vault](../key-vault/general/overview.md), хранилище ключей можно создать с помощью [портал Azure](../key-vault//general/quick-create-portal.md), [Azure CLI](../key-vault//general/quick-create-cli.md)или [Azure PowerShell](../key-vault//general/quick-create-powershell.md). в той же подписке, что и Azure Когнитивный поиск. Для хранилища ключей должна быть включена **Защита от** **обратимого удаления** и очистки.
+ [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md). Если у вас ее нет, [Настройте новый клиент](../active-directory/develop/quickstart-create-new-tenant.md).

У вас должно быть приложение поиска, которое может создать зашифрованный объект. В этот код вы будете ссылаться на ключ хранилища ключей и Active Directory сведения о регистрации. Этот код может быть рабочим приложением или прототипом кода, таким как [пример кода C# дотнесовтоенкриптионусингкмк](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToEncryptionUsingCMK).

> [!TIP]
> Можно использовать [POST](search-get-started-rest.md), [Visual Studio Code](search-get-started-vs-code.md)или [Azure PowerShell](./search-get-started-powershell.md), чтобы вызвать интерфейсы API-интерфейсов RESTful, которые создают индексы и сопоставления синонимов, включающие параметр ключа шифрования. В настоящее время портал не поддерживает добавление ключа к индексам или картам синонимов.

## <a name="1---enable-key-recovery"></a>1. Включение восстановления ключей

Из-за особенностей шифрования с помощью управляемых клиентом ключей никто не может получить ваши данные, если ключ хранилища ключей Azure удален. Чтобы предотвратить потери данных, вызванные случайным удалением Key Vault ключей, в хранилище ключей должно быть включено обратимое удаление и защита от очистки. Обратимое удаление включено по умолчанию, поэтому вы сможете столкнуться с проблемами только в том случае, если она была отключена специально. По умолчанию защита от удаления не включена, но она необходима для шифрования ключа, управляемого клиентом, в Когнитивный поиск. Дополнительные сведения см. в статье Обзор [защиты от](../key-vault/general/soft-delete-overview.md#purge-protection) [обратимого удаления](../key-vault/general/soft-delete-overview.md) и очистки.

Оба свойства можно задать с помощью команд Portal, PowerShell или Azure CLI.

### <a name="using-azure-portal"></a>Использование портала Azure

1. [Войдите в портал Azure](https://portal.azure.com) и откройте страницу обзора хранилища ключей.

1. На странице **Обзор** в разделе **основные компоненты** включите **обратимое удаление** и **защиту от вирусов**.

### <a name="using-powershell"></a>Использование PowerShell

1. Выполните команду `Connect-AzAccount` , чтобы настроить учетные данные Azure.

1. Выполните следующую команду, чтобы подключиться к хранилищу ключей, заменив `<vault_name>` его допустимым именем:

   ```powershell
   $resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "<vault_name>").ResourceId
   ```

1. Azure Key Vault создается с возможностью обратимого удаления. Если оно отключено в вашем хранилище, выполните следующую команду:

   ```powershell
   $resource.Properties | Add-Member -MemberType NoteProperty -Name "enableSoftDelete" -Value 'true'
   ```

1. Включить защиту от очистки:

   ```powershell
   $resource.Properties | Add-Member -MemberType NoteProperty -Name "enablePurgeProtection" -Value 'true'
   ```

1. Сохраните обновления:

   ```powershell
   Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
   ```

### <a name="using-azure-cli"></a>Использование Azure CLI

+ Если у вас установлена [Azure CLI](/cli/azure/install-azure-cli), можно выполнить следующую команду, чтобы включить необходимые свойства.

   ```azurecli-interactive
   az keyvault update -n <vault_name> -g <resource_group> --enable-soft-delete --enable-purge-protection
   ```

## <a name="2---create-a-key-in-key-vault"></a>2. Создание ключа в Key Vault

Пропустите этот шаг, если у вас уже есть ключ в Azure Key Vault.

1. [Войдите в портал Azure](https://portal.azure.com) и откройте страницу обзора хранилища ключей.

1. Выберите **ключи** слева и нажмите кнопку **+ создать/импортировать**.

1. В области **Создание ключа** откройте список **Параметры** и выберите метод создания ключа. Вы можете **создать** новый ключ, **отправить** существующий или **восстановить резервную копию** ключа.

1. Введите **имя** ключа и при необходимости выберите другие ключевые свойства.

1. Нажмите кнопку **Создать**, чтобы начать развертывание.

1. Запишите идентификатор ключа — он состоит из **URI значения ключа**, **имени ключа** и **версии ключа**. Идентификатор потребуется для определения зашифрованного индекса в Когнитивный поиск Azure.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-key-identifier.png" alt-text="Создать новый ключ хранилища ключей":::

## <a name="3---register-an-app-in-active-directory"></a>3. Регистрация приложения в Active Directory

1. В [портал Azure](https://portal.azure.com)найдите ресурс Azure Active Directory для своей подписки.

1. Слева в разделе **Управление** выберите **Регистрация приложений**, а затем щелкните **Новая регистрация**.

1. Присвойте регистрационное имя, возможно, имя, похожее на имя приложения поиска. Выберите **Зарегистрировать**.

1. После создания регистрации приложения скопируйте идентификатор приложения. Необходимо предоставить эту строку приложению. 

   При пошаговом выполнении [дотнесовтоенкриптионусингкмк](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToEncryptionUsingCMK)вставьте это значение в **appsettings.jsв** файле.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-application-id.png" alt-text="Идентификатор приложения в разделе &quot;основные компоненты&quot;":::

1. Затем выберите **сертификаты & секреты** слева.

1. Выберите **Создать секрет клиента**. Присвойте секрету отображаемое имя и нажмите кнопку **Добавить**.

1. Скопируйте секрет приложения. При пошаговом выполнении образца вставьте это значение в **appsettings.js** файла.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-application-secret.png" alt-text="Секрет приложения":::

## <a name="4---grant-key-access-permissions"></a>4. Предоставление разрешений на доступ к ключам

На этом шаге вы создадите политику доступа в Key Vault. Эта политика предоставляет приложению, зарегистрированному с Active Directory разрешением на использование управляемого клиентом ключа.

Разрешения на доступ могут быть отменены в любое заданное время. После отзыва любой индекс службы поиска или карту синонимов, которая использует это хранилище ключей, станет непригодной для использования. Восстановление разрешений на доступ к хранилищу ключей в дальнейшем приведет к восстановлению доступа к карте индекс\синоним. Дополнительные сведения см. в статье [безопасный доступ к хранилищу ключей](../key-vault/general/secure-your-key-vault.md).

1. По-прежнему в портал Azure откройте страницу **обзора** хранилища ключей. 

1. Выберите **политики доступа** слева и щелкните **+ Добавить политику доступа**.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-add-access-policy.png" alt-text="Добавление новой политики доступа к хранилищу ключей":::

1. Щелкните **выбрать субъект** и выберите приложение, зарегистрированное с помощью Active Directory. Его можно найти по имени.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-access-policy-permissions.png" alt-text="Выбор субъекта политики доступа к хранилищу ключей":::

1. В **разделе Ключевые разрешения** выберите *получить*, *расобернуть ключ* и *Ключ переноса*.

1. В окне « **секретные разрешения**» выберите *получить*.

1. В окне **разрешения сертификата** выберите *получить*.

1. Выберите **Добавить** , а затем **сохранить**.

> [!Important]
> Зашифрованное содержимое в Когнитивный поиск Azure настроено для использования определенного ключа Azure Key Vault с определенной **версией**. При изменении ключа или версии необходимо обновить карту индекса или синонима, чтобы использовать новый кэй\версион **перед** удалением предыдущего кэй\версион. Если этого не сделать, то вы не сможете расшифровать содержимое после того, как теряется доступ к ключу.

<a name="encrypt-content"></a>

## <a name="5---encrypt-content"></a>5. шифрование содержимого

Чтобы добавить управляемый клиентом ключ на индекс, источник данных, набор умений, индексатор или карту синонимов, необходимо использовать [REST API поиска](/rest/api/searchservice/) или пакет SDK. Портал не предоставляет сопоставления синонимов или свойства шифрования. При использовании допустимых индексов API, источников данных, навыков, индексаторов и сопоставлений синонимов поддерживаются свойства **encryptionKey** верхнего уровня.

В этом примере используется REST API со значениями для Azure Key Vault и Azure Active Directory:

```json
{
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

> [!Note]
> Ни одно из этих сведений о хранилище ключей не считается секретным, и его можно легко получить, перейдя к соответствующей странице Azure Key Vault ключа в портал Azure.

## <a name="example-index-encryption"></a>Пример. Шифрование индекса

Создайте зашифрованный индекс с помощью [REST API когнитивный Поиск создания индекса Azure](/rest/api/searchservice/create-index). Используйте `encryptionKey` свойство, чтобы указать используемый ключ шифрования.
> [!Note]
> Ни одно из этих сведений о хранилище ключей не считается секретным, и его можно легко получить, перейдя к соответствующей странице Azure Key Vault ключа в портал Azure.

## <a name="rest-examples"></a>Примеры RESTFUL

В этом разделе показан полный код JSON для зашифрованного индекса и сопоставлений синонимов

### <a name="index-encryption"></a>Шифрование индекса

Подробные сведения о создании нового индекса с помощью REST API можно найти в статье [Создание индекса (REST API)](/rest/api/searchservice/create-index), где единственное отличие заключается в указании сведений о ключе шифрования как части определения индекса:

```json
{
 "name": "hotels",
 "fields": [
  {"name": "HotelId", "type": "Edm.String", "key": true, "filterable": true},
  {"name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": true, "facetable": false},
  {"name": "Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.lucene"},
  {"name": "Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene"},
  {"name": "Category", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
  {"name": "Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "sortable": false, "facetable": true},
  {"name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "sortable": true, "facetable": true},
  {"name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true, "facetable": true},
  {"name": "Rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true},
  {"name": "Location", "type": "Edm.GeographyPoint", "filterable": true, "sortable": true},
 ],
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Теперь можно отправить запрос на создание индекса, а затем начать использовать индекс в обычном режиме.

### <a name="synonym-map-encryption"></a>Шифрование на карте синонимов

Создайте зашифрованную карту синонимов, используя [REST API когнитивный Поиск Azure с картой создания синонимов](/rest/api/searchservice/create-synonym-map). Используйте `encryptionKey` свойство, чтобы указать используемый ключ шифрования.

```json
{
  "name" : "synonymmap1",
  "format" : "solr",
  "synonyms" : "United States, United States of America, USA\n
  Washington, Wash. => WA",
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Теперь можно отправить запрос на создание сопоставлений синонимов, а затем начать использовать его в обычном режиме.

## <a name="example-data-source-encryption"></a>Пример. шифрование источника данных

Создайте зашифрованный источник данных с помощью инструкции [Create Data Source (Azure Когнитивный поиск REST API)](/rest/api/searchservice/create-data-source). Используйте `encryptionKey` свойство, чтобы указать используемый ключ шифрования.

```json
{
  "name" : "datasource1",
  "type" : "azureblob",
  "credentials" :
  { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=datasource;AccountKey=accountkey;EndpointSuffix=core.windows.net"
  },
  "container" : { "name" : "containername" },
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Теперь можно отправить запрос на создание источника данных и приступить к его использованию в обычном режиме.

## <a name="example-skillset-encryption"></a>Пример. шифрование набора навыков

Создайте зашифрованный набор навыков с помощью [REST API когнитивный Поиск "Создание набора навыков Azure](/rest/api/searchservice/create-skillset)". Используйте `encryptionKey` свойство, чтобы указать используемый ключ шифрования.

```json
{
  "name" : "datasource1",
  "type" : "azureblob",
  "credentials" :
  { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=datasource;AccountKey=accountkey;EndpointSuffix=core.windows.net"
  },
  "container" : { "name" : "containername" },
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Теперь можно отправить запрос на создание набора навыков, а затем начать его использование в обычном режиме.

## <a name="example-indexer-encryption"></a>Пример. шифрование индексатора

Создайте зашифрованный индексатор с помощью [REST API когнитивный Поиск создания индексатора Azure](/rest/api/searchservice/create-indexer). Используйте `encryptionKey` свойство, чтобы указать используемый ключ шифрования.

```json
{
  "name": "indexer1",
  "dataSourceName": "datasource1",
  "skillsetName": "skillset1",
  "parameters": {
      "configuration": {
          "imageAction": "generateNormalizedImages"
      }
  },
  "encryptionKey": {
    "keyVaultUri": "https://demokeyvault.vault.azure.net",
    "keyVaultKeyName": "myEncryptionKey",
    "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660",
    "accessCredentials": {
      "applicationId": "00000000-0000-0000-0000-000000000000",
      "applicationSecret": "myApplicationSecret"
    }
  }
}
```

Теперь можно отправить запрос на создание индексатора и начать его использование в обычном режиме.

>[!Important]
> Хотя `encryptionKey` нельзя добавить к существующим индексам поиска или сопоставлениям синонимов, его можно обновить, предоставив разные значения для любого из трех сведений о хранилище ключей (например, обновление версии ключа). При переходе на новый ключ Key Vault или новую версию ключа необходимо сначала обновить любой индекс поиска или карту синонимов, которая использует этот ключ, чтобы использовать новый кэй\версион **перед** удалением предыдущего кэй\версион. Если не выполнить это действие, будет отображена схема индекса или синонима, которую нельзя использовать, так как она не сможет расшифровать содержимое после потери доступа к ключу. Хотя в дальнейшем восстановление разрешений на доступ к хранилищу ключей приведет к восстановлению доступа к содержимому.

## <a name="simpler-alternative-trusted-service"></a>Более простое альтернативное: доверенная служба

В зависимости от конфигурации клиента и требований к проверке подлинности вы можете реализовать более простой подход к доступу к ключу хранилища ключей. Вместо создания и использования приложения Active Directory можно сделать службу поиска доверенной, включив для нее управляемое системой удостоверение. Для доступа к ключу хранилища ключей можно использовать службу надежных поисков в качестве участника безопасности, а не зарегистрированного в AD приложения.

Этот подход позволяет опустить шаги для регистрации приложений и секретов приложений, а также упрощает определение ключа шифрования только компонентами хранилища ключей (URI, имя хранилища, версия ключа).

Как правило, управляемое удостоверение позволяет службе поиска проходить проверку подлинности в Azure Key Vault без сохранения учетных данных (ApplicationID или Аппликатионсекрет) в коде. Жизненный цикл этого типа управляемого удостоверения связан с жизненным циклом службы поиска, которая может иметь только одно управляемое удостоверение. Дополнительные сведения о работе управляемых удостоверений см. в статье [что такое управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md).

1. Сделайте службу поиска доверенной.
   ![Включение управляемого удостоверения, назначаемого системой](./media/search-managed-identities/turn-on-system-assigned-identity.png "Включение управляемого удостоверения, назначаемого системой")

1. При настройке политики доступа в Azure Key Vault выберите службу надежных поисков в качестве принципа (вместо приложения, зарегистрированного в AD). Назначьте те же разрешения (множественное получение, перенос, распаковка), как описано в шаге предоставление разрешений для ключа доступа.

1. Используйте упрощенное построение, в `encryptionKey` котором опущены свойства Active Directory.

    ```json
    {
      "encryptionKey": {
        "keyVaultUri": "https://demokeyvault.vault.azure.net",
        "keyVaultKeyName": "myEncryptionKey",
        "keyVaultKeyVersion": "eaab6a663d59439ebb95ce2fe7d5f660"
      }
    }
    ```

Ниже перечислены условия, которые не позволят внедрить этот упрощенный подход.

+ Вы не можете напрямую предоставить службе поиска доступ к хранилищу ключей (например, если служба поиска находится в другом клиенте Active Directory, отличном от Azure Key Vault).

+ Для размещения нескольких зашифрованных карт индексес\синоним требуется одна служба поиска, каждый из которых использует другой ключ из другого хранилища ключей, где каждое хранилище ключей должно использовать для проверки подлинности **другое удостоверение** . Так как служба поиска может иметь только одно управляемое удостоверение, требования к нескольким удостоверениям отменяют упрощенный подход к вашему сценарию.  

## <a name="work-with-encrypted-content"></a>Работа с зашифрованным содержимым

При шифровании с помощью управляемого клиентом ключа вы увидите задержку как для индексирования, так и для запросов из-за дополнительной работы шифрования или расшифровки. Azure Когнитивный поиск не регистрирует действия шифрования, но вы можете отслеживать доступ к ключам с помощью ведения журнала хранилища ключей. Рекомендуется [включить ведение журнала](../key-vault/general/logging.md) в рамках настройки хранилища ключей.

В течение времени должно произойти вращение ключа. При каждом смене ключей важно следовать следующей последовательности:

1. [Определите ключ, используемый индексом или картой синонимов](search-security-get-encryption-keys.md).
1. [Создайте новый ключ в хранилище ключей](../key-vault/keys/quick-create-portal.md), но оставьте исходный ключ доступным.
1. [Обновите свойства encryptionKey](/rest/api/searchservice/update-index) в индексе или сопоставлении синонимов, чтобы использовать новые значения. Только те объекты, которые изначально были созданы с помощью этого свойства, могут быть обновлены для использования другого значения.
1. Отключите или удалите предыдущий ключ в хранилище ключей. Отслеживайте доступ к ключам, чтобы проверить, используется ли новый ключ.

По соображениям производительности Служба поиска кэширует ключ до нескольких часов. Если отключить или удалить ключ без предоставления нового, запросы будут продолжать работать на временной основе до истечения срока действия кэша. Однако после того, как служба поиска не сможет расшифровать содержимое, вы получите следующее сообщение: "доступ запрещен. Возможно, используемый ключ запроса был отозван. Повторите попытку ". 

## <a name="next-steps"></a>Дальнейшие действия

Если вы не знакомы с архитектурой безопасности Azure, ознакомьтесь с [документацией по безопасности Azure](../security/index.yml)и в частности, в этой статье:

> [!div class="nextstepaction"]
> [Шифрование неактивных данных](../security/fundamentals/encryption-atrest.md)