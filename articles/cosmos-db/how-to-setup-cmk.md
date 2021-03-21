---
title: Настройка управляемых клиентом ключей для учетной записи Azure Cosmos DB
description: Подробные сведения о настройке управляемых клиентом ключей для учетной записи Azure Cosmos DB с Azure Key Vault
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 02/19/2021
ms.author: thweiss
ms.openlocfilehash: 3ee566a598ea7fdf060712c934305ef63467e548
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101656522"
---
# <a name="configure-customer-managed-keys-for-your-azure-cosmos-account-with-azure-key-vault"></a>Настройка управляемых клиентом ключей для учетной записи Azure Cosmos с Azure Key Vault
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

> [!NOTE]
> Использование управляемых клиентом ключей с Azure Cosmos DBным [аналитическим хранилищем](analytical-store-introduction.md) в настоящее время требует дополнительной настройки вашей учетной записи. Обратитесь за дополнительными [azurecosmosdbcmk@service.microsoft.com](mailto:azurecosmosdbcmk@service.microsoft.com) сведениями.

Данные, хранимые в учетной записи Azure Cosmos, автоматически шифруются с помощью ключей, управляемых корпорацией Майкрософт (**ключи, управляемые службой**). При необходимости можно добавить второй уровень шифрования с ключами, которыми вы управляете (**управляемыми клиентом ключами**).

:::image type="content" source="./media/how-to-setup-cmk/cmk-intro.png" alt-text="Уровни шифрования данных клиента":::

Необходимо сохранить управляемые клиентом ключи в [Azure Key Vault](../key-vault/general/overview.md) и предоставить ключ для каждой учетной записи Azure Cosmos, которая их поддерживает. Этот ключ используется для шифрования всех данных, хранящихся в данной учетной записи.

> [!NOTE]
> Сейчас управляемые клиентом ключи доступны только для новых учетных записей Azure Cosmos. Их следует настраивать во время создания учетной записи.

## <a name="register-the-azure-cosmos-db-resource-provider-for-your-azure-subscription"></a><a id="register-resource-provider"></a> Регистрация поставщика ресурсов Azure Cosmos DB в своей подписке Azure

1. Войдите на [портал Azure](https://portal.azure.com/), перейдите к своей подписке Azure и на вкладке **Параметры** выберите **Поставщики ресурсов**.

   :::image type="content" source="./media/how-to-setup-cmk/portal-rp.png" alt-text="Элемент поставщиков ресурсов в меню слева":::

1. Найдите поставщик ресурсов **Microsoft.DocumentDB**. Убедитесь, что поставщик ресурсов уже помечен как зарегистрированный. В противном случае выберите выделите его и выберите **Зарегистрировать**.

   :::image type="content" source="./media/how-to-setup-cmk/portal-rp-register.png" alt-text="Регистрация поставщика ресурсов Microsoft.DocumentDB":::

## <a name="configure-your-azure-key-vault-instance"></a>Настройка экземпляра Azure Key Vault

Для использования управляемых клиентом ключей с Azure Cosmos DB требуется установка двух свойств в экземпляре Azure Key Vault, который планируется использовать для размещения ключей шифрования: **Обратимое удаление** и **Защита от очистки**.

Если вы создаете новый экземпляр Azure Key Vault, включите эти свойства во время создания:

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-prop.png" alt-text="Включение обратимого удаления и защиты от очистки для нового экземпляра Azure Key Vault":::

Если вы используете имеющийся экземпляр Azure Key Vault, проверьте, включены ли эти свойства, на портале Azure в разделе **Свойства**. Если какое-либо из этих свойств не включено, см. разделы "Включение обратимого удаления" и "Включение защиты от очистки" в одной из следующих статей:

- [Как использовать обратимое удаление в Key Vault с помощью PowerShell](../key-vault/general/key-vault-recovery.md)
- [Как использовать обратимое удаление в Key Vault с помощью интерфейса командной строки](../key-vault/general/key-vault-recovery.md)

## <a name="add-an-access-policy-to-your-azure-key-vault-instance"></a>Добавление политики доступа к экземпляру Azure Key Vault

1. На портале Azure перейдите к экземпляру Azure Key Vault, который планируется использовать для размещения ключей шифрования. В меню слева выберите **Политики доступа**.

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-ap.png" alt-text="Политики доступа из меню слева":::

1. Нажмите **+ Добавить политику доступа**.

1. В раскрывающемся меню **Разрешения ключей** щелкните **Получить**, **Распаковка ключа** и выберите разрешения **Упаковка ключа**.

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-add-ap-perm2.png" alt-text="Выбор нужных разрешений":::

1. В разделе **Выбор субъекта** выберите пункт **Не выбрано**. Затем выполните поиск субъекта **Azure Cosmos DB** и выберите его (для удобства поиск можно выполнить по идентификатору субъекта: `57506a73-e302-42a9-b869-6f12d9ec29e9` регионах Azure для государственных организаций и `a232010e-820c-4083-83bb-3ace5fc29d0b` во всех остальных регионах Azure). Наконец, в нижней части нажмите кнопку **Выбрать**. Если субъект **Azure Cosmos DB** отсутствует в списке, может потребоваться повторная регистрация поставщика ресурсов **Microsoft.DocumentDB**, как описано в разделе [Регистрация поставщика ресурсов](#register-resource-provider) в этой статье.

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-add-ap.png" alt-text="Выбор субъекта Azure Cosmos DB":::

1. Выберите **Добавить**, чтобы добавить новую политику доступа.

1. Выберите **сохранить** на экземпляре Key Vault, чтобы сохранить все изменения.

## <a name="generate-a-key-in-azure-key-vault"></a>Создание нового ключа в Azure Key Vault

1. На портале Azure перейдите к экземпляру Azure Key Vault, который планируется использовать для размещения ключей шифрования. Затем в меню слева выберите **Ключи**.

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-keys.png" alt-text="Элемент Keys в меню слева":::

1. Выберите **Создать или импортировать**, введите имя нового ключа и выберите размер ключа RSA. Для максимальной безопасности рекомендуется использовать как минимум 3072. Щелкните **Создать**.

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-gen.png" alt-text="Создание ключа":::

1. После создания ключа выберите только что созданный ключ, а затем его текущую версию.

1. Скопируйте **идентификатор ключа**, кроме части после последней косой черты:

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-keyid.png" alt-text="Копирование идентификатора ключа":::

## <a name="create-a-new-azure-cosmos-account"></a>Создание учетной записи Azure Cosmos

### <a name="using-the-azure-portal"></a>Использование портала Azure

При создании учетной записи Azure Cosmos DB на портале Azure на шаге **Шифрование** выберите **Управляемый клиентом ключ**. В поле **URI ключа** вставьте URI или идентификатор ключа Azure Key Vault, скопированный на предыдущем шаге.

:::image type="content" source="./media/how-to-setup-cmk/portal-cosmos-enc.png" alt-text="Настройка параметров CMK на портале Azure":::

### <a name="using-azure-powershell"></a><a id="using-powershell"></a> Использование Azure PowerShell

При создании учетной записи Azure Cosmos DB с помощью PowerShell придерживайтесь следующих рекомендаций.

- Передайте скопированный ранее универсальный код ресурса (URI) ключа Azure Key Vault в свойство **keyVaultKeyUri** в **PropertyObject**.

- В качестве версии API используйте **2019-12-12** или более новую.

> [!IMPORTANT]
> Для успешного создания учетной записи с управляемыми клиентом ключами необходимо явно задать свойство `locations`.

```powershell
$resourceGroupName = "myResourceGroup"
$accountLocation = "West US 2"
$accountName = "mycosmosaccount"

$failoverLocations = @(
    @{ "locationName"="West US 2"; "failoverPriority"=0 }
)

$CosmosDBProperties = @{
    "databaseAccountOfferType"="Standard";
    "locations"=$failoverLocations;
    "keyVaultKeyUri" = "https://<my-vault>.vault.azure.net/keys/<my-key>";
}

New-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2019-12-12" -ResourceGroupName $resourceGroupName `
    -Location $accountLocation -Name $accountName -PropertyObject $CosmosDBProperties
```

Чтобы после создания учетной записи проверить, включены ли управляемые клиентом ключи, получите URI ключа Azure Key Vault:

```powershell
Get-AzResource -ResourceGroupName $resourceGroupName -Name $accountName `
    -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    | Select-Object -ExpandProperty Properties `
    | Select-Object -ExpandProperty keyVaultKeyUri
```

### <a name="using-an-azure-resource-manager-template"></a>Использование шаблона Azure Resource Manager

При создании новой учетной записи Azure Cosmos с помощью шаблона Azure Resource Manager придерживайтесь следующих рекомендаций.

- Передайте скопированный ранее универсальный код ресурса (URI) ключа Azure Key Vault в свойство **keyVaultKeyUri** в объекте **properties**.

- В качестве версии API используйте **2019-12-12** или более новую.

> [!IMPORTANT]
> Для успешного создания учетной записи с управляемыми клиентом ключами необходимо явно задать свойство `locations`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "keyVaultKeyUri": {
            "type": "string"
        }
    },
    "resources": 
    [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "name": "[parameters('accountName')]",
            "apiVersion": "2019-12-12",
            "kind": "GlobalDocumentDB",
            "location": "[parameters('location')]",
            "properties": {
                "locations": [ 
                    {
                        "locationName": "[parameters('location')]",
                        "failoverPriority": 0,
                        "isZoneRedundant": false
                    }
                ],
                "databaseAccountOfferType": "Standard",
                "keyVaultKeyUri": "[parameters('keyVaultKeyUri')]"
            }
        }
    ]
}
```

Разверните шаблон, используя следующий скрипт PowerShell:

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$accountLocation = "West US 2"
$keyVaultKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-key>"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "deploy.json" `
    -accountName $accountName `
    -location $accountLocation `
    -keyVaultKeyUri $keyVaultKeyUri
```

### <a name="using-the-azure-cli"></a><a id="using-azure-cli"></a> Использование Azure CLI

При создании новой учетной записи Azure Cosmos с помощью Azure CLI передайте универсальный код ресурса (URI) ключа Azure Key Vault, скопированный ранее из параметра `--key-uri`.

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
keyVaultKeyUri = 'https://<my-vault>.vault.azure.net/keys/<my-key>'

az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --key-uri $keyVaultKeyUri
```

Чтобы после создания учетной записи проверить, включены ли управляемые клиентом ключи, получите URI ключа Azure Key Vault:

```azurecli-interactive
az cosmosdb show \
    -n $accountName \
    -g $resourceGroupName \
    --query keyVaultKeyUri
```

## <a name="key-rotation"></a>Смена ключей

Поворот управляемого клиентом ключа, используемого вашей учетной записью Azure Cosmos, может осуществляться двумя способами.

- Создайте новую версию ключа, которая сейчас используется из Azure Key Vault:

  :::image type="content" source="./media/how-to-setup-cmk/portal-akv-rot.png" alt-text="Создание новой версии ключа":::

- Замените ключ, используемый в настоящий момент, на совершенно другой, изменив URI ключа в своей учетной записи. В портал Azure перейдите к учетной записи Azure Cosmos и выберите **Шифрование данных** в меню слева:

    :::image type="content" source="./media/how-to-setup-cmk/portal-data-encryption.png" alt-text="Запись меню шифрования данных":::

    Затем замените **универсальный код ресурса (URI) ключа** новым ключом, который вы хотите использовать, и выберите **сохранить**:

    :::image type="content" source="./media/how-to-setup-cmk/portal-key-swap.png" alt-text="Обновление URI ключа":::

    Вот как добиться того же результата в PowerShell:

    ```powershell
    $resourceGroupName = "myResourceGroup"
    $accountName = "mycosmosaccount"
    $newKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-new-key>"
    
    $account = Get-AzResource -ResourceGroupName $resourceGroupName -Name $accountName `
        -ResourceType "Microsoft.DocumentDb/databaseAccounts"
    
    $account.Properties.keyVaultKeyUri = $newKeyUri
    
    $account | Set-AzResource -Force
    ```

Предыдущая версия ключа или ключа может быть отключена через 24 часа или после того, как [Azure Key Vault журналы аудита](../key-vault/general/logging.md) не показывают действия из Azure Cosmos DB в этом ключе или версии ключа.
    
## <a name="error-handling"></a>Обработка ошибок

При использовании ключей Customer-Managed (CMK) в Azure Cosmos DB при возникновении ошибок Azure Cosmos DB возвращает сведения об ошибке вместе с кодом подсостояния HTTP в ответе. Этот код подсостояния можно использовать для отладки основной причины проблемы. Список поддерживаемых кодов подсостояния HTTP см. в статье [коды состояния HTTP для Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) .

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="is-there-an-additional-charge-to-enable-customer-managed-keys"></a>Взимается ли дополнительная плата за включение управляемых клиентом ключей?

Нет, плата за включение этой функции не взимается.

### <a name="how-do-customer-managed-keys-impact-capacity-planning"></a>Как управляемые клиентом ключи влияют на планирование ресурсов?

При использовании управляемых клиентом ключей количество [единиц запроса](./request-units.md), потребляемых операциями базы данных, возрастает из-за дополнительной обработки, необходимой для шифрования и расшифровки данных. Это может немного повысить использование подготовленной емкости. При расчетах ориентируйтесь на следующую таблицу.

| Operation type (Тип операции) | Увеличение количества единиц запроса |
|---|---|
| Точечные операции чтения (получение элементов по их идентификаторам) | +5 % на операцию |
| Любая операция записи | +6 % на операцию<br/>примерно +0,06 единиц запроса на индексированное свойство |
| Запросы, канал изменений для чтения или канал с конфликтами | +15 % на операцию |

### <a name="what-data-gets-encrypted-with-the-customer-managed-keys"></a>Какие данные шифруются с помощью управляемых клиентом ключей?

С помощью управляемых клиентом ключей шифруются все данные, хранящиеся в учетной записи Azure Cosmos, за исключением следующих метаданных:

- имена [учетных записей, баз данных и контейнеров](./account-databases-containers-items.md#elements-in-an-azure-cosmos-account) Azure Cosmos DB;

- имена [хранимых процедур](./stored-procedures-triggers-udfs.md);

- пути свойств, объявленные в [политиках индексирования](./index-policy.md);

- значения [ключей секции](./partitioning-overview.md) контейнеров.

### <a name="are-customer-managed-keys-supported-for-existing-azure-cosmos-accounts"></a>Поддерживаются ли управляемые клиентом ключи для имеющихся учетных записей Azure Cosmos?

Сейчас эта возможность доступна только для новых учетных записей.

### <a name="is-it-possible-to-use-customer-managed-keys-in-conjunction-with-the-azure-cosmos-db-analytical-store"></a>Можно ли использовать управляемые клиентом ключи в сочетании с Azure Cosmos DBным [аналитическим хранилищем](analytical-store-introduction.md)?

Да, но в настоящее время для вашей учетной записи требуется дополнительная настройка. Обратитесь за дополнительными [azurecosmosdbcmk@service.microsoft.com](mailto:azurecosmosdbcmk@service.microsoft.com) сведениями.

### <a name="is-there-a-plan-to-support-finer-granularity-than-account-level-keys"></a>Планируется ли поддержка большей детализации по сравнению с ключами уровня учетной записи?

В настоящее время нет, но рассматривается возможность поддержки ключей уровня контейнера.

### <a name="how-can-i-tell-if-customer-managed-keys-are-enabled-on-my-azure-cosmos-account"></a>Как узнать, включены ли управляемые клиентом ключи для учетной записи Azure Cosmos?

В портал Azure перейдите к учетной записи Azure Cosmos и просмотрите запись **шифрования данных** в меню слева. Если эта запись существует, в вашей учетной записи будут включены ключи, управляемые клиентом.

:::image type="content" source="./media/how-to-setup-cmk/portal-data-encryption.png" alt-text="Запись меню шифрования данных":::

Можно также программно получить сведения об учетной записи Azure Cosmos и найти `keyVaultKeyUri` свойство. Сведения о том, как это можно сделать [в PowerShell](#using-powershell) и [с помощью Azure CLI](#using-azure-cli), см. выше.

### <a name="how-do-customer-managed-keys-affect-a-backup"></a>Как управляемые клиентом ключи влияют на резервное копирование?

Azure Cosmos DB [регулярно выполняет автоматическое резервное копирование](./online-backup-and-restore.md) данных, хранящихся в вашей учетной записи. В результате этой операции создается резервная копия зашифрованных данных. Чтобы использовать восстановленную резервную копию, требуется ключ шифрования, который применялся во время резервного копирования. Поэтому ключ не отзывается, и та его версия, которая использовалась во время резервного копирования, будет по-прежнему активна.

### <a name="how-do-i-revoke-an-encryption-key"></a>Как отозвать ключ шифрования?

Отзыв ключа выполняется путем отключения его последней версии.

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-rev2.png" alt-text="Отключение версии ключа":::

Кроме того, чтобы отозвать все ключи из экземпляра Azure Key Vault, можно удалить политику доступа, предоставленную субъекту Azure Cosmos DB.

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-rev.png" alt-text="Удаление политики доступа для субъекта-Azure Cosmos DB":::

### <a name="what-operations-are-available-after-a-customer-managed-key-is-revoked"></a>Какие операции доступны после отзыва управляемого клиентом ключа?

Единственная операция, возможная после отзыва ключа шифрования, — это удаление учетной записи.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с дополнительными сведениями о [шифровании данных в Azure Cosmos DB](./database-encryption-at-rest.md).
- Ознакомьтесь с обзором [безопасного доступа к данным в Cosmos DB](secure-access-to-data.md).